<script setup lang="ts">

import { computed, ref, watch, watchEffect } from 'vue'
import { refThrottled, useDevicesList, useUserMedia, watchThrottled } from '@vueuse/core'

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


import { useDeviceMotion } from '@vueuse/core'

const {
  acceleration,
  accelerationIncludingGravity,
  rotationRate,
  interval,
} = useDeviceMotion()

// const stopNow = ref(false);
const stillVal = 2

const stopNow = computed(() => {
  if(!acceleration.value) return false
  return ((Math.abs(acceleration.value?.x ?? 0) > stillVal) || (Math.abs(acceleration.value?.y ?? 0) > stillVal) || (Math.abs(acceleration.value?.z ?? 0 )> stillVal))
})
const throttled = refThrottled(stopNow, 4000)

// watchThrottled(acceleration, (v) => {
//   if(v && ((v?.x ?? 0 > 4) || (v?.y ?? 0 > 4) || (v?.z ?? 0 > 4))) {
//     console.log('acceleration', v)
//     window.alert("brake");
//   }
// },{ throttle: 2000 },)
</script>

<template>
  <h1>hello</h1>
  <h1 style="color: red; font-size: 4rem;" v-if="throttled">Brake</h1>

  {{(acceleration?.x?.toFixed(2))}}<br>
  {{acceleration?.y?.toFixed(2)}}<br>
  {{acceleration?.z?.toFixed(2)}}<br>
  <div class="flex flex-col gap-4 text-center">
    <div>
      <button @click="enabled = !enabled">
        {{ enabled ? 'Stop' : 'Start' }}
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
}

</style>

