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

        <video
          ref="video"
          autoplay
          playsinline="true"
          v-if="showCamera"
          class="absolute inset-0 w-full h-full object-cover"
          @loadedmetadata="videoLoaded"
        >
          <img
            src="@/public/picture.png"
            alt="Placeholder"
            class="absolute flex justify-center items-center w-10 h-10 object-cover z-10"
          />
        </video>

        <!-- Captured Image -->
        <img
          v-if="capturedImage"
          :src="capturedImage"
          class="absolute inset-0 w-full h-full object-cover"
          @click="toggleCamera"
        />
      </div>

      <!-- Capture button only shown when live camera feed is displayed -->
      <div
        v-if="showCamera"
        class="flex justify-center mt-4"
      >
        <button
          @click="captureImage"
          class="bg-blue-900 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
        >
          Capture
        </button>
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

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const video = ref<HTMLVideoElement | null>(null)
const capturedImage = ref<string | null>(null)
const coordinates = ref<{ latitude: number; longitude: number } | null>(null)
const locationLoading = ref(true)
const notes = ref('')
const showCamera = ref(false)
const videoLoaded = () => {
  if (video.value) {
    initCamera()
  }
}
const toggleCamera = () => {
  showCamera.value = !showCamera.value
  console.log(showCamera.value, 'showCamera.value')
  if (showCamera.value) {
    initCamera()
  }
}

const initCamera = async () => {
  try {
    console.log('Initializing camera...')
    const stream = await navigator.mediaDevices.getUserMedia({ video: true })
    console.log('Got stream:', stream)
    if (video.value) {
      video.value.srcObject = stream
      video.value.play()
    } else {
      console.error('Video element is null')
    }
  } catch (error) {
    console.error('Error accessing camera:', error)
  }
}

const captureImage = () => {
  const canvas = document.createElement('canvas')
  if (video.value) {
    canvas.width = video.value.videoWidth
    canvas.height = video.value.videoHeight
    const context = canvas.getContext('2d')
    if (context) {
      context.drawImage(video.value, 0, 0, canvas.width, canvas.height)
      const imageData = canvas.toDataURL('image/png')
      capturedImage.value = imageData
      showCamera.value = false
    } else {
      console.error('Canvas context is null')
    }
  } else {
    console.error('Video element is null')
  }
}

const getLocation = async () => {
  try {
    navigator.geolocation.getCurrentPosition(
      position => {
        coordinates.value = {
          latitude: position.coords.latitude,
          longitude: position.coords.longitude,
        }
        locationLoading.value = false
      },
      error => {
        console.error('Error getting location:', error)
        locationLoading.value = false
      },
      { enableHighAccuracy: true },
    )
  } catch (error) {
    console.error('Error getting location:', error)
  }
}

onMounted(async () => {
  getLocation()
  await initCamera()
})

onUnmounted(() => {
  if (video.value && video.value.srcObject) {
    const tracks = (video.value.srcObject as MediaStream).getTracks()
    tracks.forEach(track => track.stop())
  }
})
</script>

<style scoped>
.divider {
  border-bottom: 1px solid #ccc;
  margin-bottom: 20px;
}

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
