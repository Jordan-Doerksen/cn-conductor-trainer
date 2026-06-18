# CN Conductor Trainer — Technical Handoff & Context

> **Purpose of this file.** Drop this into a fresh Claude/agent session (or read it yourself)
> to get enough context to make *meaningful* changes to this project without re-deriving
> everything. It captures the goal, the architecture, the exact data models, the
> safety-critical content rules, the environment gotchas, and the roadmap.
> Last updated: 2026-06-17.

---

## 0. TL;DR for a new session

- **What:** a browser-based training game that helps Jordan re-qualify as a **CN (Canadian National) freight conductor** — learn **CROR** (Canadian Rail Operating Rules), read switch lists, and switch/build trains in **his real yard (CN Grande Prairie)**.
- **How it's built:** a hub page (`index.html`) linking to several **single-file, vanilla-JS HTML modules**. No build step, no framework, no dependencies. Double-click to run; also published via **GitHub Pages**.
- **Where it lives:** the canonical git repo is on **Jordan's own machine**; he pushes to GitHub himself. The agent edits files in the connected folder but must **not** run git here (see §2).
- **Golden rules before you touch anything:** read **§2 (Hard rules)**, and **this file is the source of truth — check against it before you start and update it before you finish (§2.4).** The ones that bite: (a) **environment matters** — on **native Windows (the normal setup)** git + the `Read`/`Write`/`Edit` tools are the working flow; only on the *legacy Linux sandbox mount* are git-ops and `sed -i`/rename-edits forbidden (they corrupt files) — see §2.1; (b) never guess CROR content — scan the rulebook and cite, and flag CN-operating-practice phrasing as *not a rule*; (c) when you change a module/reference/decision, reflect it in §3/§5/§8/§12 the same session.

---

## 1. The "why" (don't lose this)

Jordan got very close to qualifying as a conductor before and wants to rebuild confidence through repetition. His words: *"i want to try and be a conductor again real bad… i don't want to be scared anymore."* This is the emotional spine of the project. Design implication: **build confidence through low-stakes reps, ramp difficulty gently, never condescend, and keep the content trustworthy** (a wrong "rule" in a trainer is worse than no trainer). Reframe user struggle as a UX gap, not their failing — when the first tester said he was "too dumb to work it out," the correct response was to add onboarding, not to assume the tester was the problem.

---

## 2. HARD RULES — read before editing (these are the "memory rules")

These are persisted in the agent's long-term memory. Reproduced here so they travel with the repo.

### 2.1 Environment / tooling — know which environment you're in
The rules here depend on where the repo is mounted:
- **Native Windows (current / normal setup):** the repo lives on Jordan's real machine at `C:\projects\CN Conductor Trainer` on NTFS. Here the `Read`/`Write`/`Edit` tools **and git are safe and are the working flow** — commits and pushes to `origin/main` happen straight from this folder (batches 1–5 of the signal encoding were committed + pushed from here). The hostile-mount cautions below do **not** apply.
- **Cowork / Linux sandbox mount (legacy):** the project folder was once mounted into a Linux sandbox over a 9p/virtio-style mount. **Rename-based writes on that mount corrupt files**, and the mount's directory cache goes stale. If you're ever back in that setup, the bullets below apply.

When in the legacy sandbox mount:

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

### 2.4 Source of truth — read this file first, and keep it current (Jordan's standing rule)
**This `HANDOFF.md` is the project's source of truth.** It's a two-way obligation, every session:
- **Before you touch anything:** read this file and check your plan against it (architecture, data models, the GP yard reference, the CROR-vs-GOI distinctions, the safety rules). Don't re-derive what's already written here, and don't silently contradict it.
- **After any meaningful change:** update this file in the same session so it never drifts from reality — new/changed modules (§3, §5), new reference sources (§8), status + roadmap (§12), and any rule or decision a future session would otherwise re-litigate. Bump the "Last updated" date. **A stale source of truth is worse than none.**
- Same discipline for the other living docs: **`reference/signal-questions.md`** (the signal-aspect Q&A + encoding log) and the project memory. If a doc disagrees with the code, the **code is reality** — fix the doc.

---

## 3. Repository layout

