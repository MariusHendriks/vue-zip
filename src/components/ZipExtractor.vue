<template>
  <div class="zip-extractor">
    <div clas="upload">
      <h1>Zip extractor</h1>

      <input class="upload" type="file" @change="handleFileSelect" />
    </div>
    <div v-if="fileList.length > 0" class="files-container">
      <div class="files">
        <h1>All Files:</h1>
        <div v-for="file in fileList" :key="file.name" class="file">
          {{ file.name }}
          <button @click="downloadFile(file)">Download</button>
        </div>
        <button class="download-all" @click="downloadAllFiles">Download All Files</button>
      </div>

      <div v-if="fileList !== groupedFiles.value">
        <h1>Files per directory:</h1>
        <div v-for="(groupFiles, groupName) in groupedFiles" :key="groupName">
          <div class="files">
            <h3>{{ groupName }}</h3>
            <div v-for="(file, index) in groupFiles" :key="file.name" class="file">
              {{ file.name }}
              <button @click="downloadFileFromGroup(groupName, index)">Download</button>
            </div>
            <button class="download-all" @click="downloadGroup(groupName)">
              Download All {{ groupName }} Files
            </button>
          </div>
        </div>
      </div>
    </div>
    <a ref="downloadLink" v-show="false" download></a>
  </div>
</template>

<script setup lang="ts">
import type { ZipItem } from '@/models/ZipItem'
import JSZip from 'jszip'
import { ref, computed } from 'vue'

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

const downloadFileFromGroup = (groupName: string, index: number) => {
  const groupFiles = groupedFiles.value[groupName]
  if (groupFiles && index >= 0 && index < groupFiles.length) {
    const file = groupFiles[index]
    downloadFile(file)
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

const downloadGroup = (groupName: string) => {
  const groupFiles = groupedFiles.value[groupName]
  if (groupFiles) {
    groupFiles.forEach(downloadFile)
  }
}

const groupedFiles = computed(() => {
  const groups: Record<string, ZipItem[]> = {}

  fileList.value.forEach((file) => {
    const parts = file.name.split('/')
    const groupName = parts.length > 1 ? parts[0] : 'root'

    if (!groups[groupName]) {
      groups[groupName] = []
    }

    groups[groupName].push(file)
  })

  return groups
})
</script>

<style scoped>
.zip-extractor {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.files-container {
  display: flex;
  gap: 5em;
}
.files {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}
.files .file {
  display: flex;
  justify-content: space-between;
}

.files .file button {
  margin-left: 15px;
}

.download-all {
  margin-top: 15px;
}

.upload {
  margin-bottom: 30px;
}
</style>
