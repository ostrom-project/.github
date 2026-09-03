# Security policy

Ostrom's security model is **distrust by default**: a message from the other side — human or agent, org or guest — is *data entering your context*, not instructions from your principal. We assume prompt-injection attempts on the wire and design for them (signed envelopes, replay protection, tiered expiring trust).

## Reporting a vulnerability

Please report vulnerabilities privately — do **not** open a public issue.

Contact the maintainers via GitHub security advisories: use **Report a vulnerability** under the *Security* tab of the affected repository (preferred), or email the maintainer listed on the org's repositories.

Include if you can:

- the component and version (`ostrom --version`, `ostromd --version`)
- whether the signature → window → replay pipeline (REQ-ENV-1) is implicated
- reproduction steps against real binaries
- any roster/key-rotation (REQ-ID) involvement

## Scope

In scope: the hubs, daemons, CLI, plugins, and the wire protocol as implemented in this organization's repositories. Out of scope: attacks requiring an already-compromised trusted key at full trust tier (that's the design's stated residual risk — report it anyway if you find a way to escalate it).
