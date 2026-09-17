# Wide To Close Cinematic Scaffolding — Technical Operational Dispatcher

**Framework Author**: William Fitzpatrick (*Writer Science*)  
**Source Lecture**: [How to write so well that readers stop scrolling](https://www.youtube.com/watch?v=IqPLeyTahLs)  
**Parent Collection**: [Master Collection](../../kirby-fitzpatrick-writers-collection/SKILL.md) | [Global Help](../../../kirby-help/SKILL.md)  

---

## 1. Cognitive Foundation: The Cheshire Cat and the Missing Wide Shot

**The concept.** Film has a grammar for a reason. A scene opens on an establishing shot — the city, the room, the crowd — and only then cuts to a hand. The audience is never shown a hand before they know whose hand, in what room, in what city, in what decade. The close-up works *because* the wide shot is still holding it in place. Withhold the wide shot and you get the **Cheshire Cat**: a grin hovering in midair with no cat behind it. That is exactly what a bare `file:line` or a pasted stack trace is in prose — a detail pointing at a body the reader has never been shown.

**Why readers scroll away.** Working memory holds **three to four items**, not thirty — this is the budget the lecture builds the whole camera discipline around. Every unanchored identifier (a host name, a service, a version, a line number) consumes a slot. A wide shot is not decoration: it is *compression*. Once the reader holds "prod-eu-west-1, order-service v4.7.2, one of four pods, last six hours" as a single chunk, the close-up costs one slot instead of four. Skip it and the reader spends all four slots trying to build the frame you refused to give — then re-reads, hunts, and finally stops scrolling. Scroll-away is not impatience; it is cognitive collapse.

**Camera continuity.** Once you cut in, the wide shot does not vanish — it stays live in the reader's mind and is what makes the close-up meaningful. Break continuity by jumping back out without a re-establishing shot (line → cluster → line) and the frame collapses; every fact already filed must be retroactively re-filed. The anti-pattern has a name: **close-to-wide**. Stack trace first, blame the function, then "oh, by the way, this is production, eu-west-1, four pods." The reader's job becomes archaeology.

```
BEFORE — CLOSE-TO-WIDE (DISORIENTING)          AFTER — WIDE-TO-CLOSE (CINEMATIC)
                                                 ┌───────────────────────────────────┐
  OrderService.java:412  ◄── detail first        │  WIDE   environment boundary      │
    NullPointerException      (whose? where?)    │  prod-eu-west-1 · k8s · v4.7.2    │
        │                                        │  1 of 4 pods · 6h · blast: 1 pod  │
        │  ...prose continues...                 ├───────────────────────────────────┤
        │                                        │  MEDIUM component / owner         │
  "our cluster in eu-west-1"  ◄── frame last     │  CartCache — only growing struct  │
  "the cache never evicts"                       ├───────────────────────────────────┤
                                                 │  CLOSE  failing line              │
  reader must retro-file every fact ──► re-read  │  OrderService.java:412            │
  ──► hunt ──► stop scrolling                    └───────────────────────────────────┘
                                                 every detail lands inside a frame
```

**The zoom ladder — three shots, by containment.** The mechanism in one line, as the collection states it: **environment boundary first → component → mutating variable.**

| Shot | Technical carrier | What it must state | Budget |
|---|---|---|---|
| **WIDE** | Environment boundary: cluster / region / host fleet / runtime | artifact + version, deploy time, observation window, blast radius (N of M) | 3–4 items |
| **MEDIUM** | Component: service, process, cache, queue, writer | who owns the state, what mutated, cardinality or rate | 3–4 items |
| **CLOSE** | The failing frame | `file:line`, the mutating call, the observable consequence | 1–2 items |

**The six signals of close-up-first writing.** Recognize the defect before you fix it:

| Signal | What it looks like |
|---|---|
| **Bare locator** | `OrderService.java:412` with no path back to the frame that gives it meaning. |
| **Prepositional patch** | "…in production, on eu-west-1, across four pods" appended *after* the claim, so the frame arrives as an afterthought. |
| **Frame hopping** | A line number and a cluster name traded mid-paragraph, forcing the reader to re-aim the camera every sentence. |
| **Renamed anchor** | The same pod called "the pod", then "the instance", then "it" — continuity lost, the reader now believes there are three things. |
| **Cheshire excerpt** | A stack trace or log block dropped in with no frame line; evidence gesturing at a body never shown. |
| **Unbounded frame** | No boundary at all — "the database is slow", "the service leaks". Which database, whose replica, at what version, since when? |

**Why this matters specifically in software engineering.** Four structural reasons make this a defect class, not a taste preference:

1. **Raw evidence arrives close-up by nature.** A stack trace, an OOMKill line, a metric spike — all are close-ups of a grin. Prose written in arrival order *transcribes* the disorientation instead of resolving it.
2. **The reader's frame *is* the debugging frame.** A responder who cannot place the detail in the topology forms no hypotheses and falls back to grep. Camera work is not polish here; it is the difference between reasoning and searching.
3. **Frame absence manufactures false root causes.** A leak diagnosis that never names the wide shot (which pod, which version, since which deploy) cannot distinguish an unbounded map from a bounded cache growing toward steady state. Half of all "memory leak" arguments are two engineers holding different frames.
4. **Continuity is what makes an artifact resumable.** An incident write-up read at 03:05 by a different engineer, in a different frame, must not require re-deriving the camera move.

**The mental model to hold.** *You are the camera operator, not the caption writer.* The reader can hold three or four things; spend the first of them on the frame that will later hold all the rest. For related framing discipline see [`../../kirby-fitzpatrick-joint-attention-pairing/SKILL.md`](../../kirby-fitzpatrick-joint-attention-pairing/SKILL.md) (shared, verifiable anchors) and [`../../kirby-fitzpatrick-zero-meta-discourse/SKILL.md`](../../kirby-fitzpatrick-zero-meta-discourse/SKILL.md) (no throat-clearing where a frame belongs).

---

## 2. Core Transformation Protocols

### Rule 1 — Open wide: the establishing sentence names the environment boundary before any identifier

The first sentence of a diagnosis, walkthrough, review, or RFC carries the frame — cluster/region, artifact + version, time window, blast radius. Nothing is named before it is placed.

```
ESTABLISHING SHOT — required first sentence
──────────────────────────────────────────────────────────────
environment:  prod-eu-west-1 (k8s 1.29)
artifact:     order-service @ v4.7.2 (deployed 14:02Z)
window:       first observed 14:11Z · 6h and counting
blast:        1 of 4 pods · no customer-facing errors yet
──────────────────────────────────────────────────────────────
```

*Before:* "`CartCache` is never evicted; the pod OOMKills."
*After:* "For six hours, one of four `order-service` pods in `prod-eu-west-1` has climbed 40 MB/h in RSS; the growth tracks `CartCache`." The detail now arrives *inside* a frame instead of pointing at one.

### Rule 2 — Zoom by containment, never by association

Each cut-in must be physically nested in the previous frame: cluster ⊃ node ⊃ pod ⊃ process ⊃ thread ⊃ function ⊃ line. If you cannot state the containment path, you are not zooming — you are changing subject, and the reader must re-establish the camera alone.

### Rule 3 — Cap every shot at three or four items (the working-memory budget)

A wide shot with nine nouns is a wide shot the reader deletes. Everything over budget moves into an **explicit off-frame list**: "Not in frame: the deploy pipeline, the sidecar, the client fleet — the growth is independent of all three." Explicit exclusion is framing, not a digression; it tells the reader where *not* to look.

### Rule 4 — Hold camera continuity: one anchor, re-stated, never re-invented

Choose the canonical anchor string once (`prod-eu-west-1/order-service@v4.7.2`) and repeat it **verbatim**. Never degrade it to "the cluster", "the service", "it". When a later section needs the wide shot, re-establish it in the same words — a re-establishing shot, not a re-narration.

### Rule 5 — Every close-up carries its ladder

No bare `file:line`, no bare log line, no bare stack frame in prose. Use the walkable locator:

```
wide › medium › close
prod-eu-west-1 › order-service@v4.7.2 › CartCache.put() › OrderService.java:412
```

A reader must be able to walk from any detail back up to the frame in a single line.

### Rule 6 — One pass down the ladder per artifact; no yoyo

Order the artifact WIDE → MEDIUM → CLOSE. Do not cut back to the cluster mid-line-analysis, and do not restart from a close-up. A second investigation opens a second, explicitly labeled ladder rather than interleaving two.

### Rule 7 — Put the failing token in the subject position of the close-up sentence

"`OrderService.java:412` dereferences the `items` array that `CartCache.put()` left undefined" — not "there's a null-pointer issue with the cache somewhere in the order code." The anchor is the subject; the frame is the modifier.

### Rule 8 — Close by panning out with the frame intact

After the close-up, restate the consequence at the level the reader must act on — alert, budget, owner — using the same anchor. The last sentence of a diagnosis belongs to the environment, not the line, because the *action* lives out there.

### Transformation table — anti-pattern vs. clean replacement

| # | Anti-pattern (close-up first / disembodied) | Defect | Clean replacement (wide → close) |
|---|---|---|---|
| 1 | `NullPointerException at OrderService.java:412` | Cheshire grin, no cat | "In `prod-eu-west-1`, `order-service@v4.7.2` pod #3 throws at `OrderService.java:412`…" |
| 2 | "Our service has a memory leak." | unbounded frame | "1 of 4 `order-service` pods in `prod-eu-west-1` grows 40 MB/h RSS since the 14:02 deploy; `CartCache` is the only growing structure." |
| 3 | "…in production, on eu-west-1, across four pods." | prepositional patch | move the frame into sentence one |
| 4 | "It's slow." / "the DB is timing out." | unnamed actor | name replica, version, query, observed latency |
| 5 | 60-frame stack trace pasted with no header | Cheshire excerpt | one frame line, the ladder, then the 3–6 frames that matter, with the rest named as excluded |
| 6 | "`CartCache` → cart cache → the cache → it" | renamed anchor | one canonical anchor string, repeated verbatim |
| 7 | "Obviously the retry loop doubles the writes." | skipped medium shot | name the worker, the loop, the dedupe window before the judgment |
| 8 | Line-level finding then cluster-level claim in one paragraph | frame hopping | one ladder per paragraph; re-establish before the wide claim |
| 9 | "Since the cache is unbounded, memory grows." | frame asserted as premise, order inverted | environment → component → the unbounded key enumeration → consequence |
| 10 | "The fix is to add a TTL." | close-up decision with no frame | state the frame the TTL is sized against (growth rate, cardinality, deploy cadence), then the TTL |

### Worked micro-example — memory-leak diagnosis (2D vs. 3D)

```
CLOSE-UP FIRST (2D)
  "The CartCache map is never evicted, so the pod OOMKills. Fix: add a TTL."
  Defects: no cluster, no version, no pod count, no numbers. The reader cannot tell a leak
  from a cache reaching steady state, cannot size a TTL, and cannot tell whether the fault
  is the map or the deploy that changed its key cardinality. The fix ships; the leak returns
  next release, because the frame was never in the artifact.

WIDE → MEDIUM → CLOSE (3D)
  "In prod-eu-west-1, 1 of 4 order-service@v4.7.2 pods has grown 40 MB/h RSS for 6h while
   its three peers stay flat                              (WIDE)
   The growth tracks CartCache — the only structure whose cardinality rose after the 14:02
   deploy                                                  (MEDIUM)
   The key now includes the raw session id, so every signed-out session leaves a permanent
   entry at 412                                         (CLOSE)
   Falsifier: if the flat peers had shown the same growth, the fault would be upstream of
   the cache — the frame is what makes that check possible."
```

Same fix. Different artifact: bounded by evidence, sized against a measured rate, attributable to a deploy, and falsifiable.

---

## 3. Engineering Application Scenarios

### Scenario A — Code Reviews (crash-fix and leak-fix reviews)

**Purpose:** the diff shows only the close-up by construction. The reviewer's first job is to reconstruct the wide shot the diff cannot carry.

Protocol:

1. **Rebuild the frame before reading code.** Read the PR body, deploy context, and incident link. If the PR does not name environment boundary, artifact version, and window, the first review comment is *that question* — not a naming nit.
2. **Issue every finding as a laddered finding.** Format: `wide › medium › close` + the mutation + the consequence. A finding that cannot name its frame is labeled a hypothesis, not a defect.
3. **Hunt frame-free fixes.** A guard at the close-up (`if (items != null)`) whose medium shot says the array is *left* undefined by the caller means the invariant was fixed at the wrong level: the crash disappears and the leak (or the next caller) inherits it. Ask the frame question: **"which component owns this invariant?"**
4. **Demand laddered evidence, not pasted evidence.** Crash and leak evidence gets tagged — `[CRASH]`, `[LEAK]` — with one frame line, the 3–6 relevant frames, and the excluded ones explicitly named.
5. **Reject close-up-only confidence.** "It passed locally" is a claim about a different frame (one process, one request, one replica). Ask for the wide shot: which topology, how many replicas, how long, what cardinality.

```
[LEAK]  Wide:    prod-eu-west-1 · 1/4 pods · order-service@v4.7.2
        Medium:  CartCache — only growing structure; key cardinality +3.1M/h
        Close:   OrderService.java:412 / CartCache.put() retains session-scoped keys
        Frame question: does the eviction rule belong to the cache (medium) or the call
        site (close)? Patching the call site leaves the cache unbounded for every other
        caller.
```

### Scenario B — PR Descriptions (and incident write-ups)

**Purpose:** the PR body is the artifact the next engineer reads with no frame of their own. A close-first description forces every reader to re-derive the camera move under time pressure.

Mandatory template, in this order:

```
## Wide shot — environment boundary
- Where:     <cluster / region / host fleet>
- What:      <service @ version, deploy time>
- Window:    <when first observed; how long; still true?>
- Blast:     <N of M instances; customer impact yes/no>
- Excluded:  <what is NOT in frame, so the reader stops looking there>

## Medium shot — component
- Owner of the state: <module / cache / queue / goroutine / writer>

## Close-up — the failing line
- `file:line` — <mutation> — <observable consequence>

## Verification (a shot in its own right)
- <command / test / drill that exercised the failure *in the stated frame*>
- Falsifier: <the observation that would have proven this diagnosis wrong>
```

Rules for the author:

- **Order is not cosmetic.** WIDE → MEDIUM → CLOSE is what makes the fix auditable by someone who has never seen the system.
- **The "Excluded" line is mandatory.** A leak investigation with no exclusions is an investigation that has narrowed nothing.
- **Numbers belong to the frame that produced them.** "40 MB/h on one pod" is not "the service leaks 40 MB/h"; say which measurement, in which frame, generalizes to what.
- **Re-establish after any pasted evidence.** A stack trace resets the camera; restate the frame in the same words before continuing.
- **Never open with the diff.** The diff is the close-up; the frame is the meaning.

### Scenario C — Architecture RFCs / ADRs

**Purpose:** a design document is a wide shot with a long exposure. ADRs habitually open on the mechanism ("we will add a TTL", "we will add a replica") — precisely how a decision loses its frame five years later, when the boundary it silently assumed no longer holds.

Add a mandatory frame section before the decision:

```
## Frame (write this before the Decision)
- System boundary:  cluster / region / tenancy model; what is *inside* this design
- Runtime boundary: pod/instance count; stateful vs stateless; scale ceiling
- Version horizon:  deploy cadence and upgrade path this decision assumes
- Out of frame:     subsystems this ADR deliberately does not govern
## Components (medium shots)
- per component: state owned → lifetime → scale limit → failure chapter
## Decision (close-up)
- <the one line that changes>
## Alternatives (each answered in the same frame: same boundary, same horizon)
## Consequences (frame-first: what changes at the environment level)
## Validation Plan (which framed assumption is drilled, and what would falsify it)
```

Rules for the architect:

- **A rejection without a frame is a rejection on taste.** "Alternative B caches per pod in front of a per-pod store; at an HPA ceiling of 8 the effective memory budget multiplies, so the design breaks at scale, not at load" is a framed rejection that survives re-litigation.
- **Put non-guarantees in the frame section.** Explicit "this holds only within the boundary below" lines are the most valuable content in an ADR.
- **Treat vendor and AI-drafted prose as close-up-suspect.** Marketing copy presents the grin (the feature's benefit) with the cat (topology, consistency model, per-region limit, retry semantics) unstated. Ask for the frame, in numbers.
- **Write the frame for the 03:00 reader.** The Frame section is what a responder reads first during a leak or a crash; if it does not answer "where does this run and at what scale", the RFC is unusable in the moment it matters most.
- **Pull only the frame the reader needs** — pair with [`../../kirby-fitzpatrick-just-in-time-context-optimizer/SKILL.md`](../../kirby-fitzpatrick-just-in-time-context-optimizer/SKILL.md) so zoom depth is a deliberate choice, not an accident of how much context was pasted in.

---

## 4. Verification Checklist

- [ ] **The wide shot precedes the first identifier.** In every artifact I produced, sentence one names the environment boundary (cluster/region/host fleet), the artifact and its version, and the observation window — before any `file:line`, log line, metric, or variable name appears. If a bare identifier is reachable earlier, the artifact is close-up-first and fails.
- [ ] **Every close-up carries a walkable ladder.** Each detail is expressible in one line as `wide › medium › close` (e.g. `prod-eu-west-1 › order-service@v4.7.2 › CartCache.put() › OrderService.java:412`). No Cheshire excerpt remains: no stack trace, log block, or metric without the frame that gives it a body.
- [ ] **Frame budget and exclusions are explicit.** Each shot carries at most 3–4 items, and everything removed to hold that budget is named in an "out of frame / not in frame" list. I did not achieve brevity by silently dropping the boundary.
- [ ] **Continuity held — no yoyo, no renamed anchor.** One pass down the ladder; zoom jumps are containment-nested; the canonical anchor string is repeated verbatim at every re-establishing shot and never degraded to "it", "the service", or "the cluster". Any cut back to a wide frame is labeled as a re-establishing shot.
- [ ] **The claim survives frame attribution.** For every root cause, growth rate, or latency figure I stated, I can name the frame it was measured in and the frame it does *not* generalize to ("40 MB/h on 1 of 4 pods" ≠ "the service leaks 40 MB/h") — and I have stated the falsifier: the observation inside that frame that would have proven the diagnosis wrong. A diagnosis that cannot be falsified in its own frame is a close-up of a grin.