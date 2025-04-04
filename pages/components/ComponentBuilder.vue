<script setup>
import {
  CheckSquareIcon,
  InfoIcon,
  LayersIcon,
  ListIcon,
  MessageSquareIcon,
  SliderIcon,
  SquareIcon,
  TabsIcon,
  ToggleLeftIcon,
  TypeIcon,
  XIcon,
} from 'lucide-vue-next'
import { defineComponent, h, onMounted, onUnmounted, ref } from 'vue'

// Mock shadcn components for the example
// In a real implementation, you would import these from your shadcn-vue library
const Button = defineComponent({ render: () => h('div') })
const Card = defineComponent({ render: () => h('div') })
const Input = defineComponent({ render: () => h('div') })
const Select = defineComponent({ render: () => h('div') })
const Checkbox = defineComponent({ render: () => h('div') })
const Switch = defineComponent({ render: () => h('div') })
const Slider = defineComponent({ render: () => h('div') })
const Tabs = defineComponent({ render: () => h('div') })
const Accordion = defineComponent({ render: () => h('div') })
const Dialog = defineComponent({ render: () => h('div') })
const Tooltip = defineComponent({ render: () => h('div') })

// Component definitions
const components = [
  { id: 'button', name: 'Button', icon: SquareIcon, preview: 'Button', component: Button },
  { id: 'card', name: 'Card', icon: LayersIcon, preview: 'Card', component: Card },
  { id: 'input', name: 'Input', icon: TypeIcon, preview: 'Input', component: Input },
  { id: 'select', name: 'Select', icon: ListIcon, preview: 'Select', component: Select },
  { id: 'checkbox', name: 'Checkbox', icon: CheckSquareIcon, preview: 'Checkbox', component: Checkbox },
  { id: 'switch', name: 'Switch', icon: ToggleLeftIcon, preview: 'Switch', component: Switch },
  { id: 'slider', name: 'Slider', icon: SliderIcon, preview: 'Slider', component: Slider },
  { id: 'tabs', name: 'Tabs', icon: TabsIcon, preview: 'Tabs', component: Tabs },
  { id: 'accordion', name: 'Accordion', icon: LayersIcon, preview: 'Accordion', component: Accordion },
  { id: 'dialog', name: 'Dialog', icon: MessageSquareIcon, preview: 'Dialog', component: Dialog },
  { id: 'tooltip', name: 'Tooltip', icon: InfoIcon, preview: 'Tooltip', component: Tooltip },
]

// Canvas items
const canvasItems = ref([])
const draggedItem = ref(null)
const dragOffset = ref({ x: 0, y: 0 })
const isDraggingOver = ref(false)

// Drag from sidebar to canvas
function onDragStart(event, component) {
  event.dataTransfer.setData('componentId', component.id)
}

function onDragOver(event) {
  event.preventDefault()
  isDraggingOver.value = true
}

function onDragLeave() {
  isDraggingOver.value = false
}

function onDrop(event) {
  event.preventDefault()
  isDraggingOver.value = false

  const componentId = event.dataTransfer.getData('componentId')
  const component = components.find(c => c.id === componentId)

  if (component) {
    const canvasRect = event.currentTarget.getBoundingClientRect()
    const x = event.clientX - canvasRect.left - 50 // Center the component
    const y = event.clientY - canvasRect.top - 20

    canvasItems.value.push({
      component,
      x: Math.max(0, x),
      y: Math.max(0, y),
    })
  }
}

// Drag within canvas
function startDrag(event, index) {
  draggedItem.value = index
  const element = event.currentTarget
  const rect = element.getBoundingClientRect()

  dragOffset.value = {
    x: event.clientX - rect.left,
    y: event.clientY - rect.top,
  }

  document.addEventListener('mousemove', onMouseMove)
  document.addEventListener('mouseup', stopDrag)

  event.preventDefault()
}

function onMouseMove(event) {
  if (draggedItem.value !== null) {
    const canvasRect = document.getElementById('canvas').getBoundingClientRect()
    const x = event.clientX - canvasRect.left - dragOffset.value.x
    const y = event.clientY - canvasRect.top - dragOffset.value.y

    // Ensure the component stays within the canvas
    const boundedX = Math.max(0, Math.min(x, canvasRect.width - 100))
    const boundedY = Math.max(0, Math.min(y, canvasRect.height - 50))

    canvasItems.value[draggedItem.value].x = boundedX
    canvasItems.value[draggedItem.value].y = boundedY
  }
}

function stopDrag() {
  draggedItem.value = null
  document.removeEventListener('mousemove', onMouseMove)
  document.removeEventListener('mouseup', stopDrag)
}

// Remove item from canvas
function removeItem(index) {
  canvasItems.value.splice(index, 1)
}

// Clear canvas
function clearCanvas() {
  canvasItems.value = []
}

