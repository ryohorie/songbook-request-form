<template>
  <v-layout column justify-center align-center class="request-container">
    <v-flex xs12 sm12 md12>
      <h2>ソングブックリクエストフォーム</h2>
      <template v-show="!isWaiting">
        <template v-if="isLogin">
          <div>
            <p>あなたのリクエスト曲：&ensp;{{ song }}</p>
            <v-form onsubmit="return false">
              <v-layout row wrap>
                <v-flex xs8>
                  <v-text-field
                    v-model="requestSong"
                    label="リクエスト曲を入力してください"
                    single-line
                    @keydown.enter="saveTrigger"
                  ></v-text-field>
                </v-flex>
                <v-flex xs4 class="wrap-btn">
                  <div>
                    <v-btn
                      small
                      color="primary"
                      :disabled="!requestSong"
                      @click="save"
                      >保存</v-btn
                    >
                  </div>
                </v-flex>
              </v-layout>
            </v-form>
          </div>
          <song-book-request-list :songs="songs" @request="request" />
          <div class="wrap-btn">
            <v-btn color="error" @click="logOut">ログアウト</v-btn>
          </div>
        </template>
      </template>
      <div id="firebaseui-auth-container" />
    </v-flex>
    <notifications position="top" />
  </v-layout>
</template>

<script>
import firebase from '@/plugins/firebase'
import 'firebase/auth'
import * as firebaseui from 'firebaseui'
import SongBookRequestList from '@/components/SongBookRequestList'

export default {
  components: {
    SongBookRequestList: SongBookRequestList
  },
  data() {
    return {
      song: '',
      requestSong: '',
      isWaiting: true,
      isLogin: false,
      user: {},
      songs: [],
      db: {}
    }
  },
  mounted() {
    /* eslint no-console: ["error", { allow: ["log"] }] */
    firebase.auth().onAuthStateChanged(async user => {
      this.isWaiting = false
      if (user) {
        this.isLogin = true
        this.user = user
        this.db = await firebase.firestore()
        this.setSongList()
      } else {
        this.isLogin = false
        this.user = []
        if (process.browser) {
          require('firebaseui/dist/firebaseui.css')
          const ui =
            firebaseui.auth.AuthUI.getInstance() ||
            new firebaseui.auth.AuthUI(firebase.auth())
          const uiConfig = {
            signInSuccessUrl: '/',
            signInOptions: [
              firebase.auth.GoogleAuthProvider.PROVIDER_ID,
              firebase.auth.EmailAuthProvider.PROVIDER_ID
            ]
          }
          ui.start('#firebaseui-auth-container', uiConfig)
        }
      }
    })
  },
  methods: {
    async save() {
      await this.request(this.requestSong)
      this.requestSong = ''
    },
    logOut() {
      firebase.auth().signOut()
      this.$forceUpdate()
    },
    saveTrigger() {
      if (event.keyCode !== 13) return
      if (!this.requestSong) return
      this.save()
    },
    async request(name) {
      await this.db
        .collection('songs')
        .doc(this.user.uid)
        .set({
          name: name
        })
      await this.setSongList()
      this.$notify({
        title: `『${name}』のリクエストを受け付けました`
      })
    },
    async setSongList() {
      const data = await this.db.collection('songs').get()
      if (data.size > 0) {
        const songs = data.docs
        const s = songs.find(s => s.id === this.user.uid)
        console.log(s)
        this.song = s ? s.data().name : '未設定'
        // 名前とカウントのオブジェクトを作る
        const songList = songs.map(s => s.data().name)
        const dict = []
        for (const key of songList) {
          dict[key] = songList.filter(function(x) {
            return x === key
          }).length
        }
        this.songs = []
        for (const key in dict) {
          this.songs.push({
            name: key,
            count: dict[key]
          })
        }
        this.songs.sort((a, b) => {
          return b.count - a.count
        })
      } else {
        this.song = '未設定'
      }
    }
  }
}
</script>
<style lang="stylus">
button
  margin-right 0
.wrap-btn
  margin auto
.request-container
  .v-input .v-label
    font-size .7em
    line-height 2.5em
</style>
