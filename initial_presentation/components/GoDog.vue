<script setup lang="ts">
// Stylised Unitree Go2, side view, facing right.
// Origin = ground under the body centre. Legs are solved with 2-link IK in world
// coordinates; "looking up" pitches the WHOLE body (rear legs crouch, front legs extend).
import { computed, onBeforeUnmount, onMounted, shallowRef, watch } from 'vue'
import { useIsSlideActive } from '@slidev/client'

const props = withDefaults(
  // speed: ground speed in the dog's own units per second. When given, the gait is locked to it
  // (stance feet move back exactly as fast as the body moves forward, so nothing skates).
  defineProps<{ walking?: boolean; sit?: boolean; pitch?: number; speed?: number }>(),
  { walking: false, sit: false, pitch: 0, speed: undefined },
)

const L = 14.3 // thigh = calf
const HIP_SEP = 34
const STAND_H = 26
const RX = -17 // rear hip x (world)

type S = { th: number; h: number; amp: number; ph: number }

function target(): S {
  const th = props.sit ? 31 : props.pitch
  const h = props.sit ? 9.4 : STAND_H - 0.55 * th
  return { th, h, amp: props.walking ? 1 : 0, ph: 0 }
}

const reduce = typeof window !== 'undefined' && window.matchMedia?.('(prefers-reduced-motion: reduce)').matches
const frame = shallowRef<S>({ ...target(), amp: reduce ? 0 : target().amp })
const cur = { ...frame.value }
const active = useIsSlideActive()

let raf = 0
let last = 0
const CYCLE = 0.62

function settled(t: S) {
  return Math.abs(cur.th - t.th) < 0.05 && Math.abs(cur.h - t.h) < 0.05 && Math.abs(cur.amp - t.amp) < 0.01
}

function tick(now: number) {
  const dt = Math.min(0.05, (now - last) / 1000)
  last = now
  const t = target()
  if (reduce) t.amp = 0
  const k = 1 - Math.exp(-dt * 8)
  cur.th += (t.th - cur.th) * k
  cur.h += (t.h - cur.h) * k
  cur.amp += (t.amp - cur.amp) * (1 - Math.exp(-dt * 6))
  if (cur.amp > 0.02) cur.ph = (cur.ph + dt * (props.speed != null ? props.speed / (4 * STRIDE) : 1 / CYCLE)) % 1
  frame.value = { ...cur }
  if (active.value && (props.walking || !settled(t))) raf = requestAnimationFrame(tick)
  else raf = 0
}

function kick() {
  const t = target()
  if (reduce) t.amp = 0
  if (!active.value) {
    // not on screen: snap so the pose is right when we arrive
    Object.assign(cur, { th: t.th, h: t.h, amp: t.amp })
    frame.value = { ...cur }
    return
  }
  if (!raf && (props.walking || !settled(t))) {
    last = performance.now()
    raf = requestAnimationFrame(tick)
  }
}

onMounted(kick)
watch(() => [props.walking, props.sit, props.pitch, active.value], kick)
onBeforeUnmount(() => cancelAnimationFrame(raf))

type P = [number, number]
function ik(hip: P, foot: P): { knee: P } {
  const dx = foot[0] - hip[0]
  const dy = foot[1] - hip[1]
  const d = Math.min(Math.hypot(dx, dy), 2 * L - 0.05)
  const hh = Math.sqrt(Math.max(0, L * L - (d / 2) ** 2))
  const nx = -dy / (Math.hypot(dx, dy) || 1)
  const ny = dx / (Math.hypot(dx, dy) || 1) // perpendicular; for a downward leg this points backward
  return { knee: [hip[0] + dx / 2 + nx * hh, hip[1] + dy / 2 + ny * hh] }
}

const KNEE_Y = 3.5
const STRIDE = 7
const LIFT = 6.5

const geo = computed(() => {
  const s = frame.value
  const a = (s.th * Math.PI) / 180
  const c = Math.cos(a)
  const sn = Math.sin(a)
  const bob = s.amp * 0.7 * Math.sin(4 * Math.PI * s.ph)
  const Ry = -(s.h + bob)
  const Fx = RX + HIP_SEP * c
  const Fy = Ry - HIP_SEP * sn

  const defs = [
    { id: 'rf', far: true, rear: true, off: 0, hip: [RX + 2.6 * c, Ry - 2.6 * sn] as P },
    { id: 'ff', far: true, rear: false, off: 0.5, hip: [Fx + 2.6 * c, Fy - 2.6 * sn] as P },
    { id: 'rn', far: false, rear: true, off: 0.5, hip: [RX, Ry] as P },
    { id: 'fn', far: false, rear: false, off: 0, hip: [Fx, Fy] as P },
  ]
  const legs = defs.map((d) => {
    const p = (s.ph + d.off) % 1
    let gx: number
    let gy = 0
    if (p < 0.5) gx = STRIDE * (1 - 4 * p)
    else {
      const u = (p - 0.5) * 2
      gx = -STRIDE + 2 * STRIDE * u
      gy = LIFT * Math.sin(Math.PI * u)
    }
    // crouched rear legs: land the knee just above the ground; otherwise a small forward step
    const kh = s.h - KNEE_Y
    const rest = d.rear
      ? Math.min(0.3 * s.th, kh < L ? Math.sqrt(L * L - KNEE_Y * KNEE_Y) - Math.sqrt(L * L - kh * kh) : 99)
      : 0.1 * s.th
    const foot: P = [d.hip[0] + rest + s.amp * gx, -s.amp * gy]
    return { ...d, foot, knee: ik(d.hip, foot).knee }
  })
  return { legs, body: `translate(${RX} ${Ry}) rotate(${-s.th})` }
})

