<script setup lang="ts">
// The locked target and the Go2 behind it, walking steadily across the scene and wrapping around
// (like everyone else) so nobody ever reverses. Gait speed is tied to ground speed → no skating.
import { onBeforeUnmount, ref, watch } from 'vue'
import { useIsSlideActive } from '@slidev/client'
import PersonFig from './PersonFig.vue'
import GoDog from './GoDog.vue'

const W = 980
const V = 44 // ground speed, scene px/s
const GAP = 128 // dog stays this far behind the target
const DOG_S = 1.7
const P_S = 1.25
const START = 500
const LC = W + 140 + GAP // length of one lap (target x runs -70 … LC-70)
const HALF = (2 * 30 * Math.sin((26 * Math.PI) / 180)) / (V / P_S) // seconds per person leg sweep

const u = ref(START) // target x
const active = useIsSlideActive()
const reduce = typeof window !== 'undefined' && window.matchMedia?.('(prefers-reduced-motion: reduce)').matches

let raf = 0
let last = 0
function tick(now: number) {
  const dt = Math.min(0.05, (now - last) / 1000)
  last = now
  u.value = ((u.value + 70 + V * dt) % LC) - 70
  raf = requestAnimationFrame(tick)
}
watch(
  active,
  (a) => {
    cancelAnimationFrame(raf)
    u.value = START
    if (a && !reduce && typeof requestAnimationFrame !== 'undefined') {
      last = performance.now()
      raf = requestAnimationFrame(tick)
    }
  },
  { immediate: true },
)
onBeforeUnmount(() => cancelAnimationFrame(raf))
</script>

<template>
  <!-- Go2 following the target -->
  <g :transform="`translate(${u - GAP} 314)`">
    <g :transform="`scale(${DOG_S})`"><GoDog :walking="!reduce" :speed="V / DOG_S" /></g>
  </g>

  <!-- locked target -->
  <g :transform="`translate(${u} 304)`">
    <ellipse class="cs-ring" cx="0" cy="3" rx="26" ry="7" />
    <g :transform="`scale(${P_S})`"><PersonFig color="#14935f" :walking="!reduce" :half="HALF" /></g>
    <g class="cs-lock">
      <rect class="cs-tbox" x="-23" y="-94" width="46" height="102" rx="3" />
      <path class="cs-brk" d="M-23,-82 V-94 H-11 M11,-94 H23 V-82 M-23,-4 V8 H-11 M11,8 H23 V-4" />
      <rect class="cs-tlabel" x="-23" y="-114" width="86" height="16" rx="8" />
      <text class="cs-ttext" x="-14" y="-103">Target 0.94</text>
    </g>
  </g>
</template>
