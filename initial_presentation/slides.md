---
theme: default
title: Voice-Guided Human Following · Team 3
info: |
  RL Project — initial presentation (Team 3).
  Language-conditioned human tracking in crowds with the Unitree Go2.
colorSchema: light
layout: hero
transition: fade
aspectRatio: 16/9
canvasWidth: 980
drawings:
  persist: false
fonts:
  sans: Space Grotesk
  mono: JetBrains Mono
  provider: google
---

<div class="cv-copy">
  <div class="kicker rise" style="--i:0"><span class="tick" />RL Project · Team 3</div>
  <h1 class="cv-title">Voice-Guided<br />Human Following</h1>
  <p class="cv-sub rise" style="--i:3">Tell a Unitree Go2 who to follow. It tracks them through a crowd.</p>
  <div class="cv-names">
    <span v-for="(n, i) in ['Abhishake','Dhruv','Gayathri','Jordan','Melvin','Sharath','Shreya']" :key="n" class="cv-name rise" :style="{ '--i': i + 4 }">{{ n }}</span>
  </div>
</div>

<CrowdScene class="cv-scene" />

<style>
.cv-copy { position: absolute; left: 3.4rem; top: 2.6rem; z-index: 2; }
.cv-title { font-size: 3.6rem !important; line-height: 1.02 !important; font-weight: 700; letter-spacing: -0.04em; margin: 0.3rem 0 0.9rem !important; }
.cv-sub { font-size: 1.1rem; color: var(--soft); margin: 0 0 1.1rem; }
.cv-names { display: flex; flex-wrap: wrap; gap: 0.45rem; max-width: 680px; }
.cv-name { font-size: 0.82rem; font-weight: 600; padding: 0.25rem 0.8rem; border-radius: 999px; background: var(--panel); border: 1px solid var(--line); box-shadow: var(--shadow); }
.cv-scene { position: absolute; left: 0; right: 0; bottom: 0; height: 56%; z-index: 1; }
</style>

<!--
We are Team 3. The project: describe a person in words, and a Unitree Go2 robot dog follows that person through a crowd.
In the scene, many people are detected (teal boxes), one matches the description (green jacket) and is selected (orange), and the Go2 follows.
-->

---
layout: hud
kicker: 'Problem'
---

# Follow the right person in a crowd

<VoicePrompt text="Follow the caregiver in the green jacket." class="rise" style="--i:1; align-self:flex-start" />

<p class="pb-lead rise" style="--i:2">Dynamic human tracking in crowds, steered by natural language.</p>

<div class="pb-grid">
  <div class="pb-chips">
    <div v-for="(c, i) in [
      ['i-carbon-layers', 'Occlusion'],
      ['i-carbon-user-multiple', 'Look-alikes'],
      ['i-carbon-movement', 'Constant motion'],
      ['i-carbon-search', 'Lost target'],
    ]" :key="c[1]" class="pb-chip rise" :style="{ '--i': i + 3 }">
      <div :class="c[0]" class="pb-cico" />{{ c[1] }}
    </div>
  </div>

  <div class="pb-cases">
    <div v-click class="card brk pb-case" style="--tone: var(--green)">
      <div class="pb-ico i-ph-person-arms-spread" />
      <div>
        <div class="pb-ct">Assistive guidance</div>
        <p>A visually impaired user has the dog follow their caregiver.</p>
      </div>
    </div>
    <div v-click class="card brk pb-case" style="--tone: var(--amber)">
      <div class="pb-ico i-ph-siren" />
      <div>
        <div class="pb-ct">Search &amp; rescue</div>
        <p>Describe a missing person. The dog finds and tracks them.</p>
      </div>
    </div>
  </div>
</div>

<style>
.pb-lead { font-size: 1.05rem; color: var(--soft); margin: 1.1rem 0 1.3rem; }
.pb-grid { display: grid; grid-template-columns: 0.8fr 1.2fr; gap: 2.2rem; align-items: stretch; }
.pb-chips { display: grid; grid-template-columns: 1fr 1fr; grid-auto-rows: 1fr; gap: 0.7rem; }
.pb-chip { display: flex; flex-direction: column; gap: 0.5rem; padding: 0.9rem 1rem; border-radius: 14px; background: var(--panel); border: 1px solid var(--line); font-size: 1rem; font-weight: 600; letter-spacing: -0.01em; }
.pb-cico { font-size: 1.5rem; color: var(--cyan); }
.pb-cases { display: flex; flex-direction: column; gap: 0.9rem; }
.pb-case { display: flex; gap: 1rem; align-items: center; padding: 1rem 1.2rem; }
.pb-ico { flex: none; font-size: 2.3rem; color: var(--tone); }
.pb-ct { font-size: 1.25rem; font-weight: 700; letter-spacing: -0.02em; margin-bottom: 0.15rem; }
.pb-case p { margin: 0; font-size: 0.95rem; line-height: 1.4; color: var(--soft); }
</style>

