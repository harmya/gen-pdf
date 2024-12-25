<template>
  <h1>Use this page to create pdfs</h1>
  <div class="container">
    <!-- Assets Section -->
    <div class="section section-1">
      <h2>Assets</h2>
      <input type="file" multiple @change="handleImageUpload" />
      <div class="assets-preview">
        <div
          class="asset-wrapper"
          v-for="(asset, index) in assets"
          :key="index"
          draggable="true"
          @dragstart="ondragstart"
        >
          <img :src="asset" alt="Asset" class="asset" />
        </div>
      </div>
    </div>

    <!-- Canvas Section -->
    <div class="section section-2">
      <canvas id="canvas" ref="canvas" @drop="ondrop" @dragover.prevent> </canvas>
    </div>

    <!-- Save Section -->
    <div class="section section-3">
      <h2>Additional Content</h2>
    </div>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  width: 100%;
  height: 100%;
}

.section {
  padding: 10px;
  box-sizing: border-box;
}

.section-1 {
  flex: 0 0 20%;
}

.section-2 {
  flex: 0 0 60%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.section-3 {
  flex: 0 0 20%;
}

.assets-preview {
  display: flex;
  flex-direction: column;
  align-items: center;
  overflow-y: hidden;
  overflow-x: auto;
  border: 1px solid #ddd;
  border-radius: 8px;
  min-height: 100px;
}

.asset {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100px;
  height: 100px;
  margin: 10px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

#canvas {
  border: 1px solid white;
  display: block;
}
</style>
<script lang="ts">
import * as fabric from 'fabric'
import { console } from 'inspector'

export default {
  data() {
    return {
      assets: [] as string[],
      canvas: null as fabric.Canvas | null,
    }
  },

  mounted() {
    const ref = this.$refs.canvas as HTMLCanvasElement
    const section = ref.parentElement as HTMLElement

    ref.height = section.clientHeight - 50
    ref.width = section.clientHeight * 0.7069

    this.canvas = new fabric.Canvas(ref)
  },

  methods: {
    handleImageUpload(event: Event) {
      const input = event.target as HTMLInputElement
      if (input.files) {
        Array.from(input.files).forEach((file) => {
          const reader = new FileReader()
          reader.onload = (e) => {
            if (e.target && e.target.result) {
              this.assets.push(e.target.result as string)
            }
          }
          reader.readAsDataURL(file)
        })
      }
    },
    ondragstart(event: DragEvent) {
      const target = event.target as HTMLElement
      const img_src = target.getAttribute('src') as string
      event.dataTransfer?.setData('text/plain', img_src)
    },

    async ondrop(event: DragEvent) {
      console.log(event)
      const data = event.dataTransfer?.getData('text/plain')
      if (data) {
        fabric.FabricImage.fromURL(data).then((img) => {
          img.set({
            left: event.offsetX,
            top: event.offsetY,
          })
          this.canvas?.add(img)
        })
      }
    },
  },
}
</script>
