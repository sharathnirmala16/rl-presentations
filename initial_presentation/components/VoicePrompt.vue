<script setup lang="ts">
defineProps<{ text: string }>()
</script>

<template>
  <div class="vp">
    <div class="vp-mic">
      <i class="vp-ring" /><i class="vp-ring r2" />
      <carbon-microphone-filled />
    </div>
    <div class="vp-bubble">
      <span class="vp-q">“</span>
      <span class="vp-typed" :style="{ '--n': text.length }">{{ text }}</span>
      <i class="vp-caret" />
    </div>
  </div>
</template>

<style>
.vp { display: inline-flex; align-items: center; gap: 0.9rem; padding: 0.6rem 1.2rem 0.6rem 0.7rem; border-radius: 999px;
  background: var(--panel); box-shadow: var(--shadow); border: 1px solid var(--line); }
.vp-mic { position: relative; width: 38px; height: 38px; flex: none; display: grid; place-items: center; font-size: 1.15rem;
  border-radius: 50%; background: var(--violet); color: #fff; }
.vp-ring { position: absolute; inset: 0; border-radius: 50%; border: 2px solid var(--violet); opacity: 0; }
.stage.on .vp-ring { animation: vp-ping 1.8s ease-out infinite; }
.stage.on .vp-ring.r2 { animation-delay: 0.9s; }
@keyframes vp-ping { from { transform: scale(1); opacity: 0.55; } to { transform: scale(2); opacity: 0; } }
.vp-bubble { font-family: var(--ff-code); font-size: 1rem; color: var(--ink); display: flex; align-items: center; white-space: nowrap; }
.vp-q { color: var(--violet); margin-right: 0.15rem; }
.vp-typed { display: inline-block; overflow: hidden; white-space: nowrap; width: calc(var(--n) * 1ch); }
.stage.on .vp-typed { width: 0; animation: vp-type calc(var(--n) * 45ms) steps(var(--n)) 0.7s forwards; }
@keyframes vp-type { to { width: calc(var(--n) * 1ch); } }
.vp-caret { display: inline-block; width: 2px; height: 1.05em; background: var(--amber); margin-left: 3px; animation: vp-blink 1s steps(1) infinite; }
@keyframes vp-blink { 50% { opacity: 0; } }
@media (prefers-reduced-motion: reduce) { .stage.on .vp-typed { animation: none; width: calc(var(--n) * 1ch); } .stage.on .vp-ring { animation: none; } }
</style>
