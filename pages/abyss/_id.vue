<template>
  <div class="vue-abyss">
    <div class="folder-path">
      <NuxtLink to="/abyss/0" class="node">root</NuxtLink
      ><NuxtLink
        v-for="f of parents"
        :key="'parent-' + f[0]"
        :to="'/abyss/' + f[0]"
        class="node"
        >{{ f[1] }}</NuxtLink
      >{{ name }} <a class="button-link" @click="reloadFolder">⟳</a>
    </div>

    <template v-if="subfolders && subfolders.length > 0">
      <h2>&#x1f4c1; Folders</h2>
      <div v-for="f of subfolders" :key="'folder-' + f[0]" class="folder">
        <NuxtLink :to="'/abyss/' + f[0]">{{ f[1] }}</NuxtLink>
        <template v-if="f[2]"> by {{ f[2][1] }}</template>
      </div>
    </template>

    <template v-if="release">
      <h2>«{{ release.title }}» by {{ artist[1] }}</h2>
      <Release :payload="release" @play="playTrack"></Release>
    </template>
    <template v-else-if="hasAudio">
      <h2>Link release</h2>
      <AbyssForm :id="id" @update="releaseUpdated"></AbyssForm>

      <template v-if="files && files.length > 0">
        <h2>&#x1f4c4; Files</h2>
        <div v-for="f of files" :key="'file-' + f['t']" class="file">
          <div class="ajax-link">{{ f['t'] }}</div>
        </div>
      </template>
    </template>
  </div>
</template>

<script>
export default {
  async asyncData({ $axios, params }) {
    const resp = await $axios.get(`/api/abyss/${params.id}`)
    return resp.data
  },
  computed: {
    allTracks() {
      return this.release
        ? this.release.tracks.map((i) => this.trackObject(i))
        : []
    },
    hasAudio() {
      return !!(
        this.release ||
        this.files.findIndex((i) => /\.(mp3|m4a)$/.test(i.t)) !== -1
      )
    },
  },
  methods: {
    playTrack(uid) {
      this.$player.startPlaylist(this.allTracks, uid)
    },
    trackObject(track) {
      // TODO: remove duplicate (same method in artist/_id.vue)
      return {
        uid: track.uid,
        title: track.title,
        artist: this.title,
        rating: track.rating,
        src: `/download/audio/${track.uid}`,
      }
    },
    reloadFolder() {},
    releaseUpdated() {
      this.reloadFolder()
    },
  },
}
</script>

<style lang="scss" scoped>
.vue-abyss {
  margin: 0 1em;

  .folder-path {
    .node {
      display: inline-block;

      &:after {
        content: '/';
        color: inherit;
        text-decoration: none;
      }
    }
  }
  .file {
    .now-playing {
      background: #ffffa3;
    }
  }
  table.release-form {
    font-size: 0.8em;
    font-family: sans-serif;

    tr.reset {
      cursor: pointer;
      &:hover {
        background-color: #ffffa3;
      }
    }
    th {
      color: #555;
    }
    td {
      text-align: center;
    }
    input {
      border: none;
      font-size: 1.1em;
      border-bottom: 1px solid #ccc;

      &[type='search'] {
        border-color: #08f;
      }
      &[type='button'] {
        border: 1px solid #ccc;
        padding: 0.2em 0.5em;
      }
    }
  }
} // end of .vue-abyss
</style>
