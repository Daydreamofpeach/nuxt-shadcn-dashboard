<script setup>
import { computed, defineComponent, h, onMounted, onUnmounted, ref } from 'vue'

// Component definitions - no icons, just names
const components = [
  { id: 'button', name: 'Button' },
  { id: 'card', name: 'Card' },
  { id: 'input', name: 'Input' },
  { id: 'select', name: 'Select' },
  { id: 'checkbox', name: 'Checkbox' },
  { id: 'switch', name: 'Switch' },
  { id: 'slider', name: 'Slider' },
  { id: 'tabs', name: 'Tabs' },
  { id: 'accordion', name: 'Accordion' },
  { id: 'dialog', name: 'Dialog' },
  { id: 'tooltip', name: 'Tooltip' },
]

// Color options for the editor
const colorOptions = [
  { name: 'Primary', value: 'rgb(124, 58, 237)' },
  { name: 'Red', value: '#ef4444' },
  { name: 'Green', value: '#22c55e' },
  { name: 'Blue', value: '#3b82f6' },
  { name: 'Yellow', value: '#eab308' },
  { name: 'Pink', value: '#ec4899' },
  { name: 'Gray', value: '#6b7280' },
  { name: 'White', value: '#ffffff' },
  { name: 'Black', value: '#000000' },
]

// Canvas items
const canvasItems = ref([])
const draggedItem = ref(null)
const dragOffset = ref({ x: 0, y: 0 })
const isDraggingOver = ref(false)

// Editor state
const editingItem = ref(null)
const editingProperties = ref({})

// Default properties for each component type
function getDefaultProperties(componentId) {
  const common = {
    bgColor: 'rgb(124, 58, 237)',
    textColor: '#ffffff',
  }

  switch (componentId) {
    case 'button':
      return {
        ...common,
        text: 'Button',
        size: 'md',
        variant: 'solid',
      }
    case 'card':
      return {
        ...common,
        bgColor: '#ffffff',
        textColor: '#000000',
        title: 'Card Title',
        content: 'Card content goes here',
        radius: 'md',
      }
    case 'input':
      return {
        ...common,
        bgColor: '#ffffff',
        textColor: '#000000',
        placeholder: 'Enter text...',
        type: 'text',
      }
    case 'select':
      return {
        ...common,
        bgColor: '#ffffff',
        textColor: '#000000',
        options: ['Option 1', 'Option 2', 'Option 3'],
      }
    case 'checkbox':
    case 'switch':
      return {
        ...common,
        label: 'Toggle me',
        checked: false,
      }
    case 'slider':
      return {
        ...common,
        min: 0,
        max: 100,
        value: 50,
      }
    case 'tabs':
      return {
        ...common,
        tabs: ['Tab 1', 'Tab 2', 'Tab 3'],
        activeTab: 0,
      }
    case 'accordion':
      return {
        ...common,
        title: 'Accordion Title',
        content: 'Accordion content goes here',
        isOpen: false,
      }
    case 'dialog':
      return {
        ...common,
        title: 'Dialog Title',
        content: 'Dialog content goes here',
        confirmText: 'Confirm',
        cancelText: 'Cancel',
      }
    case 'tooltip':
      return {
        ...common,
        text: 'Tooltip text',
        triggerText: 'Hover me',
      }
    default:
      return common
  }
}

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
    const canvasRect = document.getElementById('canvas').getBoundingClientRect()
    const x = event.clientX - canvasRect.left - 50 // Center the component
    const y = event.clientY - canvasRect.top - 20

    canvasItems.value.push({
      component,
      properties: getDefaultProperties(component.id),
      x: Math.max(0, x),
      y: Math.max(0, y),
    })
  }
}

