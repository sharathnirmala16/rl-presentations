<script setup lang="ts">
import { useIsSlideActive } from '@slidev/client'

const active = useIsSlideActive()
</script>

<template>
  <div class="slidev-layout hero stage" :class="{ on: active }">
    <slot />
  </div>
</template>

<style>
.slidev-layout.hero {
  position: relative;
  height: 100%;
  padding: 0;
  overflow: hidden;
}
</style>
