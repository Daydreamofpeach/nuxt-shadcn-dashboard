<script setup lang="ts">
import type { HTMLAttributes } from 'vue'
import { cn } from '@/lib/utils'

const props = defineProps<{
  class?: HTMLAttributes['class']
}>()
</script>

<template>
  <div :class="cn('flex flex-col text-black dark:text-white gap-y-1.5 p-6 bg-[#fff] dark:bg-[#141c26]', props.class)">
    <slot />
  </div>
</template>
