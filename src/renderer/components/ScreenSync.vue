<template>
  <div id="screen-area">
    <div class="draggable" />
    <canvas ref="canvas" />
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
  import { remote, desktopCapturer } from 'electron'
  import { debounce } from 'lodash'

  let wait = (ms) => new Promise(resolve => setTimeout(resolve, ms))

  export default {
    data () {
      return {
        position: [0, 0],
        size: [10, 10],
        imageCapture: null,
      }
    },

    async mounted () {
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

      await this.createStream()
      this.updatePreview()
    },

    watch: {
      position () {
        this.updatePreview()
      },

      size () {
        this.updatePreview()
      },
    },

    methods: {
      async createStream () {
        let sources = await desktopCapturer.getSources({ types: ['screen'] })

        for (const source of sources) {
          if (source.id !== 'screen:0:0') continue
          const stream = await navigator.mediaDevices.getUserMedia({
            audio: false,
            video: {
              mandatory: {
                chromeMediaSource: 'desktop',
                chromeMediaSourceId: source.id,
              },
            },
          })

          const track = stream.getVideoTracks()[0]
          this.imageCapture = new ImageCapture(track)
        }
      },

      async updatePreview () {
        let frame = [...this.position, ...this.size]

        let focused = remote.BrowserWindow.getFocusedWindow()
        focused.setOpacity(0)
        await wait(10)

        let bitmap = await this.imageCapture.grabFrame()

        let canvas = this.$refs.canvas
        let context = canvas.getContext('2d')
        canvas.width = this.size[0]
        canvas.height = this.size[1]
        context.clearRect(0, 0, canvas.width, canvas.height)
        context.drawImage(bitmap, ...frame, 0, 0, ...this.size)
        bitmap.close()

        await wait(10)
        focused.setOpacity(1)
      },
    },
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
  position relative
  z-index 10

  display flex
  flex-direction column
  justify-content center
  align-items space-around

  .position, .size
    text-align center
    color darkgray
    font-size 200%
  
canvas
  position absolute
  z-index 1
  top 0
  left 0
  //width 100%
  //height 100%

.draggable
  -webkit-app-region drag

  position absolute
  z-index 5
  top 3px
  left @top
  right @top
  bottom @top
  border 2px solid white
</style>
