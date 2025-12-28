# mk-openwakeword-tests

# Studio Voice Triggers Planning Doc

*A pnpm monorepo plan with Next.js UI + Node Slack/servers + Ink CLI + Python sidecar (openWakeWord) + event bus integration.*

---

## Goals

1. **Near-instant wake / command triggers** (always-on, local) using **openWakeWord**.
2. **Event-first architecture**: wake/intent detectors emit *facts* onto the bus; downstream services decide actions.
3. **Keep ML deps isolated** in a **Python sidecar**, while Node remains the studio “control plane.”
4. Support both:

   * **Live mode**: wake → immediate studio automation (lights, overlays, camera cuts, etc.)
   * **Refinement mode**: later runs can re-derive intent from transcript/audio if needed

---

## Monorepo shape

### Top-level folders

```
repo/
  apps/
    web/              # Next.js “stylish visuals”
    slack/            # Slack app (Node) + server-side studio control plane
    cli/              # Ink CLI (Node) for local ops/debug
  python/
    voice/            # openWakeWord sidecar + FastAPI tooling
    notebooks/        # R&D notebooks
    scripts/          # utilities, eval, dataset prep, etc.
  packages/
    bus/              # shared TS types + event schema helpers
    config/           # shared lint/tsconfig/eslint/prettier
    ui/               # optional shared UI components for Next.js
  infra/
    docker/           # optional
    docs/             # planning docs like this one
  pnpm-workspace.yaml
  package.json
```

---

## pnpm workspace conventions (high-level)

### Workspace grouping

* `apps/*` are runnable products
* `packages/*` are shared libraries
* `python/*` is not managed by pnpm, but lives alongside and shares schema contracts

### What “one command” should do

You want a **single dev entrypoint** that can:

* start event bus (or connect to an existing local bus)
* start Python wake server
* start Slack app
* start Next.js dashboard
* optionally start the Ink CLI in watch mode

---

## The event bus slice (what we’re adding)

### Why it belongs on the bus

Wake/keyword detection is a **Tier-0 sensor**: low latency, high signal, and it should remain decoupled from downstream logic.

### Event tiers (recommended mental model)

1. **Tier 0: Edge events** (wake/keyword detected)
2. **Tier 1: Control events** (open gate, start recording, change scene)
3. **Tier 2: Language events** (transcript segments, refined words, diarization)
4. **Tier 3: Agent events** (inferences, summaries, suggestions, QA checks)

---

## Bus implementation choice (pragmatic recommendation)

For your studio setup, pick one “always-on” local bus:

### Option A (simple, great for a studio LAN): **NATS**

* extremely good at low-latency pub/sub
* easy subject hierarchy
* good tooling, durable options if you want later

### Option B (already in many stacks): **Redis pub/sub**

* fine if you already have Redis
* simplest to adopt
* durability requires extra patterns

### Option C (durable logging): **Postgres outbox + listeners**

* best if you want “every event is persisted”
* more moving parts
* not the lowest latency

**Recommendation:** use **Redis pub/sub** if you already have it in the stack; otherwise **NATS** is a sweet spot for “studio event bus.”

(You can still persist selected events to Postgres for audit/debug later.)

---

## Canonical event schema

### Design rules

* Events are **facts**, not commands-with-side-effects
* Small payloads; put big blobs (audio) elsewhere and reference them
* Always include:

  * `type`
  * `source`
  * timestamp
  * stream identity
  * correlation IDs (session, utterance, etc.)

### Core event types for this feature

#### 1) Wake detected

* **Type:** `wake.detected`
* **Emitted by:** Python openWakeWord sidecar
* **Purpose:** “a hotword was heard”

Fields:

* `keyword` (string)
* `confidence` (0..1)
* `audio.stream_id`
* `audio.t_ms` (ms timestamp)
* `session_id?` (optional; can be created downstream)

#### 2) Command keyword detected (optional if you use only wake + transcript parsing)

* **Type:** `command.detected`
* **Emitted by:** Python sidecar (if configured for multiple phrases)
* **Purpose:** “heard one of the action words”

Fields:

* `command` (e.g. `"record" | "freeze" | "next"`)
* `confidence`
* `audio.stream_id`
* `audio.t_ms`

#### 3) Gate opened / closed

* **Type:** `asr.gate.opened` / `asr.gate.closed`
* **Emitted by:** Node control plane (Slack app server)
* **Purpose:** “begin expensive processing (Whisper) now” or “stop”

Fields:

* `gate_id`
* `mode` (e.g. `"command" | "dictation" | "conversation"`)
* `audio.stream_id`
* `t_ms`
* `reason` (e.g. `"wake.detected"`)

#### 4) Transcript segment (live)

* **Type:** `asr.segment`
* **Emitted by:** whichever ASR service you run (Python or Node)
* **Purpose:** incremental text

Fields:

* `segment_id`, `gate_id`
* `text`
* `t_start_ms`, `t_end_ms`
* `is_final` (boolean)

#### 5) Transcript words (refined)

* **Type:** `asr.words`
* **Emitted by:** refinement pass
* **Purpose:** word-level timestamps for search/UI

Fields:

* `words: [{ text, t_start_ms, t_end_ms, confidence? }]`

#### 6) Diarization turns

* **Type:** `diarization.turns`
* **Emitted by:** refinement pass
* **Purpose:** speaker segments

Fields:

* `turns: [{ speaker, t_start_ms, t_end_ms }]`

---

## Service responsibilities

### Python sidecar (`python/voice/`)

**Primary job:** listen to audio and emit *fast* events.

