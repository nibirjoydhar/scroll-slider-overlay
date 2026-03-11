# 📜 Custom Overlay Scrollbar - Vue 3

A **production-ready custom vertical scrollbar overlay component** for Vue 3 that works seamlessly with any overflowing container (tables, divs, lists, etc.). Perfect for integrating into existing projects.

## ⚡ Quick Integration (2 Minutes)

### Step 1: Copy Component File

Copy this **entire file** to your project:

```
src/components/ui/OverlayScrollbar.vue
```

Just copy the single file - no dependencies, no extra setup needed!

### Step 2: Use in Your Component

```vue
<script setup>
import OverlayScrollbar from '@/components/ui/OverlayScrollbar.vue'
</script>

<template>
  <OverlayScrollbar class="my-table">
    <!-- Your content here -->
    <table>
      <!-- rows... -->
    </table>
  </OverlayScrollbar>
</template>

<style scoped>
.my-table {
  height: 500px; /* REQUIRED */
}
</style>
```

**That's it!** The component works out of the box.

---

## 📋 What to Copy into New Project

### **Option A: Just the Component (Recommended)**

Copy **only this file**:
- `src/components/ui/OverlayScrollbar.vue`

**Into your project:**
- `your-project/src/components/ui/OverlayScrollbar.vue`

Then import and use as shown above.

### **Option B: Full Demo Setup**

If you want the demo page too, also copy:
- `src/pages/TableDemo.vue` - Demo page with 150-row table
- `src/style.css` - Global styles (optional, merge with yours)

---

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

---

## 🚀 Usage Examples

### Basic Table

```vue
<script setup>
import OverlayScrollbar from '@/components/ui/OverlayScrollbar.vue'
import { ref } from 'vue'

const rows = ref(Array.from({ length: 100 }, (_, i) => ({ id: i + 1, name: `Item ${i + 1}` })))
</script>

<template>
  <OverlayScrollbar class="table-container">
    <table>
      <thead>
        <tr>
          <th>ID</th>
          <th>Name</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in rows" :key="row.id">
          <td>{{ row.id }}</td>
          <td>{{ row.name }}</td>
        </tr>
      </tbody>
    </table>
  </OverlayScrollbar>
</template>

<style scoped>
.table-container {
  height: 600px;
}
</style>
```

### Chat Messages List

```vue
<template>
  <div class="chat">
    <OverlayScrollbar class="messages">
      <div v-for="msg in messages" :key="msg.id" class="message">
        <strong>{{ msg.sender }}:</strong> {{ msg.text }}
      </div>
    </OverlayScrollbar>
    <input v-model="newMessage" @keyup.enter="sendMessage" placeholder="Type message..." />
  </div>
</template>

<style scoped>
.messages {
  height: 400px;
}
</style>
```

### Div with Long Content

```vue
<template>
  <OverlayScrollbar class="content-box">
    <div class="article">
      <h2>Long Article</h2>
      <p>Your long content here...</p>
    </div>
  </OverlayScrollbar>
</template>

<style scoped>
.content-box {
  height: 500px;
}
</style>
```

---

## 🎨 Customization

### Change Scrollbar Colors

```vue
<template>
  <OverlayScrollbar class="custom-scroll">
    <!-- content -->
  </OverlayScrollbar>
</template>

<style scoped>
/* Dark theme */
.custom-scroll :deep(.scrollbar-thumb) {
  background: #333 !important;
}

.custom-scroll :deep(.scrollbar-thumb:hover) {
  background: #555 !important;
}

/* Blue accent */
.custom-scroll :deep(.scrollbar-thumb) {
  background: #3b82f6 !important;
}

.custom-scroll :deep(.scrollbar-thumb:hover) {
  background: #2563eb !important;
}

.custom-scroll :deep(.scrollbar-track.is-dragging .scrollbar-thumb) {
  background: #1d4ed8 !important;
}
</style>
```

### Change Scrollbar Width

```vue
<style scoped>
.custom-scroll :deep(.scrollbar-track) {
  width: 16px !important; /* Default is 10px */
}</style>
```

