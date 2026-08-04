<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { QrcodeStream } from 'vue-qrcode-reader'
import { useToast, POSITION } from 'vue-toastification'
import coscupLogo from './assets/coscup.svg'

const decodedString = ref('')
const token = ref<string | null>(null)
const toast = useToast()
const img = ref<string | null>(null)

onMounted(() => {
  const urlParams = new URLSearchParams(window.location.search)
  const tokenFromUrl = urlParams.get('token')

  if (!tokenFromUrl) {
    return ;
  }

  token.value = tokenFromUrl

  fetch(`https://geotrainpoly.coscup.org/api/booths/${tokenFromUrl}` , {
    method: 'GET',
  })
    .then((response) => response.json())
    .then((data) => {
      img.value = data.logo
    })
    .catch((error) => {
      console.error('Error:', error)
    })
})

async function onDetect(detectedCodes: any[]) {
  const result = detectedCodes[0].rawValue
  decodedString.value = result

  const response = await fetch(`https://geotrainpoly.coscup.org/api/send`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ booth_id: token.value, user_id: result }),
  })

  const body = await response.json()


  if (!response.ok) {
    return toast.error(body.detail, {
        position: POSITION.BOTTOM_CENTER,
        timeout: 3000,
      })
  }

  toast.success(body.detail, {
    position: POSITION.BOTTOM_CENTER,
    timeout: 3000,
  })
}
</script>

<template>
  <div class="main-container">
    <img :src="img ?? coscupLogo" alt="COSCUP Logo" class="logo" />
    <div class="scanner-container">
      <qrcode-stream @detect="onDetect"></qrcode-stream>
    </div>
  </div>
</template>

<style>
html, body {
  margin: 0;
  padding: 0;
  height: 100%;
  overflow: hidden;
}
</style>

<style scoped>
.main-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  gap: 2rem;
}

.logo {
  padding: 20px;
  background-color: #fff;
  flex-grow: 1;

  max-width: 75vw;
  max-height: 15vh;

  object-fit: contain;
}

.scanner-container {
  width: 80vw;
  max-width: 600px;
  height: 80vw;
  max-height: 600px;
  border: 1px solid #ccc;
}
</style>
