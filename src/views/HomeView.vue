<template>
  <main>
    <div class='image-container'>
      <img v-if="imageUrl" :src="imageUrl" alt="Uploaded Image" ref='image'>
      <canvas v-if="imageUrl" ref="canvas"></canvas>
    </div>
    <div>
      <p>
        <input type="file" @change="handleFileChange($event)"  accept="image/*"/>
        <button @click="sendImage" :disabled='!file'>Send image</button>
      </p>
    </div>
  </main>
</template>

<script setup lang="ts">
import axios from 'axios';
import {ref, watch} from 'vue'

const file = ref(null)
const imageUrl = ref('')
const response_data = ref(null)
const image = ref(null)
const canvas = ref(null)
const colorMapping = {
  "pores": "#0000FF",
  "blackhead": "#FF0000",
  "wrinkle": "#6E9900",
  "fine_line": "#8DFE2A",
  "closed_comedones": "#00FF00",
  "acne_pustule": "#9F21F6",
  "acne_nodule": "#FF00FD",
  "acne": "#FE0100",
  "brown_spot": "#7E2A28",
  "skin_spot": "#9C4406",
  "mole": "#FFA500"
}
const url = 'https://cors-anywhere.herokuapp.com/https://www.ailabapi.com/api/portrait/analysis/skin-analysis-advanced'

function handleFileChange(event) {
    file.value = event.target.files[0]
    imageUrl.value = URL.createObjectURL(event.target.files[0])
}

watch(file, (newValue, oldValue) => {
  imageUrl.value = URL.createObjectURL(newValue)
})

async function sendImage() {
    const formData = new FormData();
    formData.append("image", file.value);
    await axios.post(url, formData, {headers: {
      'ailabapi-api-key': 'ZnSmKYtb4aIViNW2O7CRegoNW4aZUHYIvcPpxBzfwryHX26pOGm8cqJRjTFkkMtM',
      'Content-Type': 'multipart/form-data',
      'Access-Control-Allow-Origin': '*',
      'X-Requested-With': 'XMLHttpRequest'
    }}).then((response) => {
      console.log(response.data)
      response_data.value = response.data
      drawPoints(response.data)
    })
}

function drawPoints(response){
  const imageElement = image.value
  const canvasElement = canvas.value
  const ctx = canvasElement.getContext('2d')
  
  canvasElement.width = imageElement.width;
  canvasElement.height = imageElement.height;
  ctx.clearRect(0, 0, canvasElement.width, canvasElement.height)

  for (var element in response.result) {
    if (response.result[element]['rectangle']) {
      response.result[element]['rectangle'].forEach(rect => {
        ctx.strokeStyle = colorMapping[element] ?? 'black'
        ctx.lineWidth = 2
        ctx.strokeRect(rect.left, rect.top, rect.width, rect.height)
      })
      
    }
    
  }
}
</script>

<style>

img,
canvas {
  position: absolute;
  top: 0;
  left: 0;
}
</style>