<template>
  <div>
    <video
      ref="video"
      width="640"
      height="480"
    />
    <button @click="switchCamera">Switch Camera</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      stream: null,
      videoTracks: [],
    }
  },
  mounted() {
    this.requestCameraAccess()
  },
  methods: {
    async requestCameraAccess() {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true, audio: false })
        this.stream = stream
        this.videoTracks = stream.getVideoTracks()
        this.$refs.video.srcObject = stream
      } catch (error) {
        console.error('Error requesting camera access:', error)
      }
    },
    switchCamera() {
      if (this.videoTracks.length > 1) {
        this.videoTracks[0].stop()
        this.videoTracks[1].start()
        this.$refs.video.srcObject = new MediaStream([this.videoTracks[1]])
      } else {
        console.log('Only one camera available')
      }
    },
  },
}
</script>
