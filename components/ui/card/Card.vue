<script setup lang="ts">
import type { HTMLAttributes } from 'vue'
import { cn } from '@/lib/utils'

const props = defineProps<{
  class?: HTMLAttributes['class']
}>()
</script>

<template>
  <div
    :class="
      cn(
        'rounded-xl border bg-[#fff] dark:bg-[#141c26] text-card-foreground shadow',
        props.class,
      )
    "
  >
    <slot />
  </div>
</template>
