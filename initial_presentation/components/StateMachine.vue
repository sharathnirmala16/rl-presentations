<script setup lang="ts">
import { computed, onBeforeUnmount, ref, watch } from 'vue'
import PersonFig from './PersonFig.vue'
import GoDog from './GoDog.vue'

// step: -1 = nothing yet, 0..2 = states revealed (current state = step)
const props = defineProps<{ step: number }>()

const states = [
  { n: 1, title: 'Search & face anchor', sub: 'Stop, look up (+35°), confirm the face', icon: 'i-carbon-search', tone: 'cyan' },
  { n: 2, title: 'Track & pursue', sub: 'Follow the selected box', icon: 'i-carbon-location-person', tone: 'amber' },
  { n: 3, title: 'Arrival / re-verify', sub: 'Close or confidence drops: sit, look up, re-check the face', icon: 'i-carbon-face-activated', tone: 'violet' },
]
const triggers = ['Face confirmed', 'Distance < 1.5 m or confidence drops']
const CW = 260 // card width
const GAP = 50

const idx = computed(() => Math.max(0, Math.min(2, props.step)))

// State 1 plays out on its own: searching → looks up → face anchored
const phase = ref(2)
let pt: ReturnType<typeof setTimeout>[] = []
watch(
  () => props.step,
  (s) => {
    pt.forEach(clearTimeout)
    pt = []
    if (s === 0) {
      phase.value = 0
      pt.push(setTimeout(() => (phase.value = 1), 1200), setTimeout(() => (phase.value = 2), 3300))
    } else phase.value = 2
  },
  { immediate: true },
)

// animated track
const dogX = [90, 400, 570]
const dist = ['> 2 m', 'closing in…', '< 1.5 m']
const tag = computed(() => {
  if (idx.value === 0) return [{ t: 'Face not visible', c: 'var(--red)' }, { t: 'Verifying face…', c: 'var(--violet)' }, { t: 'Face anchored', c: 'var(--green)' }][phase.value]
  if (idx.value === 1) return { t: 'Following target', c: 'var(--amber)' }
  return { t: 'Re-verifying face…', c: 'var(--violet)' }
})
const locked = computed(() => idx.value > 0 || phase.value === 2)

// The dog walks at a fixed ground speed and its gait is driven by that same speed,
// so the feet stay planted instead of skating. Travel time = distance / speed.
// It only looks up / sits once it has stopped.
const V = 80 // px/s on the slide
const SCALE = 1.7 // dog drawing scale
const moving = ref(false)
const dur = ref(1)
let timer: ReturnType<typeof setTimeout> | undefined
watch(idx, (v, old) => {
  clearTimeout(timer)
  const dx = dogX[v] - dogX[old]
  if (dx > 0) {
    dur.value = dx / V
    moving.value = true
    timer = setTimeout(() => (moving.value = false), dur.value * 1000)
  } else {
    // stepping back through the states: reposition quietly, no gait
    dur.value = dx < 0 ? 0.9 : 0.6
    moving.value = false
  }
})
onBeforeUnmount(() => {
  clearTimeout(timer)
  pt.forEach(clearTimeout)
})
const pitchNow = computed(() => (!moving.value && idx.value === 0 && phase.value >= 1 ? 35 : 0))

const PX = 800
</script>

<template>
  <div class="sm-wrap">
    <!-- state cards -->
    <template v-for="(s, i) in states" :key="s.n">
      <div
        class="sm-card card brk"
        :class="[s.tone, { show: step >= i, cur: step >= 0 && idx === i, done: step > i }]"
        :style="{ left: i * (CW + GAP) + 'px' }"
      >
        <div class="sm-top">
          <span class="sm-n">State {{ s.n }}</span>
          <div :class="s.icon" class="sm-ico" />
        </div>
        <div class="sm-t">{{ s.title }}</div>
        <div class="sm-s">{{ s.sub }}</div>
      </div>
      <template v-if="i < 2">
        <svg class="sm-arrow" :class="{ show: step > i }" :style="{ left: i * (CW + GAP) + CW + 5 + 'px' }" viewBox="0 0 40 20">
          <path d="M2,10 H32" pathLength="1" /><path class="ah" d="M26,4 L35,10 L26,16" />
        </svg>
        <div class="sm-trig" :class="{ show: step > i }" :style="{ left: i * (CW + GAP) + 180 + 'px' }">
          <i />{{ triggers[i] }}
        </div>
      </template>
    </template>

    <!-- animated scene -->
    <svg class="sm-track" :class="{ show: step >= 0 }" :style="{ '--dur': dur + 's' }" viewBox="0 0 880 172">
      <line class="sm-ground" x1="0" y1="142" x2="880" y2="142" />
      <!-- distance -->
      <g class="sm-dist" :style="{ transform: `translateX(${dogX[idx] + 58}px)` }">
        <line :x1="0" y1="112" :x2="PX - 34 - dogX[idx] - 58" y2="112" />
        <text :x="(PX - 34 - dogX[idx] - 58) / 2" y="102" text-anchor="middle">{{ dist[idx] }}</text>
      </g>
      <!-- person -->
      <g :transform="`translate(${PX} 142)`">
        <g transform="scale(1.4)"><PersonFig color="#14935f" /></g>
        <g class="sm-lock" :class="{ on: locked }">
          <rect x="-22" y="-104" width="44" height="112" rx="3" />
          <path d="M-22,-92 V-104 H-10 M10,-104 H22 V-92 M-22,-4 V8 H-10 M10,8 H22 V-4" />
        </g>
        <g :style="{ transform: 'translateY(-118px)' }">
          <rect class="sm-tag" :style="{ fill: tag.c }" x="-88" y="-18" width="176" height="18" rx="9" />
          <text class="sm-tagt" x="0" y="-5.5" text-anchor="middle">{{ tag.t }}</text>
        </g>
      </g>
      <!-- dog -->
      <g class="sm-dog" :style="{ transform: `translate(${dogX[idx]}px, 142px)` }">
        <g :transform="`scale(${SCALE})`"><GoDog :walking="moving" :speed="V / SCALE" :sit="idx === 2 && !moving" :pitch="pitchNow" /></g>
      </g>
    </svg>
  </div>
