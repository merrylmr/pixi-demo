<template>
  <div class="home">
    <div id="canvas">

    </div>
  </div>
</template>

<script lang="ts">
  import {defineComponent, onMounted} from 'vue'
  import * as PIXI from 'pixi.js'

  export default defineComponent({
    name: 'home',
    setup() {

      onMounted(() => {
        const app = new PIXI.Application({
          width: window.innerWidth,
          height: window.innerHeight,
          backgroundColor: 0xffffff
        });
        (document.getElementById('canvas') as HTMLElement).appendChild(app.view)
        console.log('lalal')
      })
      return {}
    }
  })

</script>
