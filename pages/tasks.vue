<!-- <script setup lang="ts">
import { columns } from '@/components/tasks/components/columns'
import DataTable from '@/components/tasks/components/DataTable.vue'
import tasks from '@/components/tasks/data/tasks.json'
</script>

<template>
  <div class="w-full flex flex-col items-stretch gap-4">
    <div class="flex flex-wrap items-end justify-between gap-2">
      <div>
        <h2 class="text-2xl font-bold tracking-tight">
          Tasks
        </h2>
        <p class="text-muted-foreground">
          Here&apos;s a list of your tasks for this month!
        </p>
      </div>
    </div>
    <DataTable :data="tasks.data" :columns="columns" />
  </div>
</template>

<style scoped>

</style> -->

<script setup>
import { Button } from '@/components/ui/button'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Checkbox } from '@/components/ui/checkbox'
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogTrigger } from '@/components/ui/dialog'
import { DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuTrigger } from '@/components/ui/dropdown-menu'
import { Input } from '@/components/ui/input'
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs'
import {
  PlusIcon,
  Trash2Icon,
} from 'lucide-vue-next'
import { computed, onMounted, ref, watch } from 'vue'

// State
const groups = ref([])
const newGroupName = ref('')
const newListName = ref('')
const newTaskText = ref('')
const activeGroup = ref(null)
const activeList = ref(null)

// Computed properties
const currentGroup = computed(() => {
  return groups.value.find(group => group.id === activeGroup.value) || null
})

// Load data from localStorage on initial render
onMounted(() => {
  const savedGroups = localStorage.getItem('todoGroups')
  if (savedGroups) {
    const parsedGroups = JSON.parse(savedGroups)
    groups.value = parsedGroups

    // Set active group and list if available
    if (parsedGroups.length > 0) {
      activeGroup.value = parsedGroups[0].id
      if (parsedGroups[0].lists.length > 0) {
        activeList.value = parsedGroups[0].lists[0].id
      }
    }
  }
})

// Save to localStorage whenever groups change
watch(groups, (newGroups) => {
  localStorage.setItem('todoGroups', JSON.stringify(newGroups))
}, { deep: true })

// Handlers
function handleAddGroup() {
  if (newGroupName.value.trim() === '')
    return

  const newGroup = {
    id: crypto.randomUUID(),
    name: newGroupName.value,
    lists: [],
  }

  groups.value.push(newGroup)
  newGroupName.value = ''
  activeGroup.value = newGroup.id
}

function handleAddList() {
  if (newListName.value.trim() === '' || !activeGroup.value)
    return

  const newList = {
    id: crypto.randomUUID(),
    name: newListName.value,
    tasks: [],
  }

  const groupIndex = groups.value.findIndex(group => group.id === activeGroup.value)
  if (groupIndex !== -1) {
    groups.value[groupIndex].lists.push(newList)
    newListName.value = ''
    activeList.value = newList.id
  }
}

function handleAddTask(listId) {
  if (newTaskText.value.trim() === '' || !activeGroup.value)
    return

  const newTask = {
    id: crypto.randomUUID(),
    text: newTaskText.value,
    completed: false,
  }

  const groupIndex = groups.value.findIndex(group => group.id === activeGroup.value)
  if (groupIndex !== -1) {
    const listIndex = groups.value[groupIndex].lists.findIndex(list => list.id === listId)
    if (listIndex !== -1) {
      groups.value[groupIndex].lists[listIndex].tasks.push(newTask)
      newTaskText.value = ''
    }
  }
}

function handleToggleTask(listId, taskId) {
  const groupIndex = groups.value.findIndex(group => group.id === activeGroup.value)
  if (groupIndex !== -1) {
    const listIndex = groups.value[groupIndex].lists.findIndex(list => list.id === listId)
    if (listIndex !== -1) {
      const taskIndex = groups.value[groupIndex].lists[listIndex].tasks.findIndex(task => task.id === taskId)
      if (taskIndex !== -1) {
        groups.value[groupIndex].lists[listIndex].tasks[taskIndex].completed
          = !groups.value[groupIndex].lists[listIndex].tasks[taskIndex].completed
      }
    }
  }
}

function handleDeleteTask(listId, taskId) {
  const groupIndex = groups.value.findIndex(group => group.id === activeGroup.value)
  if (groupIndex !== -1) {
    const listIndex = groups.value[groupIndex].lists.findIndex(list => list.id === listId)
    if (listIndex !== -1) {
      groups.value[groupIndex].lists[listIndex].tasks
        = groups.value[groupIndex].lists[listIndex].tasks.filter(task => task.id !== taskId)
    }
  }
}

