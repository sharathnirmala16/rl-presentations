<script setup lang="ts">
// ✏️ Edit this list to change the slide. Owners are a proposed split — adjust as the team decides.
const tracks = [
  {
    tone: 'cyan', icon: 'i-carbon-image-search', title: 'Perception',
    scope: ['YOLOv8 fine-tune on CrowdHuman', 'ID tests on DanceTrack'],
    owners: ['Melvin', 'Gayathri'],
  },
  {
    tone: 'violet', icon: 'i-carbon-microphone', title: 'Voice & target selection',
    scope: ['Speech-to-text prompt', 'Target selector: Jev, Luna, d3-edge'],
    owners: ['Jordan'],
  },
  {
    tone: 'amber', icon: 'i-carbon-machine-learning-model', title: 'VLA adaptation',
    scope: ['Prompt tuning, Plan A: DoRA on Edge', 'Plan B: DoRA on OmniVLA'],
    owners: ['Sharath', 'Dhruv'],
  },
  {
    tone: 'green', icon: 'i-ph-dog', title: 'Go2 behaviour',
    scope: ['Kine2Go MLP: look up, sit', 'State machine + trot'],
    owners: ['Abhishake', 'Shreya'],
  },
]
</script>

<template>
  <div class="dl">
    <div class="dl-grid">
      <div v-for="(t, i) in tracks" :key="t.title" class="dl-card card brk rise" :class="t.tone" :style="{ '--i': i + 1 }">
        <div class="dl-head">
          <div class="dl-t">{{ t.title }}</div>
          <div :class="t.icon" class="dl-ico" />
        </div>
        <ul>
          <li v-for="s in t.scope" :key="s">{{ s }}</li>
        </ul>
        <div class="dl-owners">
          <span v-for="o in t.owners" :key="o" class="dl-av"><b>{{ o[0] }}</b>{{ o }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
.dl-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; }
.dl-card { --tone: var(--cyan); padding: 0.95rem 1.1rem 1rem; }
.dl-card.violet { --tone: var(--violet); } .dl-card.amber { --tone: var(--amber); } .dl-card.green { --tone: var(--green); }
.dl-head { display: flex; justify-content: space-between; align-items: center; margin-bottom: 0.5rem; }
.dl-ico { font-size: 1.4rem; color: var(--tone); }
.dl-t { font-size: 1.25rem; font-weight: 700; letter-spacing: -0.02em; }
.dl-card ul { list-style: none; padding: 0; margin: 0 0 0.8rem; }
.dl-card li { position: relative; padding-left: 0.9rem; font-size: 0.88rem; line-height: 1.5; color: var(--soft); }
.dl-card li::before { content: ''; position: absolute; left: 0; top: 0.62em; width: 5px; height: 5px; border-radius: 50%; background: var(--tone); }
.dl-owners { display: flex; gap: 0.5rem; flex-wrap: wrap; }
.dl-av { display: inline-flex; align-items: center; gap: 0.4rem; font-size: 0.82rem; font-weight: 600; padding: 0.15rem 0.7rem 0.15rem 0.2rem; border-radius: 999px; background: color-mix(in srgb, var(--tone) 10%, white); }
.dl-av b { display: grid; place-items: center; width: 1.45rem; height: 1.45rem; border-radius: 50%; background: var(--tone); color: #fff; font-size: 0.72rem; }
</style>
