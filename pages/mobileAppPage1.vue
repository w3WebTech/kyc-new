<template>
  <div class="border boder-2 rounded-md p-5">
    <div
      class="step"
      v-if="currentStep === 1"
    >
      <h1>Step 1 / 5</h1>
      <div>
        <VTextField
          v-model="apCode"
          label="Code "
          required
          class="py-2"
          width="300"
        ></VTextField>
      </div>
      <div v-if="!imageData">
        <video
          ref="video"
          width="300"
          height="200"
          autoplay
        ></video>
        <button
          @click="captureImage"
          width="300"
          style="
            background-color: blue;
            color: white;
            width: 280px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            padding: 3px;
            text: bold;
          "
        >
          Live Capture
        </button>
      </div>
      <div v-else>
        <img
          :src="imageData"
          alt="Captured Image"
        />
      </div>
      <div v-if="locationLoading">
        <p>Loading location...</p>
      </div>
      <div v-else-if="coordinates">
        <p>Latitude: {{ coordinates.latitude }}</p>
        <p>Longitude: {{ coordinates.longitude }}</p>
      </div>
      <div v-else>
        <p>Error loading location</p>
      </div>
      <div style="display: flex; justify-content: flex-end; margin-top: 20px">
        <button
          @click="nextStep"
          style="
            background-color: blue;
            color: white;
            padding: 8px 16px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
          "
        >
          Next
        </button>
      </div>
    </div>
    <!-- Rest of your template -->
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const video = ref<HTMLVideoElement | null>(null)
const imageData = ref<string | null>(null)
const currentStep = ref(1)
const coordinates = ref<{ latitude: number; longitude: number } | null>(null)
const locationLoading = ref(true)
const mobileNumber = ref('')
const isValidMobileNumber = ref(true)

const initCamera = async () => {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({ video: true })
    if (video.value) {
      video.value.srcObject = stream
    }
  } catch (error) {
    console.error('Error accessing camera:', error)
    // Handle error here
  }
}

const captureImage = () => {
  const canvas = document.createElement('canvas')
  if (video.value) {
    canvas.width = video.value.videoWidth
    canvas.height = video.value.videoHeight
    canvas.getContext('2d')?.drawImage(video.value, 0, 0, canvas.width, canvas.height)
    imageData.value = canvas.toDataURL('image/png')
    console.log('Captured image data:', imageData.value)
  }
}

const getLocation = async () => {
  try {
    const position = await new Promise((resolve, reject) => {
      navigator.geolocation.getCurrentPosition(resolve, reject, { enableHighAccuracy: true })
    })
    coordinates.value = {
      latitude: position.coords.latitude,
      longitude: position.coords.longitude,
    }
  } catch (error) {
    console.error('Error getting location:', error)
  } finally {
    locationLoading.value = false
  }
}

const validateMobileNumber = () => {
  // Add your validation logic here
  const pattern = /^[+]*[(]{0,1}[0-9]{1,4}[)]{0,1}[-\s\./0-9]*$/g
  isValidMobileNumber.value = pattern.test(mobileNumber.value)
}

onMounted(async () => {
  await initCamera()
  await getLocation()
})

onUnmounted(() => {
  if (video.value && video.value.srcObject) {
    const tracks = video.value.srcObject.getTracks()
    tracks.forEach(track => track.stop())
  }
})

const apCode = ref('')

function previousStep() {
  currentStep.value--
}

function nextStep() {
  currentStep.value++
}
</script>
