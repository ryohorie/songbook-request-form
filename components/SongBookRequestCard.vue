<template>
  <v-layout row wrap class="wrap-card" @click="request">
    <v-flex xs10 sm6 class="name">
      <p>{{ song.name }}</p>
    </v-flex>
    <v-flex xs2 sm4 class="vote">
      <p>{{ song.count }}票</p>
    </v-flex>
  </v-layout>
</template>
<script>
export default {
  props: {
    song: {
      type: Object,
      default: () => {}
    }
  },
  methods: {
    request() {
      this.$emit('request', this.song.name)
    }
  }
}
</script>

<style lang="stylus" scoped>
.wrap-card
  border 1px solid #ccc
  line-height 3em
  box-shadow 0px 2px 1px 0px #ccc
  cursor pointer
  margin-bottom 7px
.name
  padding-left 1em
  padding-right 0.5em
p
  margin 0
  overflow hidden
  text-overflow ellipsis
  white-space nowrap
</style>
