<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted, watch } from 'vue';

interface ScrollMetrics {
  clientHeight: number;
  scrollHeight: number;
  scrollTop: number;
  thumbHeight: number;
  thumbTop: number;
}

const containerRef = ref<HTMLDivElement | null>(null);
const trackRef = ref<HTMLDivElement | null>(null);

const isHover = ref(false);
const isDragging = ref(false);
const isScrolling = ref(false); // Track manual scrolling
const dragStartY = ref(0);
const dragStartScrollTop = ref(0);
const scrollTrigger = ref(0); // Used to trigger computed recomputation
const forceUpdateTrigger = ref(0); // Force update for zoom and other changes

// Visibility states
const visibilityState = ref({
  isContainerHovered: false,
  isThumbHovered: false,
  isDragging: false,
  isScrolling: false
});

let resizeObserver: ResizeObserver | null = null;
let mutationObserver: MutationObserver | null = null;
let zoomObserverInterval: number | null = null;

const scrollMetrics = computed((): ScrollMetrics => {
  scrollTrigger.value; // Access to track dependency
  forceUpdateTrigger.value; // Access to force updates

  const container = containerRef.value;
  if (!container) {
    return {
      clientHeight: 0,
      scrollHeight: 0,
      scrollTop: 0,
      thumbHeight: 0,
      thumbTop: 0,
    };
  }

  const clientHeight = container.clientHeight;
  const scrollHeight = container.scrollHeight;
  const scrollTop = container.scrollTop;

  if (clientHeight >= scrollHeight) {
    return {
      clientHeight,
      scrollHeight,
      scrollTop,
      thumbHeight: 0,
      thumbTop: 0,
    };
  }

  // Thumb height represents the proportion of visible content
  const thumbHeightRatio = clientHeight / scrollHeight;
  const thumbHeight = Math.max(thumbHeightRatio * clientHeight, 30);

  // Thumb position follows scroll position
  const maxScrollTop = scrollHeight - clientHeight;
  const trackHeight = clientHeight - 4; // 4px total padding (top + bottom)
  const maxThumbTop = trackHeight - thumbHeight;
  const thumbTop = maxScrollTop > 0 ? (scrollTop / maxScrollTop) * maxThumbTop : 0;

  return {
    clientHeight,
    scrollHeight,
    scrollTop,
    thumbHeight,
    thumbTop,
  };
});

const isVisible = computed(() => {
  return scrollMetrics.value.scrollHeight > scrollMetrics.value.clientHeight;
});

// Compute thumb opacity based on visibility states
const thumbOpacity = computed(() => {
  const state = visibilityState.value;
  
  // Priority order: dragging > scrolling > thumb hover > container hover > default
  if (state.isDragging) return 1.0;        // Most visible when dragging
  if (state.isScrolling) return 0.9;       // Very visible when scrolling
  if (state.isThumbHovered) return 0.7;    // More visible when thumb hovered
  if (state.isContainerHovered) return 0.5; // Visible when container hovered
  return 0.2;                              // Low visibility by default
});

// Handle container mouse enter
const handleContainerMouseEnter = () => {
  visibilityState.value.isContainerHovered = true;
};

// Handle container mouse leave
const handleContainerMouseLeave = () => {
  visibilityState.value.isContainerHovered = false;
};

// Handle thumb mouse enter
const handleThumbMouseEnter = () => {
  visibilityState.value.isThumbHovered = true;
};

// Handle thumb mouse leave
const handleThumbMouseLeave = () => {
  visibilityState.value.isThumbHovered = false;
};

// Force update scroll metrics
const forceUpdateScrollMetrics = () => {
  forceUpdateTrigger.value++;
};

