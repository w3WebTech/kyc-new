<template>
  <div>
    <video
      ref="video"
      width="640"
      height="480"
      autoplay
    ></video>
    <button @click="capture">Capture</button>
    <canvas
      ref="canvas"
      width="640"
      height="480"
      style="display: none"
    ></canvas>
    <img
      v-if="image"
      :src="image"
      alt="Captured Image"
    />
  </div>
</template>

<script>
export default {
  data() {
    return {
      image: null,
    }
  },
  mounted() {
    this.startCamera()
  },
  methods: {
    async startCamera() {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true })
        this.$refs.video.srcObject = stream
      } catch (error) {
        console.error('Error accessing camera', error)
      }
    },
    capture() {
      const canvas = this.$refs.canvas
      const video = this.$refs.video
      const context = canvas.getContext('2d')

      context.drawImage(video, 0, 0, canvas.width, canvas.height)
      this.image = canvas.toDataURL('image/png')
    },
  },
}
</script>
