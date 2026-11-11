<script setup lang="ts">
import { useIsSlideActive } from '@slidev/client'

defineProps<{ kicker?: string }>()
const active = useIsSlideActive()
</script>

<template>
  <div class="slidev-layout hud stage" :class="{ on: active }">
    <div v-if="kicker" class="kicker"><span class="tick" />{{ kicker }}</div>
    <slot />
  </div>
</template>
