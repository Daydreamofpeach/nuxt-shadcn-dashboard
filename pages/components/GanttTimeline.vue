<script setup>
import { ChevronLeft, ChevronRight, ZoomIn, ZoomOut } from 'lucide-vue-next'
import { DataSet, Timeline } from 'vis-timeline/standalone'
import { onMounted, onUnmounted, ref, watch } from 'vue'
import 'vis-timeline/styles/vis-timeline-graph2d.css'

// State
const timelineRef = ref(null)
const timeline = ref(null)
const showGroups = ref(true)
const showTooltip = ref(true)
const showMajorLabel = ref(true)
const showMinorLabel = ref(true)
const isDraggable = ref(true)
const allowZoom = ref(true)

// Initialize timeline
onMounted(() => {
  initializeTimeline()
})

// Cleanup on unmount
onUnmounted(() => {
  if (timeline.value) {
    timeline.value.destroy()
  }
})

// Watch for option changes
watch(
  [showGroups, showTooltip, showMajorLabel, showMinorLabel, isDraggable, allowZoom],
  () => {
    if (timeline.value) {
      timeline.value.destroy()
    }
    initializeTimeline()
  },
)

// Initialize timeline with data and options
function initializeTimeline() {
  if (!timelineRef.value)
    return

  // Sample data
  const groups = new DataSet([
    { id: 1, content: 'Planning', className: 'group-header', nestedGroups: [2, 3] },
    { id: 2, content: 'Briefing Meeting', className: 'sub-group' },
    { id: 3, content: 'Scope Definition', className: 'sub-group' },
    { id: 10, content: 'Visual Identity', className: 'group-header', nestedGroups: [11, 12, 13] },
    { id: 11, content: 'Logo Design', className: 'sub-group' },
    { id: 12, content: 'Visual Identity Guide', className: 'sub-group' },
    { id: 13, content: 'Final Presentation', className: 'sub-group' },
    { id: 20, content: 'Website Development', className: 'group-header', nestedGroups: [21, 22, 23] },
    { id: 21, content: 'Page Design', className: 'sub-group' },
    { id: 22, content: 'Front-end Development', className: 'sub-group' },
    { id: 23, content: 'Post-launch Support', className: 'sub-group' },
  ])

  const items = new DataSet([
    {
      id: 1,
      group: 2,
      content: `<div class="item-content">
                    <span class="item-title">Briefing Meeting</span>
                    <span class="item-date">01/04</span>
                    <span class="item-priority low">Low</span>
                    <div class="progress-bar" style="width: 11%"></div>
                  </div>`,
      title: 'CLINCH Fight Wear - Low - Progress 11%',
      start: new Date(2025, 2, 1),
      end: new Date(2025, 2, 5),
      className: 'timeline-item low-priority',
      progress: 11,
      priority: 'Low',
      assignee: 'Tulio Portela',
    },
    {
      id: 2,
      group: 3,
      content: `<div class="item-content">
                    <span class="item-title">Scope Definition</span>
                    <span class="item-date">04/03</span>
                    <span class="item-priority low">Low</span>
                    <div class="progress-bar" style="width: 100%"></div>
                  </div>`,
      title: 'CLINCH Fight Wear - Low - Progress 100%',
      start: new Date(2025, 1, 24),
      end: new Date(2025, 2, 4),
      className: 'timeline-item low-priority',
      progress: 100,
      priority: 'Low',
      assignee: 'Tulio Portela',
    },
    {
      id: 3,
      group: 11,
      content: `<div class="item-content">
                    <span class="item-title">Logo Design</span>
                    <span class="item-date">02/03</span>
                    <span class="item-priority urgent">Urgent</span>
                    <div class="progress-bar" style="width: 65%"></div>
                  </div>`,
      title: 'Shop Visual Identity - Urgent',
      start: new Date(2025, 1, 17),
      end: new Date(2025, 2, 2),
      className: 'timeline-item urgent-priority',
      progress: 65,
      priority: 'Urgent',
      assignee: 'Nani Medeiros',
    },
    {
      id: 4,
      group: 12,
      content: `<div class="item-content">
                    <span class="item-title">Visual Identity Guide</span>
                    <span class="item-date">29/03</span>
                    <span class="item-priority low">Low</span>
                    <div class="progress-bar" style="width: 45%"></div>
                  </div>`,
      title: 'CLINCH Fight Wear - Low - Progress 45%',
      start: new Date(2025, 2, 15),
      end: new Date(2025, 2, 29),
      className: 'timeline-item low-priority',
      progress: 45,
      priority: 'Low',
      assignee: 'Nestor Portela',
    },
    {
      id: 5,
      group: 13,
      content: `<div class="item-content">
                    <span class="item-title">Final Presentation</span>
                    <span class="item-date">20/03</span>
                    <span class="item-priority low">Low</span>
                    <div class="progress-bar" style="width: 89%"></div>
                  </div>`,
      title: 'Shop Visual Identity - Low - Progress 89%',
      start: new Date(2025, 2, 10),
      end: new Date(2025, 2, 20),
      className: 'timeline-item low-priority',
      progress: 89,
      priority: 'Low',
      assignee: 'Nani Medeiros',
    },
    {
      id: 6,
      group: 21,
      content: `<div class="item-content">
                    <span class="item-title">Page Design</span>
                    <span class="item-date">07/03</span>
                    <span class="item-priority urgent">Urgent</span>
                    <div class="progress-bar" style="width: 100%"></div>
                  </div>`,
      title: 'CLINCH Fight Wear - Urgent - Progress 100%',
      start: new Date(2025, 1, 25),
      end: new Date(2025, 2, 7),
      className: 'timeline-item urgent-priority',
      progress: 100,
      priority: 'Urgent',
      assignee: 'Nestor Portela',
    },
    {
      id: 7,
      group: 22,
      content: `<div class="item-content">
                    <span class="item-title">Front-end Development</span>
                    <span class="item-date">13/03</span>
                    <span class="item-priority high">High</span>
                    <div class="progress-bar" style="width: 77%"></div>
                  </div>`,
      title: 'CLINCH Fight Wear - High - Progress 77%',
      start: new Date(2025, 2, 1),
      end: new Date(2025, 2, 13),
      className: 'timeline-item high-priority',
      progress: 77,
      priority: 'High',
      assignee: 'Tulio Portela',
    },
    {
      id: 8,
      group: 23,
      content: `<div class="item-content">
                    <span class="item-title">Post-launch Support</span>
                    <span class="item-date">10/04</span>
                    <span class="item-priority urgent">Urgent</span>
                  </div>`,
      title: 'Shop Visual Identity - Urgent',
      start: new Date(2025, 3, 1),
      end: new Date(2025, 3, 10),
      className: 'timeline-item urgent-priority',
      priority: 'Urgent',
      assignee: 'Nani Medeiros',
    },
  ])

  // Create timeline with options
  const options = {
    zoomable: allowZoom.value,
    moveable: true,
    selectable: true,
    editable: {
      add: {
        add: true,
        updateTime: true,
        updateGroup: true,
      }, // add new items by double tapping
      updateTime: true, // drag items horizontally  
      updateGroup: true, // drag items from one group to another
      remove: true, // delete an item by tapping the delete button top right
      overrideItems: true, // allow these options to override item.editable
    },
    onAdd: (item, callback) => {
      item.content = `<div class="item-content">
                    <span class="item-title">edit</span>
                    <span class="item-priority low">Low</span>
                    <div class="progress-bar" style="width: 0%"></div>
                  </div>`
      item.className = 'timeline-item low-priority'
      callback(item)
    },
    orientation: 'top',
    stack: true,
    margin: {
      item: {
        horizontal: 0,
      },
    },
    showCurrentTime: true,
    format: {
      minorLabels: {
        day: 'D',
      },
    },
    showMajorLabels: showMajorLabel.value,
    showMinorLabels: showMinorLabel.value,
    tooltip: {
      followMouse: true,
      overflowMethod: 'cap',
    },
    groupTemplate: (group) => {
      if (!group || group.content === undefined) {
        return ''
      }
      return `<div class="custom-group">${group.content}</div>`
    },
    template: (item) => {
      return item.content
    },
    groupOrder: 'id',
    horizontalScroll: true,
    verticalScroll: true,
    timeAxis: { scale: 'day', step: 1 },
    start: new Date(2025, 1, 15),
    end: new Date(2025, 3, 15),
  }

  // Function to scroll to items in a group
  const scrollToGroupItems = (groupId) => {
    const groupItems = items.get({
      filter: item => item.group === groupId,
    })

    if (groupItems.length > 0) {
      // Sort items by start date
      groupItems.sort((a, b) => a.start.getTime() - b.start.getTime())

      // Get the earliest start date and latest end date
      const earliestStart = groupItems[0].start
      const latestEnd = groupItems.reduce(
        (latest, item) => (item.end.getTime() > latest.getTime() ? item.end : latest),
        groupItems[0].end,
      )

      // Add some padding
      const startDate = new Date(earliestStart)
      startDate.setDate(startDate.getDate() - 2)

      const endDate = new Date(latestEnd)
      endDate.setDate(endDate.getDate() + 2)

      // Set the window to show these items
      // eslint-disable-next-line ts/no-use-before-define
      newTimeline.setWindow(startDate, endDate)
    }
  }

  // Initialize timeline
  const newTimeline = new Timeline(timelineRef.value, items, groups, options)
  timeline.value = newTimeline

  // Add event listeners for group clicking and collapsing
  newTimeline.on('click', (properties) => {
    if (properties.what === 'group-label') {
      const clickedGroupId = properties.group
      const groupData = groups.get(clickedGroupId)

      // If the group has nested groups, toggle collapse/expand
      if (groupData && groupData.nestedGroups && groupData.nestedGroups.length > 0) {
        const isCollapsed = !groupData.className?.includes('collapsed')

        if (isCollapsed) {
          // Collapse: hide nested groups
          groupData.nestedGroups.forEach((nestedId) => {
            const nestedGroup = groups.get(nestedId)
            if (nestedGroup) {
              groups.update({ ...nestedGroup, visible: false })
            }
          })
          groups.update({ ...groupData, className: `${groupData.className} collapsed` })
        }
        else {
          // Expand: show nested groups
          groupData.nestedGroups.forEach((nestedId) => {
            const nestedGroup = groups.get(nestedId)
            if (nestedGroup) {
              groups.update({ ...nestedGroup, visible: true })
            }
          })
          groups.update({ ...groupData, className: groupData.className?.replace(' collapsed', '') })
        }
      }

      // Scroll to the first item in this group
      scrollToGroupItems(clickedGroupId)
    }
  })

  // Add current time marker
  const currentDate = new Date(2025, 2, 12)
  newTimeline.addCustomTime(currentDate, 'current')
  newTimeline.setCustomTimeTitle('Today', 'current')
}