```
CN Conductor Trainer/              (repo root; entry = index.html)
├─ index.html                      Hub / module launcher (served at site root by GitHub Pages)
├─ modules/
│  ├─ GP Switching Puzzle.html     The dig-out / build puzzle (PULL/SPOT engine, BFS pars)
│  ├─ GP Yard Board.html           Interactive yard map (clickable tracks + switch dots)
│  ├─ GP Radio Steps.html          Guided "Back to a Joint" radio call sequence
│  ├─ GP Switching Drill.html      "Classify the cut" quiz
│  ├─ Signal Reading.html          CROR signal aspects (405–440) + SVG lamp renderer + aspect study
│  └─ CROR Quiz.html               Data-driven CROR rules quiz (text rules, by component, escalating tiers)
├─ assets/
│  ├─ GP Yard (cleaned).png        Denoised black-on-white trace of the yard photo
│  └─ GP Yard (tracks).png         Cropped strip used as the puzzle reference image
├─ reference/
│  ├─ Jan_2025_Canadian_rail_operating_rules_EN.pdf   (GITIGNORED — do not commit)
│  ├─ 2015_04_10-Robitaille-presentation-with-intro-slide.pdf  (GITIGNORED — CN CRTC seminar, see notes below)
│  ├─ crtc-fundamentals-notes.md   Committed digest of the Robitaille deck (terminology + rule map)
│  ├─ CROR Signal Rules.pdf        (GITIGNORED — Hoevet third-party study aid; has the "A"-plate error, don't trust blindly)
│  ├─ signal-questions.md          SME question sheet + LIVE aspect-encoding log (count, what's done, what's next)
│  ├─ signal-aspects/              Drop zone for signal-aspect photos — README tracked, raw images gitignored
│  ├─ signal_screenshots/          (GITIGNORED) CROR Verbal Quiz app shots — the trusted aspect-encoding source
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

### 5.1 ~~Yard Conductor Trainer~~ → FOLDED into the CROR Quiz (§5.7)
The old standalone "Rule Drills" module was **retired 2026-06-16** (file deleted; recoverable from git history). All of its rule-cited questions were folded into the CROR Quiz's components (switching, switches/derails, securement, radio/hand-signals, signals); the two pure "job knowledge" switch-list questions were dropped as non-rule content. **One quiz now, not two.**

### 5.2 GP Switching Puzzle (the core sim) — most-developed module
A plan-then-run yard puzzle. The player composes a list of moves, hits **Run plan**, and watches the engine work the yard; the module grades correctness, clean-up, and efficiency vs. **par**. Full data model in **§6**.
- **Batches** (`GROUPS`): Dig-outs (`#`), Tight room (`C`, capacity-limited), Build outbound (`O`, exact order), Leave it clean (`T`, must empty scratch + lead), **Full yard** (`F`, advanced — scratch tracks START occupied, some cuts arrive already tied down via the optional `startSecured` field; the skill is *reading* the yard: which cut is tied/has room/is fouling), plus **Mixed** (random). Nav = batch tabs + numbered chips (cleaner than a row of pills). The Full-yard tier is the hardest, placed last on purpose — a later progression after kick/spot are learned on emptier tracks. Each F puzzle's `par`/`parsol` and `jpar`/`ksol` were recomputed with the secured-cut solver and re-verified by the harness (every parsol wins, every ksol wins at its jpar, kick legality holds, existing puzzles unchanged). **Securement model (updated 2026-06-17, Jordan's SME call): spotting 2+ cars AUTO-SECURES the cut (hand brake before separation) → kickable; pulling releases it. The standalone TIE-DOWN move was removed — securing isn't a separate trip. Solver re-run; jpars unchanged (securing is 0 joints), ksols dropped the redundant tiedown steps.**
- **"How to play" modal** (added 2026-06-16): explains throat/engine, PULL vs SPOT in real switching language, the controls, and a worked example (puzzle #1). Auto-opens on first visit only (`localStorage` key `gpsp_seen`, wrapped in try/catch so it degrades if storage is blocked); reopen via the **How to play** header button; backdrop click closes. **Verified** via stubbed-DOM run (all assertions pass).
- **Solution-watch affordances (cleaned up 2026-06-17):** ONE always-visible **▶ Watch solution** button (`#solveBtn`, next to Run/Reset) loads the puzzle's `parsol` into the plan and runs it narrated — the no-guess "show me the clean line." The **only other** solution affordance is the post-win **▷ watch the N-joint line** (`#wtBtn`, in `jointsLine`), which runs `ksol` (the joint-optimal *kicking* line) and appears **only when the player's joints > jpar** — i.e. when there's a genuinely tighter line to teach. The old redundant **"Show a tight (par) solution"** text button (`parBtn`/`#parsol`) was **removed** — it just printed the same par line the new button now animates. Net: par line = the button; kick line = the contextual upsell; no duplicate "here's the answer" layers. Verified live (button wins on par-optimal #4 and joint-suboptimal #5; kick-line watch correctly hidden at jpar, shown below it).
- **Step scrubber (added 2026-06-17):** any plan's steps in *Your plan* are **clickable** — clicking step *i* (`showStep(i)`) stops any animation, rebuilds the yard from `freshState()` by replaying moves 0..i, freezes it, highlights that step, and holds the move's narration + `Step i / N · joints` in `#log` so it's readable (the animation used to zoom past it). A **‹ Prev / Next ›** stepper (`#stepper`, `curStep` pointer, `prevStep`/`nextStep`) walks it; **Prev past step 1 → a "start" view** (curStep=-1) showing the untouched yard. Works for the player's own plan, Watch-solution (parsol), and the kick-line watch (ksol). Animation timers now tracked in `animTimer` and cleared on scrub/run/load so a click reliably interrupts a running animation. Verified live: frozen state at step 3 of #5 byte-matches a re-simulation of moves 0..2; Next/Prev/start all correct; the × delete still works without triggering a scrub (`event.stopPropagation`).

### 5.2.1 GP Switching Sim — animated conductor game (NEW 2026-06-17)
A **separate, standalone game** (`modules/GP Switching Sim.html`) — Jordan's "pro version" of the switching concept, built fresh, not edits to §5.2. **Pure browser, no new toolchain** — single-file vanilla JS, the one upgrade being a **`<canvas>` game loop** (`requestAnimationFrame`) for true geometry + motion (DOM rows couldn't animate cars along rails). Decided with Jordan: it's **command-and-watch**, NOT direct-drive — *"a sim for the conductor, not the engineer."* You **line the switches** and **call the move**; the engineer "works it" (animated). 
- **Geometry:** a real ladder yard — LEAD + diagonal ladder + 6 body tracks (AS71 nearest the lead … AS76 at the top). Each track's route = `[A,B,junction_i,bodyEnd]` polyline; positions are arclength `s` along it; `polyAt` interpolates x/y/angle. Throat-anchored car slots.
- **The new mechanic vs §5.2 = switch lining + routing.** To reach track *i*: line *its* switch **reverse** and every switch nearer the lead **normal** (`lineCheck`). Call a move into an unlined road and the engineer refuses with a teaching message (which switch is wrong + why). Switches toggle by tapping the canvas or the chips (touch-friendly — he studies on his phone).
- **Reused the §5.2 rules brain** (the validated logic, not the renderer): `place`/`pull`/`spot`/`kick`/`reSecure`, securing (spot 2+ auto-ties → kickable; pull releases), kick legality (kickable track + secured 2+ standing cut), joints (pull +1, spot onto occupied +1, kick 0). `KICKABLE={AS72,AS73}`.
- **5 puzzles** (`PUZZLES`), each `{start, goal:{track,cars}, startSecured?}`. Win = goal set on the goal track + engine clear + no untied 2+ cut (Rule 112).
- **Verified live (preview + canvas screenshots):** static render correct; lining gate refuses a NORM-switch pull; full PULL→SPOT→SPOT solve of #1 → "Job done… 1 joint"; KICK path on #5 (pull 4, kick C-D onto the tied X-Y cut, spot A-B) → win; console clean throughout.
- **Per-track car offset (2026-06-17):** each track carries `state.offset[T]` = how far down the track the cut sits (px from clearance to slot-0 centre). Spotting onto an **empty** track picks a **random depth** (`randOffset`, 92–155px) so it feels real and leaves room; spotting/kicking onto an **occupied** track grows the cut **toward the throat** (`offset -= n*CARLEN`, `nextOffset`) so the **standing cut doesn't move** and kicked cars have room to roll in (no crowding); pulling recedes it (`offset += n*CARLEN`) so remaining cars stay put. Render + animation targets (`sBody`, doSpot/doKick) all read the offset. Verified deterministically (cut stays put on kick; spot depth random; pulled cars render to the **right/yard side** of the parked engine — earlier they wrongly drew on the left). KICK coast animation also rewritten cleaner.
- **Render model (refactored 2026-06-17):** every car is a **persistent object drawn each frame straight from the model** — no hide/overlay. `draw()` renders (1) cars standing on tracks at their slots, (2) the engine + coupled cars along `engRoute` at arclength `engS` (cars trail on the yard side), (3) a `flying` cut (only mid-kick). A move just animates `engS` (and, for a kick, `flying.s0`) and applies the model change at the couple/cut-away instant. This fixed the bug where cars blinked out while the engine moved them. Car-conservation verified (track ∪ engine ∪ flying is invariant through PULL/SPOT/KICK).
- **Verify note:** the preview throttles `requestAnimationFrame` and **screenshotting mid-animation can hang the renderer** — screenshot only idle/after-completion states, or apply moves directly + `draw()` for end-state shots.
- **Lead clearance (2026-06-17):** the drill track holds the engine + `LEAD_CAP` cars clear of the switch (`FOUL=dLead-8`; LEAD_CAP≈4). A PULL that would leave more than that on the engine is refused ("not enough room to clear the switch — spot some off first"). `parkS()` is now load-aware: the engine parks further toward the bumper the more it holds, so the whole cut sits **clear of the points** (furthest car right-edge at FOUL), never fouling up the ladder.
- **Body-track fouling (2026-06-17, Rule 114):** each track has a **clearance point** `FZONE` (42px from the switch), drawn as a small amber tick. A spot/kick onto an occupied track grows the cut toward the throat; if the throat-most car would sit closer than the clearance point (`new offset < MINOFF = FZONE-CLEAR+CARLEN/2 = 47`) the move is **refused** ("that'd foul the switch"). Cars now park **deeper** (`randOffset` 185–225px, was 92–155) so cuts sit well clear and there's room to kick onto them. A UI note + a how-to paragraph explain it; Rule **114** added to the how-to rule list. **Verified:** 640 replays (40 random rolls × 16) — every optimal line still wins at exactly par and never trips the foul rule; positive test confirms over-stuffing is refused.
- **Restore switches to normal (2026-06-17, Rule 104 — checked against the Jan 2025 CROR PDF):** **104(h)** — main-track switches must be left lined and locked in **normal** position, *except* the **104(i)** cases (occupied by equipment, attended, GBO/clearance/special instruction). **104(c)** — must not turn a switch with a car/engine between the points and the **fouling point** (validates our clearance rule). **104(b)** — examine points/target after turning. Honest nuance: the "always normal" requirement is specifically *main-track*; the sim's ladder is *yard (non-main)*, where you line per move, so we reflect it as **good practice, not a hard gate**: a how-to paragraph cites 104(h)/(c); a win leaves a gentle "restore your switches to normal (Rule 104)" nudge **if any switch is left reversed**; and **Watch optimal restores all switches to normal when it finishes** (models the practice). Not enforced on win (yard, non-main).
- **Occupied yard (2026-06-17):** each puzzle seeds **stray standing cuts of 2+ cars** (never lone cars) on unused tracks (mostly AS75/AS76) so the yard feels lived-in; `freshState` auto-secures **any** start cut of 2+ (Rule 112 — `startSecured` field now redundant). Strays are scenery/obstacles, placed off the solution tracks so puzzles stay solvable within LEAD_CAP.
- **Puzzle set (2026-06-17): 16 puzzles, 4 tiers** (`tier` field, grouped in `#pz`): **Basics** (move/dig/spot, two-track), **Digging & room** (buried pairs, 6-car cut that won't fit so you stage it, clear-an-occupied-track-and-rebuild, one-from-each-of-three), **Kicking** (first kick, make-a-kick-track, kick-to-two, long kick), **Yard work** (Reverse it + Build in order = **ordered goals**; Crowded yard; The transfer capstone). Goals can be `ordered:true` → win uses `eqArr` (exact slot order) instead of `setEq`. **Every puzzle machine-verified solvable** by a throwaway BFS solver that mirrors the engine (lining ignored since always satisfiable; respects LEAD_CAP, kick legality, securing, and the "all 2+ cuts tied" win rule) — solver deleted after use, rebuild from the model if needed. Strays/scenery use lowercase labels, interactive cars uppercase.
- **Par joints + Watch-optimal (2026-06-17):** every puzzle carries `par` (min couplings) and `opt` (the joint-optimal move line), computed offline by a Dijkstra-over-joints solver (cost: pull 1, spot-onto-occupied 1, spot-empty/kick 0; tiebreak fewest moves) — throwaway, rebuild from the model. Par shows in the stat line + the win verdict ("Par — N joints!" vs "N joints — par is M, kick onto a tied cut"). A **▶ Watch optimal** button (`#optBtn`) resets the puzzle and **auto-plays `opt` in slow motion** (`slowmo=1.7`× on `durFor`): for each move it auto-lines the switches (`autoLine`), shows it in the move UI, then runs the real `doPull/doSpot/doKick`, chained via a `moveDone` hook in `after()`; `auto`/`moveDone` globals, guards in `workIt`/`throwSwitch`/chip-clicks/Reset so manual input can't collide. **All 16 opt lines replay through the LIVE engine to a win at exactly par** (verified), and the auto-player completes end-to-end (verified on #1). Kicking is still never *forced* (spotting always works) — par is what rewards it. Linked from the hub as a **"Game"** card (orange LED, `.tag.g`).

### 5.3 GP Yard Board (interactive map)
Clickable track map built **on top of** `assets/GP Yard (cleaned).png` with switch positions overlaid (mapped from Jordan's marked photo via OpenCV homography). This file was once NUL-corrupted by `sed -i` and restored via `tr -d '\000'` — see §2.1.

### 5.4 GP Radio Steps (radio call trainer)
A fully hand-held "Back to a Joint" sequence with a visual of where the conductor stands. Exact wording is in **§9** — it was corrected by Jordan several times and is real-world-accurate, so don't "improve" it without checking with him. Module currently ends at "going for hand brakes"; later layers are queued (see §12).

### 5.5 GP Switching Drill ("classify the cut")
Quick quiz on reading a cut/switch list. Slated for more volume + layering kick/shove compliance.

### 5.6 Signal Reading (CROR aspects 405–440)
Two-section trainer (Basics / Full); three modes per section — **Verbiage drills**, **The rules** (reference), and **Read the aspect** — plus the **"Read the signal" capstone** (`startRead`/`readQuestion`, added 2026): `drawSignal` renders a random hardware variant and you name the indication (recognition, not recall; reuses the verbiage quiz engine via `state.mode==='read'`). Its pool is **`var VARIANTS`** — ~102 forms across all 38 charted indications (masts/dwarfs/multi-head/DV-R-L plates/flash/stagger), derived from the screenshot reads. **We render our own SVG — the copyrighted screenshots are never shipped.** NOTE: the 136 quiz items collapse to ~100 DISTINCT signals (24 exact repeats deduped + 9 trap/conflict/uncertain shots correctly excluded), so this is the complete distinct set — **don't "pad to 136," it only re-renders identical signals.** The pool was audited consistent with the canonical `ASPECTS` (incl. single-green Clear 405, single-red Stop 439, 407 = Y/G). The **words** (`var IND`, all 42 rules' names + meanings) are sourced exact and verified. The **aspects** — what each signal physically *looks like* — are encoded in **`var ASPECTS`** and drawn by **`drawSignal(spec)`**, an SVG lamp renderer that handles N heads, mast/dwarf, DV/R/L plates, red stagger, and **per-head flashing via a `'f'` suffix** on a head code (`"Yf"` = flashing yellow over a *steady* red; colours are `G/R/Y/L/D` top→bottom).
- **Encoding source of truth = the CROR Verbal Quiz app screenshots** in `reference/signal_screenshots/` (Jordan passed his 2025 conductor test with this app; the official rulebook corroborates it). Each shot fully specifies one aspect (rule# + name + lamps + flash caption + plate + mast/dwarf). The official CN rulebook app is a bonus cross-check.
- **Policy (Jordan's call): one card per *indication*, variants noted in the card's tip** — not one card per hardware combo. (A future separate Quiz module will use all ~136 individual variant cards; the study set stays grouped.)
- **`ASPECTS_DRAFT` stays `true`** until the full set is encoded. Progress, the resolved/open questions, and the "what's next" pointer live in **`reference/signal-questions.md`** (the live log — update it every batch).
- **Verify** new aspects with the stubbed-DOM harness (asserts lamp count + flash-vs-steady per head against the real `drawSignal`); screenshots of the live page are blocked by the infinite SMIL flash animations, so trust the renderer's counts.

### 5.7 CROR Quiz (text-rules quiz, the "use the CROR as law" track)
A **data-driven** quiz over the **text** of the CROR (everything except the signal lamp pictures, which live in §5.6). Unlike the signals work, the CROR text *is* the authoritative source, so questions are pulled straight from the rulebook and cite the rule — no screenshots, no SME loop.
- **Structured by the CROR's own components** (`var COMPONENTS`, 16): Definitions · Speeds · Crew & responsibilities · Switches/derails/track · Securing (112) · Coupling/kicking/shoving (113–115) · Radio & hand signals (12, 120–123.2) · Signals/flags/plates (26, 405–440, 564) · OCS (301–308) · ABS (505–515) · CTC (560–566) · Interlocking (601–614) · **General Rules & Time (A/C, 1–2)** · **Grade crossings (103.1)** · **Bulletins/clearances/forms (131–153)** · **Track-work protection (40–42)**. Plus a "Full CROR exam" mixed mode. The retired Rule Drills questions are folded in here. Easy to add components.
- **Escalating tiers** every run: Tier 1 Recall → Tier 2 Application → Tier 3 Distinctions (`sampleEscalating` orders tiers ascending). Each answer shows a **rule citation + the CROR text**; wrong answers collect into an end-of-run review.
- **Definitions are verbatim CROR** (`var DEFS`, 63 terms) and generate MC at runtime in both directions (term→def, def→term); Tier-3 terms pull *confusable* distractors (e.g., the CONTROLLED Block/Signal/Location/Point cluster). Operating-rule questions are hand-authored in `var BANK`, each cited to its rule.
- **Source extraction:** parsed from `reference/Jan_2025_Canadian_rail_operating_rules_EN.pdf` with `pymupdf` (working JSON in `%TEMP%\sigshots\`: `cror_defs.json`, `cror_rules.json` 224 rules, `cror_bodies.json`). **Reuse this to expand the bank** — the rulebook text has no lamp/aspect descriptions but is complete for the rules themselves.
- **Verify** with the stubbed-DOM harness (`%TEMP%\sigshots\verify_quiz.js`): every static + generated question has 4 distinct options, a valid answer index, a citation, and a tier; every component builds a non-empty escalating session.

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

**Signal aspects (Rules 405–440) — see §5.6; running count + open questions live in `reference/signal-questions.md`.** Confirmed framework, banked from the quiz app + rulebook + Robitaille: **only DV, R, and L plates exist — there is NO "A"/absolute plate in Canada** (US practice; the quiz app draws an A-plate only as a trap, captioned "not used"). **A DV plate upgrades the indication** (strip it and Diverging downgrades to Slow). The **proceed lamp's position encodes speed** (green high = Clear, green low = Slow); **flashing green = Limited, steady green = Medium/Clear**; **two staggered reds, no plate = non-controlled** (437 Stop & Proceed) vs a single/absolute red the RTC holds (439 Stop). Straight-route ("Clear to ___") aspects mean **blocks-of-warning-ahead, not speed past the signal** — speed governs only the turnout/diverging family.

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

**Done:** all **six** modules built and live on GitHub Pages (incl. the **CROR Quiz**, §5.7 — data-driven text-rules quiz, **~325 questions (63 verbatim definitions + 262 cited operating questions)** across 16 components spanning definitions → every control system → bulletins/crossings/track-work. **Every component hits the 20-question target where the rules support it (11 of 16); the naturally-thin sets are capped deep-as-useful, not padded** — ABS 13, Speeds 16, Tow 12, Roles 10, Crossings 10. Escalating tiers (recall → application → **scenario**, the tier-3 "invented situation, CROR-defensible answer" format); every question cites its rule; verified by the stubbed-DOM harness (`%TEMP%\sigshots\verify_quiz.js`). The old Rule Drills module was folded into it and retired); folder restructured (hub/modules/assets/reference) with relative links; outbound + mixed + **Full-yard** puzzle batches; **KICKING shipped** as a puzzle move (PULL/SPOT/KICK; **spotting 2+ auto-secures the cut → kickable, no separate tie-down move** (2026-06-17); two scores moves+joints, BFS pars recomputed + verified) — ⚠ the secured-cut model (kick only onto a secured 2+-car standing cut; SME Kourtney) **supersedes §7.1's simpler KICKABLE list**; expand §7.1 from the code the next time the puzzle is touched; radio module through "going for hand brakes"; How-to-play onboarding on the Switching Puzzle (verified); **Signal Reading module built** (§5.6) with the SVG lamp renderer + aspect-study mode.

**In progress:**
- **Signal-aspect encoding** (`Signal Reading.html`, §5.6): **38 of 42 indications encoded** from all 136 CROR Verbal Quiz shots (full pass done 2026-06-16, verified via stubbed-DOM harness + live in-browser render, 0 errors). `ASPECTS_DRAFT` stays `true` until the last 4 + the 407 question close. Full status + the system rules + open conflicts are in `reference/signal-questions.md` (the FULL PASS COMPLETE block).

**Open tasks (in priority order):**
1. **Get the last 4 aspects, then flip `ASPECTS_DRAFT=false`.** The quiz app didn't contain **410** (Clear to Restricting), **418** (Limited to Medium), **433A** (Diverging to Medium), or **440** (Direction Indicator — needs an arrow fixture in the renderer), and the **CROR rule text has no lamp/aspect descriptions** (verified against the Jan 2025 PDF — only indications), so these can't be sourced yet. Get them from the official rulebook's aspect diagrams or a CTC conductor; do NOT guess. (407 resolved: yellow/green confirmed by Jordan; 406 aligned to the same family.)
2. **Expand the "classify the cut" quiz** volume; layer kick/shove **compliance** into the puzzle (e.g., flag an illegal kick into AS76 or a south-yard track).
3. **Radio module later layers:** the move/departure after hand brakes and the closing **"out"** (121c); a player-supplies-the-distances mode; half-distance stop (123.2 iii); hand signals (123.1).
4. **Separate Quiz module** — a polished CROR Verbal Quiz built on all ~136 individual aspect-variant cards (the study set stays grouped, one-per-indication).

**First-tester feedback that's driving this:** "Cool program!! I'm too dumb to understand how to work it out" + a description of the efficient dig-out (kick spare cars back to AS71 instead of making multiple joints). Takeaway: onboarding gap (now addressed) + the kicking mechanic is the realism the experienced user expects.

---

## 13. Working style Jordan expects

- **Be concise and direct** (his stated preference). Don't pad.
- **Never guess on rules.** Scan, cite, and flag GOI-vs-CROR. He will (rightly) push back hard on invented rule content.
- **Match real-world operating language.** He has corrected radio wording, the coupling call, kicking geometry, and tail-is-cars-not-loco. When in doubt about an operational detail, ask him rather than infer.
- **Respect the environment rules in §2** — corrupting his files or his git repo erodes trust fast.
- **Shipped copy is HR-facing (2026-06-16).** Jordan shows the live trainer to recruiters. Keep user-visible text professional: label the yard **"Grande Prairie"**, not "your yard / your map / your photos"; no dev/in-progress notes, version tags, personal names, or "draft/not signed off" language in shipped files (visible text *or* source comments). **Keep the source/resource disclaimers** (e.g., aspects reproduced from the CROR Verbal Quiz study app; "training aid only — the official CROR governs"). Instructional second person ("you're the conductor", "your block is clear") is fine.
