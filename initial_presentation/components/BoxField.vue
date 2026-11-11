<script setup lang="ts">
// Tiny dataset "thumbnails" drawn with SVG.
//  crowd → many overlapping, partly occluded person boxes (CrowdHuman)
//  dance → identical dancers whose paths cross, so IDs are easy to swap (DanceTrack)
defineProps<{ kind: 'crowd' | 'dance' }>()

const boxes: [number, number, number, number, boolean][] = [
  [8, 14, 22, 52, false], [24, 20, 22, 48, true], [42, 10, 24, 56, false], [58, 18, 20, 50, true],
  [72, 12, 24, 54, false], [92, 20, 22, 46, false], [106, 8, 24, 58, true], [124, 16, 22, 50, false],
  [140, 12, 24, 54, false], [156, 22, 22, 44, true], [172, 10, 24, 56, false], [34, 30, 20, 38, true],
  [84, 28, 20, 40, false], [118, 30, 20, 38, true], [148, 28, 20, 40, false],
]
const paths = [
  'M10,18 C60,18 90,58 188,58',
  'M10,58 C60,58 90,18 188,18',
  'M10,38 C70,38 110,38 188,38',
  'M10,70 C50,50 120,70 188,66',
]
const dots: [number, number][] = [[188, 58], [188, 18], [188, 38], [188, 66]]
</script>

<template>
  <svg class="bf" viewBox="0 0 196 80" aria-hidden="true">
    <template v-if="kind === 'crowd'">
      <rect
        v-for="(b, i) in boxes"
        :key="i"
        class="bf-box"
        :class="{ occ: b[4] }"
        :style="{ '--i': i }"
        :x="b[0]" :y="b[1]" :width="b[2]" :height="b[3]" rx="3"
      />
    </template>
    <template v-else>
      <path v-for="(p, i) in paths" :key="i" class="bf-path" :style="{ '--i': i }" :d="p" pathLength="1" />
      <circle v-for="(d, i) in dots" :key="i" class="bf-dot" :style="{ '--i': i }" :cx="d[0]" :cy="d[1]" r="3.5" />
    </template>
  </svg>
</template>

<style>
.bf { width: 100%; height: 84px; display: block; }
.bf-box { fill: rgba(11, 143, 168, 0.07); stroke: #0b8fa8; stroke-width: 1.1; }
.bf-box.occ { stroke-dasharray: 3 2; stroke: #8f99a8; fill: rgba(143, 153, 168, 0.07); }
.stage.on .bf-box { animation: bf-pop 0.5s ease both; animation-delay: calc(var(--i) * 60ms + 500ms); }
@keyframes bf-pop { from { opacity: 0; transform: scale(0.8); } to { opacity: 1; transform: none; } }
.bf-box { transform-box: fill-box; transform-origin: center bottom; }

.bf-path { fill: none; stroke: #0b8fa8; stroke-width: 1.8; stroke-linecap: round; stroke-dasharray: 1; stroke-dashoffset: 0; opacity: 0.8; }
.bf-dot { fill: #0b8fa8; }
.stage.on .bf-path { stroke-dashoffset: 1; animation: bf-draw 1.4s ease forwards; animation-delay: calc(var(--i) * 140ms + 600ms); }
.stage.on .bf-dot { opacity: 0; animation: fadein 0.3s ease forwards; animation-delay: calc(var(--i) * 140ms + 1700ms); }
@keyframes bf-draw { to { stroke-dashoffset: 0; } }
</style>
