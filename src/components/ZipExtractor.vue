<template>
  <div>
    <input type="file" @change="handleFileSelect" />
    <div v-if="fileList.length > 0">
      <h1>Files:</h1>
      <ul>
        <li v-for="file in fileList" :key="file.name">
          {{ file.name }}
          <button @click="downloadFile(file)">Download</button>
        </li>
      </ul>
      <button @click="downloadAllFiles">Download All Files</button>
      <a ref="downloadLink" v-show="false" download></a>
    </div>
  </div>
</template>

<script lang="ts" setup>
import type { ZipItem } from '@/models/ZipItem'
import JSZip from 'jszip'
import { ref } from 'vue'

const fileList = ref<ZipItem[]>([])
const downloadLink = ref<HTMLAnchorElement | null>(null)

const handleFileSelect = async (event: Event) => {
  const input = event.target as HTMLInputElement
  if (input.files && input.files.length > 0) {
    const file = input.files[0]
    if (file.type === 'application/zip' || file.type === 'application/x-zip-compressed') {
      const zip = new JSZip()
      const content = await file.arrayBuffer()
      const loadedZip = await zip.loadAsync(content)

      const files: ZipItem[] = []
      const fileKeys = Object.keys(loadedZip.files)

      for (const key of fileKeys) {
        const zipEntry = loadedZip.files[key]
        if (!zipEntry.dir) {
          const fileData = await zipEntry.async('blob')
          files.push({ name: key, file: fileData })
        }
      }

      fileList.value = files
    } else {
      alert('Please upload a valid zip file.')
    }
  }
}

const downloadFile = (file: ZipItem) => {
  const url = URL.createObjectURL(file.file)
  if (downloadLink.value) {
    downloadLink.value.href = url
    downloadLink.value.download = file.name
    downloadLink.value.click()
    URL.revokeObjectURL(url)
  }
}

const downloadAllFiles = () => {
  fileList.value.forEach((fileItem) => {
    const url = URL.createObjectURL(fileItem.file)
    if (downloadLink.value) {
      downloadLink.value.href = url
      downloadLink.value.download = fileItem.name
      downloadLink.value.click()
      URL.revokeObjectURL(url)
    }
  })
}
</script>
