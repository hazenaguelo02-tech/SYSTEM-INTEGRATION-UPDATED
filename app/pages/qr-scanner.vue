<template>
  <div class="mx-auto d-flex justify-center align-center" style="height: 90vh;">
    <v-card>
      <v-card-text>

        <!-- Show QR Scanner -->
        <video ref="videoRef" class="qr-video"></video>

        <!-- Start Scanner -->
        <v-btn color="primary" @click="startScanner" block>
          Start Scanner
        </v-btn>

        <!-- Stop Scanner -->
        <v-btn color="error" class="mt-5" @click="stopScanner" block>
          Stop Scanner
        </v-btn>

      </v-card-text>
    </v-card>
  </div>
</template>

<script lang="ts" setup>
// @ts-nocheck
import QrScanner from 'qr-scanner'

const videoRef = ref<HTMLVideoElement | null>(null)

// const result = ref('')

let scanner: QrScanner | null = null

const startScanner = async () => {
  if (!videoRef.value) return

  scanner = new QrScanner(
    videoRef.value,

    (scanResult) => {
      // result.value = scanResult.data
      // emit('scanned', scanResult.data)

      // optional: stop after successful scan
      scanner?.stop()
    },

    {
      preferredCamera: 'environment',
      highlightScanRegion: true,
      highlightCodeOutline: true,
    }
  )

  await scanner.start()
}

const stopScanner = () => {
  scanner?.stop()
}
</script>

<style scoped>
.qr-video {
  width: 100%;
  max-width: 400px;
  border-radius: 12px;
  background: #000;
}
</style>