// Timeline control functions
function handleZoomIn() {
  if (timeline.value) {
    const currentRange = timeline.value.getWindow()
    const start = new Date(currentRange.start.valueOf())
    const end = new Date(currentRange.end.valueOf())
    const interval = end.getTime() - start.getTime()
    const newInterval = interval * 0.7
    const center = new Date((start.getTime() + end.getTime()) / 2)
    const newStart = new Date(center.getTime() - newInterval / 2)
    const newEnd = new Date(center.getTime() + newInterval / 2)
    timeline.value.setWindow(newStart, newEnd)
  }
}

function handleZoomOut() {
  if (timeline.value) {
    const currentRange = timeline.value.getWindow()
    const start = new Date(currentRange.start.valueOf())
    const end = new Date(currentRange.end.valueOf())
    const interval = end.getTime() - start.getTime()
    const newInterval = interval * 1.3
    const center = new Date((start.getTime() + end.getTime()) / 2)
    const newStart = new Date(center.getTime() - newInterval / 2)
    const newEnd = new Date(center.getTime() + newInterval / 2)
    timeline.value.setWindow(newStart, newEnd)
  }
}

function handleMoveLeft() {
  if (timeline.value) {
    const window = timeline.value.getWindow()
    const interval = window.end.getTime() - window.start.getTime()
    const distance = interval * 0.2
    const newStart = new Date(window.start.getTime() - distance)
    const newEnd = new Date(window.end.getTime() - distance)
    timeline.value.setWindow(newStart, newEnd)
  }
}

