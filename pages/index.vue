<template>
  <div class="browser-content">
    <template v-if="mode === 'index'">
      <input v-model="filterValue" />
      <div id="scroller" class="artists-list">
        <div v-for="a in filteredArtists" :key="'artist-' + a.id">
          <NuxtLink :to="'/artist/' + a.id">{{ a.title }}</NuxtLink>
        </div>
      </div>
    </template>
  </div>
</template>

<script>
export default {
  async asyncData({ $axios, params }) {
    const resp = await $axios.get('/api/index')
    return { artists: resp.data }
  },
  data() {
    return {
      mode: 'index',
      artist: {},
      artists: [],
      filterValue: '',
      abyssFolderId: 0,
      tracklists: { playlist: [], simple: [] },
    }
  }, // end of methods()
  computed: {
    filteredArtists() {
      if (this.filterValue) {
        const r = new RegExp(this.filterValue, 'i')
        return this.artists.filter(
          (i) => i.title.match(r) || (i.aliases && i.aliases.match(r))
        )
      } else {
        return this.artists
      }
    },
  },
  mounted() {
    const el = document.querySelector('#scroller')

    function scrollHorizontally(e) {
      e = window.event || e
      e.preventDefault()
      el.scrollLeft -= e.wheelDelta || -e.detail * 20
    }

    el.addEventListener('mousewheel', scrollHorizontally, false) // IE9, Chrome, Safari, Opera
    el.addEventListener('DOMMouseScroll', scrollHorizontally, false) // Firefox
  },
}
</script>

<style lang="scss" scoped>
.browser-content {
  overflow-x: hidden;
  overflow-y: scroll;
  grid-area: content;

  .artists-list {
    padding: 0 1em;
    display: flex;
    flex-flow: wrap column;
    align-content: start;
    max-height: 80vh;
    overflow-x: auto;

    div {
      width: 15em;
      margin-right: 1em;
    }
  }
}
</style>
