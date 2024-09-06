<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const video = ref<HTMLVideoElement | null>(null)
const imageData = ref<string | null>(null)
const currentStep = ref(1)

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
    // Set imageData.value instead of directly assigning
    // Now you can do something with the captured image data
    console.log('Captured image data:', imageData.value)
  }
}

onMounted(() => {
  initCamera()
  // Request location permission
  navigator.geolocation.getCurrentPosition(
    position => {
      console.log(`Location permission granted: ${position}`)
    },
    error => {
      console.error(`Error getting location permission: ${error}`)
    },
    { enableHighAccuracy: true },
  )
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

function validateMobileNumber() {
  // Add your validation logic here
}
</script>