const handleThumbMouseDown = (e: MouseEvent) => {
  isDragging.value = true;
  visibilityState.value.isDragging = true;
  dragStartY.value = e.clientY;
  dragStartScrollTop.value = containerRef.value?.scrollTop ?? 0;

  document.body.style.userSelect = 'none';
  document.body.style.cursor = 'grabbing';

  // Prevent text selection on drag
  e.preventDefault();
  e.stopPropagation();

  document.addEventListener('mousemove', handleMouseMove, { passive: false });
  document.addEventListener('mouseup', handleMouseUp, { once: true });
};

const handleMouseMove = (e: MouseEvent) => {
  if (!isDragging.value || !containerRef.value) return;

  const container = containerRef.value;
  const deltaY = e.clientY - dragStartY.value;
  const { scrollHeight, clientHeight, thumbHeight } = scrollMetrics.value;

  const maxScrollTop = scrollHeight - clientHeight;
  const maxThumbTop = clientHeight - thumbHeight;

  if (maxThumbTop > 0 && maxScrollTop > 0) {
    const scrollDelta = (deltaY / maxThumbTop) * maxScrollTop;
    const newScrollTop = Math.max(
      0,
      Math.min(dragStartScrollTop.value + scrollDelta, maxScrollTop)
    );
    container.scrollTop = newScrollTop;
  }

  e.preventDefault();
  e.stopPropagation();
};

const handleMouseUp = () => {
  isDragging.value = false;
  visibilityState.value.isDragging = false;
  // Only reset if not hovering
  if (!isHover.value) {
    document.body.style.cursor = '';
  }
  document.body.style.userSelect = '';
  document.removeEventListener('mousemove', handleMouseMove);
};

let scrollTimeout: number | null = null;

const handleScroll = () => {
  // Trigger reactivity by incrementing this
  scrollTrigger.value++;
  
  // Set scrolling state to true
  visibilityState.value.isScrolling = true;
  
  // Clear existing timeout
  if (scrollTimeout) {
    window.clearTimeout(scrollTimeout);
  }
  
  // Set timeout to reset scrolling state after scrolling stops
  scrollTimeout = window.setTimeout(() => {
    visibilityState.value.isScrolling = false;
  }, 300); // Reset after 300ms of no scrolling
};

// Detect zoom changes using visual viewport
const handleZoomChange = () => {
  forceUpdateScrollMetrics();
};

// Detect window resize
const handleWindowResize = () => {
  forceUpdateScrollMetrics();
};

const initResizeObserver = () => {
  const container = containerRef.value;
  if (!container) return;

  // ResizeObserver for container and content changes
  resizeObserver = new ResizeObserver(() => {
    forceUpdateScrollMetrics();
  });

  resizeObserver.observe(container);
  Array.from(container.children).forEach((child) => {
    if (child !== trackRef.value) {
      resizeObserver?.observe(child);
    }
  });

  // MutationObserver for DOM changes
  mutationObserver = new MutationObserver(() => {
    // Use a small delay to ensure DOM is fully updated
    setTimeout(() => {
      forceUpdateScrollMetrics();
    }, 0);
  });

  mutationObserver.observe(container, {
    childList: true,
    subtree: true,
    attributes: true,
    attributeFilter: ['style', 'class']
  });
};

// Zoom detection using visual viewport
const initZoomDetection = () => {
  // Visual viewport zoom detection
  if (window.visualViewport) {
    window.visualViewport.addEventListener('resize', handleZoomChange);
  }
  
  // Periodic check for zoom changes (fallback)
  zoomObserverInterval = window.setInterval(() => {
    forceUpdateScrollMetrics();
  }, 100); // Check every 100ms for instant updates
  
  // Window resize detection
  window.addEventListener('resize', handleWindowResize);
};

const destroyResizeObserver = () => {
  if (resizeObserver) {
    resizeObserver.disconnect();
    resizeObserver = null;
  }
  
  if (mutationObserver) {
    mutationObserver.disconnect();
    mutationObserver = null;
  }
  
  if (window.visualViewport) {
    window.visualViewport.removeEventListener('resize', handleZoomChange);
  }
  
  if (zoomObserverInterval) {
    window.clearInterval(zoomObserverInterval);
    zoomObserverInterval = null;
  }
  
  window.removeEventListener('resize', handleWindowResize);
};

