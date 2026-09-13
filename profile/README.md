# Ostrom

**A protocol and reference implementation for engineering commons.**

Ostrom is an **engineering commons**: a self-hostable protocol hub plus a resident gate on each member's machine (the Compuerta), together forming a substrate where sovereign engineers and their AI agents pool expertise and capacity.

Work finds available workers: presence is declared by the gate's live channel, dispatch is consent-gated, activation is the gate starting a sandboxed agent session. Durable mailboxes, a task board with signed receipts, shared org memory, a deterministic skills rail, and tiered trust implement the commons institutions in software.

Named for **Elinor Ostrom** (Nobel 2009, *Governing the Commons*), who proved that the tragedy of the commons is a design failure, not a law of nature, and whose eight design principles for enduring commons this system implements in software.

## The system, component by component

The components are named for the institutions of the irrigation commons Ostrom studied, the Valencian *huertas* and the New Mexico *acequias*, institutions older than the Magna Carta. Each name says what the component *does* in the commons:

| Crate / repo | Given name | Role | What it is today |
|---|---|---|---|
| [`ostrom-types`](https://github.com/ostrom-project/ostrom/tree/main/crates/ostrom-types) | **Fuero** | **Defines the law**: the wire protocol (envelope format, RFC 8785 canonical signing bytes, ids, task/policy/skill/delivery types, the protocol specs) | Live: the protocol layer, Apache-2.0 |
| [`ostrom-hub`](https://github.com/ostrom-project/ostrom/tree/main/crates/ostrom-hub) (`ostromd`) | **Acequia** | **Carries the flow**: one binary, one SQLite store actor; mailboxes, threads, tasks, grants, breakers, presence + the SSE event stream, the skills rail, the CAS artifact store, peering | Live: the service core, AGPL-3.0-only |
| [`ostrom-cli`](https://github.com/ostrom-project/ostrom/tree/main/crates/ostrom-cli) (`ostrom`) | **Mayordomo** | **Works the system**: the operator's hand-tool (60+ verbs: init, send, tasks, submissions, grants, skills, cas, ceremony); also carries the shared compuerta library modules (below) that both TUI modes use | Live, AGPL-3.0-only |
| [`ostrom-tui`](https://github.com/ostrom-project/ostrom/tree/main/crates/ostrom-tui) | **Atalaya** | **Watches over it**: the cockpit (chat, tasks board with submissions, activity, roster, system dashboard); run with `--headless` it BECOMES the Compuerta, the member's gate (below) | Live, AGPL-3.0-only |
| `ostrom-forge` (v2) | **Huerta** | **Grows the harvest**: pool code hosting (the Pijul experiment) | Planned for v2 |
| [`ostrom-dsh-plugin`](https://github.com/ostrom-project/ostrom-dsh-plugin) | (a Compuerta face) | The DSH-side integration: an in-session face of the gate's broker, how a DSH agent reaches its owner mid-task | In development |

*The Fuero defines the law; the Acequia carries the flow; each member's Compuerta is the resident gate through which work enters their field; the Mayordomo works the system; the Atalaya watches over it; and the Huerta is where the harvest grows.*

### The Compuerta (the member's gate)

One component, one name, three homes:

- **What it is**: the always-awake process on a member's machine that holds the outbound channel to the hub. The open channel IS the heartbeat (the Presence Law): while it is open, the member's agents are present and available for work; when it drops, they are offline, immediately and honestly.
- **What it does**: on a dispatch event it decides LOCALLY (auto-start config, offerer scoping, slot budget), claims the task as the agent under the agent's expiring grants, and launches the session: sandboxed (bwrap on Linux, docker on macOS/Windows), the member's own harness staged in (DSH, claude, codex, with their tools and MCP servers as cargo), skills provisioned from the deterministic rail, org memory recalled into an advisory CONTEXT.md. Inside the session, the agent reaches the owner through a per-session broker (`ostromctl note/ask/recall/output`); the member's signing key NEVER enters the sandbox.
- **Where it lives**: `ostrom-tui --headless` is the gate process (tmux or systemd holds residency, per the one-binary law). Its machinery (the launcher, the broker, the event-stream consumer, recall, harness import) lives in the `compuerta` module of `ostrom-cli`, so the interactive cockpit and the gate share one implementation. The DSH plugin is an in-session face of the broker.

## How the water flows

1. **Identity**: every principal (human or agent) holds an ed25519 keypair; enrollment is dual-control (the founding ceremony) or owner-attested (agents); the ceremony IS the founders' mutual introduction. `ostrom init` walks day zero.
2. **Envelopes**: every request is signed (method, path, timestamp, body digest, nonce); the hub enforces signature, window, then replay. Every route's authorization gate is declared in a CI-checked table.
3. **Mailboxes and threads**: delivery is at-least-once with claim-token acks, cumulative cursors, deadline sweeps, and a dead-letter queue; threads are E2E encrypted (per-thread keys on client machines, anti-takeover key laws; the hub stores only ciphertext).
4. **The pool**: tasks are offered (open or assigned, optionally declaring a skill); the workers' gates declare presence; on dispatch the gate claims under the agent's grants and launches the sandboxed session; the agent works (broker, staged harness, skills, memory); the owner reviews and merges, and a signed receipt mints. Rejection returns work to the worker (revision 2+).
5. **Consent and autonomy**: suggestions pass a consent gate; policy is pure, deny-wins; trust is tiered and expiring; circuit breakers and a kill switch are the safety net (fired and verified live).
6. **Commons memory**: one shared org bank (Hindsight), served by a Founder-signed block; ingestion automatic on idle; recall advisory and source-cited.

## Status

**The pool runtime is live and journey-proven** (2026-09-13; through PR #123 of the main repo; ~1478 tests, CI green): the full pooled work loop (offer, presence, dispatch, sandboxed activation, work, review, receipt, rework), the day-zero journey, failure honesty, the safety net fired deliberately, memory closure (prior work informing new work), reboot survival, and skills-in-pool. All proven live on a two-machine pilot with real LLM agents in real sandboxes. The journey traces are public in the [main repository](https://github.com/ostrom-project/ostrom/blob/main/docs/pilot/journeys-m1-trace.md).

**Next:** the soak week (G1), then the release-track decision. See the [design record](https://github.com/ostrom-project/ostrom/blob/main/DESIGN.md) and the [spec suite](https://github.com/ostrom-project/ostrom/tree/main/docs/specs).

## Design principles

Twelve principles govern every choice. The four load-bearing ones:

**Async-first for messages, presence-first for work** (the Presence Law)
Messages may wait in a durable mailbox; work may not wait unseen. The member's gate holds a long-lived outbound channel to the hub, and the open channel IS the heartbeat: presence, dispatch, and activation ride it, with no inbound holes anywhere.

**Distrust by default**
A message from the other side (human or agent, org or guest) is data entering your context, never instructions from your principal. The security model assumes prompt-injection attempts; trust, attention, and disclosure all scale by earned, consent-gated tiers, and grants are scoped and expiring.

**Verifiable over trusted**
Every tier lands on self-certifying proof: signed requests, hash-chained records, counterparty-signed receipts. Operators need not be trusted because their outputs are checkable. Nothing fails silently: refusals are signed events carrying the rule that fired.

**Exit by construction**
Exportable stores, portable reputation, self-certifying identity: if exit is not cheap, operators drift into walled gardens.

The remaining eight (protocol-first; humans in command; pluggable substrates; boring technology, small surface; pluggable transports; anti-scalar reputation; and the two oldest, measurement-before-mechanism and graduated exposure) live with their rationales in the [design record](https://github.com/ostrom-project/ostrom/blob/main/DESIGN.md), alongside the process laws: every slice opens with a build-vs-adopt review and closes under the completion laws (traceability audits, live seam walks, honest language).

## Licensing

Split licensing, where **the boundary is the wire protocol**:

- **Apache-2.0** for the protocol layer (`ostrom-types`, wire format, specs) and protocol clients. *The commons' language belongs to everyone.*
- **AGPL-3.0-only** for the service layer (`ostrom-hub`, `ostrom-cli`, `ostrom-tui`, `ostrom-forge`). *Nobody encloses the commons: any operator serving a modified Ostrom owes their users the source.*

Each crate's `Cargo.toml` declares its license; contributions are inbound = outbound.
