<script setup lang="ts">
// Stylised pedestrian. Origin = feet, ~70 units tall. Renders a <g> (use inside <svg>).
withDefaults(
  // half: seconds per leg sweep. Pick it from the ground speed so the feet don't skate:
  // half = 2 * legLength * sin(26deg) / speed  (all in the figure's own units)
  defineProps<{ color?: string; walking?: boolean; flip?: boolean; half?: number }>(),
  { color: '#7d8da6', walking: false, flip: false, half: 0.9 },
)
</script>

<template>
  <g class="pf" :class="{ walking }" :style="{ '--c': color, '--half': half + 's' }">
    <g :transform="flip ? 'scale(-1 1)' : undefined">
      <line class="pf-leg pf-a" x1="0" y1="-30" x2="0" y2="0" />
      <line class="pf-leg pf-b" x1="0" y1="-30" x2="0" y2="0" />
      <rect class="pf-torso" x="-8" y="-53" width="16" height="27" rx="8" />
      <circle class="pf-head" cx="0" cy="-61" r="7" />
    </g>
  </g>
</template>

<style>
.pf-leg { stroke: color-mix(in srgb, var(--c) 65%, #1b2330); stroke-width: 5; stroke-linecap: round; transform-origin: 0 -30px; }
.pf-torso { fill: var(--c); }
.pf-head { fill: #f0cfb4; }
.pf.walking .pf-a { animation: pf-swing var(--half, 0.9s) linear infinite alternate; }
.pf.walking .pf-b { animation: pf-swing var(--half, 0.9s) linear infinite alternate-reverse; }
@keyframes pf-swing { from { transform: rotate(-26deg); } to { transform: rotate(26deg); } }
@media (prefers-reduced-motion: reduce) { .pf.walking .pf-leg { animation: none; } }
</style>