// Drag within canvas
function startDrag(event, index) {
  // Don't start drag if we're clicking on the remove button
  if (event.target.tagName === 'BUTTON') {
    return
  }

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

// Component editor functions
function openEditor(index) {
  editingItem.value = index
  // Clone the properties to avoid direct mutation
  editingProperties.value = JSON.parse(JSON.stringify(canvasItems.value[index].properties))
}

function closeEditor() {
  editingItem.value = null
  editingProperties.value = {}
}

function updateProperty(property, value) {
  editingProperties.value[property] = value
}

function saveChanges() {
  if (editingItem.value !== null) {
    canvasItems.value[editingItem.value].properties = editingProperties.value
    closeEditor()
  }
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

    // Generate HTML based on component type and properties
    const props = item.properties || {}

    switch (item.component.id) {
      case 'button':
        html += `      <button class="px-4 py-2 rounded-md" style="background-color: ${props.bgColor}; color: ${props.textColor};">${props.text || 'Button'}</button>\n`
        break
      case 'card':
        html += `      <div class="p-4 border rounded-md shadow-sm w-64" style="background-color: ${props.bgColor};">\n`
        html += `        <h3 class="text-lg font-medium" style="color: ${props.textColor};">${props.title || 'Card Title'}</h3>\n`
        html += `        <p style="color: ${props.textColor};">${props.content || 'Card content goes here'}</p>\n`
        html += '      </div>\n'
        break
      case 'input':
        html += `      <input type="${props.type || 'text'}" class="px-3 py-2 border rounded-md w-full" placeholder="${props.placeholder || 'Enter text...'}" style="background-color: ${props.bgColor}; color: ${props.textColor};">\n`
        break
      case 'select':
        html += `      <select class="px-3 py-2 border rounded-md w-full" style="background-color: ${props.bgColor}; color: ${props.textColor};">\n`
        if (props.options && Array.isArray(props.options)) {
          props.options.forEach((option) => {
            html += `        <option>${option}</option>\n`
          })
        }
        else {
          html += '        <option>Option 1</option>\n'
          html += '        <option>Option 2</option>\n'
        }
        html += '      </select>\n'
        break
      case 'checkbox':
        html += '      <label class="flex items-center gap-2">\n'
        html += `        <input type="checkbox" class="rounded" ${props.checked ? 'checked' : ''}>\n`
        html += `        <span style="color: ${props.textColor};">${props.label || 'Checkbox'}</span>\n`
        html += '      </label>\n'
        break
      case 'switch':
        html += '      <label class="relative inline-flex items-center cursor-pointer">\n'
        html += '        <input type="checkbox" class="sr-only peer">\n'
        html += `        <div class="w-11 h-6 bg-gray-200 peer-focus:outline-none rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all" style="background-color: ${props.checked ? props.bgColor : '#d1d5db'};"></div>\n`
        html += `        <span class="ml-3" style="color: ${props.textColor};">${props.label || 'Toggle'}</span>\n`
        html += '      </label>\n'
        break
      default:
        html += `      <div class="p-2 border rounded-md" style="background-color: ${props.bgColor}; color: ${props.textColor};">${item.component.name} ${props.label || ''}</div>\n`
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
const ComponentPreview = defineComponent({
  props: {
    component: Object,
    properties: Object,
  },
  setup(props) {
    return () => {
      const component = props.component
      const properties = props.properties || {}

      // Common styles based on properties
      const getStyles = () => {
        return {
          backgroundColor: properties.bgColor || 'rgb(124, 58, 237)',
          color: properties.textColor || '#ffffff',
        }
      }

      switch (component.id) {
        case 'button':
          return h('button', {
            class: 'px-4 py-2 rounded-md',
            style: getStyles(),
          }, properties.text || 'Button')

        case 'card':
          return h('div', {
            class: 'p-4 border rounded-md shadow-sm w-64',
            style: { backgroundColor: properties.bgColor || '#ffffff' },
          }, [
            h('h3', {
              class: 'text-lg font-medium',
              style: { color: properties.textColor || '#000000' },
            }, properties.title || 'Card Title'),
            h('p', {
              style: { color: properties.textColor || '#000000' },
            }, properties.content || 'Card content goes here'),
          ])

        case 'input':
          return h('input', {
            type: properties.type || 'text',
            class: 'px-3 py-2 border rounded-md w-full',
            placeholder: properties.placeholder || 'Input',
            style: {
              backgroundColor: properties.bgColor || '#ffffff',
              color: properties.textColor || '#000000',
            },
          })

        case 'select':
          return h('select', {
            class: 'px-3 py-2 border rounded-md w-full',
            style: {
              backgroundColor: properties.bgColor || '#ffffff',
              color: properties.textColor || '#000000',
            },
          }, [
            h('option', null, 'Option 1'),
            h('option', null, 'Option 2'),
          ])

        case 'checkbox':
          return h('label', { class: 'flex items-center gap-2' }, [
            h('input', { type: 'checkbox', class: 'rounded' }),
            h('span', {
              style: { color: properties.textColor || '#000000' },
            }, properties.label || 'Checkbox'),
          ])

        case 'switch':
          return h('div', {
            class: 'w-10 h-5 rounded-full relative',
            style: { backgroundColor: properties.bgColor ? `${properties.bgColor}50` : 'rgba(124, 58, 237, 0.3)' },
          }, [
            h('div', {
              class: 'absolute w-4 h-4 rounded-full top-0.5 left-0.5',
              style: { backgroundColor: properties.bgColor || 'rgb(124, 58, 237)' },
            }),
          ])

        case 'slider':
          return h('div', {
            class: 'w-full h-2 rounded-full relative',
            style: { backgroundColor: properties.bgColor ? `${properties.bgColor}20` : 'rgba(124, 58, 237, 0.2)' },
          }, [
            h('div', {
              class: 'absolute w-4 h-4 rounded-full top-1/2 -translate-y-1/2',
              style: {
                backgroundColor: properties.bgColor || 'rgb(124, 58, 237)',
                left: '30%',
              },
            }),
          ])

        case 'tabs':
          return h('div', { class: 'border-b' }, [
            h('div', { class: 'flex gap-4' }, [
              h('div', {
                class: 'px-4 py-2 border-b-2',
                style: {
                  borderColor: properties.bgColor || 'rgb(124, 58, 237)',
                  color: properties.bgColor || 'rgb(124, 58, 237)',
                },
              }, 'Tab 1'),
              h('div', {
                class: 'px-4 py-2',
                style: { color: properties.textColor || '#000000' },
              }, 'Tab 2'),
            ]),
          ])

        case 'accordion':
          return h('div', { class: 'border rounded-md overflow-hidden' }, [
            h('div', {
              class: 'p-3 font-medium flex justify-between items-center',
              style: {
                backgroundColor: properties.bgColor ? `${properties.bgColor}10` : 'rgba(124, 58, 237, 0.05)',
                color: properties.textColor || '#000000',
              },
            }, [
              h('span', null, properties.title || 'Accordion Title'),
              h('span', null, '+'),
            ]),
          ])

        case 'dialog':
          return h('div', {
            class: 'border rounded-md p-4 w-64 shadow-sm',
            style: { backgroundColor: properties.bgColor || '#ffffff' },
          }, [
            h('div', {
              class: 'text-lg font-medium mb-2',
              style: { color: properties.textColor || '#000000' },
            }, properties.title || 'Dialog Title'),
            h('p', {
              class: 'text-sm mb-4',
              style: { color: properties.textColor || '#000000' },
            }, properties.content || 'Dialog content goes here'),
            h('div', { class: 'flex justify-end gap-2' }, [
              h('button', { class: 'px-3 py-1 border rounded-md' }, properties.cancelText || 'Cancel'),
              h('button', {
                class: 'px-3 py-1 rounded-md',
                style: {
                  backgroundColor: properties.bgColor || 'rgb(124, 58, 237)',
                  color: '#ffffff',
                },
              }, properties.confirmText || 'Confirm'),
            ]),
          ])

        case 'tooltip':
          return h('div', { class: 'relative inline-block' }, [
            h('button', {
              class: 'px-3 py-1 rounded-md',
              style: {
                backgroundColor: properties.bgColor ? `${properties.bgColor}10` : 'rgba(124, 58, 237, 0.1)',
                color: properties.textColor || '#000000',
              },
            }, properties.triggerText || 'Hover me'),
            h('div', {
              class: 'absolute bottom-full mb-2 px-2 py-1 text-xs rounded',
              style: {
                backgroundColor: properties.bgColor || 'rgb(124, 58, 237)',
                color: '#ffffff',
              },
            }, properties.text || 'Tooltip text'),
          ])

        default:
          return h('div', {
            style: getStyles(),
          }, component.name)
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
        <h1 class="text-2xl text-muted-forground font-bold">
          Component Builder
        </h1>
        <p class="text-sm text-primary/80">
          Drag and drop components to build your UI
        </p>
      </header>

      <div class="flex  gap-6 flex-row">
        <!-- Sidebar -->
        <div class="h-[calc(100vh-8rem)] w-44 overflow-y-auto rounded-lg bg-white p-4 shadow-md md:w-64 dark:bg-[#1e2736]">
          <h2 class="mb-4 text-lg text-muted-forground font-semibold">
            Components
          </h2>
          <div class="space-y-2">
            <div
              v-for="component in components"
              :key="component.id"
              class="cursor-move rounded-md bg-primary/5 p-3 transition-colors hover:bg-primary/30"
              draggable="true"
              @dragstart="onDragStart($event, component)"
            >
              <div class="flex items-center gap-2">
                <span class="text-sm text-muted-forground active:text-primary font-medium">{{ component.name }}</span>
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
              @dblclick="openEditor(index)"
            >
              <div class="mb-1 flex items-center justify-between">
                <span class="text-xs text-primary/70">{{ item.component.name }}</span>
                <button class="text-red-500 hover:text-red-600" @click="removeItem(index)">
                  ✕
                </button>
              </div>
              <ComponentPreview :component="item.component" :properties="item.properties" />
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Component Editor Modal -->
    <div v-if="editingItem !== null" class="fixed inset-0 z-50 flex items-center justify-center bg-black/50">
      <div class="max-h-[80vh] max-w-md w-full overflow-auto rounded-lg bg-white shadow-xl dark:bg-[#1e2736]">
        <div class="flex items-center justify-between border-b border-gray-200 p-4 dark:border-gray-700">
          <h3 class="text-lg text-primary font-semibold">
            Edit {{ canvasItems[editingItem]?.component.name }}
          </h3>
          <button class="text-gray-500 dark:text-gray-400 hover:text-gray-700 dark:hover:text-gray-200" @click="closeEditor">
            ✕
          </button>
        </div>

        <div class="p-4">
          <!-- Common Properties -->
          <div class="mb-4">
            <label class="mb-1 block text-sm font-medium">Background Color</label>
            <div class="flex gap-2">
              <button
                v-for="color in colorOptions"
                :key="color.value"
                class="h-8 w-8 border rounded-full"
                :class="{ 'ring-2 ring-primary': editingProperties.bgColor === color.value }"
                :style="{ backgroundColor: color.value }"
                @click="updateProperty('bgColor', color.value)"
              />
            </div>
          </div>

          <div class="mb-4">
            <label class="mb-1 block text-sm font-medium">Text Color</label>
            <div class="flex gap-2">
              <button
                v-for="color in colorOptions"
                :key="color.value"
                class="h-8 w-8 border rounded-full"
                :class="{ 'ring-2 ring-primary': editingProperties.textColor === color.value }"
                :style="{ backgroundColor: color.value }"
                @click="updateProperty('textColor', color.value)"
              />
            </div>
          </div>

          <!-- Component-specific properties -->
          <template v-if="editingItem !== null">
            <template v-if="canvasItems[editingItem]?.component.id === 'button'">
              <div class="mb-4">
                <label class="mb-1 block text-sm font-medium">Button Text</label>
                <input
                  v-model="editingProperties.text"
                  type="text"
                  class="w-full border rounded-md px-3 py-2"
                >
              </div>
              <div class="mb-4">
                <label class="mb-1 block text-sm font-medium">Button Size</label>
                <select v-model="editingProperties.size" class="w-full border rounded-md px-3 py-2">
                  <option value="sm">
                    Small
                  </option>
                  <option value="md">
                    Medium
                  </option>
                  <option value="lg">
                    Large
                  </option>
                </select>
              </div>
              <div class="mb-4">
                <label class="mb-1 block text-sm font-medium">Button Style</label>
                <select v-model="editingProperties.variant" class="w-full border rounded-md px-3 py-2">
                  <option value="solid">
                    Solid
                  </option>
                  <option value="outline">
                    Outline
                  </option>
                  <option value="ghost">
                    Ghost
                  </option>
                </select>
              </div>
            </template>

            <template v-else-if="canvasItems[editingItem]?.component.id === 'card'">
              <div class="mb-4">
                <label class="mb-1 block text-sm font-medium">Card Title</label>
                <input
                  v-model="editingProperties.title"
                  type="text"
                  class="w-full border rounded-md px-3 py-2"
                >
              </div>
              <div class="mb-4">
                <label class="mb-1 block text-sm font-medium">Card Content</label>
                <textarea
                  v-model="editingProperties.content"
                  class="w-full border rounded-md px-3 py-2"
                  rows="3"
                />
              </div>
              <div class="mb-4">
                <label class="mb-1 block text-sm font-medium">Border Radius</label>
                <select v-model="editingProperties.radius" class="w-full border rounded-md px-3 py-2">
                  <option value="none">
                    None
                  </option>
                  <option value="sm">
                    Small
                  </option>
                  <option value="md">
                    Medium
                  </option>
                  <option value="lg">
                    Large
                  </option>
                </select>
              </div>
            </template>

            <template v-else-if="canvasItems[editingItem]?.component.id === 'input'">
              <div class="mb-4">
                <label class="mb-1 block text-sm font-medium">Placeholder</label>
                <input
                  v-model="editingProperties.placeholder"
                  type="text"
                  class="w-full border rounded-md px-3 py-2"
                >
              </div>
              <div class="mb-4">
                <label class="mb-1 block text-sm font-medium">Input Type</label>
                <select v-model="editingProperties.type" class="w-full border rounded-md px-3 py-2">
                  <option value="text">
                    Text
                  </option>
                  <option value="email">
                    Email
                  </option>
                  <option value="password">
                    Password
                  </option>
                  <option value="number">
                    Number
                  </option>
                </select>
              </div>
            </template>

            <template v-else>
              <div class="mb-4">
                <label class="mb-1 block text-sm font-medium">Label</label>
                <input
                  v-model="editingProperties.label"
                  type="text"
                  class="w-full border rounded-md px-3 py-2"
                >
              </div>
            </template>
          </template>
        </div>

        <div class="flex justify-end gap-2 border-t border-gray-200 p-4 dark:border-gray-700">
          <button
            class="border border-gray-300 rounded-md px-4 py-2 dark:border-gray-600"
            @click="closeEditor"
          >
            Cancel
          </button>
          <button
            class="rounded-md bg-primary px-4 py-2 text-white"
            @click="saveChanges"
          >
            Save Changes
          </button>
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
