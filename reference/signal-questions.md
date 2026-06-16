# Signal aspects — questions for a working conductor

**Purpose:** unblock the "Read the aspect" section of `modules/Signal Reading.html`. The
CROR *indications* (Rules 405–440, the words) are verified and solid. What's missing is
what each signal physically **looks like** — the lamps — so they can be drawn and signed off.

**Why this exists:** the only aspect source on hand is a third-party study chart
(`reference/CROR Signal Rules.pdf`, a Joseph Hoevet aid — "not an official reference") and
it has at least one confirmed error (it shows an **"A" / absolute plate, which does not
exist in Canada** — that's US practice; the CROR only names DV and R plates, Rules 403/573).
So aspects must be confirmed by an authoritative source or a working conductor before they
go in. Right now only 3 of 42 are encoded and the section is flagged `ASPECTS_DRAFT`.

**Who answers:** any current conductor. **CTC territory is ideal** — that's where the full
speed-signaling set (Limited / Medium / Diverging / Advance, plates, flashing) actually
appears. The questions don't change based on who answers.

---

## ⭐ The one ask (best case)

> Do you have CN's **official signal aspect chart** — the signal page in the GOI / special
> instructions / timetable? A clear **photo of that page** lets all 42 aspects be encoded
> correctly in one shot, from the authoritative source. If not, no problem — answer the
> questions below from what you actually see out there.

A photo of the real CN chart removes the correctness risk entirely. Everything below is the
fallback if the chart isn't handy.

---

## Answers captured (live)
Confirmed answers logged here as they arrive, so they don't get lost in the thread.

- **Framing correction (conductor, CTC territory):** the straight-route aspects are **not**
  "speed past the signal." They tell you **how many blocks ahead are clear before an
  occupancy** — how much warning you get. Speed only governs the turnout/diverging family.
- **Clear (405):** green on top — single green, *or* all-green — both read as Clear. *(Jordan, firsthand)*
- **Clear to Stop (411):** **yellow over red.** Block you're entering clear, the next occupied;
  prepare to stop at the next signal. *(Jordan, firsthand)* — **ENCODED**
- **Advance Clear to Stop (415):** **flashing yellow over red.** Your block + next clear, the
  third occupied; prepare to stop at the second signal. *(Jordan, firsthand)* — **ENCODED**
  (renderer now flashes a single head via a `'f'` suffix, e.g. `"Yf"` = flashing yellow over steady red).
- **Grammar:** green on top **+ a non-green lower head = a "Clear to [speed]"**; all-green = plain Clear. *(Jordan, firsthand)*

**Partially captured — modifier logic, but lamp stack NOT yet confirmed (do not encode until it is):**
A **DV plate** + **flashing** are the discriminators on the diverging / slow / restricting group —
> - **DV plate, flashing → Diverging to Stop (429)**
> - **DV plate, steady → Diverging (430)**
> - no plate, **flashing → Slow to Stop (435)**
> - no plate, **steady, just yellow → Restricting (436)**
>
> **Flashing (SME-confirmed):** Advance Clear to Stop (415, encoded) · the **Limited** family ·
> Diverging to Stop (429) · Slow to Stop (435). Slow to Stop flashes *without* a DV plate;
> Diverging to Stop flashes *with* one — the plate is the only difference between those two.
>
> Still need the actual lamp colours (top→bottom) to draw these — same format as 411/415:
> Diverging (430) = ___/___ +DV · Diverging to Stop (429) = ___/___ flashing +DV ·
> Slow to Stop (435) = ___/___ flashing · Restricting (436) = ___ (single yellow? R plate?) ·
> Limited to Clear (416) = ___/___ (flashing green?). 2-head or 3-head mast?

**Encoded in the module so far (5 of 42):** 405 Clear · 411 Clear to Stop · 415 Advance Clear
to Stop · 437 Stop & Proceed · 439 Stop. `ASPECTS_DRAFT` stays **true** until the set is complete.

**Still needed (next):**
- The **exact lower-head colour** for each "Clear to [speed]": Clear to Limited (406) /
  Medium (407) / Slow (409) / Diverging (408) / Restricting (410) — green over *what*?
- The **turnout/diverging family** (Limited / Medium / Diverging / Slow *to* ___) — first word is a real speed.
- **Restricting (436)**, **Diverging (430)**, **Direction Indicator (440)**, **Take/Leave (438)**.
- Confirm **Clear** on a 2-head mast: single green, or green-over-green?

---

## The question sheet (fallback)

**Paste this framing first:**

> I've got the CROR indications (Rules 405–440) word-for-word — that part's solid. What I
> need is what each signal physically LOOKS like, since you read CTC signals daily. Don't
> worry about rule numbers, just tell me the lamps. Where you're not 100% sure, say so —
> I'd rather have a gap than a wrong lamp.

### A — the system (this generates most of the 42)

> **Corrected by SME:** the straight-route aspects are **not** "speed past the signal." They
> tell you **how many blocks ahead are clear before an occupancy** — how much warning you get.
> Speed only governs the *turnout/diverging* family. So there are really two families:

1. **Hardware on your sub:** colour-light (separate R/Y/G lens per head), searchlight (one
   lens that changes colour), or position-light? Typical mast — **2 heads or 3?**

**A1 · Approach family ("Clear to ___") — block warning, normal speed:**
2. Walk the warning ladder, top→bottom lamps for each:
   - **Clear (405):** all clear ahead → single ___? (green?)
   - **Clear to Stop (411):** your block clear, next occupied → top lamp ___ (steady yellow?)
     over a ___ lower head?
   - **Advance Clear to Stop (415):** your block clear, next clear, third occupied →
     **flashing yellow on top** *(SME-confirmed)* over a ___ lower head?
   - For these — is it a **single head** showing the yellow, or **yellow over red** (2 heads)?
3. Same block-warning idea for **Clear to Limited / Medium / Slow / Diverging / Restricting** —
   what changes in the lamps vs Clear to Stop?

**A2 · Turnout/diverging family ("Limited / Medium / Diverging / Slow to ___") — speed through
the turnouts:**
4. Here the first word *is* a speed. What do these look like (top head colour, lower head,
   flashing) — e.g. **Medium to Clear (422)**, **Limited to Clear (416)**, **Slow to Clear
   (431)**, **Diverging (430)**? When does a **third (bottom) head** appear?
5. Which indications use a **flashing** lamp, and **which lamp** flashes? (Confirmed so far:
   flashing yellow top = Advance Clear to Stop. What else flashes?)

### B — the stand-alones (these break the grammar)
6. **Clear (405)** — single green, one head, right?
7. **Restricting (436)** — aspect? (flashing red / red-over-yellow / lunar white?) Does it
   carry an **R** plate on your sub?
8. **Stop and Proceed (437) vs Stop (439)** — describe exactly what your eyes use to tell an
   *absolute* Stop from a permissive (stop-then-proceed) one. Is it a **number plate
   present/absent**, the **two-red stagger**, or something else? **← the big one, say more here.**
9. **Diverging (430)** — aspect, and does it **always** carry a **DV** plate?
10. **Direction Indicator (440)** — the flashing arrow: separate fixture? colour?
    left / right / straight?
11. **Take or Leave Siding or Other Track (438)** — does GP / your territory even use it?
    If so, what's the aspect?

### C — plates & dwarfs
12. Plates you **actually** see out there: DV, R, anything else? And can you confirm you've
    **never seen an "A" (absolute) plate** — established there's no such plate in Canada,
    want a working conductor to back that.
13. **Dwarf (ground) signals** — which indications show as dwarfs, and do the colours differ
    from the mast version?

**Caveat to include:** some indications have more than one legal aspect — just give the one
you'd actually see on your territory, and note **which sub** that is.

---

## The 3 that must get nailed no matter what
Even a partial answer should cover these — they're what's blocking sign-off today:
1. **437 vs 439** — the physical tell for absolute vs permissive Stop.
2. **Plates** — confirm DV and R, and confirm **no "A" plate**.
3. **Flashing** — which indications flash, and which lamp.

---

## Appendix — how answers map to the encoding (for whoever encodes it)
Each aspect in `modules/Signal Reading.html` (`var ASPECTS`) is:

```js
{ r:"NNN", lvl:"basic|full",
  spec:{ heads:[ ... ],   // lamp colours TOP→BOTTOM: G green · R red · Y yellow · L lunar/white · D dark
         type:"mast|dwarf",
         stagger:true,     // optional — two heads offset L/R (the non-controlled tell)
         flash:true,       // optional — lamp(s) flash
         plaque:"DV|R" },  // optional — plate on the mast (NO "A" in Canada)
  tip:"one-line plain-language note" }
```

So from the answers: Q2/Q3/Q4 → the `heads` array · Q5 → `flash` · Q8 → `stagger` and/or
`plaque` · Q9/Q12 → `plaque` · Q13 → `type:"dwarf"`. Encode against the verified `IND`
table (the words) in the same file, then set `ASPECTS_DRAFT=false` only after a verify pass.
