<template>
  <scroll class="suggest" ref="suggest" :data="result" :pullup="pullup"
          :beforeScroll="beforeScroll" @scrollToEnd="searchMore" @beforeScroll="listScroll">
    <ul class="suggest-list">
      <li class="suggest-item" @click.stop="selectItem(item)" v-for="item in result">
        <div class="icon">
          <i :class="getIconCls(item)"></i>
        </div>
        <div class="name">
          <p class="text" v-html="getDisplayName(item)"></p>
        </div>
      </li>
      <loading v-show="hasMore" title=""></loading>
    </ul>
    <div class="no-result-wrapper" v-show="!hasMore && !result.length">
      <no-result title="抱歉，暂无搜索结果"></no-result>
    </div>
  </scroll>
</template>

<script type="text/ecmascript-6">
  import Scroll from 'base/scroll/scroll'
  import Loading from 'base/loading/loading'
  import NoResult from 'base/no-result/no-result'
  import {search} from 'api/search'
  import {ERR_OK} from 'api/config'
  import {createSong, isValidMusic} from 'common/js/song'
  import {mapMutations, mapActions} from 'vuex'
  import Singer from 'common/js/singer'

  const TYPE_SINGER = 'singer'
  const PERPAGE = 20

  //这里滑动的时候 search-box的input要失去焦点 也就是blur()

  export default{
    props: {
      showSinger: {
        type: Boolean,
        default: true
      },
      query: {
        type: String,
        default: ''
      }
    },
    data(){
      return {
        page: 1,
        pullup: true,
        beforeScroll: true,
        hasMore: true,
        result: []
      }
    },
    methods: {
      refresh(){
        this.$refs.suggest.refresh()
      },
      search(){
        this.page = 1
        this.hasMove = true
        this.$refs.suggest.scrollTo(0, 0)
        search(this.query, this.page, this.showSinger, PERPAGE).then((res) => {
          if (res.code === ERR_OK) {
            this.result = this._genResult(res.data)
            this._checkMore(res.data)
          }
        })
      },
      searchMore(){
        if (!this.hasMore) {
          return
        }
        this.page++
        search(this.query, this.page, this.showSinger, PERPAGE).then((res) => {
          if (res.code === ERR_OK) {
            this.result = this.result.concat(this._genResult(res.data))
            this._checkMore(res.data)
          }
        })
      },
      selectItem(item){
        if (item.type === TYPE_SINGER) {
          const singer = new Singer({
            id: item.singermid,
            name: item.singername
          })
          this.$router.push({
            path: `/search/${singer.id}`
          })
          this.setSinger(singer)
        } else {
          this.insertSong(item)
        }
        this.$emit('select', item)
      },
      _checkMore(data){
        const song = data.song
        if (!song.list.length || (song.curnum + (song.curpage - 1) * PERPAGE) >= song.totalnum) {
          this.hasMove = false
        }
      },
      _genResult(data){
        console.log('_genResult', data)
        let ret = []
        if (data.zhida && data.zhida.singerid) {
          //这个解构写法
          console.log('data.zhida', data.zhida)
          ret.push({...data.zhida, ...{type: TYPE_SINGER}})
        }
        if (data.song) {
          ret = ret.concat(this._normalizeSongs(data.song.list))
        }
        return ret
      },
      getIconCls(item){
        if (item.type === TYPE_SINGER) {
          return 'icon-mine'
        } else {
          return 'icon-music'
        }
      },
      getDisplayName(item){
        if (item.type === TYPE_SINGER) {
          return item.singername
        } else {
          return `${item.name} - ${item.singer}`
        }
      },
      _normalizeSongs(list){
        let ret = []
        list.forEach((musicData) => {
          if (isValidMusic(musicData)) {
            ret.push(createSong(musicData))
          }
        })
        return ret
      },
      listScroll(){
        this.$emit('listScroll')
      },
      ...mapMutations({
        setSinger: 'SET_SINGER'
      }),
      ...mapActions([
        'insertSong'
      ])
    },
    watch: {
      query(newQuery){
        if (!newQuery) {
          return
        }
        this.search(newQuery)
      }
    },
    components: {
      Scroll,
      Loading,
      NoResult
    }
  }
</script>

<style lang="stylus" scoped rel="stylesheet/stylus">
  @import "~common/stylus/variable"
  @import "~common/stylus/mixin"

  .suggest
    height 100%
    overflow: hidden
    .suggest-list
      padding 0 30px
      .suggest-item
        display flex
        align-items center
        padding-bottom 20px
        .icon
          flex 0 0 30px
          width 30px
          [class^="icon-"]
            font-size $font-size-medium
            color $color-text-d
        .name
          flex 1
          font-size: $font-size-medium
          color: $color-text-d
          overflow: hidden
          .text
            no-wrap()

    .no-result-wrapper
      position absolute
      width 100%
      top 50%
      transform translateY(-50%)
</style>
