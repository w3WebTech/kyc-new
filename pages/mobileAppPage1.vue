<template>
  <div class="p-4 rounded-md shadow-md bg-white">
    <div class="flex justify-between items-center mb-4">
      <h2 class="text-xl font-bold">A101</h2>
    </div>
    <div class="mb-4">
      <h3 class="text-lg font-medium mb-2">Finy Wealth</h3>
      <p class="text-gray-600">Andhra Pradesh</p>
      <p class="text-gray-600">
        GPS {{ coordinates ? `${coordinates.latitude}, ${coordinates.longitude}` : 'Unable to get location' }}
      </p>
    </div>
    <VDivider />
    <div class="mb-4">
      <h4 class="text-lg font-medium mb-2">Step 1:</h4>
      <p class="text-gray-600">Please take a live photo of Name</p>

      <div class="w-full border rounded h-64 py-4 relative">
        <div v-if="showCamera">
          <!-- Live Camera Feed -->
          <video
            ref="video"
            autoplay
            playsinline
            class="absolute inset-0 w-full h-full object-cover"
          ></video>

          <div class="flex justify-center items-center absolute inset-0 z-10">
            <button
              @click="captureImage"
              class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
            >
              Capture
            </button>
          </div>
        </div>

        <!-- Captured Image -->
        <img
          v-if="capturedImage && !showCamera"
          :src="capturedImage"
          alt="Captured"
          class="absolute inset-0 w-full h-full object-cover"
        />
      </div>

      <div
        v-if="capturedImage && !showCamera"
        class="flex justify-center items-center mt-4"
      >
        <button
          @click="showCamera = true"
          class="bg-gray-400 hover:bg-gray-500 text-white font-bold py-2 px-4 rounded"
        >
          Retake
        </button>
      </div>
    </div>
    <div class="mb-4">
      <label
        for="message"
        class="block text-gray-700 font-bold mb-2"
        >Optional Message / Notes</label
      >
      <textarea
        id="message"
        rows="4"
        class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
        v-model="notes"
      ></textarea>
    </div>
    <div class="flex justify-end">
      <button
        @click="previousStep"
        class="bg-gray-400 hover:bg-gray-500 text-white font-bold py-2 px-4 rounded mr-4"
      >
        ← Previous
      </button>
      <button
        @click="nextStep"
        class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
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
const showCamera = ref(true)

const initCamera = async () => {
  try {
    // Request camera access
    const stream = await navigator.mediaDevices.getUserMedia({ video: true })

    // Set the video source
    if (video.value) {
      video.value.srcObject = stream
      video.value.play()
    }
  } catch (error) {
    console.error('Error accessing camera:', error)
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

const captureImage = () => {
  const canvas = document.createElement('canvas')
  if (video.value) {
    canvas.width = video.value.videoWidth
    canvas.height = video.value.videoHeight
    const context = canvas.getContext('2d')
    if (context) {
      context.drawImage(video.value, 0, 0, canvas.width, canvas.height)
      capturedImage.value = canvas.toDataURL('image/png')
      console.log('Captured image data:', capturedImage.value)
      showCamera.value = false // Hide camera after capture
    }
  }
}

onMounted(async () => {
  await initCamera()
  getLocation()
})

onUnmounted(() => {
  if (video.value && video.value.srcObject) {
    const tracks = (video.value.srcObject as MediaStream).getTracks()
    tracks.forEach(track => track.stop())
  }
})
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