const far = computed(() => geo.value.legs.filter((l) => l.far))
const near = computed(() => geo.value.legs.filter((l) => !l.far))
</script>

<template>
  <g class="dog">
    <ellipse class="dg-shadow" cx="0" cy="1" rx="32" ry="3.4" />

    <!-- far-side legs -->
    <g v-for="l in far" :key="l.id" class="dg-far">
      <line class="o" :x1="l.hip[0]" :y1="l.hip[1]" :x2="l.knee[0]" :y2="l.knee[1]" stroke-width="8.6" />
      <line class="o" :x1="l.knee[0]" :y1="l.knee[1]" :x2="l.foot[0]" :y2="l.foot[1]" stroke-width="6.6" />
      <line class="f" :x1="l.hip[0]" :y1="l.hip[1]" :x2="l.knee[0]" :y2="l.knee[1]" stroke-width="7" />
      <line class="f" :x1="l.knee[0]" :y1="l.knee[1]" :x2="l.foot[0]" :y2="l.foot[1]" stroke-width="5" />
      <circle class="dg-foot" :cx="l.foot[0]" :cy="l.foot[1] - 0.5" r="3.4" />
    </g>

    <!-- body -->
    <g :transform="geo.body">
      <rect class="dg-shell" x="-7" y="-8" width="57" height="14" rx="7" />
      <rect class="dg-top" x="-2" y="-10.5" width="34" height="9" rx="4.5" />
      <rect class="dg-belly" x="0" y="2" width="44" height="4" rx="2" />
      <rect class="dg-panel" x="12" y="-6" width="14" height="6" rx="2.5" />
      <circle class="dg-led" cx="31" cy="-5" r="1.1" />
      <!-- head module -->
      <rect class="dg-head" x="39" y="-9.5" width="13" height="14.5" rx="5" />
      <circle class="dg-lens" cx="48" cy="-2.4" r="3.6" />
      <circle class="dg-glint" cx="47" cy="-3.4" r="1.1" />
      <rect class="dg-chin" x="41" y="1.2" width="8" height="2" rx="1" />
    </g>

    <!-- near-side legs -->
    <g v-for="l in near" :key="l.id">
      <line class="o" :x1="l.hip[0]" :y1="l.hip[1]" :x2="l.knee[0]" :y2="l.knee[1]" stroke-width="9.4" />
      <line class="o" :x1="l.knee[0]" :y1="l.knee[1]" :x2="l.foot[0]" :y2="l.foot[1]" stroke-width="7.2" />
      <line class="n" :x1="l.hip[0]" :y1="l.hip[1]" :x2="l.knee[0]" :y2="l.knee[1]" stroke-width="7.8" />
      <line class="n" :x1="l.knee[0]" :y1="l.knee[1]" :x2="l.foot[0]" :y2="l.foot[1]" stroke-width="5.6" />
      <circle class="dg-knee" :cx="l.knee[0]" :cy="l.knee[1]" r="3.3" />
      <circle class="dg-foot" :cx="l.foot[0]" :cy="l.foot[1] - 0.5" r="3.8" />
    </g>
    <g v-for="l in near" :key="l.id + 'h'">
      <circle class="dg-hip" :cx="l.hip[0]" :cy="l.hip[1]" r="6.2" />
      <circle class="dg-hipin" :cx="l.hip[0]" :cy="l.hip[1]" r="2.3" />
    </g>
  </g>
</template>

<style>
.dog .dg-shadow { fill: rgba(27, 35, 48, 0.11); }
.dog .o { stroke: #a9b2bf; stroke-linecap: round; }
.dog .n { stroke: #e4e8ee; stroke-linecap: round; }
.dog .f { stroke: #bcc4cf; stroke-linecap: round; }
.dog .dg-far .o { stroke: #8d97a6; }
.dog .dg-shell { fill: #e0e5ec; stroke: #a9b2bf; stroke-width: 1.1; }
.dog .dg-top { fill: #f1f3f7; stroke: #b9c1cc; stroke-width: 0.8; }
.dog .dg-belly { fill: #c3cad5; }
.dog .dg-panel { fill: #d2d8e1; stroke: #aab3c0; stroke-width: 0.6; }
.dog .dg-led { fill: #0b8fa8; }
.dog .dg-head { fill: #262b35; }
.dog .dg-chin { fill: #3b4250; }
.dog .dg-lens { fill: #0e1117; stroke: #5b6475; stroke-width: 1.1; }
.dog .dg-glint { fill: #6fd3e6; }
.dog .dg-hip { fill: #e9edf2; stroke: #a9b2bf; stroke-width: 1.1; }
.dog .dg-hipin { fill: #c3cad5; }
.dog .dg-knee { fill: #e9edf2; stroke: #a9b2bf; stroke-width: 0.9; }
.dog .dg-foot { fill: #1d2129; }
.dog .dg-far .dg-foot { fill: #2c323d; }
</style>
