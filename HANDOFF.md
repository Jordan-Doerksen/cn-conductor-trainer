# CN Conductor Trainer — Technical Handoff & Context

> **Purpose of this file.** Drop this into a fresh Claude/agent session (or read it yourself)
> to get enough context to make *meaningful* changes to this project without re-deriving
> everything. It captures the goal, the architecture, the exact data models, the
> safety-critical content rules, the environment gotchas, and the roadmap.
> Last updated: 2026-06-16.

---

## 0. TL;DR for a new session

- **What:** a browser-based training game that helps Jordan re-qualify as a **CN (Canadian National) freight conductor** — learn **CROR** (Canadian Rail Operating Rules), read switch lists, and switch/build trains in **his real yard (CN Grande Prairie)**.
- **How it's built:** a hub page (`index.html`) linking to several **single-file, vanilla-JS HTML modules**. No build step, no framework, no dependencies. Double-click to run; also published via **GitHub Pages**.
- **Where it lives:** the canonical git repo is on **Jordan's own machine**; he pushes to GitHub himself. The agent edits files in the connected folder but must **not** run git here (see §2).
- **Golden rules before you touch anything:** read **§2 (Hard rules)**. Two of them have bitten us repeatedly: (a) never `git init`/git-op or `sed -i`/rename-edit inside the mounted folder — it corrupts files; (b) never guess CROR content — scan the rulebook and cite, and flag CN-operating-practice phrasing as *not a rule*.

---

## 1. The "why" (don't lose this)

Jordan got very close to qualifying as a conductor before and wants to rebuild confidence through repetition. His words: *"i want to try and be a conductor again real bad… i don't want to be scared anymore."* This is the emotional spine of the project. Design implication: **build confidence through low-stakes reps, ramp difficulty gently, never condescend, and keep the content trustworthy** (a wrong "rule" in a trainer is worse than no trainer). Reframe user struggle as a UX gap, not their failing — when the first tester said he was "too dumb to work it out," the correct response was to add onboarding, not to assume the tester was the problem.

---

## 2. HARD RULES — read before editing (these are the "memory rules")

These are persisted in the agent's long-term memory. Reproduced here so they travel with the repo.

### 2.1 Environment / tooling (the sandbox mount is hostile)
The project folder is mounted into a Linux sandbox over a 9p/virtio-style mount. **Rename-based writes on that mount corrupt files**, and the mount's directory cache goes stale.

- **NEVER `git init` or run git operations inside the mounted folder.** It produces a broken `.git/config` ("bad config line 1", reproducible). The canonical repo lives on Jordan's real Windows machine; **he pushes from there.** Do not touch git here. Do not handle his GitHub token/credentials.
- **NEVER use `sed -i` (or any in-place/rename edit) on mounted files.** A `sed -i` once left **54 NUL bytes** in `GP Yard Board.html`. The **`Edit` tool also uses a rename-write and corrupted a file on the `outputs` mount** during this very session — so prefer full-file methods.
- **Safe write methods (confirmed working):** full-file **truncate-writes** — the `Write` tool *for host project paths* (`C:\projects\...`), bash heredoc `cat > file <<'EOF'`, or python `open(p,'w').write(...)`. These do not rename.
- **To recover a NUL-corrupted file:** `tr -d '\000' < f > clean && cat clean > f`, then verify with `node --check` / a structure grep.
- **The bash sandbox cannot reliably read the project subtree this session** (e.g. `modules/` reads as empty, `wc` on a file `find` just listed says "No such file"). **Treat the `Read`/`Write`/`Edit` file tools (host path) as the source of truth.** For executable verification, copy the snippet to the **`outputs`** mount (which *is* reachable as the bash cwd) and run it there.
- **Verify on the host path, not the stale mount.** There are leftover renamed folders in the mount (`CN Conductor Training`, `CN RAIL Switching`) from past renames — ignore those; the live one is **`CN Conductor Trainer`**.

