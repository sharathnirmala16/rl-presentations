<script setup lang="ts">
import { computed, onBeforeUnmount, ref, watch } from 'vue'
import { useIsSlideActive } from '@slidev/client'

const props = withDefaults(
  defineProps<{ to: number; dur?: number; delay?: number; decimals?: number; prefix?: string; suffix?: string; when?: boolean }>(),
  { dur: 1400, delay: 350, decimals: 0, prefix: '', suffix: '', when: true },
)

const active = useIsSlideActive()
const val = ref(props.to) // final value whenever the slide isn't playing (overview, export)
let raf = 0

watch(
  () => active.value && props.when,
  (isActive) => {
    cancelAnimationFrame(raf)
    if (!isActive) {
      val.value = props.to
      return
    }
    val.value = 0
    const start = performance.now() + props.delay
    const tick = (t: number) => {
      const p = Math.min(1, Math.max(0, (t - start) / props.dur))
      val.value = props.to * (1 - Math.pow(1 - p, 3))
      if (p < 1) raf = requestAnimationFrame(tick)
    }
    raf = requestAnimationFrame(tick)
  },
  { immediate: true },
)
onBeforeUnmount(() => cancelAnimationFrame(raf))

const text = computed(
  () =>
    props.prefix +
    val.value.toLocaleString('en-US', { minimumFractionDigits: props.decimals, maximumFractionDigits: props.decimals }) +
    props.suffix,
)
</script>

<template>
  <span class="cu">{{ text }}</span>
</template>

<style>
.cu { font-variant-numeric: tabular-nums; }
</style>
