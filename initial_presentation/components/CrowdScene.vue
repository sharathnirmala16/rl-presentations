<script setup lang="ts">
import PersonFig from './PersonFig.vue'
import ChaseDuo from './ChaseDuo.vue'

// [baseline y, scale, direction (1 = →, -1 = ←), seconds to cross, start offset (s), torso colour, detector score]
const palette = ['#8aa0bf', '#a597d0', '#b6bfcc', '#d98fa0', '#7fb3b0', '#9a8fcf', '#d1ab7c', '#89a3c4']
type W = [number, number, 1 | -1, number, number, number, string]
const walkers: W[] = [
  [206, 0.62, 1, 95, 12, 0, '0.78'],
  [208, 0.62, -1, 110, 60, 3, '0.74'],
  [204, 0.6, 1, 120, 85, 5, '0.71'],
  [210, 0.64, -1, 88, 30, 1, '0.81'],
  [205, 0.6, 1, 100, 50, 7, '0.69'],
  [244, 0.82, -1, 74, 20, 2, '0.88'],
  [246, 0.82, 1, 82, 55, 4, '0.9'],
  [242, 0.8, -1, 90, 5, 6, '0.86'],
  [248, 0.84, 1, 70, 40, 7, '0.92'],
  [250, 0.84, -1, 78, 62, 0, '0.89'],
  [290, 1.05, 1, 62, 18, 5, '0.95'],
  [292, 1.05, -1, 58, 44, 1, '0.93'],
  [288, 1.02, 1, 66, 8, 3, '0.91'],
]

const W = 980
const span = (d: 1 | -1) => (d === 1 ? ['-70px', `${W + 70}px`] : [`${W + 70}px`, '-70px'])

// floor perspective lines
const vp = { x: 490, y: 150 }
const rays = Array.from({ length: 17 }, (_, i) => -900 + i * 220)
const rows = [168, 186, 208, 236, 272, 318]
</script>

<template>
  <svg class="crowd" viewBox="0 0 980 340" preserveAspectRatio="xMidYMax slice" aria-hidden="true">
    <defs>
      <linearGradient id="cs-fade" x1="0" y1="0" x2="0" y2="1">
        <stop offset="0" stop-color="#f6f3ec" stop-opacity="1" />
        <stop offset="0.55" stop-color="#f6f3ec" stop-opacity="0.4" />
        <stop offset="1" stop-color="#f6f3ec" stop-opacity="0" />
      </linearGradient>
      <linearGradient id="cs-floorfill" x1="0" y1="0" x2="0" y2="1">
        <stop offset="0" stop-color="#ece6d8" stop-opacity="0" />
        <stop offset="1" stop-color="#e6dfcf" stop-opacity="0.9" />
      </linearGradient>
      <linearGradient id="cs-scan" x1="0" y1="0" x2="1" y2="0">
        <stop offset="0" stop-color="#0b8fa8" stop-opacity="0" />
        <stop offset="0.75" stop-color="#0b8fa8" stop-opacity="0.06" />
        <stop offset="1" stop-color="#0b8fa8" stop-opacity="0" />
      </linearGradient>
    </defs>

    <!-- floor -->
    <rect x="0" y="150" width="980" height="190" fill="url(#cs-floorfill)" />
    <g class="cs-floor">
      <line v-for="x in rays" :key="x" :x1="vp.x" :y1="vp.y" :x2="vp.x + x" y2="345" />
      <line v-for="y in rows" :key="y" x1="0" :y1="y" x2="980" :y2="y" />
    </g>
    <rect x="0" y="130" width="980" height="130" fill="url(#cs-fade)" />

    <!-- crowd -->
    <g v-for="(w, i) in walkers" :key="i" :transform="`translate(0 ${w[0]})`">
      <g
        class="cs-mover"
        :style="{
          '--x0': span(w[2])[0],
          '--x1': span(w[2])[1],
          animationDuration: `${w[3]}s`,
          animationDelay: `-${w[4]}s`,
        }"
      >
        <g :transform="`scale(${w[1]})`">
          <ellipse class="cs-sh" cx="0" cy="2" rx="12" ry="3" />
          <PersonFig :color="palette[w[5]]" :flip="w[2] === -1" walking :half="(26.3 * w[1] * w[3]) / (W + 140)" />
          <g class="cs-box">
            <rect x="-17" y="-76" width="34" height="80" vector-effect="non-scaling-stroke" />
            <text x="-17" y="-80">{{ w[6] }}</text>
          </g>
        </g>
      </g>
    </g>

    <!-- sweep -->
    <rect class="cs-scan" x="-200" y="150" width="200" height="190" fill="url(#cs-scan)" />

    <!-- Go2 + locked target -->
    <ChaseDuo />
  </svg>
</template>

<style>
.crowd { display: block; width: 100%; height: 100%; }
.cs-floor line { stroke: #ddd5c3; stroke-width: 1; opacity: 0.85; }
.cs-sh { fill: rgba(27, 35, 48, 0.08); }
.cs-mover { animation: cs-cross linear infinite; }
@keyframes cs-cross { from { transform: translateX(var(--x0)); } to { transform: translateX(var(--x1)); } }

.cs-box rect { fill: rgba(11, 143, 168, 0.05); stroke: #0b8fa8; stroke-width: 1; opacity: 0.45; }
.cs-box text { font: 600 8px var(--ff-code); fill: #0b8fa8; opacity: 0.7; }

.cs-scan { animation: cs-sweep 8s linear infinite; mask-image: linear-gradient(to bottom, transparent, #000 45%); }
@keyframes cs-sweep { from { transform: translateX(0); } to { transform: translateX(1200px); } }

.cs-ring { fill: none; stroke: #e9750d; stroke-width: 1.5; transform-box: fill-box; transform-origin: center; animation: cs-pulse 2s ease-out infinite; }
@keyframes cs-pulse { from { transform: scale(0.7); opacity: 0.9; } to { transform: scale(1.9); opacity: 0; } }

.cs-tbox { fill: rgba(233, 117, 13, 0.07); stroke: #e9750d; stroke-width: 1.2; }
.cs-brk { fill: none; stroke: #e9750d; stroke-width: 2.5; stroke-linecap: round; stroke-linejoin: round; }
.cs-tlabel { fill: #e9750d; }
.cs-ttext { font: 700 9px var(--ff-sans); fill: #fff; letter-spacing: 0.02em; }
/* scale around the person's own centre (local coords), not the box + label's bounding box */
.cs-lock { transform-box: view-box; transform-origin: 0px -43px; }
.stage.on .cs-lock { animation: cs-acquire 0.9s cubic-bezier(0.2, 0.8, 0.2, 1) 1s both; }
@keyframes cs-acquire { 0% { opacity: 0; transform: scale(1.7); } 60% { opacity: 1; } 100% { opacity: 1; transform: scale(1); } }

@media (prefers-reduced-motion: reduce) {
  .cs-mover, .cs-scan, .cs-ring { animation: none; }
}
</style>
