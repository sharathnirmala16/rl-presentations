<script setup lang="ts">
import { useNav } from '@slidev/client'

const { currentPage, total } = useNav()
const pad = (n: number) => String(n).padStart(2, '0')
</script>

<template>
  <div class="gb">
    <div class="gb-glow" />
    <div v-if="currentPage > 1" class="gb-foot">
      <span>Team 3 · RL Project</span>
      <div class="gb-ticks">
        <i
          v-for="n in total"
          :key="n"
          :class="{ cur: n === currentPage, done: n < currentPage }"
        />
      </div>
      <span>{{ pad(currentPage) }} / {{ pad(total) }}</span>
    </div>
  </div>
</template>

<style>
.gb { position: absolute; inset: 0; pointer-events: none; overflow: hidden; }
.gb-glow {
  position: absolute; inset: 0;
  background:
    radial-gradient(560px 320px at 100% 0%, rgba(11, 143, 168, 0.09), transparent 70%),
    radial-gradient(520px 300px at 0% 100%, rgba(233, 117, 13, 0.08), transparent 70%);
}
.gb-foot {
  position: absolute; left: 3rem; right: 3rem; bottom: 0.85rem;
  display: flex; align-items: center; justify-content: space-between;
  font-size: 0.72rem; font-weight: 500; letter-spacing: 0.02em;
  color: var(--mute);
}
.gb-ticks { display: flex; gap: 5px; }
.gb-ticks i {
  display: block; width: 26px; height: 4px; border-radius: 2px;
  background: var(--line); transition: background 0.4s;
}
.gb-ticks i.done { background: color-mix(in srgb, var(--cyan) 45%, var(--line)); }
.gb-ticks i.cur { background: var(--amber); }
</style>
