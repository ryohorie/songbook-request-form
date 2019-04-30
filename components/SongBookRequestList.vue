<template>
  <div>
    <song-book-request-card
      v-for="(song, index) in songs"
      :key="index"
      :song="song"
      @request="request"
    />
  </div>
</template>
<script>
import SongBookRequestCard from '@/components/SongBookRequestCard'
export default {
  components: {
    SongBookRequestCard: SongBookRequestCard
  },
  props: {
    songs: {
      type: Array,
      default: () => []
    }
  },
  methods: {
    request(name) {
      this.$emit('request', name)
    }
  }
}
</script>

<style></style>
