<template>
  <div class="container">
    <div
      class="step"
      v-if="currentStep === 1"
    >
      <h1>Step 1 / 5</h1>
      <div class="input-group">
        <label for="apCode">App Code</label>
        <input
          type="text"
          id="apCode"
          v-model="apCode"
          placeholder="Example: AP001"
        />
      </div>
      <div v-if="!imageData">
        <video
          ref="video"
          width="300"
          height="200"
          autoplay
        ></video>
        <button @click="captureImage">Live Capture</button>
      </div>
      <div v-else>
        <img
          :src="imageData"
          alt="Captured Image"
        />
      </div>
      <div v-if="!coordinates">
        <p>Loading...</p>
      </div>
      <div v-else>
        <p>Latitude: {{ coordinates.latitude }}</p>
        <p>Longitude: {{ coordinates.longitude }}</p>
      </div>
      <button @click="nextStep">Next</button>
    </div>
    <div
      class="step"
      v-if="currentStep === 2"
    >
      <h1>Step 2 / 5</h1>
      <h2>Mobile Number</h2>
      <div class="input-group">
        <label for="mobileNumber">Enter your mobile number</label>
        <input
          type="tel"
          id="mobileNumber"
          v-model="mobileNumber"
          placeholder="+1 (555) 555-5555"
        />
        <span v-if="!isValidMobileNumber">Invalid mobile number</span>
      </div>
      <button @click="previousStep">Previous</button>
      <button @click="nextStep">Next</button>
    </div>
    <div
      class="step"
      v-if="currentStep === 3"
    >
      <h1>Step 3 / 5</h1>
      <p>Success! Your request has been submitted.</p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const video = ref<HTMLVideoElement | null>(null)
const imageData = ref<string | null>(null)
const currentStep = ref(1)
const coordinates = ref<{ latitude: number; longitude: number } | null>(null)
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

const getLocation = () => {
  navigator.geolocation.getCurrentPosition(
    position => {
      coordinates.value = {
        latitude: position.coords.latitude,
        longitude: position.coords.longitude,
      }
    },
    error => {
      console.error('Error getting location:', error)
    },
    { enableHighAccuracy: true },
  )
}

const validateMobileNumber = () => {
  // Add your validation logic here
  const pattern = /^[+]*[(]{0,1}[0-9]{1,4}[)]{0,1}[-\s\./0-9]*$/g
  isValidMobileNumber.value = pattern.test(mobileNumber.value)
}

onMounted(() => {
  initCamera()
  getLocation()
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

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.step {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 300px;
  padding: 20px;
  border: 1px solid #ccc;
  margin: 20px;
}

.input-group {
  display: flex;
  flex-direction: column;
  margin-bottom: 10px;
}

.input-group label {
  margin-bottom: 5px;
}

.input-group input {
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  background-color: #4caf50;
  color: white;
  cursor: pointer;
  margin: 10px;
}

button:hover {
  background-color: #45a049;
}
</style>