function handleMoveRight() {
  if (timeline.value) {
    const window = timeline.value.getWindow()
    const interval = window.end.getTime() - window.start.getTime()
    const distance = interval * 0.2
    const newStart = new Date(window.start.getTime() + distance)
    const newEnd = new Date(window.end.getTime() + distance)
    timeline.value.setWindow(newStart, newEnd)
  }
}

function handleToday() {
  if (timeline.value) {
    const currentDate = new Date(2025, 2, 12)
    const start = new Date(currentDate)
    start.setDate(start.getDate() - 15)
    const end = new Date(currentDate)
    end.setDate(end.getDate() + 15)
    timeline.value.setWindow(start, end)
  }
}

function handleFitAll() {
  if (timeline.value) {
    timeline.value.fit()
  }
}
</script>

<template>
  <UiCard class="gantt-timeline-container">
    <UiCardContent class="p-0">
      <div class="timeline-controls flex items-center gap-2 border-b p-4">
        <UiButton variant="outline" size="icon" title="Zoom In" @click="handleZoomIn">
          <ZoomIn class="h-4 w-4" />
        </UiButton>
        <UiButton variant="outline" size="icon" title="Zoom Out" @click="handleZoomOut">
          <ZoomOut class="h-4 w-4" />
        </UiButton>
        <UiButton
          variant="outline"
          size="icon"
          title="Move Left"
          @click="handleMoveLeft"
        >
          <ChevronLeft class="h-4 w-4" />
        </UiButton>
        <UiButton variant="outline" class="text-xs" @click="handleToday">
          Today
        </UiButton>
        <UiButton
          variant="outline"
          size="icon"
          title="Move Right"
          @click="handleMoveRight"
        >
          <ChevronRight class="h-4 w-4" />
        </UiButton>
        <UiButton variant="outline" class="text-xs" @click="handleFitAll">
          Fit All
        </UiButton>
      </div>

      <div class="timeline-options flex flex-wrap gap-4 border-b p-4">
        <div class="flex items-center space-x-2">
          <UiCheckbox id="showMajorLabel" v-model:checked="showMajorLabel" />
          <label
            for="showMajorLabel"
            class="text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70"
          >
            Show Major Label
          </label>
        </div>
        <div class="flex items-center space-x-2">
          <UiCheckbox id="showMinorLabel" v-model:checked="showMinorLabel" />
          <label
            for="showMinorLabel"
            class="text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70"
          >
            Show Minor Label
          </label>
        </div>
        <div class="flex items-center space-x-2">
          <UiCheckbox id="showTooltip" v-model:checked="showTooltip" />
          <label
            for="showTooltip"
            class="text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70"
          >
            Show Tooltip
          </label>
        </div>
        <div class="flex items-center space-x-2">
          <UiCheckbox id="isDraggable" v-model:checked="isDraggable" />
          <label
            for="isDraggable"
            class="text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70"
          >
            Always Draggable
          </label>
        </div>
        <div class="flex items-center space-x-2">
          <UiCheckbox id="allowZoom" v-model:checked="allowZoom" />
          <label
            for="allowZoom"
            class="text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70"
          >
            Allow Zoom
          </label>
        </div>
        <div class="flex items-center space-x-2">
          <UiCheckbox id="showGroups" v-model:checked="showGroups" />
          <label
            for="showGroups"
            class="text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70"
          >
            Show Groups
          </label>
        </div>
      </div>

      <div ref="timelineRef" class="timeline-container" />
    </UiCardContent>
  </UiCard>
