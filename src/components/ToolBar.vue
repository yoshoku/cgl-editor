<script setup lang="ts">
import { ref } from 'vue'

interface Props {
  isRunning: boolean
}

defineProps<Props>()

const emit = defineEmits<{
  run: []
  stop: []
  fullscreen: []
  download: []
  upload: [file: File]
}>()

const fileInput = ref<HTMLInputElement | null>(null)

const handleFileChange = (e: Event) => {
  const target = e.target as HTMLInputElement
  const file = target.files?.[0]
  if (file && file.name.endsWith('.js')) {
    emit('upload', file)
  }
  // Reset input value to allow re-uploading the same file
  if (target) {
    target.value = ''
  }
}

const triggerUpload = () => {
  fileInput.value?.click()
}

const openAbout = () => {
  window.open('https://github.com/yoshoku/cgl-editor', '_blank')
}
</script>

<template>
  <div class="toolbar">
    <button
      class="icon-button run-button"
      :disabled="isRunning"
      data-tooltip="Run"
      @click="emit('run')"
    >
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="18"
        height="18"
        viewBox="0 0 24 24"
        fill="currentColor"
      >
        <path d="M8 5v14l11-7z" />
      </svg>
    </button>
    <button
      class="icon-button fullscreen-button"
      :disabled="isRunning"
      data-tooltip="Run in fullscreen"
      @click="emit('fullscreen')"
    >
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="18"
        height="18"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        stroke-width="2"
        stroke-linecap="round"
        stroke-linejoin="round"
      >
        <path
          d="M8 3H5a2 2 0 0 0-2 2v3m18 0V5a2 2 0 0 0-2-2h-3m0 18h3a2 2 0 0 0 2-2v-3M3 16v3a2 2 0 0 0 2 2h3"
        />
      </svg>
    </button>
    <button
      class="icon-button stop-button"
      :disabled="!isRunning"
      data-tooltip="Stop"
      @click="emit('stop')"
    >
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="18"
        height="18"
        viewBox="0 0 24 24"
        fill="currentColor"
      >
        <rect x="6" y="6" width="12" height="12" />
      </svg>
    </button>
    <div class="divider"></div>
    <button class="icon-button upload-button" data-tooltip="Upload" @click="triggerUpload">
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="18"
        height="18"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        stroke-width="2"
        stroke-linecap="round"
        stroke-linejoin="round"
      >
        <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4" />
        <polyline points="17 8 12 3 7 8" />
        <line x1="12" y1="3" x2="12" y2="15" />
      </svg>
    </button>
    <button
      class="icon-button download-button"
      data-tooltip="Download"
      @click="emit('download')"
    >
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="18"
        height="18"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        stroke-width="2"
        stroke-linecap="round"
        stroke-linejoin="round"
      >
        <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4" />
        <polyline points="7 10 12 15 17 10" />
        <line x1="12" y1="15" x2="12" y2="3" />
      </svg>
    </button>
    <button class="icon-button about-button" data-tooltip="About" @click="openAbout">
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="18"
        height="18"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        stroke-width="2.5"
        stroke-linecap="round"
        stroke-linejoin="round"
      >
        <line x1="12" y1="18" x2="12" y2="12" />
        <line x1="10" y1="11" x2="12" y2="11" />
        <line x1="12" y1="19" x2="14" y2="19" />
        <line x1="12" y1="5" x2="12.01" y2="5" />
      </svg>
    </button>
    <input
      ref="fileInput"
      type="file"
      accept=".js"
      style="display: none"
      @change="handleFileChange"
    />
  </div>
</template>

<style scoped>
.toolbar {
  display: flex;
  align-items: center;
  padding: 8px;
  background-color: #f0f0f0;
  border-bottom: 1px solid #ccc;
}

.icon-button {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  padding: 0;
  border: none;
  border-radius: 50%;
  cursor: pointer;
  transition:
    background-color 0.2s,
    color 0.2s;
  margin-right: 8px;
  background-color: #fff;
  color: #333;
}

.icon-button::after {
  content: attr(data-tooltip);
  position: absolute;
  bottom: -32px;
  left: 50%;
  transform: translateX(-50%);
  padding: 4px 8px;
  background-color: #333;
  color: white;
  font-size: 12px;
  white-space: nowrap;
  border-radius: 4px;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.2s;
  z-index: 1000;
}

.icon-button:hover:not(:disabled)::after {
  opacity: 1;
}

.icon-button:hover:not(:disabled) {
  background-color: #e91e63;
  color: white;
}

.run-button:disabled {
  background-color: #e0e0e0;
  color: #999;
  cursor: not-allowed;
}

.stop-button:disabled {
  background-color: #e0e0e0;
  color: #999;
  cursor: not-allowed;
}

.fullscreen-button:disabled {
  background-color: #e0e0e0;
  color: #999;
  cursor: not-allowed;
}

.divider {
  width: 1px;
  background-color: #666;
  margin: 0 8px 0 0;
  align-self: stretch;
}

.about-button {
  margin-left: auto;
  margin-right: 0;
}
</style>
