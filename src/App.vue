<script setup lang="ts">
import { ref } from 'vue'
import { createFFmpeg, fetchFile } from '@ffmpeg/ffmpeg'

const videoSrc = ref('')
const ffmpeg = createFFmpeg({
  log: true,
})

const handleChange = async (e: Event) => {
  const target = e.target as HTMLInputElement
  const files = target.files

  if (!files) {
    return
  }

  const file = files[0]
  await ffmpeg.load()
  ffmpeg.FS('writeFile', file.name, await fetchFile(file))
  await ffmpeg.run(
    '-loop',
    '1',
    '-i',
    file.name,
    '-c:v',
    'libx264',
    '-t',
    '10',
    '-pix_fmt',
    'yuv420p',
    '-vf',
    'scale=1920:1080',
    'out.mp4'
  )

  const videoData = ffmpeg.FS('readFile', 'out.mp4')
  ffmpeg.FS('unlink', file.name)
  const src = URL.createObjectURL(
    new Blob([videoData.buffer], { type: 'video/mp4' })
  )

  videoSrc.value = src
}
</script>

<template>
  <input type="file" @change="handleChange" />
  <video :src="videoSrc"></video>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