### 2.2 Content / safety (this is safety-critical training material)
- **NEVER guess or infer CROR content.** Jordan's instruction, verbatim: *"please never make guesses or infer, take the time to answer questions by cror rules scanning."* Scan the rulebook PDF, quote/cite the rule number, and if something is **CN operating practice (GOI)** rather than a CROR rule, **say so explicitly** (e.g. "set and center" is GOI, not CROR; nearest CROR rule is 109).
- **Do NOT republish the official CROR rulebook PDF.** It is gitignored via `reference/*.pdf`. Keep it local; never commit/redistribute it.
- **Don't hand-draw the yard from eye.** Three hand-drawn SVG attempts got the topology wrong and frustrated Jordan ("this is so wrong it hurts"). The yard geometry must come from the **processed photo trace** (see §6.3), not interpretation.

### 2.3 Verification discipline
Every non-trivial change to a module gets checked before it's called done: `node --check` on the extracted `<script>`, a **stubbed-DOM run-through** (mock `document`/`localStorage`/`setTimeout`, then exercise the new code), and for the puzzle, **re-run the BFS solver / re-validate every `parsol`**. For SVG/image work, render with `cairosvg` and eyeball it. When the project mount is unreadable, do these on the `outputs` mount.

---

## 3. Repository layout

```
CN Conductor Trainer/              (repo root; entry = index.html)
├─ index.html                      Hub / module launcher (served at site root by GitHub Pages)
├─ modules/
│  ├─ Yard Conductor Trainer.html  CROR rule drills (~33 Q across 4 domains)
│  ├─ GP Switching Puzzle.html     The dig-out / build puzzle (PULL/SPOT engine, BFS pars)
│  ├─ GP Yard Board.html           Interactive yard map (clickable tracks + switch dots)
│  ├─ GP Radio Steps.html          Guided "Back to a Joint" radio call sequence
│  └─ GP Switching Drill.html      "Classify the cut" quiz
├─ assets/
│  ├─ GP Yard (cleaned).png        Denoised black-on-white trace of the yard photo
│  └─ GP Yard (tracks).png         Cropped strip used as the puzzle reference image
├─ reference/
│  ├─ Jan_2025_Canadian_rail_operating_rules_EN.pdf   (GITIGNORED — do not commit)
│  ├─ 2015_04_10-Robitaille-presentation-with-intro-slide.pdf  (GITIGNORED — CN CRTC seminar, see notes below)
│  ├─ crtc-fundamentals-notes.md   Committed digest of the Robitaille deck (terminology + rule map)
│  ├─ signal-questions.md          SME question sheet + live answer log for the Signal Reading aspects
│  ├─ gp-yard-large.jpg            Source photo the trace was built from
│  └─ Game Plan.md
├─ README.md
├─ PUBLISH.md                      Git + GitHub Pages instructions (Jordan pushes from his PC)
├─ .gitignore                      contains `reference/*.pdf`
├─ .gitattributes
└─ HANDOFF.md                      (this file)
```

**Linking convention (relative paths — keep them intact):**
- Hub → module: `modules/GP%20Switching%20Puzzle.html` (spaces are `%20`).
- Module → hub (back-link): `../index.html`.
- Module → image: `../assets/GP%20Yard%20(tracks).png`.

The entry file was deliberately renamed to **`index.html`** so GitHub Pages serves it at the site root. Don't rename it back.

---

## 4. Architecture & design choices

- **Single-file modules, vanilla JS.** Each module is one self-contained `.html` (inline `<style>` + `<script>`, no imports). Rationale: runs by double-click with zero install, trivially portable, easy to host statically, and a non-developer can open any one in isolation. **Keep modules separate — do not merge into one mega-file.**
- **Hub-and-spoke UI.** `index.html` is a card grid of module launchers; each module has a back-link to the hub. New modules = new card in the hub + new file in `modules/`.
- **Dark, high-contrast theme** via CSS variables (`--bg/--card/--line/--ink/--muted/--yellow/--blue/--green/--red/--orange/--grey`). Reuse these tokens for visual consistency across modules.
- **Pedagogy: concept first, layers later.** Early puzzles teach the *mechanic*; rules/compliance/efficiency get layered in as the learner is ready. Don't front-load every rule.
- **Progressive disclosure / onboarding.** Modules that confused the first tester get an explicit "How to play" (auto-opens once via guarded `localStorage`, reopen button, click-outside to close). See §5.2.
- **Trust the source.** Anything rule-related is tied to a CROR citation or explicitly flagged as operating practice.

---

## 5. The modules

### 5.1 Yard Conductor Trainer (CROR rule drills)
~33 multiple-choice questions spanning the four domains (switch lists, CROR recall, switching moves, radio/signals). Straight Q&A drill; the volume is the value. Future: expand the bank, add explanations citing rule numbers.

### 5.2 GP Switching Puzzle (the core sim) — most-developed module
A plan-then-run yard puzzle. The player composes a list of moves, hits **Run plan**, and watches the engine work the yard; the module grades correctness, clean-up, and efficiency vs. **par**. Full data model in **§6**.
- **Batches** (`GROUPS`): Dig-outs (`#`), Tight room (`C`, capacity-limited), Build outbound (`O`, exact order), Leave it clean (`T`, must empty scratch + lead), **Full yard** (`F`, advanced — scratch tracks START occupied, some cuts arrive already tied down via the optional `startSecured` field; the skill is *reading* the yard: which cut is tied/has room/is fouling), plus **Mixed** (random). Nav = batch tabs + numbered chips (cleaner than a row of pills). The Full-yard tier is the hardest, placed last on purpose — it's a later progression after kick/spot/tie-down are learned on emptier tracks. Each F puzzle's `par`/`parsol` and `jpar`/`ksol` were recomputed with the secured-cut solver and re-verified by the stubbed-DOM harness (every parsol wins, every ksol wins at its jpar, kick/tie-down legality holds, existing puzzles unchanged).
- **"How to play" modal** (added 2026-06-16): explains throat/engine, PULL vs SPOT in real switching language, the controls, and a worked example (puzzle #1). Auto-opens on first visit only (`localStorage` key `gpsp_seen`, wrapped in try/catch so it degrades if storage is blocked); reopen via the **How to play** header button; backdrop click closes. **Verified** via stubbed-DOM run (all assertions pass).

### 5.3 GP Yard Board (interactive map)
Clickable track map built **on top of** `assets/GP Yard (cleaned).png` with switch positions overlaid (mapped from Jordan's marked photo via OpenCV homography). This file was once NUL-corrupted by `sed -i` and restored via `tr -d '\000'` — see §2.1.

### 5.4 GP Radio Steps (radio call trainer)
A fully hand-held "Back to a Joint" sequence with a visual of where the conductor stands. Exact wording is in **§9** — it was corrected by Jordan several times and is real-world-accurate, so don't "improve" it without checking with him. Module currently ends at "going for hand brakes"; later layers are queued (see §12).

### 5.5 GP Switching Drill ("classify the cut")
Quick quiz on reading a cut/switch list. Slated for more volume + layering kick/shove compliance.

---

## 6. The Switching Puzzle engine (data model + algorithms)

This is the part most likely to need edits (new puzzles, the kicking feature). Here's everything needed to extend it safely.

### 6.1 State & the two moves
State is `{ tracks: {TRACK: [cars...]}, engine: [cars...] }`. **Tracks store cars throat-first** (index 0 = nearest the throat = the left/`⟵` end shown in the UI). The locomotive 🚂 sits at the head of the engine array; the engine's **working end is the tail** of the array.

```js
// PULL n from T: take the n cars nearest the throat onto the engine tail.
function pull(s,T,n){
  var t=s.tracks[T];
  if(t===undefined) return 'unknown';
  if(n<1) return 'pull >=1';
  if(n>t.length) return 'only '+t.length+' on '+T;
  s.engine = s.engine.concat(t.slice(0,n));   // append nearest-throat cars to engine tail
  s.tracks[T] = t.slice(n);
  return null;
}
// SPOT n to T: shove the n cars at the engine's working end into T (at its throat end), cut off.
function spot(s,T,n){
  var t=s.tracks[T];
  if(t===undefined) return 'unknown';
  if(n<1) return 'spot >=1';
  if(n>s.engine.length) return 'only '+s.engine.length+' on engine';
  var cap=P().cap;
  if(cap && cap[T]!==undefined && t.length+n>cap[T]) return T+' is full (holds '+cap[T]+')';
  var b=s.engine.slice(s.engine.length-n);     // last n on the engine
  s.engine=s.engine.slice(0,s.engine.length-n);
  s.tracks[T]=b.concat(t);                      // placed at the throat end of T
  return null;
}
```

**Mental model = a stack.** The *last* cars you PULL sit at the working end, so they're the *first* you can SPOT. That ordering constraint is the whole puzzle.

### 6.2 Puzzle object schema
`PUZZLES` is an array of objects. Title's **first character selects the batch** (`#`, `C`, `O`, `T`).

```js
{
  type?: "outbound" | "digout",  // omitted => basic dig-out
  tidy?: true,                   // if set, finish requires every scratch track AND the engine empty
  title: "#1 — Warm-up",         // 1st char = batch code
  order: "Set out B to AS73, engine clear.",   // shown in the yellow work-order bar
  inbound: "AS71",               // BLUE track (cut starts here)
  tracks: ["AS71","AS72","AS73"],// tracks shown, in display order
  start:  { AS71:["A","B","C"], AS72:[], AS73:[] }, // initial positions (throat-first)
  cap?:   { AS71:99, AS72:2, AS73:9 },  // per-track capacity; >=90 renders as "unlimited"
  goal:   "AS73",                // GOLD track (destination)
  setouts:["B"],                 // cars that must end up on goal (set membership; order ignored)
  target?:["A","B","C"],         // OUTBOUND only: exact ordered consist required on goal
  par: 3,                        // minimum move count (verified by BFS, see 6.4)
  parsol: [["pull","AS71",2],["spot","AS73",1],["spot","AS72",1]] // one optimal line
}
```

### 6.3 Win condition (mirror this if you add a move type)
```
scratch  = tracks where t != goal and t != inbound
tidyOK   = every scratch track empty
clear    = engine empty
setdone  = (type==='outbound') ? eqArr(goal, target)   // exact order
                               : setEq(goal, setouts)  // set membership
WIN = setdone && clear && (!tidy || tidyOK)
```
`eqArr` = element-wise equality; `setEq` = same multiset. Helpers already exist in the file.

### 6.4 Par / BFS solver (kept OUT of the shipped HTML)
Par values are computed offline by a **Node BFS** over the state space (moves = all valid PULL/SPOT of every length from/to every track, deduped by serialized state) to find the minimum move count, then one optimal path is stored as `parsol`. When you add puzzles or a new move (kicking), **re-run the solver and update both `par` and `parsol`**; otherwise the "wasted moves" feedback lies to the learner. The solver script is a throwaway in scratch, not committed — re-create it from this description if needed.

### 6.5 Reference strip rendering (the little yard map under the puzzle)
SVG bands are drawn over `assets/GP Yard (tracks).png`. Coordinates come from `TXY[track] = [x, y, width]` in the *original* image space; the SVG is offset by the crop origin `OX=130, OY=470` and uses `viewBox="0 0 1170 220"`. Switch dots come from `SW = [[x,y],...]`. Band color encodes role: **gold** = goal, **blue** = inbound, **grey** = scratch; **orange** dots = switches. If you re-crop the image, update `OX/OY` and the `viewBox`.

```js
// Track geometry (original-image px): [x, y, width]
TXY = {AS65:[340,492,1006],AS66:[997,520,191],AS67:[815,539,358],AS72:[484,542,188],
       AS68:[807,559,255],AS73:[471,560,218],AS69:[790,576,264],AS74:[457,578,244],
       AS70:[783,595,168],AS75:[447,596,262],AS71:[602,613,352],AS76:[188,614,368],
       AS77:[466,636,543],AS78:[484,656,505]};
OX=130; OY=470; // crop offset for the reference strip
```

---

## 7. Grande Prairie (GP) yard reference

- **14 tracks: AS65–AS78.** `AS65 = STK` (the through track / Grande Cache Sub).
- **North yard:** AS72–AS78. **South yard:** AS65–AS71.
- **Subdivisions through GP:** Grande Prairie Sub (north to **Rycroft Jct**, ~M49.4), Dawson Creek Sub (west to **Dawson Creek**), Grande Cache Sub (M230.7–232.9, to **Poskar/Poelzer**), Edson Sub (to **Swan Landing**).
- **Throat switches/derails:** AS81–AS93, AS99, by the CN Yard Office.
- **Source diagram:** Sept-2020 operating-practices sheet (not to scale); the digital trace is `assets/GP Yard (cleaned).png`, built from `reference/gp-yard-large.jpg`.

### 7.1 KICKING rules at GP (drives the §12 feature)
Kicking is allowed **only from the NORTH end** (there's a grade there) and **only into tracks AS71, AS72, AS73, AS74, AS75, AS77, AS78.** NOT AS76 (it's the north-end track) and NOT AS65–AS70 (those must be shoved/coupled). The north-end grade also matters for **securement** (Rule 112 hand-brake counts).

```js
KICKABLE = ['AS71','AS72','AS73','AS74','AS75','AS77','AS78'];
```

---

## 8. CROR rule index used in the game

Verify wording against `reference/Jan_2025_Canadian_rail_operating_rules_EN.pdf` before quoting. Quick map of what we reference:

| Rule(s) | Topic |
|---|---|
| 104–115 | Switches, coupling/uncoupling, kicking / running switch / gravity drop, fouling, shoving |
| 109 | Nearest CROR rule to "set and center" (which itself is **CN GOI, not CROR**) |
| 112 | Securement / hand-brake table (grade-dependent) |
| 113.0(i), 113.2 | Couple + **stretch to verify** the joint |
| 113.2(c) | Air charged before releasing hand brakes |
| 113.4, 113.5 | Kicking restrictions |
| 115 | Point protection |
| 120(b) | Switching-by-radio exception (routine switching calls don't each carry "over") |
| 121 | Radio positive ID: initial call commences with railway initials of the party being called (121a); initiator ends the initial call "**over**" (121b); each party ends its final transmission "**out**" (121c). NOTE: 121(a) is positive ID only — the engineer's *repeat-back* of counts is **123(c)**, not 121(a). |
| 122, 123, 123.2 | 122 = brevity; **123(c) = safety-affecting radio instructions must be repeated back to the sender** (this governs the engineer repeating each car/distance count — modules cite this correctly); switching by radio = 123.2; 123.2(i) direction off front of loco, 123.2(iii) = half-distance stop, 123.2(v) doubt = stop, 123.2(vi) = 50 ft/car length |
| 123.1 | Hand signals (radio module future layer) |
| 134 | Engine number designation |
| R12 | Hand signals |
| R26 | Blue flag |
| R105 | Non-main track |

**Always distinguish CROR rules from CN GOI (operating instructions).** "Set and center" is the canonical example of GOI phrasing that is *not* a numbered CROR rule.

**Control-system framework source (added 2026-06-16):** `reference/2015_04_10-Robitaille-presentation-with-intro-slide.pdf` (GITIGNORED — local only, like the rulebook) is a 2015 CN seminar, *Canadian Rail Traffic Control Fundamentals* (Sean Robitaille, CN Transportation Engineer, U. of Illinois). Best for the **operating language** and how the control methods fit together; digested in `reference/crtc-fundamentals-notes.md`. Rule numbers it anchors: OCS **301–315** · ABS **505–515** · CTC **560–578** · Interlocking **601–620** · pass a controlled signal at stop **564** · Cautionary Limits **94** · non-main track **105** · Special Control System **351–353** · signal aspects **405–440** (speed-signal system; **L/DV/R** markers — confirms there's no "A"). **It's 2015 — verify time-sensitive claims** ("ABS only on CP", "no PTC mandate in Canada") against current sources before repeating.

---

## 9. Radio call sequence (Jordan-verified wording — do not "improve" without asking)

Engine ID is **"CN ####" said digit-by-digit** ("C-N, two-two-zero-four"), never "GP engine." The coupling move call is **"back up X cars to a joint"** ("joint" = a coupling; direction is relative to the **front of the controlling locomotive**) — **not** "shove ahead to couple." The **tail is CARS, not the loco.**

The guided "Back to a Joint" flow in `GP Radio Steps.html`:

1. **Positive ID** → ends "over".
2. Conductor: **"I'm on the ground and in the clear, I can see your tail — back ten cars to a joint."** → ends "over". (Leads with point-protection + visibility before the back-up + distance.)
3. **Car counts:** "five cars, five, CN 2204." The **engineer repeats each count.** **Drop the engine number at/below two cars.** (Don't double the info — it's count, short count, ID; not "five cars five CN 22-04, five cars…")
4. **Transition to feet:** "last car (one)" is the last *car* count before distance takes over → "half a car, twenty feet, ten feet, five feet."
5. **"That'll do — stop and stretch."** ONE combined call (113.0(i)/113.2(a)): brakes on + stop, the drift completes the joint, then stretch to verify.
6. **"Good stretch, set and center."** ("Set and center" = **CN GOI, not CROR**; nearest rule 109. Wait for engineer confirmation before going between equipment.)
7. **"Hoses coupled, air's cut in."**
8. **Engineer calls "air to the tail."** Conductor **repeats it back** + **"going for hand brakes."** (113.2(c) — air must be charged before releasing hand brakes.)

Module **ends** at step 8. Initial transmissions end "over"; the conversation's final transmission ends "out" (121c) — that closing "out" lands with the *departure* layer (queued, §12).

---

## 10. Build & verification toolchain

- **Image → clean trace (OpenCV):** `ImageOps.exif_transpose` to orient, crop to the sheet, `adaptiveThreshold` to isolate linework, despeckle by connected-component area → `GP Yard (cleaned).png`. Switch markings detected in **HSV**, then mapped onto the board with **ORB feature homography**. SVG previews rendered to PNG with **`cairosvg`** for eyeballing.
- **Puzzle correctness:** Node **BFS** solver computes/verifies `par` + `parsol`.
- **JS sanity:** `node --check` on the extracted `<script>`; a **stubbed-DOM** harness (mock `document`/`localStorage`/`setTimeout`) to run module init + new handlers without a browser.
- **Run executable checks on the `outputs` mount** when the project subtree isn't readable from bash (see §2.1). Example just used: a stub firing click events confirmed the new How-to modal opens once, closes on "Got it"/backdrop, reopens via button.

---

## 11. Product decisions already made (so you don't re-litigate them)

- **Domains to cover (all four):** (1) reading switch lists & building trains in order, (2) CROR recall, (3) switching moves — shoves/point protection, kicking, coupling, securement, (4) radio & hand signals.
- **Format:** *both* quick rule drills *and* a visual yard simulator with a "rules referee" that flags violations.
- **Scope:** start tight on the yard/switching core, expand to full CROR over time.
- **Ramping:** the first puzzle block is concept-only; rules/habits (clean yard, no cars on the lead, kicking compliance) are added as later layers.
- **Naming/nav:** batch tabs + numbered chips, not an endless row of pill buttons.

---

## 12. Status & roadmap

**Done:** all five modules built and live on GitHub Pages; folder restructured (hub/modules/assets/reference) with relative links; outbound + mixed puzzle batches; radio module through "going for hand brakes"; **How-to-play onboarding added to the Switching Puzzle (2026-06-16, verified).**

**Open tasks (in priority order):**
1. **Add KICKING as a puzzle move** + update the BFS solver so par credits the kicking line (kicking avoids multiple joints — exactly what the first tester described). Honor §7.1: north-end only, into AS71/72/73/74/75/77/78. Update the How-to to mention kicking once it's in.
2. **Expand the "classify the cut" quiz** volume; layer kick/shove **compliance** into the puzzle (e.g., flag an illegal kick into AS76 or a south-yard track).
3. **Radio module later layers:** the move/departure after hand brakes and the closing **"out"** (121c); a player-supplies-the-distances mode; half-distance stop (123.2 iii); hand signals (123.1).

**First-tester feedback that's driving this:** "Cool program!! I'm too dumb to understand how to work it out" + a description of the efficient dig-out (kick spare cars back to AS71 instead of making multiple joints). Takeaway: onboarding gap (now addressed) + the kicking mechanic is the realism the experienced user expects.

---

## 13. Working style Jordan expects

- **Be concise and direct** (his stated preference). Don't pad.
- **Never guess on rules.** Scan, cite, and flag GOI-vs-CROR. He will (rightly) push back hard on invented rule content.
- **Match real-world operating language.** He has corrected radio wording, the coupling call, kicking geometry, and tail-is-cars-not-loco. When in doubt about an operational detail, ask him rather than infer.
- **Respect the environment rules in §2** — corrupting his files or his git repo erodes trust fast.
