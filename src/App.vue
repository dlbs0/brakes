<script setup lang="ts">

import { computed, onMounted, ref,  watchEffect } from 'vue'
import { refThrottled, useDevicesList, useUserMedia, useDeviceMotion } from '@vueuse/core'

const currentCamera = ref<string>()
const { videoInputs: cameras } = useDevicesList({
  requestPermissions: true,
  onUpdated() {
    if (!cameras.value.find(i => i.deviceId === currentCamera.value))
      currentCamera.value = cameras.value[0]?.deviceId
  },
})

const video = ref<HTMLVideoElement>()
const { stream, enabled } = useUserMedia({
  constraints: { video: { deviceId: currentCamera.value ??''} },
})

watchEffect(() => {
  if (video.value)
    video.value.srcObject = stream.value!
})


const {
  acceleration,
  accelerationIncludingGravity,
  rotationRate,
  interval,
} = useDeviceMotion()

const stillVal = 2

const stopNow = computed(() => {
  if(!acceleration.value) return false
  return ((Math.abs(acceleration.value?.x ?? 0) > stillVal) || (Math.abs(acceleration.value?.y ?? 0) > stillVal) || (Math.abs(acceleration.value?.z ?? 0 )> stillVal))
})
const throttled = refThrottled(stopNow, 2000, true , true)

onMounted(() => {
  enabled.value = true
})
</script>

<template>
  <h1>Very helpful app</h1>
  <h1 style="color: red; font-size: 4rem;" v-if="throttled">Brake</h1>
  <h1 style="color: red; font-size: 4rem;" v-if="stopNow">Slow Down</h1>

  {{(acceleration?.x?.toFixed(2))}}<br>
  {{acceleration?.y?.toFixed(2)}}<br>
  {{acceleration?.z?.toFixed(2)}}<br>
  <div class="flex flex-col gap-4 text-center">
    <div>
      <button @click="enabled = !enabled">
        {{ enabled ? 'Stop Video' : 'Start Video' }}
      </button>
    </div>

    
    <div>
      <video ref="video" muted autoplay controls class="h-100 w-auto vid" />
    </div>
  </div>
</template>


<style scoped>
.vid{
  max-width: 100%;
  height: 100vh;
  position: absolute;
  top:0;
  left:0;
  z-index: -1;
  object-fit: cover;
  object-position: 50% 50%;
}

</style>

