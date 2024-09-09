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

      <div class="w-full border rounded h-20 py-4">
        <div class="flex justify-center items-center">
          <img
            src="@/public/picture.png"
            alt=""
            class="h-10 w-10"
          />
        </div>

        <div
          class="flex justify-center items-center"
          @click="captureImage"
        >
          Live Capture
        </div>
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
      ></textarea>
    </div>
    <div class="flex justify-end">
      <button class="bg-gray-400 hover:bg-gray-500 text-white font-bold py-2 px-4 rounded mr-4">← Previous</button>
      <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">Next →</button>
    </div>
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
const name = ref('')
const address = ref('')
const notes = ref('')

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
    canvas.getContext('2d')?.drawImage(video.value, 0, 0, canvas.width, canvas.height)
    capturedImage.value = canvas.toDataURL('image/png')
    console.log('Captured image data:', capturedImage.value)
  }
}

const validateMobileNumber = () => {
  // Add your validation logic here
  const pattern = /^[+]*[(]{0,1}[0-9]{1,4}[)]{0,1}[-\s\./0-9]*$/g
  isValidMobileNumber.value = pattern.test(mobileNumber.value)
}

onMounted(async () => {
  await initCamera()
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

function submit() {
  // Here you would send the collected data to your backend
  console.log('Collected data:', {
    apCode: apCode.value,
    imageData: imageData.value,
    coordinates: coordinates.value,
    mobileNumber: mobileNumber.value,
    name: name.value,
    address: address.value,
    notes: notes.value,
  })
}
</script>

<style scoped>
.step {
  display: none;
}

.step.active {
  display: block;
}
</style>
