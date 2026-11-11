<script setup lang="ts">
// Perception → language → policy pipeline, with Plan A / Plan B fork.
// `step` = number of clicks so far on the slide; each element appears at its `at` step.
defineProps<{ step: number }>()

type Node = { id: string; x: number; y: number; at: number; tone: string; icon: string; title: string; sub: string }
const NW = 146
const NH = 66
const nodes: Node[] = [
  { id: 'voice', x: 0, y: 0, at: 0, tone: 'violet', icon: 'i-carbon-microphone', title: 'Voice to text', sub: 'the prompt' },
  { id: 'cam', x: 0, y: 140, at: 0, tone: 'cyan', icon: 'i-carbon-camera', title: 'Go2 camera', sub: 'live frames' },
  { id: 'yolo', x: 183, y: 140, at: 1, tone: 'cyan', icon: 'i-carbon-image-search', title: 'YOLOv8', sub: 'person boxes' },
  { id: 'sel', x: 366, y: 70, at: 2, tone: 'violet', icon: 'i-carbon-select-window', title: 'Target selector', sub: 'Jev · Luna · d3-edge' },
  { id: 'vla', x: 549, y: 70, at: 3, tone: 'amber', icon: 'i-carbon-machine-learning-model', title: 'OmniVLA', sub: 'image + box + prompt' },
  { id: 'act', x: 732, y: 70, at: 3, tone: 'green', icon: 'i-ph-dog', title: 'Go2 control', sub: 'waypoint PD · trot' },
]
// edges: path, arrowhead position + angle, step it appears
const edges: { d: string; h?: number[]; at: number; tone?: string }[] = [
  { d: 'M146,173 H183', h: [183, 173, 0], at: 1 },
  { d: 'M329,173 H349 V118 H366', h: [366, 118, 0], at: 2 },
  { d: 'M146,33 H349 V88 H366', h: [366, 88, 0], at: 2 },
  { d: 'M512,103 H549', h: [549, 103, 0], at: 3 },
  { d: 'M695,103 H732', h: [732, 103, 0], at: 3 },
  // OmniVLA → three ways to adapt it, tried in order
  { d: 'M622,136 V232 H131', at: 4 },
  { d: 'M131,232 V262', h: [131, 262, 90], at: 4, tone: 'violet' },
  { d: 'M440,232 V262', h: [440, 262, 90], at: 5, tone: 'green' },
  { d: 'M622,232 H749 V262', h: [749, 262, 90], at: 6, tone: 'amber' },
  // "if it fails" hand-offs between the cards
  { d: 'M266,317 H305', h: [305, 317, 0], at: 5, tone: 'mute' },
  { d: 'M575,317 H614', h: [614, 317, 0], at: 6, tone: 'mute' },
]
const fails = [{ x: 285, at: 5 }, { x: 594, at: 6 }]
const packet = 'M146,173 H349 V118 H440 V103 H732'
</script>

<template>
  <div class="pf-wrap">
    <!-- edges -->
    <svg class="pf-edges" viewBox="0 0 880 372">
      <g v-for="(e, i) in edges" :key="i" :class="['pf-e', e.tone, { show: step >= e.at }]">
        <path :d="e.d" pathLength="1" />
        <path v-if="e.h" class="pf-ah" d="M-8,-5 L0,0 L-8,5" :transform="`translate(${e.h[0]} ${e.h[1]}) rotate(${e.h[2]})`" />
      </g>
      <text v-for="f in fails" :key="f.x" class="pf-fail" :class="{ show: step >= f.at }" :x="f.x" y="308" text-anchor="middle">fails</text>
      <circle v-if="step >= 4 && step < 7" class="pf-packet" r="4">
        <animateMotion dur="3.6s" repeatCount="indefinite" :path="packet" />
      </circle>
    </svg>

    <!-- nodes -->
    <div
      v-for="n in nodes"
      :key="n.id"
      class="pf-node card brk"
      :class="[n.tone, { show: step >= n.at }]"
      :style="{ left: n.x + 'px', top: n.y + 'px', width: NW + 'px', height: NH + 'px' }"
    >
      <div :class="n.icon" class="pf-ico" />
      <div class="pf-t">{{ n.title }}</div>
      <div class="pf-s">{{ n.sub }}</div>
    </div>

    <!-- adaptation ladder: each step only if the previous one falls short -->
    <div class="pf-plan card brk violet" :class="{ show: step >= 4 }" style="left: 0; top: 262px">
      <span class="chip">First</span>
      <div class="pf-pt">Prompt tuning</div>
      <p>Frozen <b>OmniVLA-Edge</b>. Try box styles and prompt wording.</p>
      <div class="pf-tag">No training</div>
    </div>
    <div class="pf-plan card brk green" :class="{ show: step >= 5 }" style="left: 309px; top: 262px">
      <span class="chip">Plan A</span>
      <div class="pf-pt">DoRA on OmniVLA-Edge</div>
      <p>Fine-tune on boxed examples so it gets used to boxes.</p>
      <div class="pf-tag">Lab GPU</div>
    </div>
    <div class="pf-plan card brk amber" :class="{ show: step >= 6 }" style="left: 618px; top: 262px">
      <span class="chip">Plan B</span>
      <div class="pf-pt">DoRA on OmniVLA</div>
      <p>Same fine-tuning on the full <b>7.5B</b> model.</p>
      <div class="pf-tag hot">Rented RTX 5090</div>
    </div>
  </div>