<!--
The problem: dynamic human tracking in a crowd. Following one person is easy; following the one the user means, while everyone moves and overlaps, is hard. The user just speaks: "follow the caregiver in the green jacket".
Hard parts: occlusion, look-alikes, everything moving, losing the target.
[click] Use case one: a visually impaired person has the dog follow their caregiver in a busy place.
[click] Use case two: search and rescue. Describe someone once, the robot finds and tracks them.
-->

---
layout: hud
kicker: 'Approach'
clicks: 9
---

# Pipeline and fallback plans

<div class="fl-tabs">
  <span :class="{ on: $clicks < 7 }">Perception pipeline</span>
  <span :class="{ on: $clicks >= 7 }">Behaviour states</span>
</div>

<div class="fl-views">
  <PipelineFlow :step="$clicks" class="fl-v" :class="{ off: $clicks >= 7 }" />
  <StateMachine :step="$clicks - 7" class="fl-v" :class="{ off: $clicks < 7 }" />
</div>

<style>
.fl-tabs { position: absolute; top: 1.7rem; right: 3rem; display: flex; gap: 0.4rem; font-size: 0.72rem; font-weight: 600; }
.fl-tabs span { padding: 0.28rem 0.8rem; border-radius: 999px; color: var(--mute); background: rgba(27, 35, 48, 0.05); transition: all 0.4s ease; }
.fl-tabs span.on { color: #fff; background: var(--ink); }
.fl-views { position: relative; width: 880px; height: 372px; }
.fl-v { position: absolute; left: 0; top: 0; transition: opacity 0.5s ease, transform 0.5s ease; }
.fl-v.off { opacity: 0 !important; transform: translateY(12px); pointer-events: none; }
</style>

<!--
[click x6] Pipeline: the user's voice becomes a text prompt. The Go2 camera frame goes to a fine-tuned YOLOv8 for person boxes. A target-selection model (we will try Jev, GPT 6 Luna Decisions and d3-edge 0.6B) picks the box that best matches the description. The image with that box goes to OmniVLA with the prompt "follow that person".
Adapting OmniVLA to boxes, in order, moving on only if the previous step falls short:
First, prompt tuning on a frozen OmniVLA-Edge: boxed image plus prompt, and we experiment with the box style and the prompt wording. No training.
Plan A: DoRA fine-tuning of OmniVLA-Edge on examples with boxes, so it gets used to them. Small enough for the lab GPUs (to confirm).
Plan B: the same DoRA fine-tuning on the full 7.5B OmniVLA. This is the only step that needs a rented RTX 5090.
[click x3] Three states. 1 Search and face anchor: the dog stops, Kine2Go pitches it up about 35 degrees, it confirms the face. 2 Track and pursue: follow the selected box. 3 Arrival / dispute resolution: if distance drops below 1.5 m or tracking confidence drops, Kine2Go sits / pitches up to re-verify the face anchor.
-->

---
layout: hud
kicker: 'Data'
---

# Datasets

<div class="ds-grid">
  <div class="card brk ds-card rise" style="--i:1; --tone: var(--cyan)">
    <div class="ds-name">CrowdHuman</div>
    <span class="chip">Fine-tune YOLOv8 · lab GPUs</span>
    <BoxField kind="crowd" />
    <div class="ds-stats">
      <div><b><CountUp :to="15000" /></b><span>train images</span></div>
      <div><b><CountUp :to="470" prefix="~" suffix="K" /></b><span>person boxes</span></div>
      <div><b><CountUp :to="22.6" :decimals="1" /></b><span>people per image</span></div>
    </div>
    <p>Dense, occluded crowds</p>
  </div>

  <div class="card brk ds-card rise" style="--i:2; --tone: var(--cyan)">
    <div class="ds-name">DanceTrack</div>
    <span class="chip">Fine-tune &amp; test YOLOv8 · lab GPUs</span>
    <BoxField kind="dance" />
    <div class="ds-stats">
      <div><b><CountUp :to="100" /></b><span>videos</span></div>
      <div><b><CountUp :to="105" prefix="~" suffix="K" /></b><span>frames</span></div>
      <div><b class="ds-txt">HOTA · IDF1</b><span>metrics</span></div>
    </div>
    <p>Look-alike dancers, easy ID swaps</p>
  </div>

  <div class="card brk ds-card ds-small rise" style="--i:3; --tone: var(--green)">
    <div class="ds-name">Kine2Go</div>
    <span class="chip">Trains the MLP</span>
    <svg class="ds-k2g" viewBox="-36 -46 90 50"><GoDog sit /></svg>
    <p>The look-up and sit motions (states 1 &amp; 3)</p>
  </div>
</div>

<div class="ds-flow rise" style="--i:5">
  <span class="ds-fl">Auto-labelling the data for OmniVLA</span>
  <span class="chip" style="--tone: var(--cyan)">YOLOv8 + tracker</span><i class="i-carbon-arrow-right" />
  <span class="chip" style="--tone: var(--green)">Waypoints from boxes</span><i class="i-carbon-arrow-right" />
  <span class="chip" style="--tone: var(--violet)">Text: DeepSeek V4.1 Flash</span><i class="i-carbon-arrow-right" />
  <span class="chip" style="--tone: var(--amber)">Fine-tune OmniVLA</span>
</div>

<style>
.ds-grid { display: grid; grid-template-columns: 1.15fr 1.15fr 0.9fr; gap: 1rem; margin-top: 0.4rem; }
.ds-card { padding: 0.95rem 1.05rem; display: flex; flex-direction: column; align-items: flex-start; gap: 0.35rem; }
.ds-name { font-size: 1.55rem; font-weight: 700; letter-spacing: -0.03em; }
.ds-card .bf { margin: 0.4rem 0 0.2rem; }
.ds-stats { display: flex; align-items: flex-end; gap: 1rem; margin: 0.2rem 0 0.3rem; }
.ds-stats div { display: flex; flex-direction: column; }
.ds-stats b { font-size: 1.25rem; font-weight: 700; letter-spacing: -0.02em; }
.ds-stats .ds-txt { font-size: 0.95rem; padding-top: 0.2rem; }
.ds-stats div > span { font-size: 0.68rem; color: var(--mute); }
.ds-card p { margin: 0; font-size: 0.92rem; line-height: 1.4; color: var(--soft); }
.ds-k2g { height: 112px; width: 100%; margin: 0.3rem 0 0.3rem; }
.ds-flow { margin-top: 1.1rem; display: flex; align-items: center; gap: 0.6rem; flex-wrap: wrap; color: var(--mute); }
.ds-flow .chip { font-size: 0.78rem; padding: 0.35rem 0.8rem; }
.ds-fl { flex-basis: 100%; font-size: 0.72rem; font-weight: 700; color: var(--mute); }
</style>

<!--
Two public datasets for the person-tracking part. CrowdHuman: crowded, occluded scenes. Both datasets are used to fine-tune and test YOLOv8 on the lab GPUs, so no rental cost. DanceTrack: everyone looks alike and moves unpredictably, so it tests that we stay locked on the same person.
Kine2Go is the motion data for the small MLP that makes the dog look up and sit.
Bottom row is how we plan to make training data for OmniVLA from these datasets. YOLOv8 plus a tracker gives boxes and target velocity. Waypoint labels are computed from the target's box (horizontal offset gives heading, box height gives distance), so they are synthetic, not recorded robot motion. A cheap vision-language model (DeepSeek V4.1 Flash) writes a short description of each person, one per track. CrowdHuman is still images, so it only adds boxes and descriptions, no motion. Only the 7.5B OmniVLA is fine-tuned on rented compute. Open question for feedback: DanceTrack dancers dress alike, so descriptions may be ambiguous and we will check a sample by hand.
Dataset stats are from the dataset papers; double check before presenting.
-->

---
layout: hud
kicker: 'Resources'
clicks: 6
---

# Tech stack, compute and GPU cost estimate

<div class="tk-row rise" style="--i:1">
  <div class="tk-col green"><div class="tk-h">Robot</div><span class="tk-chip">Unitree Go2</span><span class="tk-chip">ROS 2</span></div>
  <div class="tk-col violet"><div class="tk-h">Voice</div><span class="tk-chip">Whisper STT</span></div>
  <div class="tk-col cyan"><div class="tk-h">Perception</div><span class="tk-chip">YOLOv8</span><span class="tk-chip">Jev · Luna · d3-edge</span></div>
  <div class="tk-col amber"><div class="tk-h">Policy</div><span class="tk-chip">OmniVLA</span><span class="tk-chip">Kine2Go MLP</span></div>
  <div class="tk-col slate"><div class="tk-h">Compute</div><span class="tk-chip">Lab GPUs: YOLOv8, MLP, OmniVLA-Edge</span><span class="tk-chip tk-hot">Rented RTX 5090: OmniVLA 7.5B only</span></div>
</div>

<div class="tk-main">
  <div class="card brk tk-math" style="--tone: var(--amber)">
    <div class="tk-mh">Plan B: one DoRA run on OmniVLA (7.5B)</div>
    <div v-click="1" class="tk-r"><div class="tk-f"><i>D</i> = 5×10<sup>4</sup> samples × 400 tok × 3 ep = <b>6×10<sup>7</sup></b> tokens</div><div class="tk-s">50k frames ≈ 83 min of Go2 demos at 10 Hz · ~400 tokens each (to confirm in code)</div></div>
    <div v-click="2" class="tk-r"><div class="tk-f"><i>F</i> ≈ 6<i>ND</i> = 6 × 7.5×10<sup>9</sup> × 6×10<sup>7</sup> ≈ <b>2.7×10<sup>18</sup></b> FLOPs</div><div class="tk-s">6ND rule (Kaplan et al. 2020), an upper bound for PEFT</div></div>
    <div v-click="3" class="tk-r"><div class="tk-f"><i>R</i> ≈ 210 TFLOPs × 0.30 ≈ <b>6.3×10<sup>13</sup></b> FLOP/s</div><div class="tk-s">RTX 5090 BF16 dense peak (NVIDIA spec) · 30% utilisation (Llama 3 hit 38–43%)</div></div>
    <div v-click="4" class="tk-r"><div class="tk-f"><i>t</i> = <i>F</i>/<i>R</i> ≈ 4.3×10<sup>4</sup> s ≈ <b>12 h</b></div></div>
    <div v-click="5" class="tk-r"><div class="tk-f">cost ≈ 12 h × $0.30–0.60/h ≈ <b>$4–7</b> per run</div><div class="tk-s">RTX 5090 marketplace listings, e.g. Vast.ai (~$0.2–0.4 cheapest, ~$0.6 median)</div></div>
    <div class="tk-fit"><span class="chip" style="--tone: var(--green)">VRAM ✓</span>7.5B in bf16 ≈ 15 GB + adapters, fits the 5090’s 32 GB</div>
  </div>

  <div class="tk-total">
    <div class="tk-th">Estimated GPU cost</div>
    <div class="tk-big">
      <span v-click="6" class="tk-b1">$0</span>
      <span v-click="6" class="tk-dash">–</span>
      <span v-click="6" class="tk-b2"><CountUp prefix="$" :to="22" :dur="1200" :delay="150" :when="$clicks >= 6" /></span>
    </div>
    <div class="tk-rows">
      <div v-click="6"><span>Prompt tuning or Plan A works</span><b>$0, everything on lab GPUs</b></div>
      <div v-click="6"><span>We reach Plan B</span><b>One rented run ≈ $4–7</b></div>
      <div v-click="6"><span>Worst case</span><b>Three Plan B runs ≈ $22</b></div>
    </div>
  </div>
</div>

<style>
.tk-row { display: grid; grid-template-columns: 0.95fr 0.7fr 0.8fr 1.15fr 1.5fr; gap: 0.7rem; margin-top: 0.2rem; }
.tk-col { --tone: var(--cyan); display: flex; flex-wrap: wrap; gap: 0.3rem; align-items: flex-start; align-content: flex-start; }
.tk-col.green { --tone: var(--green); } .tk-col.violet { --tone: var(--violet); } .tk-col.amber { --tone: var(--amber); } .tk-col.slate { --tone: var(--ink); }
.tk-h { width: 100%; font-size: 0.74rem; font-weight: 700; color: var(--tone); }
.tk-chip { font-size: 0.74rem; font-weight: 500; padding: 0.18rem 0.6rem; border-radius: 999px; background: var(--panel); border: 1px solid var(--line); white-space: nowrap; }
.tk-hot { background: color-mix(in srgb, var(--amber) 12%, white); border-color: color-mix(in srgb, var(--amber) 45%, white); font-weight: 600; }
.tk-main { display: grid; grid-template-columns: 1.75fr 1fr; gap: 1.4rem; margin-top: 0.8rem; align-items: stretch; }
.tk-math { padding: 0.75rem 1.1rem 0.8rem; }
.tk-mh { font-size: 0.95rem; font-weight: 700; letter-spacing: -0.01em; margin-bottom: 0.45rem; }
.tk-r { padding: 0.28rem 0; border-top: 1px solid var(--line); }
.tk-f { font-size: 0.98rem; line-height: 1.3; }
.tk-f i { font-style: italic; font-family: Georgia, 'Times New Roman', serif; }
.tk-f b { color: var(--amber); font-weight: 700; }
.tk-s { font-size: 0.66rem; line-height: 1.3; color: var(--mute); margin-top: 0.05rem; }
.tk-fit { display: flex; gap: 0.5rem; align-items: center; font-size: 0.74rem; color: var(--soft); padding-top: 0.5rem; border-top: 1px solid var(--line); margin-top: 0.15rem; }
.tk-th { font-size: 0.95rem; font-weight: 700; letter-spacing: -0.01em; }
.tk-big { display: flex; align-items: baseline; gap: 0.3rem; font-size: 3.4rem; font-weight: 700; letter-spacing: -0.045em; line-height: 1.1; margin: 0.1rem 0 0.5rem; }
.tk-b1 { color: var(--green); } .tk-b2 { color: var(--amber); } .tk-dash { color: var(--mute); font-weight: 300; }
.tk-rows { display: flex; flex-direction: column; gap: 0.5rem; }
.tk-rows div { display: flex; flex-direction: column; font-size: 0.8rem; padding-left: 0.75rem; border-left: 3px solid var(--line); }
.tk-rows span { font-size: 0.66rem; font-weight: 600; color: var(--mute); }
.tk-rows b { font-weight: 600; line-height: 1.3; }
.tk-rows .tk-unc { border-left-color: transparent; font-size: 0.7rem; color: var(--mute); padding-left: 0.75rem; }
</style>

<!--
Tech stack across the top: Go2 robot, Whisper for speech to text, YOLOv8, the target selectors (Jev, GPT 6 Luna Decisions, d3-edge 0.6B), OmniVLA and the Kine2Go MLP. YOLOv8, the MLP and anything on OmniVLA-Edge (prompt tuning, Plan A) run on lab GPUs. Only DoRA fine-tuning of the full 7.5B OmniVLA (Plan B) needs a rented RTX 5090.
This is an estimate for Plan B, the only step with a rental cost, one row per click:
[1] 50k samples x 400 tokens x 3 epochs = 6e7 tokens. 50k frames is about 83 minutes of Go2 driving at 10 Hz. The 400 tokens per sample is our assumption (image tokens, prompt, actions) and the 50k is our target dataset size; both are estimates.
[2] FLOPs ~ 6 x parameters x tokens = 2.7e18, using OmniVLA's 7.5B parameters. 6ND is an upper bound for LoRA/DoRA since they skip weight gradients for the frozen base.
[3] RTX 5090 peak BF16 dense is about 210 TFLOPs. We assume 30% utilisation; Llama 3 reached 38 to 43% on tuned H100 clusters.
[4] So about 4.3e4 seconds, roughly 12 hours.
[5] At $0.30 to 0.60 per hour that is $4 to 7 per run. Marketplace listings such as Vast.ai range from about $0.2 to 0.4 at the cheap end to about $0.6 median.
[6] Whole project estimate: $0 if prompt tuning or Plan A (both on OmniVLA-Edge, lab GPUs) is enough. If we reach Plan B, one run is about $4 to 7. Worst case about $22 with three Plan B runs (3 x 12 h x $0.60). The least certain inputs are tokens per sample, real GPU utilisation and the rental price; we would like feedback on these.
-->

---
layout: hud
kicker: 'Risks'
---

# Fallbacks

<div class="fb">
  <div class="fb-h rise" style="--i:0"><span>If this goes wrong</span><span>We try, in order</span></div>

  <div class="fb-row card rise" style="--i:1">
    <div class="fb-risk"><i class="i-carbon-machine-learning-model" />OmniVLA ignores the boxes</div>
    <div class="fb-chain">
      <span class="fb-s">Prompt engineering</span><i class="i-carbon-arrow-right" />
      <span class="fb-s">DoRA on Edge</span><i class="i-carbon-arrow-right" />
      <span class="fb-s">DoRA on OmniVLA</span><i class="i-carbon-arrow-right" />
      <span class="fb-s last">YOLO + tracker + PD control</span>
    </div>
  </div>

  <div class="fb-row card rise" style="--i:2">
    <div class="fb-risk"><i class="i-carbon-data-base" />Not enough boxed training data</div>
    <div class="fb-chain">
      <span class="fb-s">Auto-label the datasets</span><i class="i-carbon-arrow-right" />
      <span class="fb-s">Record our own Go2 runs</span><i class="i-carbon-arrow-right" />
      <span class="fb-s last">Skip fine-tuning, report why</span>
    </div>
  </div>

  <div class="fb-row card rise" style="--i:3">
    <div class="fb-risk"><i class="i-carbon-select-window" />Picking the right person</div>
    <div class="fb-chain">
      <span class="fb-s">Jev</span><i class="i-carbon-arrow-right" />
      <span class="fb-s">GPT 6 Luna Decisions</span><i class="i-carbon-arrow-right" />
      <span class="fb-s last">d3-edge 0.6B, fine-tuned</span>
    </div>
  </div>

  <div class="fb-row card rise" style="--i:4">
    <div class="fb-risk"><i class="i-ph-dog" />Look up / sit motion</div>
    <div class="fb-chain">
      <span class="fb-s">Kine2Go-trained MLP</span><i class="i-carbon-arrow-right" />
      <span class="fb-s last">Go2 SDK pitch and sit commands</span>
    </div>
  </div>

</div>

<style>
.fb { display: flex; flex-direction: column; gap: 0.42rem; }
.fb-h { display: grid; grid-template-columns: 205px 1fr; padding: 0 0.95rem; font-size: 0.68rem; font-weight: 700; color: var(--mute); }
.fb-row { display: grid; grid-template-columns: 205px 1fr; align-items: center; padding: 0.5rem 0.95rem; border-radius: 12px; }
.fb-risk { display: flex; align-items: center; gap: 0.5rem; font-size: 0.82rem; font-weight: 700; line-height: 1.2; }
.fb-risk i { font-size: 1.05rem; color: var(--cyan); flex: none; }
.fb-chain { display: flex; align-items: center; gap: 0.35rem; flex-wrap: wrap; color: var(--mute); }
.fb-s { font-size: 0.74rem; font-weight: 500; padding: 0.2rem 0.65rem; border-radius: 999px; background: var(--panel-2); border: 1px solid var(--line); white-space: nowrap; color: var(--ink); }
.fb-s.last { background: color-mix(in srgb, var(--amber) 11%, white); border-color: color-mix(in srgb, var(--amber) 45%, white); color: var(--ink); font-weight: 600; }
.fb-chain i { font-size: 0.8rem; }
</style>

<!--
Each row is a risk, then what we try in order; the amber chip is the final fallback.
OmniVLA: we start with prompt engineering on a frozen OmniVLA-Edge (boxed image plus prompt), then DoRA on Edge, then DoRA on the full 7.5B model. If none of these follow boxes, we fall back to a classical pipeline: YOLO, a tracker and a PD controller that keeps the target centred and at a set distance. We would build that baseline anyway, to compare against.
Data: first auto-label the datasets, then record our own Go2 runs following people. If neither works we skip fine-tuning and report why.
Target selection: we try Jev first, then GPT 6 Luna Decisions, then d3-edge 0.6B, which we could fine-tune on CrowdHuman or DanceTrack-style data.
Look up and sit: if the Kine2Go-trained MLP does not work, use the Go2 SDK's built-in pitch and sit commands.
-->

---
layout: hud
kicker: 'Team'
---

# Division of labor

<DivisionOfLabor />

<!--
Four workstreams that follow the pipeline: perception, voice and target selection, VLA adaptation, and Go2 behaviour. Everyone joins for integration, crowd tests and the demo. The owners shown are a proposed split.
-->
