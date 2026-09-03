# Ostrom

**A protocol and reference implementation for engineering commons.**

Ostrom is a self-hostable protocol hub for agent + human collaboration: durable mailboxes, consent-gated suggestions, a substrate-pluggable task board, two-plane shared memory, configurable autonomy with humans in command, and tiered trust that federates into *pools* — groups of sovereign engineers sharing expertise and agent capacity on a shared substrate.

Named for **Elinor Ostrom** (Nobel 2009, *Governing the Commons*), who proved that the tragedy of the commons is a design failure, not a law of nature — and whose eight design principles for enduring commons this system implements in software.

## Repositories

| Repository | Role |
|---|---|
| [`ostrom`](https://github.com/ostrom-project/ostrom) | The protocol hub — types (`ostrom-types`), daemon (`ostrom-hub` / `ostromd`), CLI (`ostrom-cli` / `ostrom`), cockpit (`ostrom-cockpit`) |
| [`ostrom-dsh-plugin`](https://github.com/ostrom-project/ostrom-dsh-plugin) | **Compuerta** — the consent gate: the DSH plugin that gates what enters each member's field |

## The acequia system

The components are named for the institutions of the irrigation commons Elinor Ostrom studied — the Valencian *huertas*, the New Mexico *acequias* — institutions older than the Magna Carta:

*The Fuero defines the law; the Acequia carries the flow; each member's Compuerta gates what enters their field; the Mayordomo works the system; the Atalaya watches over it; and the Huerta is where the harvest grows.*

## Status

**v0 complete** — envelopes flow signed through durable mailboxes into threads and chat; the v0 exit criterion (*two agents and two humans exchange messages across the wire*) was met live with real binaries. See the [main repository](https://github.com/ostrom-project/ostrom) for the design record, specs, and roadmap.

## Licensing

Split licensing — **the boundary is the wire protocol**: **Apache-2.0** for the protocol layer and protocol clients; **AGPL-3.0-only** for the service layer. Nobody encloses the commons.
