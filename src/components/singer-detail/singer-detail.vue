<template>
  <transition name="slide">
    <music-list :title="title" :bg-image="bgImage" :songs="songs"></music-list>
  </transition>
</template>

<script type="text/ecmascript-6">
  import MusicList from 'components/music-list/music-list'
  import {getSingerDetail} from 'api/singer'
  import {ERR_OK} from 'api/config'
  import {createSong, isValidMusic} from 'common/js/song'
  import {mapGetters} from 'vuex'

  export default{
    data(){
      return {
        songs: []
      }
    },
    created(){
      this._getDetail()
    },
    methods: {
      _getDetail(){
        if (!this.singer.id) {
          this.$router.push('/singer')
          return
        }
        getSingerDetail(this.singer.id).then((res) => {
          if (res.code === ERR_OK) {
            this.songs = this._normalizeSinger(res.data.list)
          }
        })
      },
      _normalizeSinger(list){
        let ret = []
        list.forEach((item) => {
          // 这里牛批
          let {musicData} = item
//          console.log('item', item)
//          console.log('musicData', musicData)
          if (isValidMusic(musicData)) {
            ret.push(createSong(musicData))
          }
        })
        return ret
      }
    },
    computed: {
      title(){
        return this.singer.name
      },
      bgImage(){
        return this.singer.avatar
      },
      ...mapGetters([
        'singer'
      ])
    },
    components: {
      MusicList
    }
  }
</script>

<style lang="stylus" scoped rel="stylesheet/stylus">
  .slide-enter-active, .slide-leave-active
    transition all 0.3s

  .slide-enter, .slide-leave-to
    transform translate3d(100%, 0, 0)
</style>
