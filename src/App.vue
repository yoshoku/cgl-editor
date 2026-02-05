<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import JsEditor from './components/JsEditor.vue'
import JsPreview from './components/JsPreview.vue'
import ToolBar from './components/ToolBar.vue'

const generateId = (): string => {
  return `${Date.now()}-${Math.random().toString(36).substring(2, 9)}`
}

let initialCode = `title = "";

description = "";

characters = [];

options = {};

function update() {
  if (!ticks) {
  }
}`

const code = ref(initialCode)
const contentArea = ref<HTMLElement | null>(null)
const editorWidth = ref(50)
const iframeKey = ref(generateId())
const isDragging = ref(false)
const isRunning = ref(false)
const jsPreviewInstance = ref<InstanceType<typeof JsPreview> | null>(null)

const runGame = () => {
  const iframe = jsPreviewInstance.value?.iframe as HTMLIFrameElement
  if (!iframe) return

  const html = `<!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8" />
      <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
      <meta name="viewport" content="width=device-width, height=device-height, user-scalable=no, initial-scale=1, maximum-scale=1" />
      <script src="https://unpkg.com/algo-chip@1.0.2/packages/core/dist/algo-chip.umd.js"><\/script>
      <script src="https://unpkg.com/algo-chip@1.0.2/packages/util/dist/algo-chip-util.umd.js"><\/script>
      <script src="https://unpkg.com/gif-capture-canvas@1.1.0/build/index.js"><\/script>
      <script src="https://unpkg.com/pixi.js@5.3.0/dist/pixi.min.js"><\/script>
      <script src="https://unpkg.com/pixi-filters@3.1.1/dist/pixi-filters.js"><\/script>
      <script src="https://unpkg.com/crisp-game-lib@latest/docs/bundle.js"><\/script>
      <script>${code.value}<\/script>
      <script>window.addEventListener("load", onLoad);<\/script>
      <style>
        * { margin: 0; padding: 0; }
        html, body { width: 100%; height: 100%; overflow: hidden; }
        body { display: flex; justify-content: center; align-items: center; }
        canvas { display: block; }
      </style>
    </head>
    <body>
    </body>
    </html>`

  iframe.srcdoc = html

  isRunning.value = true
}

const runGameFullscreen = async () => {
  const wrapper = jsPreviewInstance.value?.wrapper
  if (!wrapper) return

  if (wrapper.requestFullscreen) {
    await wrapper.requestFullscreen()
    setTimeout(() => runGame(), 100)
  }
}

const stopGame = () => {
  iframeKey.value = generateId()
  isRunning.value = false
}

const downloadGame = () => {
  const blob = new Blob([code.value], { type: 'text/javascript' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = 'main.js'
  document.body.appendChild(a)
  a.click()
  document.body.removeChild(a)
  URL.revokeObjectURL(url)
}

const uploadGame = (file: File) => {
  const reader = new FileReader()
  reader.onload = e => {
    const content = e.target?.result
    if (typeof content === 'string') {
      code.value = content
      initialCode = content
      if (isRunning.value) {
        stopGame()
      }
    }
  }
  reader.readAsText(file)
}

const handleBeforeUnload = (e: BeforeUnloadEvent) => {
  if (code.value !== initialCode) {
    e.preventDefault()
    e.returnValue = ''
  }
}

const startResize = (e: MouseEvent) => {
  isDragging.value = true
  e.preventDefault()
}

const handleResize = (e: MouseEvent) => {
  if (!isDragging.value || !contentArea.value) return

  const rect = contentArea.value.getBoundingClientRect()
  const offsetX = e.clientX - rect.left
  const newWidth = (offsetX / rect.width) * 100

  if (newWidth >= 20 && newWidth <= 80) {
    editorWidth.value = newWidth
  }
}

const stopResize = () => {
  isDragging.value = false
}

onMounted(() => {
  window.addEventListener('mousemove', handleResize)
  window.addEventListener('mouseup', stopResize)
  window.addEventListener('mouseleave', stopResize)
  window.addEventListener('beforeunload', handleBeforeUnload)
})

onUnmounted(() => {
  window.removeEventListener('mousemove', handleResize)
  window.removeEventListener('mouseup', stopResize)
  window.removeEventListener('mouseleave', stopResize)
  window.removeEventListener('beforeunload', handleBeforeUnload)
})
</script>

<template>
  <div class="container">
    <ToolBar
      :is-running="isRunning"
      @run="runGame"
      @fullscreen="runGameFullscreen"
      @stop="stopGame"
      @upload="uploadGame"
      @download="downloadGame"
    />
    <div ref="contentArea" class="main-content" :class="{ dragging: isDragging }">
      <div
        class="editor-section"
        :style="{ width: editorWidth + '%' }"
        :class="{ 'no-pointer-events': isDragging }"
      >
        <JsEditor :code="code" @update:code="code = $event" />
      </div>
      <div class="divider" :class="{ dragging: isDragging }" @mousedown="startResize"></div>
      <div class="preview-section" :class="{ 'no-pointer-events': isDragging }">
        <JsPreview ref="jsPreviewInstance" :key="iframeKey" />
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
}

.main-content {
  display: flex;
  flex-direction: row;
  flex: 1;
  min-height: 0;
}

.main-content.dragging {
  cursor: col-resize;
  user-select: none;
}

.editor-section {
  display: flex;
  flex-direction: column;
  min-height: 0;
  min-width: 20%;
  max-width: 80%;
}

.preview-section {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 0;
}

.divider {
  width: 8px;
  background-color: #ccc;
  cursor: col-resize;
  position: relative;
  flex-shrink: 0;
}

.divider:hover {
  background-color: #999;
}

.divider.dragging {
  background-color: #666;
}

.no-pointer-events {
  pointer-events: none;
}
</style>
