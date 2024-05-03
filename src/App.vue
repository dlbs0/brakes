<script setup lang="ts">

import { ref, watch, watchEffect } from 'vue'
import { useDevicesList, useUserMedia } from '@vueuse/core'

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
  constraints: { video: { deviceId: currentCamera } },
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

watch(acceleration, (v) => {
  if(v && ((v?.x ?? 0 > 0.1) || (v?.y ?? 0 > 0.1) || (v?.z ?? 0 > 0.1))) {
    console.log('acceleration', v)
    window.alert("brake");
  }
})
</script>

<template>
  <h1>hello</h1>
  <div class="flex flex-col gap-4 text-center">
    <div>
      <button @click="enabled = !enabled">
        {{ enabled ? 'Stop' : 'Start' }}
      </button>
    </div>

    
    <div>
      <video ref="video" muted autoplay controls class="h-100 w-auto" />
    </div>
  </div>
</template>