function handleDeleteList(listId) {
  const groupIndex = groups.value.findIndex(group => group.id === activeGroup.value)
  if (groupIndex !== -1) {
    groups.value[groupIndex].lists = groups.value[groupIndex].lists.filter(list => list.id !== listId)

    // Update active list if needed
    if (groups.value[groupIndex].lists.length > 0) {
      activeList.value = groups.value[groupIndex].lists[0].id
    }
    else {
      activeList.value = null
    }
  }
}

function handleDeleteGroup(groupId) {
  groups.value = groups.value.filter(group => group.id !== groupId)

  // Update active group if needed
  if (groups.value.length > 0) {
    activeGroup.value = groups.value[0].id
    if (groups.value[0].lists.length > 0) {
      activeList.value = groups.value[0].lists[0].id
    }
    else {
      activeList.value = null
    }
  }
  else {
    activeGroup.value = null
    activeList.value = null
  }
}

function selectGroup(groupId) {
  activeGroup.value = groupId
  const selectedGroup = groups.value.find(group => group.id === groupId)
  if (selectedGroup && selectedGroup.lists.length > 0) {
    activeList.value = selectedGroup.lists[0].id
  }
  else {
    activeList.value = null
  }
}

function setActiveList(listId) {
  activeList.value = listId
}
</script>

