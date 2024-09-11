<template>
  <div class="rounded-md shadow-md bg-white">
    <div class="flex justify-between items-center p-3">
      <h2 class="text-xl text-blue-900 font-bold">A101</h2>
    </div>
    <div class="border-b">
      <h3 class="font-medium mb-1 px-3">Finy Wealth</h3>
      <p class="text-gray-600 font-medium px-3">Andhra Pradesh</p>
      <p class="text-gray-600 px-3">
        <span class="text-blue-900">GPS</span>
        {{ coordinates ? `${coordinates.latitude}, ${coordinates.longitude}` : 'Unable to get location' }}
      </p>
    </div>

    <div class="mb-2 p-3">
      <h4 class="text-lg text-blue-900 font-medium mb-2">Step 1:</h4>
      <p class="text-gray-600">Please take a live photo of Name</p>

      <div class="w-full border-dotted border-2 rounded h-60 relative">
        <!-- Show picture image initially -->
        <div
          class="absolute inset-0 flex flex-col justify-center items-center py-20"
          v-if="!showCamera && !capturedImage"
          @click="toggleCamera"
        >
          <img
            src="@/public/picture.png"
            alt="Placeholder"
            class="w-10 h-10 object-cover"
          />
          <div class="mt-2 font-bold">Live Capture</div>
        </div>

        <!-- Live Camera Feed -->

        <!-- <video
          ref="video"
          autoplay
          playsinline="true"
          v-if="showCamera"
          class="absolute inset-0 w-full h-full object-cover"
          @loadedmetadata="videoLoaded"
        ></video> -->
        <video
          ref="video"
          v-if="showCamera && !capturedImage"
          class="absolute inset-0 w-full h-full object-cover"
          autoplay
        ></video>

        <!-- Captured Image -->
        <img
          v-if="capturedImage"
          :src="capturedImage"
          class="absolute inset-0 w-full h-full object-cover"
        />
      </div>

      <!-- Capture button only shown when live camera feed is displayed -->
      <div
        v-if="showCamera && !capturedImage"
        class="p-3"
      >
        <button
          @click="capture"
          class="bg-blue-900 hover:bg-blue-700 text-white font-bold py-2 rounded mr-2 w-full"
        >
          Capture
        </button>
        <!-- <button
          @click="switchCamera"
          class="bg-blue-900 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded w-100"
        >
          Switch Camera
        </button> -->
      </div>
    </div>
    <div class="p-3">
      <textarea
        id="message"
        rows="3"
        v-model="notes"
        placeholder="  Optional Message / Notes"
        class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
      ></textarea>
    </div>
    <div class="flex justify-between p-3">
      <button
        @click="previousStep"
        class="border border-gray-500 text-gray-500 font-bold py-2 px-4 w-100 mr-2 rounded"
      >
        ← Previous
      </button>
      <button
        @click="nextStep"
        class="bg-blue-900 hover:bg-blue-700 text-white font-bold py-2 px-4 ml-2 w-100 rounded"
      >
        Next →
      </button>
    </div>
  </div>
</template>
<script  lang="ts">
export default {
  data() {
    return {
      image: null,
      currentStream: null,
      facingMode: 'environment',
      showCamera: false, // 'user' for front camera, 'environment' for back camera
    }
  },
  mounted() {
    this.startCamera(), this.getLocation()
  },
  methods: {
    toggleCamera() {
      this.showCamera = !this.showCamera
      if (this.showCamera) {
        this.startCamera()
      }
    },
    async startCamera() {
      this.currentStream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: this.facingMode } })
      this.$refs.video.srcObject = this.currentStream
    },
    // async switchCamera() {
    //   this.facingMode = this.facingMode === 'user' ? 'environment' : 'user'
    //   await this.startCamera()
    // },
    capture() {
      const canvas = document.createElement('canvas')
      const video = this.$refs.video
      const context = canvas.getContext('2d')
      canvas.width = video.videoWidth
      canvas.height = video.videoHeight
      context.drawImage(video, 0, 0, canvas.width, canvas.height)
      this.capturedImage = canvas.toDataURL('image/png')
      console.log(this.capturedImage, '  this.capturedImage ')
    },
    async getLocation() {
      debugger
      try {
        const permission = await navigator.permissions.query('geolocation')
        if (permission.state === 'denied') {
          console.error('Location permission denied')
          return
        }

        if (permission.state === 'prompt') {
          const result = await navigator.geolocation.requestPermission()
          if (result !== 'granted') {
            console.error('Location permission denied')
            return
          }
        }

        navigator.geolocation.getCurrentPosition(
          position => {
            this.coordinates = {
              latitude: position.coords.latitude,
              longitude: position.coords.longitude,
            }
            this.locationLoading = false
          },
          error => {
            console.error('Error getting location:', error)
            this.locationLoading = false
          },
          { enableHighAccuracy: true },
        )
      } catch (error) {
        console.error('Error getting location:', error)
      }
    },
  },
}

// ... (no changes)
</script>

<style scoped>
.relative {
  position: relative;
}
.absolute {
  position: absolute;
}
.inset-0 {
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
}
.object-cover {
  object-fit: cover;
}
</style>