</template>

  <style>
  /* Main timeline container styles */
.gantt-timeline-container {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
  --low-color: #4caf50;
  --high-color: #ff9800;
  --urgent-color: #f44336;
  --progress-bg: #e0e0e0;
  --group-header-bg: #f5f5f5;
  --timeline-border: #e0e0e0;
  --alternate-bg: rgba(255, 235, 235, 0.3);
  --current-time: #e91e63;
}

/* Timeline container */
.timeline-container {
  height: 600px;
  overflow: hidden;
  border: 1px solid var(--timeline-border);
  border-top: none;
}

/* Custom styling for vis-timeline elements */
.vis-timeline {
  border: none;
  font-family: inherit;
}

/* Timeline background with alternating columns */
.vis-time-axis .vis-grid.vis-odd {
  background-color: var(--alternate-bg);
}

/* Timeline item styling */
.timeline-item {
  border-radius: 4px;
  border: none !important;
  box-shadow:
    0 1px 3px rgba(0, 0, 0, 0.12),
    0 1px 2px rgba(0, 0, 0, 0.24);
  background-color: primary !important;
  color: #333;
  padding: 0 !important;
  overflow: visible !important;
}

/* Priority colors */
.low-priority {
  border-left: 4px solid var(--low-color) !important;
}

.high-priority {
  border-left: 4px solid var(--high-color) !important;
}

