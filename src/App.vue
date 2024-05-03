<script setup lang="ts">

import { computed, ref, watch, watchEffect } from 'vue'
import { useDevicesList, useUserMedia, watchThrottled } from '@vueuse/core'

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
const stillVal = 8

const stopNow = computed(() => {
  if(!acceleration.value) return false
  return (((acceleration.value?.x ?? 0) > stillVal) || ((acceleration.value?.y ?? 0) > stillVal) || ((acceleration.value?.z ?? 0 )> stillVal))
})

// watchThrottled(acceleration, (v) => {
//   if(v && ((v?.x ?? 0 > 4) || (v?.y ?? 0 > 4) || (v?.z ?? 0 > 4))) {
//     console.log('acceleration', v)
//     window.alert("brake");
//   }
// },{ throttle: 2000 },)
</script>

<template>
  <h1>hello</h1>
  <h1 style="color: red; font-size: 4rem;" v-if="stopNow">Brake</h1>

  {{acceleration?.x}}
  {{acceleration?.y}}
  {{acceleration?.z}}
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