</template>

<style>
.sm-wrap { position: relative; width: 880px; height: 372px; }
.sm-card { position: absolute; top: 0; width: 260px; height: 124px; padding: 0.7rem 0.9rem;
  opacity: 0; transform: translateY(10px); transition: opacity 0.5s ease, transform 0.5s cubic-bezier(0.2, 0.8, 0.2, 1), box-shadow 0.5s ease; }
.sm-card.cyan { --tone: var(--cyan); } .sm-card.violet { --tone: var(--violet); }
.sm-card.amber { --tone: var(--amber); }
.sm-card.show { opacity: 0.5; transform: none; }
.sm-card.done { opacity: 0.8; }
.sm-card.cur { opacity: 1; box-shadow: 0 0 0 2px color-mix(in srgb, var(--tone) 55%, white), var(--shadow); }
.sm-top { display: flex; align-items: center; justify-content: space-between; }
.sm-n { font-size: 0.68rem; font-weight: 600; color: var(--tone); }
.sm-ico { font-size: 1.1rem; color: var(--tone); }
.sm-t { font-size: 1.15rem; font-weight: 700; letter-spacing: -0.02em; margin: 0.3rem 0 0.15rem; }
.sm-s { font-size: 0.74rem; line-height: 1.3; color: var(--soft); }

.sm-arrow { position: absolute; top: 54px; width: 40px; height: 20px; overflow: visible; }
.sm-arrow path { fill: none; stroke: var(--mute); stroke-width: 1.8; stroke-linecap: round; stroke-linejoin: round; stroke-dasharray: 1; stroke-dashoffset: 1; transition: stroke-dashoffset 0.6s ease; }
.sm-arrow .ah { stroke-dasharray: none; stroke-dashoffset: 0; opacity: 0; transition: opacity 0.3s ease 0.45s; }
.sm-arrow.show path { stroke-dashoffset: 0; } .sm-arrow.show .ah { opacity: 1; }

.sm-trig { position: absolute; top: 142px; width: 230px; margin-left: -10px; padding: 0.28rem 0.5rem; font-size: 0.7rem; font-weight: 500; line-height: 1.3; text-align: center; color: var(--soft);
  border-radius: 999px; background: var(--panel); border: 1px solid var(--line);
  opacity: 0; transform: translateY(6px); transition: opacity 0.5s ease 0.35s, transform 0.5s ease 0.35s; }
.sm-trig.show { opacity: 1; transform: none; }
.sm-trig i { position: absolute; left: 50%; top: -15px; width: 1px; height: 14px; background: var(--line); }

.sm-track { position: absolute; left: 0; top: 196px; width: 880px; height: 172px; opacity: 0; transition: opacity 0.6s ease 0.2s; overflow: visible; }
.sm-track.show { opacity: 1; }
.sm-ground { stroke: #d8d0bd; stroke-width: 2; stroke-dasharray: 6 6; }
.sm-dog { transition: transform var(--dur, 1s) cubic-bezier(0.3, 0.2, 0.7, 0.8); }
.sm-dist { transition: transform var(--dur, 1s) cubic-bezier(0.3, 0.2, 0.7, 0.8); }
.sm-dist line { stroke: var(--amber); stroke-width: 1.6; stroke-dasharray: 4 4; transition: all var(--dur, 1s) cubic-bezier(0.3, 0.2, 0.7, 0.8); }
.sm-dist text { font: 700 12px var(--ff-sans); fill: var(--amber); }
.sm-lock { opacity: 0; transition: opacity 0.4s ease; }
.sm-lock.on { opacity: 1; }
.sm-lock rect { fill: rgba(233, 117, 13, 0.07); stroke: var(--amber); stroke-width: 1.2; }
.sm-lock path { fill: none; stroke: var(--amber); stroke-width: 2.5; stroke-linecap: round; stroke-linejoin: round; }
.sm-tag { transition: fill 0.4s ease; }
.sm-tagt { font: 700 10px var(--ff-sans); fill: #fff; }
</style>
