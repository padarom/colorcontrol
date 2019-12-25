<template>
  <div id="screen-area">
    <div class="draggable" />
    <main>
      <div class="position">
        <i class="fad fa-arrow-to-left" />
        {{ position[0] }}
      </div>
      <div class="position">
        <i class="fad fa-arrow-to-top" />
        {{ position[1] }}
      </div>
      <div class="size">
        <i class="fad fa-retweet-alt" />
        {{ size[0] }} x {{ size[1] }}
      </div>
    </main>
  </div>
</template>

<script>
  import { remote } from 'electron'
  import { debounce } from 'lodash'

  export default {
    data () {
      return {
        position: [0, 0],
        size: [10, 10],
      }
    },

    created () {
      let currentWindow = remote.BrowserWindow.getFocusedWindow()

      let onUpdate = debounce(() => {
        this.position = currentWindow.getPosition()
        this.size = currentWindow.getSize()
      }, 5, {
        leading: true,
        trailing: true,
      })

      onUpdate()

      currentWindow.on('move', onUpdate)
      currentWindow.on('resize', onUpdate)
    }
  }
</script>

<style lang="stylus" scoped>
html, body, #app
  background transparent !important

#screen-area
  height 100%
  padding 10px

main
  height 100%
  display flex
  flex-direction column
  justify-content center
  align-items space-around

  .position, .size
    text-align center
    color darkgray
    font-size 200%
  
.draggable
  -webkit-app-region drag

  position absolute
  top 10px
  left @top
  right @top
  bottom @top
</style>