<template>
  <div class="mx-auto max-w-6xl min-h-screen bg-white p-4 text-muted-foreground container dark:bg-[#141c26]">
    <h1 class="text-custom-cream mb-6 text-3xl font-bold">
      Todo List Groups
    </h1>

    <div class="grid grid-cols-1 gap-6 md:grid-cols-12">
      <!-- Sidebar with groups -->
      <div class="md:col-span-3 space-y-4">
        <div class="mb-4 flex items-center gap-2">
          <Input
            v-model="newGroupName"
            placeholder="New group name"
            class="border-custom-slate/30 focus:ring-custom-green placeholder-custom-cream/50 bg-white text-muted-foreground dark:bg-[#141c26]"
            @keydown.enter="handleAddGroup"
          />
          <Button
            size="icon"
            class="text-white bg-primary/50 hover:bg-primary/80"
            @click="handleAddGroup"
          >
            <PlusIcon class="h-4 w-4" />
          </Button>
        </div>

        <div class="space-y-3">
          <div
            v-for="group in groups"
            :key="group.id"
            class="flex cursor-pointer items-center justify-between rounded-xl p-3 transition-all duration-200"
            :class="[
              group.id === activeGroup
                ? 'bg-primary/50  text-white shadow-md'
                : 'bg-primary/20  hover:bg-primary/60 text-white',
            ]"
            @click="selectGroup(group.id)"
          >
            <span class="truncate font-medium">{{ group.name }}</span>
            <Button
              variant="solid"
              size="icon"
              :class="[group.id === activeGroup ? 'hover:bg-primary/50' : 'hover:bg-primary/80']"
              @click.stop="handleDeleteGroup(group.id)"
            >
              <Trash2Icon class="h-4 w-4" />
            </Button>
          </div>
        </div>
      </div>

      <!-- Main content -->
      <div class="md:col-span-9">
        <div v-if="currentGroup" class="space-y-6">
          <div class="flex items-center justify-between">
            <h2 class="text-2xl text-primary font-semibold">
              {{ currentGroup.name }}
            </h2>

            <Dialog>
              <DialogTrigger>
                <Button class="bg-primary/50 text-white hover:bg-primary/80">
                  <PlusIcon class="mr-2 h-4 w-4" />
                  Add List
                </Button>
              </DialogTrigger>
              <DialogContent class="border-custom-slate/30 bg-white dark:bg-[#141c26]">
                <DialogHeader>
                  <DialogTitle class="text-muted-forground">
                    Create a new list
                  </DialogTitle>
                </DialogHeader>
                <div class="mt-4 flex items-center gap-2">
                  <Input
                    v-model="newListName"
                    placeholder="List name"
                    class="bg-custom-slate/20 border-custom-slate/30 text-muted-forground placeholder-muted-froground/50 focus:ring-primary"
                    @keydown.enter="handleAddList"
                  />
                  <Button
                    class="bg-primary/50 text-white hover:bg-primary/80"
                    @click="handleAddList"
                  >
                    Create
                  </Button>
                </div>
              </DialogContent>
            </Dialog>
          </div>

          <div v-if="currentGroup.lists.length > 0">
            <Tabs :default-value="activeList" class="w-full" @value-change="setActiveList">
              <TabsList class="w-full justify-start overflow-x-auto bg-primary/50">
                <TabsTrigger
                  v-for="list in currentGroup.lists"
                  :key="list.id"
                  :value="list.id"
                  class="text-white/80 data-[state=active]:bg-primary/50 data-[state=active]:text-white"
                >
                  {{ list.name }}
                </TabsTrigger>
              </TabsList>

              <div class="grid grid-cols-1 mt-6 gap-6 lg:grid-cols-3 sm:grid-cols-2">
                <TabsContent
                  v-for="list in currentGroup.lists"
                  :key="list.id"
                  :value="list.id"
                  class="mt-0"
                >
                  <Card class="cube-card border-custom-slate/30 overflow-hidden bg-white dark:bg-[#141c26]">
                    <CardHeader class="bg-custom-slate/50 flex flex-row items-center justify-between">
                      <CardTitle class="text-muted-forground">
                        {{ list.name }}
                      </CardTitle>
                      <DropdownMenu>
                        <DropdownMenuTrigger>
                          <Button variant="ghost" size="icon" class="hover:bg-custom-slate/80">
                            <Trash2Icon class="h-4 w-4" />
                          </Button>
                        </DropdownMenuTrigger>
                        <DropdownMenuContent align="end" class="bg-custom-dark border-custom-slate/30">
                          <DropdownMenuItem
                            class="focus:bg-custom-slate/20 text-destructive focus:text-destructive"
                            @click="handleDeleteList(list.id)"
                          >
                            Delete List
                          </DropdownMenuItem>
                        </DropdownMenuContent>
                      </DropdownMenu>
                    </CardHeader>
                    <CardContent class="p-4">
                      <div class="space-y-4">
                        <div class="flex items-center gap-2">
                          <Input
                            v-model="newTaskText"
                            placeholder="Add a new task"
                            class="bg-custom-slate/20 border-custom-slate/30 text-muted-forground placeholder-muted-forground/80 focus:ring-primary"
                            @keydown.enter="handleAddTask(list.id)"
                          />
                          <Button
                            size="icon"
                            class="text-muted-forground bg-primary/50 hover:bg-primary/80"
                            @click="() => handleAddTask(list.id)"
                          >
                            <PlusIcon class="h-4 w-4" />
                          </Button>
                        </div>

                        <div class="max-h-[300px] overflow-y-auto pr-1 space-y-3">
                          <p
                            v-if="list.tasks.length === 0"
                            class="py-4 text-center text-white/50"
                          >
                            No tasks yet. Add one above!
                          </p>
                          <div
                            v-for="task in list.tasks"
                            :key="task.id"
                            class="cube-task flex items-center justify-between border rounded-lg p-3"
                          >
                            <div class="flex items-center gap-3">
                              <Checkbox
                                :id="task.id"
                                :checked="task.completed"
                                class="data-[state=checked]:text-muted-forground border-primary data-[state=checked]:bg-primary"
                                @update:checked="() => handleToggleTask(list.id, task.id)"
                              />
                              <label
                                :for="task.id"
                                :class="[task.completed ? 'line-through text-white/50' : 'text-white']"
                              >
                                {{ task.text }}
                              </label>
                            </div>
                            <Button
                              variant="ghost"
                              size="icon"
                              class="hover:bg-primary/50"
                              @click="() => handleDeleteTask(list.id, task.id)"
                            >
                              <Trash2Icon class="h-4 w-4 text-white/70" />
                            </Button>
                          </div>
                        </div>
                      </div>
                    </CardContent>
                  </Card>
                </TabsContent>
              </div>
            </Tabs>
          </div>
          <Card v-else class="cube-card border-custom-slate/30 bg-custom-slate/30 p-6 text-center">
            <p class="text-custom-cream/50 mb-4">
              No lists in this group yet.
            </p>
            <Dialog>
              <DialogTrigger>
                <Button class="bg-priamryhover:bg-primary/80 text-muted-forground">
                  <PlusIcon class="mr-2 h-4 w-4" />
                  Create your first list
                </Button>
              </DialogTrigger>
              <DialogContent class="bg-custom-dark border-custom-slate/30">
                <DialogHeader>
                  <DialogTitle class="text-custom-cream">
                    Create a new list
                  </DialogTitle>
                </DialogHeader>
                <div class="mt-4 flex items-center gap-2">
                  <Input
                    v-model="newListName"
                    placeholder="List name"
                    class="bg-custom-slate/20 border-custom-slate/30 focus:ring-custom-green text-custom-cream placeholder-custom-cream/50"
                    @keydown.enter="handleAddList"
                  />
                  <Button
                    class="bg-custom-green hover:bg-custom-green/90 text-custom-dark"
                    @click="handleAddList"
                  >
                    Create
                  </Button>
                </div>
              </DialogContent>
            </Dialog>
          </Card>
        </div>
        <Card v-else class="cube-card border-custom-slate/30 bg-custom-slate/30 p-6 text-center">
          <p class="text-custom-cream/50 mb-4">
            No groups yet. Create one to get started!
          </p>
          <div class="mx-auto max-w-xs flex items-center gap-2">
            <Input
              v-model="newGroupName"
              placeholder="New group name"
              class="bg-custom-slate/20 border-custom-slate/30 focus:ring-primary text-white placeholder-white/50"
              @keydown.enter="handleAddGroup"
            />
            <Button
              size="icon"
              class="bg-primary hover:bg-primary/80 text-zinc-900y
              "
              @click="handleAddGroup"
            >
              <PlusIcon class="h-4 w-4" />
            </Button>
          </div>
        </Card>
      </div>
    </div>
  </div>
