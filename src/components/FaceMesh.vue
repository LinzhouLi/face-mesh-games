<script setup>

import { onMounted, ref } from 'vue';
import { FaceMeshDetection, DIRECTION } from './FaceMeshDetection.js';

const direction = ref('none');
let loading = ref(true);
let faceMeshDet = new FaceMeshDetection(dir => {
  if (loading) { // 加载完成
    loading.value = false;
  }
  console.log(dir)
  direction.value = dir;
});

onMounted(() => {
  const canvasElement = document.getElementById('output_canvas');

  faceMeshDet.setCanvas(canvasElement);
  faceMeshDet.init();
});

</script>

<template>
  <div v-loading="loading" element-loading-text="Loading...">
    <canvas id="output_canvas" width="1280" height="720" />
    <div id="direction">{{ direction }}</div>
  </div>
</template>

<style scoped>
#output_canvas {
  width: 1280px;
  height: 720px;
}

#direction {
  margin-top: 20px;
  text-align: center;
  font-size: 30px;
}
</style>