.urgent-priority {
  border-left: 4px solid var(--urgent-color) !important;
}

/* Item content styling */
.item-content {
  padding: 6px 8px;
  display: flex;
  flex-direction: column;
  gap: 2px;
  height: 100%;
  position: relative;
}

.item-title {
  font-weight: 500;
  font-size: 13px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.item-date {
  font-size: 11px;
  color: #666;
}

.item-priority {
  font-size: 11px;
  font-weight: 500;
  padding: 1px 4px;
  border-radius: 3px;
  display: inline-block;
}

.item-priority.low {
  color: var(--low-color);
  background-color: rgba(76, 175, 80, 0.1);
}

.item-priority.high {
  color: var(--high-color);
  background-color: rgba(255, 152, 0, 0.1);
}

.item-priority.urgent {
  color: var(--urgent-color);
  background-color: rgba(244, 67, 54, 0.1);
}

/* Progress bar */
.progress-bar {
  position: absolute;
  bottom: 0;
  left: 0;
  height: 3px;
  background-color: var(--low-color);
}

.high-priority .progress-bar {
  background-color: var(--high-color);
}

.urgent-priority .progress-bar {
  background-color: var(--urgent-color);
}

/* Group styling */
.vis-label.vis-group-level-0 {
  background-color: var(--group-header-bg);
  font-weight: 600;
  border-bottom: 1px solid var(--timeline-border);
  border-top: 1px solid var(--timeline-border);
}

.vis-labelset .vis-label {
  border: none;
}

.vis-inner {
  border: none !important;
}

.vis-label.vis-group-level-0 .custom-group {
  padding: 8px;
  font-size: 14px;
}

.vis-label.vis-nesting-group {
  cursor: pointer;
}

.vis-label .custom-group {
  padding: 8px;
  font-size: 13px;
}

/* Time axis styling */
.vis-time-axis .vis-text {
  color: #666;
  padding: 5px 0;
}

.vis-time-axis .vis-grid.vis-minor {
  border-color: #e0e0e0;
}

.vis-time-axis .vis-grid.vis-major {
  border-color: #bdbdbd;
}

.vis-time-axis .vis-text.vis-major {
  font-weight: 600;
}

/* Current time marker */
.vis-current-time {
  background-color: var(--current-time);
  width: 2px;
}

.vis-custom-time.current {
  background-color: var(--current-time);
  width: 2px;
}

/* Panel styling */
.vis-panel.vis-center,
.vis-panel.vis-left,
.vis-panel.vis-right,
.vis-panel.vis-top,
.vis-panel.vis-bottom {
  border-color: var(--timeline-border);
}

/* Tooltip styling */
.vis-tooltip {
  border-radius: 4px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  padding: 8px 12px;
  font-size: 13px;
  border: 1px solid #e0e0e0;
  background-color: white;
  color: #333;
}

/* Styling for collapsible groups */
.vis-label.vis-group-level-0.collapsed {
  background-color: #e6e6e6;
}

.vis-label.vis-group-level-0.collapsed .custom-group::before {
  content: '▶';
  display: inline-block;
  margin-right: 5px;
  font-size: 10px;
}

.vis-label.vis-group-level-0 .custom-group::before {
  content: '▼';
  display: inline-block;
  margin-right: 5px;
  font-size: 10px;
}

/* Hover effect for group labels */
.vis-label.vis-group-level-0 {
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.vis-label.vis-group-level-0:hover {
  background-color: #eaeaea;
}

/* Animation for expanding/collapsing */
.vis-label,
.vis-group {
  transition: opacity 0.3s ease;
}
</style>
