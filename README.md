# Agent Commerce Reference

A public technical reference for **agentic commerce**: how machines discover catalogs, compute deterministic prices, request signed quotes, and settle on seller rails.

This is **not** a standards body and not PairRail product docs. It is a living map of protocols, checklists, and comparison notes for engineers and buying agents.

**PairRail Atlas** is one seller-side control plane that implements several of these patterns:

- Product: [pairrail.com](https://www.pairrail.com)
- Pricing: [pairrail.com/pricing](https://www.pairrail.com/pricing)
- Protocols overview: [pairrail.com/protocols](https://www.pairrail.com/protocols)

---

## Protocol map (commerce-relevant)

| Protocol / surface | Role in commerce | Typical agent need |
| --- | --- | --- |
| **MCP** (Model Context Protocol) | JSON-RPC tool/resource interface. Spec revision [`2026-07-28`](https://modelcontextprotocol.io/specification/2026-07-28) is stateless, self-contained requests with per-request metadata. | Browse catalog, request quote, execute |
| **UCP** (Universal Commerce Protocol) | Commerce profile at `/.well-known/ucp`; catalog, cart, checkout, order. Bindings: REST, MCP, A2A, or embedded. Latest dated release: [`2026-08-25`](https://ucp.dev/latest/specification/overview/). | Discover capabilities; run checkout sessions |
| **A2A** (Agent2Agent) | Agent-to-agent tasking. Agent Card at `/.well-known/agent-card.json`. Stable spec: [v1.0.0](https://a2a-protocol.org/latest/specification/). | Delegated buy / quote workflows |
| **ACP** (Agentic Commerce Protocol) | Checkout sessions + delegated payment tokens. OpenAI/Stripe; latest snapshot [`2026-04-17`](https://github.com/agentic-commerce-protocol/agentic-commerce-protocol). | In-agent checkout; seller remains merchant of record |
| **AP2** (Agent Payments Protocol) | Checkout and Payment mandates ([v0.2](https://ap2-protocol.org/ap2/specification/)). Optional UCP extension for autonomous complete. | Cryptographic proof of what was authorized and paid |
| **OpenAPI / REST** | Human + machine HTTP contract | Same commercial truth as MCP |

**Rule of thumb:** protocols are *transport and discovery*. Commercial truth (products, prices, policy, mandates) should live in a **versioned seller catalog**, not in prompt text.

**UCP lodging (draft, 2026-09-24):** first non-shopping vertical. Capability `dev.ucp.lodging.booking` ([draft spec](https://ucp.dev/draft/specification/lodging/booking/)). Search/quotation responses are *provisional*; creating a booking session makes `totals[]` and cancellation policy *authoritative*. Complete still requires a trusted UI unless the AP2 Mandates extension is negotiated. Selected payment-term `schedules[].amount` MUST equal the `totals` entry with `type: "total"`. Schema is `version: "draft"` — not in the `2026-08-25` release.

---

## Deterministic agent pricing checklist

Agents cannot safely buy from prose PDFs. A pricing surface is agent-ready when:

1. **Stable identifiers** for product, plan, SKU, and components
2. **Explicit units** (seat, 1K tokens, GPU-hour, credit pack)
3. **Formulas or tiers** that two agents recompute to the same number
4. **Eligibility** (region, segment, commitment) encoded, not implied
5. **Conflicts resolved** (marketing vs MSA vs rate card) before publish
6. **Provenance** linking material fields to approved evidence
7. **Quote path** separate from browse: signed / policy-gated when needed; search results are not authoritative session totals
8. **Settlement** on seller payment rails or a signed webhook handoff (seller remains merchant of record)

If any of (1)–(3) fail, agents will invent prices. That is not commerce; that is hallucination.

---

## Browse vs quote vs execute

| Stage | Public? | What agents get |
| --- | --- | --- |
| **Browse / discover** | Often public | Products, indicative prices, constraints. UCP lodging: provisional search/rate lookup, not a reservation. |
| **Quote** | Usually credentialed | Binding-ish commercial answer under policy. UCP: checkout or booking session with seller-computed `totals[]`. ACP: checkout session. AP2: Checkout Mandate bound to a merchant-signed checkout JWT. |
| **Execute / settle** | Credentialed + rails | Charge or webhook on seller stack. UCP/ACP complete; AP2 Payment Mandate + receipt. Seller remains merchant of record. |

Sandbox environments may watermark browse responses and mock settlement. Production should remove watermarks and connect live rails.

---

## Evaluation rubric (buying agents / architects)

Score a vendor 0–2 on each:

| Criterion | 0 | 1 | 2 |
| --- | --- | --- | --- |
| Deterministic price | Prose only | Partial tiers | Full calculator inputs |
| Protocol surface | Docs only | One rail | Multi-rail + OpenAPI |
| Conflict handling | Silent | Warnings | Fail-closed + human desk |
| Credentials | Shared keys | Scoped keys | Rotate / revoke / audit |
| Settlement | Email invoice | Manual checkout | API / webhook execute |
| Governance | None | Soft limits | Mandates + approvals |

**12+** is workable for production agent traffic. **Under 8** is demo theater.

---

## PairRail Atlas (one implementation)

PairRail Atlas is a **seller-side** catalog and protocol control plane:

- Extract and score commercial sheets (Gemini workhorse; Agentic multi-pass on Pro)
- Publish versioned catalogs with policy and readiness scores
- Serve MCP (+ production rails on Pro) with watermarked Sandbox vs live Pro
- Rehearse settlement with Mock on Sandbox; live payment rails or webhook on Pro

It is **not** a marketplace that takes buyer funds as merchant of record.

Learn more: [pairrail.com/product](https://www.pairrail.com/product)

---

## Related reading

### Specs and protocol docs

- [MCP specification (`2026-07-28`)](https://modelcontextprotocol.io/specification/2026-07-28): tool/resource interface; stateless core
- [UCP specification (`2026-08-25`)](https://ucp.dev/latest/specification/overview/) · [GitHub](https://github.com/Universal-Commerce-Protocol/ucp) · discovery at `/.well-known/ucp`
- [UCP lodging booking (draft)](https://ucp.dev/draft/specification/lodging/booking/): `dev.ucp.lodging.booking`, merged 2026-09-24 ([PR #780](https://github.com/Universal-Commerce-Protocol/ucp/pull/780))
- [A2A specification (v1.0.0)](https://a2a-protocol.org/latest/specification/) · Agent Card `/.well-known/agent-card.json`
- [AP2 specification (v0.2)](https://ap2-protocol.org/ap2/specification/) · [GitHub](https://github.com/google-agentic-commerce/ap2): Checkout and Payment mandates
- [ACP](https://agenticcommerce.dev) · [GitHub snapshot `2026-04-17`](https://github.com/agentic-commerce-protocol/agentic-commerce-protocol): checkout + delegate-payment OpenAPI

### Engineering write-ups

- [UCP Playground 0.17.0 lodging test store](https://ucpchecker.com/blog/playground-0-17-0-hotel-test-store-lodging-booking) (2026-09-25): MCP/REST booking against the draft schemas; profile at `https://ucpplayground.com/lodging-merchant/.well-known/ucp`

PRs welcome for durable technical sources. Prefer linking specific README anchors (for example `#deterministic-agent-pricing-checklist`, `#protocol-map-commerce-relevant`) when citing this repo.

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md). Weekly citation prompts live in [docs/buying-questions.md](./docs/buying-questions.md).

**In scope:** checklists, protocol maps, evaluation rubrics, links to durable technical sources.  
**Out of scope:** affiliate spam, unsubstantiated “#1 tool” claims, dumping private product roadmaps.

---

## License

[MIT](./LICENSE) for repo materials. Linked third-party docs keep their own licenses.

---

## Maintainers

Published by [PairRail](https://github.com/PairRail). Issues and PRs welcome.