// Export HTML
function exportHTML() {
  let html = '<!DOCTYPE html>\n<html lang="en">\n<head>\n'
  html += '  <meta charset="UTF-8">\n'
  html += '  <meta name="viewport" content="width=device-width, initial-scale=1.0">\n'
  html += '  <title>Exported Components</title>\n'
  html += '  <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">\n'
  html += '</head>\n<body class="bg-white dark:bg-gray-900 min-h-screen p-8">\n'
  html += '  <div class="container mx-auto relative" style="height: 600px;">\n'

  canvasItems.value.forEach((item) => {
    html += `    <div class="absolute p-2 border border-gray-200 rounded-md bg-white dark:bg-gray-800 shadow-sm" style="left: ${item.x}px; top: ${item.y}px;">\n`
    html += `      <div class="text-sm font-medium mb-1">${item.component.name}</div>\n`

    // Generate placeholder HTML based on component type
    switch (item.component.id) {
      case 'button':
        html += '      <button class="px-4 py-2 bg-blue-500 text-white rounded-md">Button</button>\n'
        break
      case 'card':
        html += '      <div class="p-4 border rounded-md shadow-sm w-64">\n'
        html += '        <h3 class="text-lg font-medium">Card Title</h3>\n'
        html += '        <p class="text-gray-500">Card content goes here</p>\n'
        html += '      </div>\n'
        break
      case 'input':
        html += '      <input type="text" class="px-3 py-2 border rounded-md w-full" placeholder="Input">\n'
        break
      case 'select':
        html += '      <select class="px-3 py-2 border rounded-md w-full">\n'
        html += '        <option>Option 1</option>\n'
        html += '        <option>Option 2</option>\n'
        html += '      </select>\n'
        break
      case 'checkbox':
        html += '      <label class="flex items-center gap-2">\n'
        html += '        <input type="checkbox" class="rounded">\n'
        html += '        <span>Checkbox</span>\n'
        html += '      </label>\n'
        break
      case 'switch':
        html += '      <label class="relative inline-flex items-center cursor-pointer">\n'
        html += '        <input type="checkbox" class="sr-only peer">\n'
        html += '        <div class="w-11 h-6 bg-gray-200 peer-focus:outline-none rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[\'\'] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-blue-600"></div>\n'
        html += '      </label>\n'
        break
      default:
        html += `      <div class="p-2 border rounded-md">${item.component.name} Placeholder</div>\n`
    }

    html += '    </div>\n'
  })

  html += '  </div>\n</body>\n</html>'

  // Create a download link
  const blob = new Blob([html], { type: 'text/html' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = 'exported-components.html'
  document.body.appendChild(a)
  a.click()
  document.body.removeChild(a)
  URL.revokeObjectURL(url)
}

// Component preview
// eslint-disable-next-line vue/one-component-per-file
const ComponentPreview = defineComponent({
  props: {
    component: Object,
  },
  setup(props) {
    return () => {
      const component = props.component

      switch (component.id) {
        case 'button':
          return h('button', { class: 'px-4 py-2 bg-primary text-white rounded-md' }, 'Button')
        case 'card':
          return h('div', { class: 'p-4 border rounded-md shadow-sm w-64' }, [
            h('h3', { class: 'text-lg font-medium text-primary' }, 'Card Title'),
            h('p', { class: 'text-primary/70' }, 'Card content goes here'),
          ])
        case 'input':
          return h('input', {
            type: 'text',
            class: 'px-3 py-2 border rounded-md w-full',
            placeholder: 'Input',
          })
        case 'select':
          return h('select', { class: 'px-3 py-2 border rounded-md w-full' }, [
            h('option', null, 'Option 1'),
            h('option', null, 'Option 2'),
          ])
        case 'checkbox':
          return h('label', { class: 'flex items-center gap-2' }, [
            h('input', { type: 'checkbox', class: 'rounded' }),
            h('span', null, 'Checkbox'),
          ])
        case 'switch':
          return h('div', { class: 'w-10 h-5 bg-primary/30 rounded-full relative' }, [
            h('div', { class: 'absolute w-4 h-4 bg-white rounded-full top-0.5 left-0.5' }),
          ])
        case 'slider':
          return h('div', { class: 'w-full h-2 bg-primary/20 rounded-full relative' }, [
            h('div', { class: 'absolute w-4 h-4 bg-primary rounded-full top-1/2 -translate-y-1/2', style: 'left: 30%' }),
          ])
        case 'tabs':
          return h('div', { class: 'border-b' }, [
            h('div', { class: 'flex gap-4' }, [
              h('div', { class: 'px-4 py-2 border-b-2 border-primary text-primary' }, 'Tab 1'),
              h('div', { class: 'px-4 py-2 text-primary/50' }, 'Tab 2'),
            ]),
          ])
        case 'accordion':
          return h('div', { class: 'border rounded-md overflow-hidden' }, [
            h('div', { class: 'p-3 bg-primary/5 font-medium flex justify-between items-center' }, [
              h('span', null, 'Accordion Title'),
              h('span', null, '+'),
            ]),
          ])
        case 'dialog':
          return h('div', { class: 'border rounded-md p-4 w-64 shadow-sm' }, [
            h('div', { class: 'text-lg font-medium mb-2' }, 'Dialog Title'),
            h('p', { class: 'text-sm mb-4' }, 'Dialog content goes here'),
            h('div', { class: 'flex justify-end gap-2' }, [
              h('button', { class: 'px-3 py-1 border rounded-md' }, 'Cancel'),
              h('button', { class: 'px-3 py-1 bg-primary text-white rounded-md' }, 'Confirm'),
            ]),
          ])
        case 'tooltip':
          return h('div', { class: 'relative inline-block' }, [
            h('button', { class: 'px-3 py-1 bg-primary/10 rounded-md' }, 'Hover me'),
            h('div', { class: 'absolute bottom-full mb-2 px-2 py-1 bg-primary text-white text-xs rounded' }, 'Tooltip text'),
          ])
        default:
          return h('div', null, component.name)
      }
    }
  },
})

// Clean up event listeners
onUnmounted(() => {
  document.removeEventListener('mousemove', onMouseMove)
  document.removeEventListener('mouseup', stopDrag)
})
</script>

<template>
  <div class="min-h-screen bg-white text-primary-foreground dark:bg-[#141c26]">
    <div class="mx-auto p-4 container">
      <header class="mb-6">
        <h1 class="text-2xl text-primary font-bold">
          Component Builder
        </h1>
        <p class="text-sm text-primary/80">
          Drag and drop components to build your UI
        </p>
      </header>

      <div class="flex flex-col gap-6 md:flex-row">
        <!-- Sidebar -->
        <div class="h-[calc(100vh-8rem)] w-full overflow-y-auto rounded-lg bg-white p-4 shadow-md md:w-64 dark:bg-[#1e2736]">
          <h2 class="mb-4 text-lg text-primary font-semibold">
            Components
          </h2>
          <div class="space-y-2">
            <div
              v-for="component in components"
              :key="component.id"
              class="cursor-move rounded-md bg-primary/5 p-3 transition-colors hover:bg-primary/10"
              draggable="true"
              @dragstart="onDragStart($event, component)"
            >
              <div class="flex items-center gap-2">
                <component :is="component.icon" class="h-4 w-4 text-primary" />
                <span class="text-sm text-primary font-medium">{{ component.name }}</span>
              </div>
            </div>
          </div>
        </div>

        <!-- Canvas -->
        <div
          class="relative h-[calc(100vh-8rem)] flex-1 overflow-auto rounded-lg bg-white p-4 shadow-md dark:bg-[#1e2736]"
          @dragover="onDragOver"
          @dragleave="onDragLeave"
          @drop="onDrop"
        >
          <div class="absolute right-4 top-4 flex gap-2">
            <button
              class="rounded-md bg-red-500 px-3 py-1.5 text-sm text-white transition-colors hover:bg-red-600"
              @click="clearCanvas"
            >
              Clear
            </button>
            <button
              class="rounded-md bg-primary px-3 py-1.5 text-sm text-white transition-colors hover:bg-primary/90"
              @click="exportHTML"
            >
              Export HTML
            </button>
          </div>

          <div
            id="canvas"
            class="relative h-full w-full border-2 rounded-md border-dashed p-4 transition-colors"
            :class="{
              'flex items-center justify-center': !canvasItems.length,
              'border-primary/50 bg-primary/5': isDraggingOver,
              'border-primary/20': !isDraggingOver,
            }"
          >
            <p v-if="!canvasItems.length" class="text-center text-primary/50">
              Drag components here to build your UI
            </p>

            <div
              v-for="(item, index) in canvasItems"
              :key="index"
              class="absolute border border-primary/20 rounded-md bg-white p-2 shadow-sm dark:bg-[#1e2736]"
              :style="{ left: `${item.x}px`, top: `${item.y}px` }"
              @mousedown="startDrag($event, index)"
            >
              <div class="mb-1 flex items-center justify-between">
                <span class="text-xs text-primary/70">{{ item.component.name }}</span>
                <button class="text-red-500 hover:text-red-600" @click="removeItem(index)">
                  <XIcon class="h-3 w-3" />
                </button>
              </div>
              <ComponentPreview :component="item.component" />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

  <style>
  :root {
  --primary: 250 95% 60%;
  --primary-foreground: 0 0% 100%;
}

.dark {
  --primary: 250 95% 60%;
  --primary-foreground: 0 0% 100%;
}

/* Prevent text selection during drag */
.cursor-move {
  user-select: none;
}
</style>
