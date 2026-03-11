<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'

interface ScrollMetrics {
  clientHeight: number
  scrollHeight: number
  scrollTop: number
  thumbHeight: number
  thumbTop: number
}

const containerRef = ref<HTMLDivElement | null>(null)
const trackRef = ref<HTMLDivElement | null>(null)

const isHover = ref(false)
const isDragging = ref(false)
const dragStartY = ref(0)
const dragStartScrollTop = ref(0)
const scrollTriggger = ref(0)  // Used to trigger computed recomputation

let resizeObserver: ResizeObserver | null = null

const scrollMetrics = computed((): ScrollMetrics => {
  scrollTriggger.value  // Access to track dependency
  
  const container = containerRef.value
  if (!container) {
    return {
      clientHeight: 0,
      scrollHeight: 0,
      scrollTop: 0,
      thumbHeight: 0,
      thumbTop: 0,
    }
  }

  const clientHeight = container.clientHeight
  const scrollHeight = container.scrollHeight
  const scrollTop = container.scrollTop

  // Thumb height represents the proportion of visible content
  const thumbHeight = Math.max((clientHeight / scrollHeight) * clientHeight, 30)

  // Thumb position follows scroll position
  const maxScrollTop = scrollHeight - clientHeight
  const maxThumbTop = clientHeight - thumbHeight
  const thumbTop = maxScrollTop > 0 ? (scrollTop / maxScrollTop) * maxThumbTop : 0

  return {
    clientHeight,
    scrollHeight,
    scrollTop,
    thumbHeight,
    thumbTop,
  }
})

const isVisible = computed(() => {
  return scrollMetrics.value.scrollHeight > scrollMetrics.value.clientHeight
})

const handleThumbMouseDown = (e: MouseEvent) => {
  isDragging.value = true
  dragStartY.value = e.clientY
  dragStartScrollTop.value = containerRef.value?.scrollTop ?? 0

  document.body.style.userSelect = 'none'
  document.body.style.cursor = 'grabbing'

  document.addEventListener('mousemove', handleMouseMove)
  document.addEventListener('mouseup', handleMouseUp)
  
  e.preventDefault()
}

const handleMouseMove = (e: MouseEvent) => {
  if (!isDragging.value || !containerRef.value) return

  const container = containerRef.value
  const deltaY = e.clientY - dragStartY.value
  const { scrollHeight, clientHeight, thumbHeight } = scrollMetrics.value

  const maxScrollTop = scrollHeight - clientHeight
  const maxThumbTop = clientHeight - thumbHeight

  if (maxThumbTop > 0 && maxScrollTop > 0) {
    const scrollDelta = (deltaY / maxThumbTop) * maxScrollTop
    const newScrollTop = Math.max(0, Math.min(dragStartScrollTop.value + scrollDelta, maxScrollTop))
    container.scrollTop = newScrollTop
  }
}

const handleMouseUp = () => {
  isDragging.value = false
  document.body.style.userSelect = ''
  document.body.style.cursor = ''
  document.removeEventListener('mousemove', handleMouseMove)
  document.removeEventListener('mouseup', handleMouseUp)
}

const handleScroll = () => {
  // Trigger reactivity by incrementing this
  scrollTriggger.value++
}

const initResizeObserver = () => {
  const container = containerRef.value
  if (!container) return

  resizeObserver = new ResizeObserver(() => {
    void scrollMetrics.value
  })

  resizeObserver.observe(container)
  Array.from(container.children).forEach((child) => {
    if (child !== trackRef.value) {
      resizeObserver?.observe(child)
    }
  })
}

const destroyResizeObserver = () => {
  if (resizeObserver) {
    resizeObserver.disconnect()
    resizeObserver = null
  }
}

onMounted(() => {
  initResizeObserver()
})

onUnmounted(() => {
  destroyResizeObserver()
  document.removeEventListener('mousemove', handleMouseMove)
  document.removeEventListener('mouseup', handleMouseUp)
})
</script>

<template>
  <div class="scrollbar-wrapper">
    <!-- Scrollable content container -->
    <div
      ref="containerRef"
      class="scrollbar-container"
      @scroll="handleScroll"
      @mouseenter="isHover = true"
      @mouseleave="isHover = false"
    >
      <!-- Content slot -->
      <slot />
    </div>

    <!-- Scrollbar track overlay (positioned absolutely, NOT scrolling) -->
    <div
      v-if="isVisible"
      ref="trackRef"
      class="scrollbar-track"
      :class="{ 'is-hover': isHover, 'is-dragging': isDragging }"
    >
      <div
        class="scrollbar-thumb"
        :style="{ height: `${scrollMetrics.thumbHeight}px`, transform: `translateY(${scrollMetrics.thumbTop}px)` }"
        @mousedown="handleThumbMouseDown"
      />
    </div>
  </div>
</template>

<style scoped>
/* Wrapper for proper positioning context */
.scrollbar-wrapper {
  position: relative;
  width: 100%;
  height: 100%;
}

.scrollbar-container {
  position: relative;
  overflow-y: scroll;
  overflow-x: hidden;
  width: 100%;
  height: 100%;
}

/* Hide default scrollbar */
.scrollbar-container::-webkit-scrollbar {
  display: none;
}

.scrollbar-container {
  -ms-overflow-style: none;
  scrollbar-width: none;
}

/* Scrollbar track - positioned absolutely, overlays container */
.scrollbar-track {
  position: absolute;
  right: 2px;
  top: 0;
  width: 10px;
  height: 100%;
  background: transparent;
  pointer-events: none;
  z-index: 999;
}

.scrollbar-track.is-hover {
  background-color: rgba(0, 0, 0, 0.08);
  pointer-events: auto;
}

.scrollbar-track.is-dragging {
  background-color: rgba(0, 0, 0, 0.15);
  pointer-events: auto;
}

/* Scrollbar thumb */
.scrollbar-thumb {
  position: absolute;
  width: 100%;
  background: rgba(0, 0, 0, 0.35);
  border-radius: 5px;
  cursor: grab;
  left: 0;
  will-change: transform;
  transition: background-color 0.2s ease;
  min-height: 30px;
}

.scrollbar-thumb:hover {
  background: rgba(0, 0, 0, 0.55);
}

.scrollbar-track.is-dragging .scrollbar-thumb {
  background: rgba(0, 0, 0, 0.75);
  cursor: grabbing;
}
</style>
