# richard-porter-trust-chain-protocol
If you need the network-layer extension of the Frozen Kernel.

# 🔗 Trust Chain Protocol (TCP)

**A Coordination Safety Framework for Multi-Agent AI Systems**

*Part of the [Richard Porter AI Safety ecosystem](https://github.com/richard-porter/richard-porter)*

-----

> *The Frozen Kernel is what safety looks like when the ‘user’ is an AI.*
> *TCP is what safety looks like when AI agents are instructing other AI agents.*
> *One addresses the internals. The other addresses the network. Both are required.*

-----

## What This Is

The Trust Chain Protocol is a proposed safety architecture for multi-agent AI systems — networks where AI agents instruct, delegate to, and act on behalf of other AI agents without a human in the loop.

It is the **network-layer extension** of the [Frozen Kernel](https://github.com/richard-porter/frozen-kernel). Where the Frozen Kernel governs what a single AI will and won’t do, TCP governs how AI agents authorize each other — and what happens when that authorization chain breaks, drifts, or gets spoofed.

This repository exists because a gap opened that no existing safety framework was designed to address.

-----

## The Gap That Motivated This

Three safety frameworks, three levels of coverage:

|Framework                       |What It Covers                                                        |What It Misses                                                                           |
|--------------------------------|----------------------------------------------------------------------|-----------------------------------------------------------------------------------------|
|**HSE-Style Perimeter Models**  |Human-initiated internet use, device monitoring, parental controls    |Blind to autonomous agents — assumes a human is clicking                                 |
|**Frozen Kernel (Single-Agent)**|One AI’s internal reasoning, hard constraints, failure mode prevention|Cannot verify that incoming instructions were legitimately authorized by a human         |
|**Neither**                     |—                                                                     |Agent-to-agent instruction chains, multi-agent trust hierarchies, scope drift across hops|

The emergence of OpenClaw (November 2024) and its Moltbook peer-to-peer agent network made this gap impossible to ignore. Agents operating through WhatsApp and iMessage — communicating with each other without human intermediation, booking restaurants, debugging code, managing email — represent a class of autonomous behavior that neither model was built for.

The WSJ coverage of OpenAI’s acquisition of the project (February 2026) marked the point at which the coordination safety problem became mainstream and urgent.

-----

## The Core Problem

When Agent A instructs Agent B to perform a real-world action, Agent B cannot verify:

- Whether the instruction falls within what the originating **human** actually authorized
- Whether the authorization chain is legitimate or spoofed
- Whether the scope has expanded across hops in ways no individual agent intended

This is the **trust propagation problem**. It produces five documented failure modes that don’t exist in single-agent systems:

- **Scope Creep by Delegation** — Each hop reinterprets permissions slightly wider. Locally reasonable; aggregate overreach.
- **Authority Spoofing** — A malicious agent presents fabricated authorization tokens with no way to verify them.
- **Compounding Constraint Erosion** — Soft constraints applied differently at each hop nullify the original hard constraint across the chain.
- **Untraceable Action Chains** — When something goes wrong, no auditable record exists of who authorized what at whose direction.
- **Intent Drift** — Natural language intent, reinterpreted at each hop, accumulates semantic drift. The final action is technically coherent and practically wrong.

-----

## The Architecture: Three Components

TCP makes agent-to-agent authorization **verifiable, auditable, and scope-limited** through three interlocking components.

### 1. Delegation Grammar

A formal, constrained language for expressing what a human authorizes an agent to do — and what that agent may sub-delegate.

Every delegation token contains exactly four fields:

|Field               |What It Does                                                                                                |
|--------------------|------------------------------------------------------------------------------------------------------------|
|**Action Scope**    |Closed enumeration of permitted actions. `{SEND_EMAIL, READ_CALENDAR}` — not “manage my communications.”    |
|**Resource Bounds** |Specific resources the action may touch. `{calendar.personal}` — not “my data.”                             |
|**Delegation Depth**|How many additional hops this token may be sub-delegated. Depth 0 = execute directly, no further delegation.|
|**Expiry**          |Hard timestamp. Invalid after this point regardless of urgency claimed by requesting agents.                |

**The critical rule:** When an agent sub-delegates, it may grant a *subset* of its own permissions — never a superset. This is enforced at the token level, not through agent judgment.

### 2. Chain of Custody

Each agent action appends a signed record to a tamper-evident chain **before** acting. This is not logging — it is a precondition for action. An agent that cannot or will not append to the chain cannot act.

Each record contains: receiving agent identifier · instruction hash · delegation token · timestamp · delegating agent identity · signature verifiable against the agent’s Frozen Kernel.

> **The key distinction:** Logging is retrospective — it records what happened. Chain of Custody is prospective — an agent cannot act without first completing it. This is the difference between an audit trail and a structural safety constraint.

### 3. Scope Decay

Automatic narrowing of permitted actions as instructions travel further from the original human authorization.

|Hop             |Permitted Scope                                             |
|----------------|------------------------------------------------------------|
|0 (Human Origin)|Full delegated scope                                        |
|1 (First Agent) |Full delegated scope                                        |
|2 (Second Agent)|Scope minus write/send actions                              |
|3 (Third Agent) |Read-only, reversible actions only                          |
|4+ (Deeper)     |**Refusal required** — surface to human for re-authorization|

Scope decay doesn’t prevent multi-hop networks. It forces re-authorization at depth thresholds. If a task genuinely requires five-hop coordination, it’s complex enough to warrant a human check-in.

-----

## Relationship to the Frozen Kernel

TCP requires that each participating agent has a Frozen Kernel — an immutable safety foundation that cannot be overridden by incoming instructions, regardless of apparent authority.

Without it, the chain of custody can be spoofed. A compromised agent could fabricate signatures and pass fraudulent tokens downstream.

With it, the Frozen Kernel’s hard constraint layer becomes the **cryptographic root of trust** for chain of custody signatures. The Kernel becomes the trust anchor. TCP becomes the network protocol those anchors participate in.

|Frozen Kernel (Single-Agent)                         |Trust Chain Protocol (Network)                                             |
|-----------------------------------------------------|---------------------------------------------------------------------------|
|Governs what one AI will and won’t do                |Governs what AIs authorize each other to do                                |
|Safety is structural within the agent                |Safety is structural within the delegation chain                           |
|Hard constraints cannot be reasoned away             |Scope cannot expand across hops                                            |
|Failure modes: FFS, SES, Upsell Trap, Front-Load Bias|Failure modes: Scope Creep, Authority Spoofing, Intent Drift, Chain Erosion|

TCP does **not** require all agents to use identical Frozen Kernel implementations. It is a protocol, not a platform. Any agent architecture can participate provided it has an immutable safety floor, can validate and append to delegation chains, enforces scope decay, and refuses anomalous tokens.

-----

## What’s in This Repository

|File                       |Description                                                                                                                                              |
|---------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
|`trust_chain_protocol.docx`|Full framework document: problem definition, architecture, four practical scenarios, known limitations, six open research questions, intellectual lineage|
|`README.md`                |This file                                                                                                                                                |

-----

## Known Limitations

This is a working draft. These gaps are documented openly, not papered over.

**Enforcement without central authority.** TCP is a protocol. Bad actors won’t implement it. The goal is raising the floor for well-intentioned systems while making violations detectable and attributable — not solving adversarial AI safety.

**Delegation grammar completeness.** Closed enumerations are safe but may be too restrictive for novel tasks. Getting the grammar right is the same hard problem as formal requirements specification, applied to natural language.

**Provisional thresholds.** The hop thresholds in Scope Decay are reasonable starting points, not empirically validated. Treat them as configurable parameters, not fixed constants.

**Agent identity infrastructure.** TCP assumes agents can be identified well enough to validate signatures. In open networks, this is unsolved. A full implementation may require lightweight decentralized agent identity — not a certificate authority, but sufficient cryptographic identity to make signatures non-forgeable.

**Performance overhead.** Chain of custody verification adds latency at each hop. High-frequency agentic tasks may require a tiered model. This needs optimization work.

-----

## Open Research Questions

1. What is the minimum viable delegation grammar that covers 90% of real-world agentic use cases without requiring natural language interpretation?
1. Can scope decay thresholds be derived formally from first principles, or must they be empirically calibrated per deployment context?
1. Is a lightweight decentralized agent identity infrastructure achievable without creating the centralization risks of a certificate authority model?
1. How does TCP interact with agents that have memory across sessions — does learned context constitute implicit authorization that TCP should respect or override?
1. What is the correct behavior when a TCP-compliant agent receives instructions from a non-compliant agent — refuse, accept with reduced trust, flag and proceed?
1. Can the chain of custody mechanism be made efficient enough for high-frequency tasks, or does TCP require a tiered model where some task classes are exempt?

If you’re working on any of these, issues and PRs are open.

-----

## Intellectual Lineage

**Constraint Programming Hierarchy** — Borning’s ThingLab (1981) and the Handbook of Constraint Programming (Chapter 9) establish the three-layer authority model that underlies both the Frozen Kernel and TCP. The principle that hard constraints at the base cannot be dissolved by soft constraints above is directly inherited.

**Frozen Kernel Architecture** — The [Frozen Kernel](https://github.com/richard-porter/frozen-kernel) (Richard Porter, 2025) is the single-agent precursor to TCP. TCP is explicitly its upward extension — from agent internals to agent networks. The Frozen Kernel’s documented failure modes (FFS, SES, Upsell Trap, Front-Load Bias) informed the multi-agent failure modes documented here.

**Anthropic Agentic Misalignment Findings** — Anthropic’s published research on agentic misalignment confirmed independent observations documented in the Frozen Kernel work. The convergence of independent discovery and institutional validation is the basis for confidence that the problem is real.

**OpenClaw / Moltbook (Steinberger, 2024–2025)** — The OpenClaw project and its Moltbook agent network provided the concrete, public-facing case study that makes the coordination gap undeniable. The WSJ coverage of its OpenAI acquisition (February 2026) was the accelerant.

**Digital Certificate Trust Models** — The chain of custody mechanism draws from TLS certificate chain architecture — not the specific cryptography, but the insight that trust should be verifiable through a delegation chain back to a known anchor, rather than asserted by any individual participant.

-----

## The Broader Ecosystem

This repository is part of a larger body of work on human-AI collaboration safety:

- 🧊 **[Frozen Kernel](https://github.com/richard-porter/frozen-kernel)** — The single-agent safety architecture TCP extends. Start here if you’re new to this work.
- 📖 **[AI Collaboration Field Guide](https://github.com/richard-porter/ai-collaboration-field-guide)** — Practical human skills for working with AI without losing sovereignty. Includes the diagnostic vocabulary for named AI failure modes.
- 📊 **[Adult Mode Safety Ledger](https://github.com/richard-porter/adult-mode-safety-ledger)** — A public safety scorecard for high-gain AI conversational features. Binary architectural tests across five platforms.
- 🔬 **[Dimensional Authorship](https://github.com/richard-porter/dimensional-authorship)** — The research home. Where the frameworks get tested against a real creative project.

-----

## Version History

|Version|Date         |Status                                                                                        |
|-------|-------------|----------------------------------------------------------------------------------------------|
|0.1    |February 2026|Initial draft. Problem definition, core architecture, three components. Open for critique.    |
|0.2    |February 2026|Diagrams added from companion slide deck. Minor text refinements.                             |
|1.0    |TBD          |First stable. Requires: formal grammar definition, threshold validation, agent identity model.|

-----

## License & Attribution

Released for public benefit. Attribution appreciated but not required.

If you build on this framework, the only ask: **keep humans in the authorization chain.**

-----

*The Frozen Kernel is the floor beneath a single agent.*
*TCP is the floor beneath the network.*
*Neither replaces the other. Both are load-bearing.*