</template>

<style>
.pf-wrap { position: relative; width: 880px; height: 372px; }
.pf-edges { position: absolute; inset: 0; width: 880px; height: 372px; overflow: visible; }

.pf-e path:first-child { fill: none; stroke: var(--cyan); stroke-width: 2; stroke-linecap: round; stroke-dasharray: 1; stroke-dashoffset: 1; transition: stroke-dashoffset 0.7s ease; opacity: 0.85; }
.pf-e.green path:first-child { stroke: var(--green); }
.pf-e.amber path:first-child { stroke: var(--amber); stroke-dasharray: 0.03 0.015; stroke-dashoffset: 0; opacity: 0; transition: opacity 0.6s ease; }
.pf-e.amber.show path:first-child { opacity: 0.95; }
.pf-e.show path:first-child { stroke-dashoffset: 0; }
.pf-ah { fill: none; stroke: var(--cyan); stroke-width: 2; stroke-linecap: round; stroke-linejoin: round; opacity: 0; transition: opacity 0.3s ease 0.55s; }
.pf-e.green .pf-ah { stroke: var(--green); }
.pf-e.amber .pf-ah { stroke: var(--amber); }
.pf-e.show .pf-ah { opacity: 1; }
.pf-packet { fill: var(--amber); }

.pf-node, .pf-plan, .pf-legend { opacity: 0; transform: translateY(10px) scale(0.98); transition: opacity 0.5s ease, transform 0.5s cubic-bezier(0.2, 0.8, 0.2, 1); position: absolute; }
.pf-node.show, .pf-plan.show, .pf-legend.show { opacity: 1; transform: none; }
.pf-node { padding: 0.5rem 0.75rem; display: flex; flex-direction: column; justify-content: center; }
.pf-node.cyan, .pf-plan.cyan { --tone: var(--cyan); }
.pf-node.violet, .pf-plan.violet { --tone: var(--violet); }
.pf-node.amber, .pf-plan.amber { --tone: var(--amber); }
.pf-node.green, .pf-plan.green { --tone: var(--green); }
.pf-ico { position: absolute; top: 0.5rem; right: 0.55rem; font-size: 1.05rem; color: var(--tone); }
.pf-t { font-weight: 700; font-size: 0.9rem; letter-spacing: -0.01em; color: var(--ink); }
.pf-s { font-size: 0.7rem; color: var(--mute); margin-top: 0.12rem; line-height: 1.25; }

.pf-e.violet path:first-child { stroke: var(--violet); }
.pf-e.violet .pf-ah { stroke: var(--violet); }
.pf-e.mute path:first-child { stroke: var(--mute); stroke-width: 1.5; }
.pf-e.mute .pf-ah { stroke: var(--mute); stroke-width: 1.5; }
.pf-fail { font: 600 9px var(--ff-sans); fill: var(--mute); opacity: 0; transition: opacity 0.4s ease 0.4s; }
.pf-fail.show { opacity: 1; }

.pf-plan { padding: 0.65rem 0.95rem; width: 262px; height: 110px; position: absolute; }
.pf-pt { font-size: 1.1rem; font-weight: 700; letter-spacing: -0.02em; color: var(--ink); margin: 0.25rem 0 0.1rem; white-space: nowrap; }
.pf-plan p { font-size: 0.78rem; line-height: 1.35; color: var(--soft); margin: 0; }
.pf-plan b { color: var(--ink); font-weight: 600; }
.pf-tag { position: absolute; top: 0.6rem; right: 0.8rem; font-size: 0.68rem; font-weight: 600; color: var(--mute); padding: 0.12rem 0.55rem; border-radius: 999px; border: 1px solid var(--line); }
.pf-tag.hot { color: var(--amber); border-color: color-mix(in srgb, var(--amber) 45%, white); background: color-mix(in srgb, var(--amber) 10%, white); }
</style>
