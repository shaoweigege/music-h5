<template>
  <transition name="list-fade">
    <div class="playlist" v-show="showFlag" @click="hide">

      <div class="list-wrapper" @click.stop>
        <div class="list-header">
          <h1 class="title">
            <i class="icon" :class="iconMode"  @click="changeMode"></i>
            <span class="text">{{modeText}}</span>
            <span class="clear" @click="clickConfirm"><i class="icon-clear"></i></span>
          </h1>
        </div>
        <scroll :data="sequenceList" :refreshDelay="refreshDelay" ref="listContent"  class="list-content" >
          <transition-group ref="list" name="list" tag="ul">
            <li ref="songItem" class="item" v-for="(item,index) in sequenceList" :key="item.id" @click="selectItem(item,index)">
              <i class="current" :class="getCurrentIcon(item)"></i>
              <span class="text">{{item.name}}</span>
              <span @click="toggleFavorite(item)" class="like">
                <i :class="getFavoriteIcon(item)"></i>
              </span>
              <span class="delete" @click.stop="deleteOne(item)">
                <i class="icon-delete"></i>
              </span>
            </li>
          </transition-group>
        </scroll>
        <div class="list-operate">
          <div @click="addSong"  class="add">
            <i class="icon-add"></i>
            <span class="text">添加歌曲到队列</span>
          </div>
        </div>
        <div @click="hide" class="list-close">
          <span>关闭</span>
        </div>
      </div>
      <confirm ref="confirm" text="是否清空播放列表" confirmBtnText="清空" @confirm="clearPlaylist"></confirm>
      <add-song ref="addSong"></add-song>
    </div>
  </transition>
</template>

<script type="text/ecmascript-6">
  import {mapMutations, mapActions} from 'vuex'
  import Scroll from 'base/scroll/scroll'
  import {Config} from 'common/js/config'
  import Confirm from 'base/confirm/confirm'
  import {playerMixin} from 'common/js/mixin'
  import AddSong from 'components/add-song/add-song'

  export default {
    mixins: [playerMixin],
    data () {
      return {
        showFlag: false,
        refreshDelay: 100
      }
    },
    computed: {
      modeText() {
        return this.mode === Config.playMode.sequence ? '顺序播放' : this.mode === Config.playMode.random ? '随机播放' : '单曲循环'
      }
    },
    watch: {
      // 如果currentSong改变，scroll到这个歌曲的位置
      currentSong(newSong, oldSong) {
        if (!this.showFlag || newSong.id === oldSong.id) {
          return
        }
        setTimeout(() => {
          this.scrollToCurrent(newSong)
        }, 20)
      }
    },
    methods: {
      show() {
        this.showFlag = true
        // 展开遮罩层手动刷新一下scroll
        setTimeout(() => {
          this.$refs.listContent.refresh()
          this.scrollToCurrent(this.currentSong)
        }, 20)
      },
      hide() {
        this.showFlag = false
      },
      getCurrentIcon(item) {
        if (item.id === this.currentSong.id) {
          return 'icon-play'
        }
        return ''
      },
      // 添加歌曲到队列
      selectItem(item, index) {
        // 如果是随机播放
        if (this.mode === Config.playMode.random) {
          index = this.playList.findIndex((song) => {
            return song.id === item.id
          })
        }
        this.setCurrentIndex(index)
        this.setPlayingStatus(true)
      },
      scrollToCurrent(current) {
        const index = this.sequenceList.findIndex((song) => {
          return current.id === song.id
        })
        this.$refs.listContent.scrollToElement(this.$refs.list.$el.children[index], 300)
      },
      // 删除某一首歌
      deleteOne(item) {
        this.deleteSong(item)
        if (!this.playList.length) {
          this.hide()
        }
      },
      addSong() {
        this.$refs.addSong.show()
      },
      clickConfirm() {
        this.$refs.confirm.show()
      },
      clearPlaylist() {
        this.deleteSongList()
        this.hide()
      },
      ...mapMutations({
        setCurrentIndex: 'SET_CURRENT_INDEX',
        setPlayingStatus: 'SET_PLAYING_STATE'
      }),
      ...mapActions([
        'deleteSong',
        'deleteSongList'
      ])
    },
    components: {
      Scroll,
      Confirm,
      AddSong
    }
  }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"
  @import "~common/stylus/mixin"

  .playlist
    position: fixed
    left: 0
    right: 0
    top: 0
    bottom: 0
    z-index: 200
    background-color: $color-background-d
    &.list-fade-enter-active, &.list-fade-leave-active
      transition: opacity 0.3s
      .list-wrapper
        transition: all 0.3s
    &.list-fade-enter, &.list-fade-leave-to
      opacity: 0
      .list-wrapper
        transform: translate3d(0, 100%, 0)
    &.list-fade-enter
    .list-wrapper
      position: absolute
      left: 0
      bottom: 0
      width: 100%
      background-color: $color-highlight-background
      .list-header
        position: relative
        padding: 20px 30px 10px 20px
        .title
          display: flex
          align-items: center
          .icon
            margin-right: 10px
            font-size: 30px
            color: $color-theme-d
          .text
            flex: 1
            font-size: $font-size-medium
            color: $color-text-l
          .clear
            extend-click()
            .icon-clear
              font-size: $font-size-medium
              color: $color-text-d
      .list-content
        max-height: 240px
        overflow: hidden
        .item
          display: flex
          align-items: center
          height: 40px
          padding: 0 30px 0 20px
          overflow: hidden
          &.list-enter-active, &.list-leave-active
            transition: all 0.1s
          &.list-enter, &.list-leave-to
            height: 0
          .current
            flex: 0 0 20px
            width: 20px
            font-size: $font-size-small
            color: $color-theme-d
          .text
            flex: 1
            no-wrap()
            font-size: $font-size-medium
            color: $color-text-d
          .like
            extend-click()
            margin-right: 15px
            font-size: $font-size-small
            color: $color-theme
            .icon-favorite
              color: $color-sub-theme
          .delete
            extend-click()
            font-size: $font-size-small
            color: $color-theme
      .list-operate
        width: 140px
        margin: 20px auto 30px auto
        .add
          display: flex
          align-items: center
          padding: 8px 16px
          border: 1px solid $color-text-l
          border-radius: 100px
          color: $color-text-l
          .icon-add
            margin-right: 5px
            font-size: $font-size-small-s
          .text
            font-size: $font-size-small
      .list-close
        text-align: center
        line-height: 50px
        background: $color-background
        font-size: $font-size-medium-x
        color: $color-text-l
</style>
