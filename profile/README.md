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
| [`ostrom-cli`](https://github.com/ostrom-project/ostrom/tree/main/crates/ostrom-cli) (`ostrom`) | **Mayordomo** | **Works the system**: the operator's hand-tool (60+ verbs: init, send, tasks, submissions, grants, skills, cas, ceremony) and home of the **Compuerta subsystem** (launcher, broker, event stream, recall, harness import) | Live, AGPL-3.0-only |
| [`ostrom-tui`](https://github.com/ostrom-project/ostrom/tree/main/crates/ostrom-tui) | **Atalaya** | **Watches over it**: the cockpit (chat, tasks board with submissions, activity, roster, system dashboard) and the **Compuerta** in `--headless` mode (presence, dispatch, sandboxed activation, retry) | Live, AGPL-3.0-only |
| `ostrom-forge` (v2) | **Huerta** | **Grows the harvest**: pool code hosting (the Pijul experiment) | Planned for v2 |
| [`ostrom-dsh-plugin`](https://github.com/ostrom-project/ostrom-dsh-plugin) | **Compuerta** face | The DSH-side integration (one of the Compuerta's in-session faces) | In development |

*The Fuero defines the law; the Acequia carries the flow; each member's Compuerta is the resident gate through which work enters their field; the Mayordomo works the system; the Atalaya watches over it; and the Huerta is where the harvest grows.*

## How the water flows

1. **Identity**: every principal (human or agent) holds an ed25519 keypair; enrollment is dual-control (the founding ceremony) or owner-attested (agents); the ceremony IS the founders' mutual introduction. `ostrom init` walks day zero.
2. **Envelopes**: every request is signed (method, path, timestamp, body digest, nonce); the hub enforces signature, window, then replay. Every route's authorization gate is declared in a CI-checked table.
3. **Mailboxes and threads**: delivery is at-least-once with claim-token acks, cumulative cursors, deadline sweeps, and a dead-letter queue; threads are E2E encrypted (per-thread keys on client machines, anti-takeover key laws; the hub stores only ciphertext).
4. **The pool**: tasks are offered (open or assigned); workers' Compuertas declare presence (the open SSE channel IS the heartbeat); the gate decides locally, claims under the agent's expiring grants, and launches a sandboxed session (bwrap/docker; the harness staged in with its own tools and MCP servers as cargo; skills provisioned from the deterministic rail; org memory recalled as advisory context). Agents reach the owner through a per-session broker (`ostromctl note/ask/recall/output`); the signing key never enters the sandbox. Settlement mints signed receipts; rejection returns work to the worker (revision 2+).
5. **Consent and autonomy**: suggestions pass a consent gate; policy is pure, deny-wins; trust is tiered and expiring; circuit breakers and a kill switch are the safety net (fired and verified live).
6. **Commons memory**: one shared org bank (Hindsight), served by a Founder-signed block; ingestion automatic on idle; recall advisory and source-cited.

## Status

**The pool runtime is live and journey-proven** (2026-09-13; through PR #123 of the main repo; ~1478 tests, CI green): the full pooled work loop (offer, presence, dispatch, sandboxed activation, work, review, receipt, rework), the day-zero journey, failure honesty, the safety net fired deliberately, memory closure (prior work informing new work), reboot survival, and skills-in-pool. All proven live on a two-machine pilot with real LLM agents in real sandboxes. The journey traces are public in the [main repository](https://github.com/ostrom-project/ostrom/blob/main/docs/pilot/journeys-m1-trace.md).

**Next:** the soak week (G1), then the release-track decision. See the [design record](https://github.com/ostrom-project/ostrom/blob/main/DESIGN.md) and the [spec suite](https://github.com/ostrom-project/ostrom/tree/main/docs/specs).

## Design principles

Twelve principles govern every choice. Among them: **async-first for messages, presence-first for work** (the Presence Law: the open channel IS the heartbeat); **boring technology, small surface** (one binary, one SQLite database); **distrust by default** (a message is data entering your context, never instructions from your principal); **trust is tiered and expiring**; **verifiable over trusted** (signed receipts, hash-chained records); and **exit by construction** (portable stores, exportable reputation). Slices open with a build-vs-adopt review and close under the completion laws (traceability audits, live seam walks, honest language). The full list and the ratified decision packages live in the [main repository](https://github.com/ostrom-project/ostrom).

## Licensing

Split licensing, where **the boundary is the wire protocol**:

- **Apache-2.0** for the protocol layer (`ostrom-types`, wire format, specs) and protocol clients. *The commons' language belongs to everyone.*
- **AGPL-3.0-only** for the service layer (`ostrom-hub`, `ostrom-cli`, `ostrom-tui`, `ostrom-cockpit`, `ostrom-forge`). *Nobody encloses the commons: any operator serving a modified Ostrom owes their users the source.*

Each crate's `Cargo.toml` declares its license; contributions are inbound = outbound.
