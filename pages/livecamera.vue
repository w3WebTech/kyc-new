<template>
  <div>
    <video
      ref="video"
      width="640"
      height="480"
      autoplay
    ></video>
    <div>
      <button @click="switchCamera">Switch Camera</button>
      <button @click="capture">Capture</button>
    </div>
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
      currentStream: null,
      facingMode: 'user', // 'user' for front camera, 'environment' for back camera
    }
  },
  mounted() {
    this.startCamera()
  },
  methods: {
    async startCamera() {
      //   const stream = await navigator.mediaDevices.getUserMedia({ video: true })
      try {
        // Stop any existing stream before starting a new one
        if (this.currentStream) {
          const tracks = this.currentStream.getTracks()
          tracks.forEach(track => track.stop())
        }

        // Get the media stream with the current facing mode
        const stream = await navigator.mediaDevices.getUserMedia({
          video: { facingMode: this.facingMode },
        })

        this.currentStream = stream
        this.$refs.video.srcObject = stream
      } catch (error) {
        console.error('Error accessing camera', error)
      }
    },
    async switchCamera() {
      this.facingMode = this.facingMode === 'user' ? 'environment' : 'user'
      await this.startCamera()
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

<style scoped>
/* Add some basic styling if needed */
</style>