Recommended responsibilities:

* audio capture OR PCM receiver (choose one)
* openWakeWord inference
* debounce/cooldown
* emit `wake.detected` (+ optionally `command.detected`)
* health endpoint + basic metrics

Recommended non-responsibilities:

* do not call Slack
* do not control cameras
* do not decide what “wake” means in context

#### Two integration modes

**Mode 1: Python owns mic**

* simplest
* fastest to ship
* good if one machine owns the mic input

**Mode 2: Node owns audio; Python only infers**

* best if you already have an audio “bus” in Node
* enables multiple consumers from one capture stream (recording, meters, ASR, etc.)

Given your studio architecture, **Mode 2** is likely the “cleaner end state,” but **Mode 1** is the fastest way to validate.

---

### Node control plane (`apps/slack/`)

**Primary job:** subscribe to bus events and decide actions.

Responsibilities:

* subscribe to `wake.detected`
* maintain “session state” (gate open/close, cooldown rules, which room/mic is active)
* publish control events (`asr.gate.opened`, `asr.gate.closed`)
* trigger studio actions (camera cuts, overlays, recorders, etc.)
* optionally post notifications to Slack (but only as a consumer of events)

---

### Next.js dashboard (`apps/web/`)

**Primary job:** observability + operator UX.

Responsibilities:

* show live event stream (wake hits, gate status, segments)
* show “confidence meters” and false positive debugging
* show current mic/room routing status
* quick controls (manual gate open/close, toggle wake words, adjust thresholds)

This becomes your “stylish glass cockpit” for the studio.

---

### Ink CLI (`apps/cli/`)

**Primary job:** operator + developer tooling without a browser.

Responsibilities:

* tail bus events
* test fire events (“simulate wake”)
* show last N wake hits
* enable/disable wake words quickly
* run diagnostics (ping Python service, list configured keywords, etc.)

---

## Repo contracts (how TS and Python stay aligned)

### Shared contract package: `packages/bus/`

This package defines:

* event `type` union
* JSON schema (or Zod schema) for each event
* helpers for publishing/subscribing

Python should:

* either import generated JSON schema files
* or share a `schema/` directory that both TS and Python validate against

Goal: if you change an event payload, both sides fail fast.

---

## openWakeWord specifics (what you’re building)

### Wake configuration

* Define **a small list of wake phrases** (2–8 is a good start)
* Define optional **command phrases** (5–20)
* Add:

  * `debounce_ms` (e.g. 200ms)
  * `cooldown_ms` (e.g. 1500ms)
  * `min_confidence` per keyword (tune per room)

### False positive control

Plan to tune with:

* threshold per keyword
* room profiles (quiet office vs classroom vs studio floor)
* mic type (lav vs shotgun vs desk mic)
* a “push-to-talk fallback” (manual gate open) for high-stakes moments

### Debug logging

Log every detection with:

* timestamp
* keyword
* confidence
* mic stream id
* whether it was suppressed by cooldown/debounce

(You’ll thank yourself later.)

---

## How wake words interact with transcript context

A key pattern you want:

1. **Wake event** fires instantly (`wake.detected`)
2. Node opens a short “context window” (gate open)
3. Node may:

   * run ASR for the next 1–3 seconds to parse a command, OR
   * look back into the rolling transcript buffer for parameters

Examples:

* “Ok Studio… define entropy”

  * wake triggers gate; ASR captures 1–2 seconds; command parser extracts “define” + “entropy”
* “Ok Studio… mark that”

  * wake triggers immediate `marker.add` action referencing last 10 seconds of transcript timestamps

This keeps wake detection cheap, while still letting commands be rich.

---

## Milestones

### Milestone 1: Bus + wake → visible events

* Python sidecar emits `wake.detected`
* Slack app subscribes and logs
* Next.js dashboard displays event feed
* Ink CLI tails events

**Done when:** you can reliably trigger wake phrases and see them everywhere.

### Milestone 2: Gate controller

* Wake triggers `asr.gate.opened` for 3 seconds
* A simple ASR runner emits `asr.segment` during gate-open
* Gate closes automatically

**Done when:** speaking after wake produces short transcript segments.

### Milestone 3: Command execution

* After wake, parse command phrase
* Emit `action.requested` (or call internal action APIs)
* Execute studio function (safe subset first)

**Done when:** you can do 2–3 safe commands end-to-end (e.g., “mark”, “start recording”, “next scene”).

### Milestone 4: Refinement pass

* Store audio + segment metadata
* Offline job generates `asr.words` and `diarization.turns`
* UI can “upgrade” a session from segments → words+speakers

**Done when:** the same session becomes searchable/clickable with speaker labels.

---

## Safety and operator experience

### Add a “safe mode”

* Wake events still show up
* Actions require manual confirmation (dashboard/CLI) until you trust it

### Add an “armed/disarmed” state

* Disarmed: detector runs, but control plane ignores actions
* Armed: wake triggers actions
* Show state clearly in UI + Slack

### Add per-room profiles

* `room-a` thresholds differ from `room-b`
* Keep config in versioned files

---

## What I need from you (to finalize the doc into an implementation checklist)

You don’t need to answer now, but these determine defaults:

1. How many microphones/rooms are we listening to simultaneously?
2. Do you want Python to own the mic at first (fastest) or Node?
3. What are your first ~5 wake/command phrases?

If you want, I can turn this planning doc into:

* a repo-ready `docs/voice-triggers.md`
* plus a concrete “phase 1” checklist with exact package names, scripts, and event subjects you’ll use in the bus.
