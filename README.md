# 📜 Custom Overlay Scrollbar - Vue 3

A **production-ready custom vertical scrollbar overlay component** for Vue 3 that works seamlessly with any overflowing container (tables, divs, lists, etc.).

## ✨ Features

- ✅ **Overlay scrollbar** - Doesn't take up layout space
- ✅ **Dynamic thumb height** - Proportional to visible content
- ✅ **Smooth scrolling** - Mouse wheel and drag support
- ✅ **Fixed track** - Stays in place while content scrolls
- ✅ **Thumb tracking** - Updates position as you scroll
- ✅ **Draggable thumb** - Click and drag to scroll
- ✅ **Hover effects** - Visual feedback on interaction
- ✅ **Responsive** - Handles container/content size changes
- ✅ **No dependencies** - Pure Vue 3 + CSS
- ✅ **TypeScript support** - Fully typed

## 🚀 Quick Start

### Installation

```bash
# Install dependencies
npm install

# Development server
npm run dev

# Production build
npm run build
```

### Basic Usage

```vue
<script setup>
import OverlayScrollbar from '@/components/ui/OverlayScrollbar.vue'
</script>

<template>
  <OverlayScrollbar class="my-container">
    <!-- Your scrollable content here -->
    <table class="data-table">
      <!-- rows... -->
    </table>
  </OverlayScrollbar>
</template>

<style scoped>
.my-container {
  height: 500px; /* Required for overflow */
}
</style>
```

## 📂 Project Structure

```
src/
├── components/
│   └── ui/
│       └── OverlayScrollbar.vue    # Main scrollbar component
├── pages/
│   └── TableDemo.vue                # Demo with 150-row table
├── main.ts
├── App.vue
└── style.css
```

## 📖 How It Works

### Scroll Calculation

The thumb height is calculated as a ratio of visible content to total content:

```
thumbHeight = (clientHeight / scrollHeight) * containerHeight
```

The thumb position reflects the scroll position:

```
thumbTop = (scrollTop / maxScrollTop) * (containerHeight - thumbHeight)
```

### Events

| Event | Handler | Purpose |
|-------|---------|---------|
| `scroll` | Updates thumb position | Tracks scroll position |
| `mousedown` (thumb) | `handleThumbMouseDown()` | Starts drag |
| `mousemove` (document) | `handleMouseMove()` | Updates scroll during drag |
| `mouseup` (document) | `handleMouseUp()` | Ends drag |

## 🎨 Styling

### Default Styles

- **Track**: 10px width, positioned right side, transparent by default
- **Thumb**: Dynamic height, rounded corners, semi-transparent dark gradient
- **Colors**:
  - Default: `rgba(0, 0, 0, 0.35)` (35% opaque)
  - Hover: `rgba(0, 0, 0, 0.55)` (55% opaque)
  - Dragging: `rgba(0, 0, 0, 0.75)` (75% opaque)

### Custom Colors

```vue
<style scoped>
.my-scrollbar :deep(.scrollbar-thumb) {
  background: #667eea !important;
}

.my-scrollbar :deep(.scrollbar-thumb:hover) {
  background: #5a67d8 !important;
}
</style>
```

## 🔧 API

### Component Props

None - the component auto-adapts to content and parent container.

### Slots

- `default` - Your scrollable content

### CSS Classes

| Class | Purpose |
|-------|---------|
| `.scrollbar-wrapper` | Outer wrapper container |
| `.scrollbar-container` | Scrollable content area |
| `.scrollbar-track` | Scrollbar track/rail |
| `.scrollbar-track.is-hover` | Track on hover |
| `.scrollbar-track.is-dragging` | Track while dragging |
| `.scrollbar-thumb` | Draggable scroll thumb |

## 💡 Usage Examples

### Table with Fixed Height

```vue
<template>
  <OverlayScrollbar class="table-scroll">
    <table>
      <thead>
        <tr>
          <th>Header 1</th>
          <th>Header 2</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in rows" :key="row.id">
          <td>{{ row.col1 }}</td>
          <td>{{ row.col2 }}</td>
        </tr>
      </tbody>
    </table>
  </OverlayScrollbar>
</template>

<style scoped>
.table-scroll {
  height: 600px;
}
</style>
```

### Chat Message List

```vue
<template>
  <div class="chat-container">
    <OverlayScrollbar class="messages">
      <div v-for="msg in messages" :key="msg.id" class="message">
        <strong>{{ msg.sender }}:</strong> {{ msg.text }}
      </div>
    </OverlayScrollbar>
    <input v-model="newMessage" @keyup.enter="sendMessage" />
  </div>
</template>

<style scoped>
.messages {
  height: 500px;
}
</style>
```

### List with Items

```vue
<template>
  <OverlayScrollbar class="item-list">
    <div v-for="item in items" :key="item.id" class="item">
      {{ item.name }}
    </div>
  </OverlayScrollbar>
</template>

<style scoped>
.item-list {
  height: 400px;
}
</style>
```

## 🎯 Demo Page

A complete demo is included at `src/pages/TableDemo.vue` with:
- 150-row data table
- Status badges with color coding
- Professional styling
- Full responsive design

Run `npm run dev` to see the demo in action.

## 🔄 What Updates the Scrollbar

The scrollbar automatically updates when:

- ✓ User scrolls with mouse wheel
- ✓ User drags the scrollbar thumb
- ✓ Container is resized (ResizeObserver)
- ✓ Content size changes
- ✓ Window is resized

## 🌐 Browser Support

- Chrome/Chromium 65+
- Firefox 55+
- Safari 12.1+
- Edge 79+

## 📊 Performance

- **ResizeObserver** - Efficient size change detection
- **Computed properties** - Reactive without watchers
- **CSS transitions** - Hardware-accelerated
- **Minimal re-renders** - Optimized Vue reactivity

## 🐛 Troubleshooting

### Scrollbar not visible

- Ensure container has a fixed `height` property
- Check that content exceeds container height
- Verify scrolling is enabled (overflow-y: scroll)

### Scrollbar not updating

- Make sure ResizeObserver is initialized
- Check that scroll event is firing
- Verify computed metrics are being calculated

### Jumpy scrolling

- Check for competing scroll listeners
- Ensure no CSS conflicts (overflow, position)
- Clear browser cache and rebuild

## 📦 Bundle Size

- Component only: ~5KB (uncompressed)
- Production build: ~65KB (gzipped minified)

## 🛠️ Technologies

- **Vue 3** - Composition API with `<script setup>`
- **TypeScript** - Full type safety
- **CSS 3** - Modern styling with transitions
- **ResizeObserver API** - Size change detection

## 📄 License

Part of scroll-slider-overlay project.

## 🙌 Contributing

This component is production-ready and can be used in any Vue 3 project. Feel free to customize the styles and behavior to match your needs.

---

**Built with ❤️ using Vue 3 Composition API**