---

## 🔑 Requirements

- Vue 3.5+ (with Composition API)
- TypeScript (optional, but recommended)
- No external dependencies needed!

Your existing project just needs Vue 3. Copy the component and use it immediately.

---

## 📖 How It Works

### Scroll Metrics

The component calculates:

```
thumbHeight = (visibleHeight / totalHeight) * containerHeight
thumbTop = (scrollPosition / maxScroll) * (containerHeight - thumbHeight)
```

### Auto-responsive

Updates automatically when:
- User scrolls with mouse wheel
- User drags the thumb
- Container/content is resized
- Window is resized

### No Dependencies

The component uses only:
- Vue 3 Composition API
- CSS 3 for styling
- ResizeObserver API (all modern browsers)

---

## 🐛 Troubleshooting

### Scrollbar not visible?

Check:
```vue
<style scoped>
.container {
  height: 500px; /* Must have fixed height */
  /* Container content must exceed this height */
}
</style>
```

### Scrollbar not responding?

- Ensure container has `height` set
- Check that content overflows the container
- Verify Vue 3 is installed

### Styling conflicts?

Use `:deep()` selector:
```vue
<style scoped>
.my-scroll :deep(.scrollbar-thumb) {
  background: red !important;
}
</style>
```

---

## 📊 CSS Classes Reference

| Class | Purpose |
|-------|---------|
| `.scrollbar-wrapper` | Outer container |
| `.scrollbar-container` | Scrollable area |
| `.scrollbar-track` | Track/rail |
| `.scrollbar-track.is-hover` | Track on hover |
| `.scrollbar-track.is-dragging` | Track while dragging |
| `.scrollbar-thumb` | Draggable thumb |

---

## 💡 Tips

1. **Always set height** - Container must have fixed height for overflow
2. **Content must overflow** - Content must be taller than container
3. **Works with any content** - Tables, divs, lists, etc.
4. **Responsive** - Automatically adapts to size changes
5. **Performance** - Uses ResizeObserver, not polling

---

## 🎯 Common Integration Pattern

```vue
<script setup lang="ts">
import OverlayScrollbar from '@/components/ui/OverlayScrollbar.vue'
import { ref } from 'vue'

const data = ref(Array.from({ length: 100 }, (_, i) => ({
  id: i + 1,
  name: `Item ${i + 1}`,
  email: `item${i + 1}@example.com`,
  status: ['Active', 'Inactive'][Math.random() > 0.5 ? 0 : 1],
})))
</script>

<template>
  <div class="wrapper">
    <h2>Data Table</h2>
    
    <!-- Wrap scrollable content with OverlayScrollbar -->
    <OverlayScrollbar class="table-scroll">
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Email</th>
            <th>Status</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in data" :key="item.id">
            <td>{{ item.id }}</td>
            <td>{{ item.name }}</td>
            <td>{{ item.email }}</td>
            <td>{{ item.status }}</td>
          </tr>
        </tbody>
      </table>
    </OverlayScrollbar>
  </div>
</template>

<style scoped>
.wrapper {
  padding: 20px;
}

.table-scroll {
  height: 500px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: #f5f5f5;
  position: sticky;
  top: 0;
}

th, td {
  padding: 12px;
  text-align: left;
  border-bottom: 1px solid #eee;
}
</style>
```

---

## ✅ Checklist for Integration

- [ ] Copy `OverlayScrollbar.vue` to `src/components/ui/`
- [ ] Import in your component
- [ ] Wrap your scrollable content
- [ ] Set height on the wrapper
- [ ] Test scrolling and dragging
- [ ] (Optional) Customize colors/width

**Done!** Your scrollbar is ready.

---

## 📞 Support

The component is self-contained and production-ready. For issues:

1. Check that container has `height` property
2. Verify content exceeds container height
3. Ensure Vue 3 is installed
4. Clear browser cache and rebuild

---

## 🙌 That's All!

Just copy one file and you're done. No configuration, no dependencies, no complexity.

**Copy `src/components/ui/OverlayScrollbar.vue` and start using it immediately!**
