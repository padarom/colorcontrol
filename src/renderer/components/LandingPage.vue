<template>
  <div id="wrapper">
    <main>
      <button @click="open">
        <i class="fad fa-expand-wide fa-2x" />
      </button>

      <div class="right-side">
        <video :src-object.prop.camel="source" autoplay />
      </div>
    </main>
  </div>
</template>

<script>
  import { remote } from 'electron'

  export default {
    data () {
      return {
        source: null,
      }
    },

    methods: {
      open () {
        remote.BrowserWindow.getFocusedWindow().minimize()

        let win = new remote.BrowserWindow({
          width: 500,
          height: 400,
          transparent: true,
          frame: false,
          show: false,
          skipTaskbar: true,
          alwaysOnTop: true,
          resizable: true,
          movable: true,
          backgroundColor: 'transparent',
          webPreferences: {
            nodeIntegration: true,
          },
        })

        win.on('blur', () => win.close())

        win.show()
        win.loadURL(remote.getGlobal('winURL') + '#screensync')
        win.closeDevTools()
      },
    },
  }
</script>