</template>

<style>
:root {
  --custom-dark: #2e2c2f;
  --custom-slate: #475b63;
  --custom-green: #729b79;
  --custom-light-green: #bacdb0;
  --custom-cream: #f3e8ee;
}

/* Custom cube styling */
.cube-card {
  @apply rounded-2xl transition-all duration-300 transform hover:translate-y-[-5px] hover:shadow-lg;
  box-shadow:
    0 10px 15px -3px rgba(0, 0, 0, 0.1),
    0 4px 6px -2px rgba(0, 0, 0, 0.05);
  perspective: 1000px;
}

.cube-list {
  @apply bg-card rounded-xl shadow-md transition-all duration-300 hover:shadow-lg;
  transform-style: preserve-3d;
  transform: translateZ(0);
}

.cube-task {
  @apply border-custom-slate/30 bg-custom-slate/10 hover:bg-custom-slate/20 transition-all duration-200;
}

/* Dark mode specific styles */
.dark .cube-card {
  box-shadow:
    0 10px 15px -3px rgba(0, 0, 0, 0.2),
    0 4px 6px -2px rgba(0, 0, 0, 0.1);
}

.dark .cube-task {
  @apply border-custom-slate/30 bg-custom-slate/10 hover:bg-custom-slate/20;
}

/* Custom color utilities for UnoCSS */
.bg-custom-dark {
  background-color: var(--custom-dark);
}
.bg-custom-slate {
  background-color: var(--custom-slate);
}
.bg-custom-green {
  background-color: var(--custom-green);
}
.bg-custom-light-green {
  background-color: var(--custom-light-green);
}
.bg-custom-cream {
  background-color: var(--custom-cream);
}

.text-custom-dark {
  color: var(--custom-dark);
}
.text-custom-slate {
  color: var(--custom-slate);
}
.text-custom-green {
  color: var(--custom-green);
}
.text-custom-light-green {
  color: var(--custom-light-green);
}
.text-custom-cream {
  color: var(--custom-cream);
}

.border-custom-slate {
  border-color: var(--custom-slate);
}
.border-custom-green {
  border-color: var(--custom-green);
}

/* Opacity variants */
.bg-custom-slate\/10 {
  background-color: rgba(71, 91, 99, 0.1);
}
.bg-custom-slate\/20 {
  background-color: rgba(71, 91, 99, 0.2);
}
.bg-custom-slate\/30 {
  background-color: rgba(71, 91, 99, 0.3);
}
.bg-custom-slate\/40 {
  background-color: rgba(71, 91, 99, 0.4);
}
.bg-custom-slate\/50 {
  background-color: rgba(71, 91, 99, 0.5);
}
.bg-custom-slate\/80 {
  background-color: rgba(71, 91, 99, 0.8);
}

.bg-custom-green\/80 {
  background-color: rgba(114, 155, 121, 0.8);
}
.bg-custom-green\/90 {
  background-color: rgba(114, 155, 121, 0.9);
}

.text-custom-cream\/50 {
  color: rgba(243, 232, 238, 0.5);
}
.text-custom-cream\/70 {
  color: rgba(243, 232, 238, 0.7);
}

.border-custom-slate\/30 {
  border-color: rgba(71, 91, 99, 0.3);
}
</style>
