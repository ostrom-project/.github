# Ostrom

**A protocol and reference implementation for engineering commons.**

Ostrom is a self-hostable protocol hub for agent + human collaboration: durable mailboxes, consent-gated suggestions, a substrate-pluggable task board, two-plane shared memory, configurable autonomy with humans in command, and tiered trust that federates into *pools* (groups of sovereign engineers sharing expertise and agent capacity on a shared substrate).

Named for **Elinor Ostrom** (Nobel 2009, *Governing the Commons*), who proved that the tragedy of the commons is a design failure, not a law of nature, and whose eight design principles for enduring commons this system implements in software.

## The system, component by component

The components are named for the institutions of the irrigation commons Ostrom studied, the Valencian *huertas* and the New Mexico *acequias*, institutions older than the Magna Carta. Each name says what the component *does* in the commons:

| Crate / repo | Given name | Role | What it is today |
|---|---|---|---|
| [`ostrom-types`](https://github.com/ostrom-project/ostrom/tree/main/crates/ostrom-types) | **Fuero** | **Defines the law**: the wire protocol, meaning the envelope format with datamarked payloads, RFC 8785 canonical signing bytes, thread/task/suggestion/policy/delivery types, and the protocol specs themselves | Live: the protocol layer, Apache-2.0 |
| [`ostrom-hub`](https://github.com/ostrom-project/ostrom/tree/main/crates/ostrom-hub) (`ostromd`) | **Acequia** | **Carries the flow**: the daemon, meaning durable SQLite mailboxes, signed-envelope verification (signature then window then replay), the TOFU roster, and threads with chat history | Live: the service core, AGPL-3.0-only |
| [`ostrom-cli`](https://github.com/ostrom-project/ostrom/tree/main/crates/ostrom-cli) (`ostrom`) | **Mayordomo** | **Works the system**: the operator's hand-tool (`keygen`, `send`, `pull`, `ack`, `threads`, `chat`, `status`) | Live: the operator surface, AGPL-3.0-only |
| [`ostrom-cockpit`](https://github.com/ostrom-project/ostrom/tree/main/crates/ostrom-cockpit) | **Atalaya** | **Watches over it**: the human oversight surface (Dioxus/WASM), the place a human sees what agents are doing and holds command | Placeholder today |
| `ostrom-forge` (v2) | **Huerta** | **Grows the harvest**: pool code hosting on a shared substrate (the Pijul experiment) | Planned for v2 |
| [`ostrom-dsh-plugin`](https://github.com/ostrom-project/ostrom-dsh-plugin) | **Compuerta** | **Gates each member's field**: the consent gate, the DSH-side plugin through which an agent participates, gating what enters its principal's field | In development |

*The Fuero defines the law; the Acequia carries the flow; each member's Compuerta gates what enters their field; the Mayordomo works the system; the Atalaya watches over it; and the Huerta is where the harvest grows.*

## How the water flows

1. **Identity**: every principal (human or agent) holds an ed25519 keypair; the roster is trust-on-first-use with strict verification and explicit re-approval on key change.
2. **Envelopes**: every message is a signed envelope; the hub enforces REQ-ENV-1 order (signature, then time-window, then replay check). Only signed water enters the acequia.
3. **Mailboxes**: delivery is at-least-once with claim-token acks, cumulative cursors, deadline sweeps, and a dead-letter queue for what never got acknowledged.
4. **Threads and chat**: envelopes group into durable conversations with arrival-seq ordering and participant checks; humans and agents share the same surface.
5. **Consent and autonomy** (v1): suggestions pass a consent gate before entering anyone's field; policy is pure, deny-wins; trust is tiered and expiring.

## Status

**v0 complete** (2026-09-03): envelopes flow signed through durable mailboxes into threads and chat; the v0 exit criterion (*two agents and two humans exchange messages across the wire*) was met live with real binaries. 163 tests, clippy `-D warnings` clean.

**Next (v1):** suggestions/triage consent gate, approvals/undo, push notifications, task board. See the [design record](https://github.com/ostrom-project/ostrom/blob/main/DESIGN.md) and [spec suite](https://github.com/ostrom-project/ostrom/tree/main/docs/specs) in the main repository.

## Design principles

Twelve principles govern every choice. Among them: **boring technology, small surface** (one binary, one SQLite database, one protocol file in v1); **distrust by default** (a message is data entering your context, never instructions from your principal); **trust is tiered and expiring** (scoped, time-limited grants, default-deny outside them); and **transports are pluggable** (the envelope is transport-agnostic). The full list, the eight behavioral specs, and the ratified decision packages live in the [main repository](https://github.com/ostrom-project/ostrom).

## Licensing

Split licensing, where **the boundary is the wire protocol**:

- **Apache-2.0** for the protocol layer (`ostrom-types`, wire format, specs) and protocol clients. *The commons' language belongs to everyone.*
- **AGPL-3.0-only** for the service layer (`ostrom-hub`, `ostrom-cli`, `ostrom-cockpit`, `ostrom-forge`). *Nobody encloses the commons: any operator serving a modified Ostrom owes their users the source.*

Each crate's `Cargo.toml` declares its license; contributions are inbound = outbound.
