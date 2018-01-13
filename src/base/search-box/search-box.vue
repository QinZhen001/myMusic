<template>
  <div class="search-box">
    <i class="icon-search"></i>
    <input class="box" ref="query" v-model="query" :placeholder="placeholder"/>
    <i class="icon-dismiss" @click="clear" v-show="query"></i>
  </div>
</template>

<script type="text/ecmascript-6">
  import {debounce} from 'common/js/util'

  export default{
    props: {
      placeholder: {
        type: String,
        default: '搜索歌曲、歌手'
      }
    },
    data(){
      return {
        query: ''
      }
    },
    created(){
      //这里在created的时候watch也很巧妙
      // 这样可以监听到最开始的值 而不止止是改变后的值
      this.$watch('query', debounce((newQuery) => {
        this.$emit('query', newQuery)
      }, 200))
    },
    methods: {
      clear(){
        this.query = ''
      },
      setQuery(query){
        this.query = query
      },
      blur() {
        this.$refs.query.blur()
      }
    }
  }
</script>

<style lang="stylus" scoped rel="stylesheet/stylus">
  @import "~common/stylus/variable"

  .search-box
    display flex
    align-items center
    box-sizing border-box
    width 100%
    padding 0 6px
    height 40px
    background: $color-highlight-background
    border-radius 6px
    .icon-search
      font-size 24px
      color $color-background
    .box
      flex 1
      margin 0 5px
      line-height 18px
      background $color-highlight-background
      color $color-text
      font-size: $font-size-medium
      outline: none
      &::placeholder
        color $color-text-d
    .icon-dismiss
      font-size 16px
      color $color-background
</style>