onMounted(() => {
  initResizeObserver();
  initZoomDetection();
  // Initial update after DOM is rendered
  setTimeout(() => {
    forceUpdateScrollMetrics();
  }, 0);
});

onUnmounted(() => {
  destroyResizeObserver();
  document.removeEventListener('mousemove', handleMouseMove);
  document.removeEventListener('mouseup', handleMouseUp);
});
</script>

<template>
  <div class="scrollbar-wrapper overlay-scrollbar-wrapper">
    <!-- Scrollable content container -->
    <div
      ref="containerRef"
      class="scrollbar-container"
      @scroll="handleScroll"
      @mouseenter="handleContainerMouseEnter"
      @mouseleave="handleContainerMouseLeave"
    >
      <!-- Content slot -->
      <slot />
    </div>

    <!-- Scrollbar track overlay (positioned absolutely, NOT scrolling) -->
    <div
      v-if="isVisible"
      ref="trackRef"
      class="scrollbar-track"
      :class="{ 'is-hover': visibilityState.isContainerHovered, 'is-dragging': isDragging }"
    >
      <div
        class="scrollbar-thumb"
        :style="{
          height: `${scrollMetrics.thumbHeight}px`,
          transform: `translateY(${scrollMetrics.thumbTop}px)`,
          opacity: thumbOpacity,
        }"
        @mousedown="handleThumbMouseDown"
        @mouseenter="handleThumbMouseEnter"
        @mouseleave="handleThumbMouseLeave"
      />
    </div>
  </div>
</template>

<style scoped>
/* Wrapper for proper positioning context */
.scrollbar-wrapper {
  position: relative !important;
  width: 100% !important;
  height: 100% !important;
  display: flex !important;
  flex-direction: column !important;
  flex: 1 !important;
}

.scrollbar-container {
  position: relative !important;
  overflow-y: scroll !important;
  overflow-x: hidden !important;
  width: 100% !important;
  height: 100% !important;
}

/* Hide default scrollbar */
.scrollbar-container::-webkit-scrollbar {
  display: none !important;
}

.scrollbar-container {
  -ms-overflow-style: none !important;
  scrollbar-width: none !important;
}

/* Scrollbar track - positioned absolutely, overlays container */
.scrollbar-track {
  position: absolute !important;
  right: 0px !important;
  top: 2px !important;
  width: 5px !important;
  height: calc(100% - 4px) !important;
  background: transparent !important;
  pointer-events: none !important;
  z-index: 999999 !important;
  transition: background-color 0.2s ease !important;
  /* Force visibility */
  display: block !important;
  visibility: visible !important;
  opacity: 1 !important;
}

.scrollbar-track.is-hover {
  background-color: rgba(0, 0, 0, 0) !important;
}

.scrollbar-track.is-dragging {
  background-color: rgba(0, 0, 0, 0) !important;
}

/* Scrollbar thumb - always interactive */
.scrollbar-thumb {
  position: absolute !important;
  width: 100% !important;
  background: rgba(0, 0, 0, 0.35) !important;
  border-radius: 5px !important;
  left: 0 !important;
  will-change: transform !important;
  min-height: 30px !important;
  pointer-events: auto !important;
  cursor: grab !important;
  user-select: none !important;
  -webkit-user-select: none !important;
  -moz-user-select: none !important;
  -ms-user-select: none !important;
  transition: all 0.15s ease, height 0s, opacity 0.15s ease !important;
  /* Force visibility */
  display: block !important;
  visibility: visible !important;
}

.scrollbar-thumb:hover {
  background: rgba(0, 0, 0, 0.45) !important;
}

.scrollbar-thumb:active,
.scrollbar-track.is-dragging .scrollbar-thumb {
  background: rgba(0, 0, 0, 0.55) !important;
  cursor: grabbing !important;
}
</style>
