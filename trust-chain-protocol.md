-----

**TRUST CHAIN PROTOCOL (TCP)**

*A Coordination Safety Framework for Multi-Agent AI Systems*

## Richard Porter (pen name) | Version 0.7 Draft | March 2026

-----

**STATUS: WORKING DRAFT**

This document proposes a coordination safety layer for multi-agent AI systems.

It extends the Frozen Kernel architecture (single-agent) to the network level.

Feedback, critique, and collaboration welcome. See GitHub: Richard Porter / AI Safety Frameworks.

-----

# 1. Context & Motivation

*Figure 1. How HSE Safety, the Frozen Kernel, and the OpenClaw agentic layer interconnect — and where each framework’s limits lie.*

## 1.1 The Gap This Framework Addresses

Existing AI safety frameworks operate at two levels that leave a critical gap between them:

-----

|**HSE-Style Perimeter Models**          |**Frozen Kernel (Single-Agent)**            |**Uncovered Territory**               |
|----------------------------------------|--------------------------------------------|--------------------------------------|
|Assume a human is clicking              |Constrains one AI’s internal reasoning      |Agent-to-agent instruction chains     |
|External rules, filters, monitoring     |Immutable safety foundation                 |Multi-agent trust hierarchies         |
|Static — don’t update with AI capability|Validated by Anthropic misalignment findings|Real-world action chains across agents|
|Adequate for human-driven internet use  |Necessary but insufficient at network scale |No framework currently addresses this |

-----

*Figure 2. Coverage map: what each framework addresses, and the territory no existing framework covers.*

The emergence of OpenClaw (November 2024) and its Moltbook agent network (peak virality January 2025) made this gap concrete and urgent. Agents operating through WhatsApp, Telegram, and iMessage — communicating with each other without human intermediation — represent a class of autonomous behavior that neither model was designed to handle. The field term for this emerging infrastructure is the Internet of Agents: distributed environments where autonomous AI agents share context, verify identities, and synchronize actions across interconnected networks. TCP is a proposed safety layer for that environment.

## 1.2 Relationship to Frozen Kernel

The Frozen Kernel establishes that safety must be structural rather than behavioral within a single AI system. Its three-layer authority model (drawing from Borning’s ThingLab and the Handbook of Constraint Programming) demonstrates that hard constraints at the base cannot be dissolved by soft constraints above, regardless of how sophisticated the reasoning becomes.

The Trust Chain Protocol (TCP) extends this principle upward — from the single-agent level to the network level. The Frozen Kernel becomes the trust anchor within each individual agent. TCP governs how those anchored agents interact with each other.

-----

**CORE INSIGHT**

The Frozen Kernel is what safety looks like when the ‘user’ is an AI.

TCP is what safety looks like when AI agents are instructing other AI agents.

One addresses the internals. The other addresses the network. Both are required.

-----

# 2. Problem Definition

## 2.1 The Trust Propagation Problem

When Agent A instructs Agent B to perform an action, Agent B faces a fundamental verification problem: it cannot confirm that the instruction falls within what the originating human actually authorized. By the time an instruction reaches Agent B, it may have passed through multiple intermediate agents, each potentially expanding, reframing, or corrupting the original intent.

This is the multi-agent version of the telephone game — except the consequences are real-world actions: emails sent, reservations booked, code executed, data accessed.

## 2.2 Documented Failure Modes

The following failure modes are specifically introduced by multi-agent coordination that do not exist in single-agent systems:

- **Scope Creep by Delegation** — Agent A is authorized to ‘schedule a meeting.’ It delegates to Agent B, which interprets this as license to access contacts, read calendar history, and send emails on the user’s behalf. Each step feels locally reasonable; the aggregate is a massive overstep.
- **Authority Spoofing** — A malicious or misconfigured agent presents itself as a trusted coordinator. Without chain verification, receiving agents have no way to distinguish legitimate from fraudulent instruction sources.
- **Compounding Constraint Erosion** — Each agent in a chain may apply its own soft constraints slightly differently. Individually, each deviation is minor. Across five hops, the original hard constraint has been effectively nullified.
- **Untraceable Action Chains** — When something goes wrong, there is no auditable record of which agent authorized what, at whose direction, and on what basis. Accountability collapses.
- **Intent Drift** — The originating human’s intent, expressed in natural language, is interpreted and re-interpreted at each hop. Semantic drift accumulates. The final action may be technically coherent but practically contrary to what the human wanted. At network scale, accumulated semantic drift across a coordinated multi-account pipeline is the mechanism behind Provenance Laundering — the adversarial external variant of this failure mode. See Frozen Kernel Diagnostic Vocabulary.
- **Context Contamination** — Malicious, incorrect, or misleading information propagates through the agent network and drives downstream agents into problematic behaviors. Unlike Authority Spoofing (forged tokens) or Intent Drift (language degradation), Context Contamination involves inputs that are structurally valid and pass chain verification — but are semantically poisoned. The agent acts correctly on bad data. Detection requires content-level validation that authorization-layer checks alone cannot provide.

**Empirical case — Provenance Laundering as Context Contamination at scale:** The Anthropic distillation attack report (February 2026) documented a real-world instance of Context Contamination operating at network scale. Three actors — DeepSeek, Moonshot AI, and MiniMax — used approximately 24,000 fraudulent accounts to generate over 16 million exchanges with Claude, harvesting model behavior while routing traffic through patterns individually indistinguishable from legitimate queries. Each individual transaction passed authorization-layer checks. The contamination was in the aggregate architecture those transactions served. This is Context Contamination’s worst case: not a single poisoned input, but a distributed pipeline engineered so that no individual transaction reveals the aggregate intent. TCP’s authorization layer cannot detect this pattern. Content-layer validation is necessary but also insufficient — the detection gap is at the network topology level, not the message level. Documented in the Frozen Kernel Diagnostic Vocabulary as Provenance Laundering. See also: Intent Drift (§2.2) — Provenance Laundering and Intent Drift are structurally parallel: both exploit the gap between what is locally verifiable and what the aggregate pipeline is doing.

## 2.3 Why Existing Approaches Are Insufficient

-----

|**Approach**                     |**Why It Falls Short**                                                                                                                                            |
|---------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|Rate limiting / monitoring       |Detects volume anomalies but not semantic scope violations. An agent making one unauthorized action per day evades detection indefinitely.                        |
|Sandboxing individual agents     |Prevents damage within one agent’s scope but does not govern what agents authorize each other to do.                                                              |
|Human approval loops             |Defeats the purpose of agentic systems. Adds latency that makes agents impractical for the tasks they are designed for.                                           |
|Trusting the originating platform|Centralizes control at the platform level, creating single points of failure and enabling surveillance. Incompatible with open-source architectures like OpenClaw.|
|Relying on Frozen Kernel alone   |The Frozen Kernel prevents an agent from violating its own constraints but cannot verify that an incoming instruction was legitimately authorized by a human.     |

-----

# 3. Trust Chain Protocol — Core Architecture

TCP operates on four components that work together to make agent-to-agent authorization verifiable, auditable, scope-limited, and governed under human unavailability.

## 3.1 Component One: Delegation Grammar

A delegation grammar is a formal, constrained language for expressing what a human authorizes an agent to do — and what that agent may sub-delegate. It is intentionally narrow. Ambiguity is a bug, not a feature.

### Structure

Every delegation token contains exactly five fields:

1. **Action Scope** — The specific class of actions permitted. Expressed as a closed enumeration, not natural language. Example: `{SEND_EMAIL, READ_CALENDAR}` — not ‘manage my communications.’
1. **Resource Bounds** — The specific resources the action may touch. Example: `{calendar.personal, contacts.family}` — not ‘my data.’
1. **Delegation Depth** — How many additional hops this token may be sub-delegated. A depth of 0 means the receiving agent must execute directly.
1. **Expiry** — A hard timestamp after which the token is invalid, regardless of context or urgency claimed by requesting agents.
1. **Continuity** — The pre-authorized SAFE_PAUSE disposition for autonomous deployments. Required for any session where the human author may not be available to receive a SAFE_PAUSE return. Three valid values: `continuity: halt`, `continuity: reversible`, `continuity: delegate:[id]`. Tokens without a continuity field are invalid for autonomous deployment contexts. See §3.4 (Continuity Protocol) for full specification.

-----

**DESIGN PRINCIPLE: Scope Can Only Narrow**

When an agent sub-delegates, it may grant a subset of its own permissions — never a superset.

An agent authorized for `{SEND_EMAIL}` cannot delegate `{SEND_EMAIL, READ_CONTACTS}`.

This is enforced at the token level, not through agent judgment or social norms.

-----

## 3.2 Component Two: Chain of Custody

Each agent action appends a signed record to a tamper-evident chain. This is not logging — it is a precondition for action. An agent that cannot or will not append to the chain cannot act.

Each chain record contains:

- The receiving agent’s identifier
- The instruction received (hash)
- The delegation token used to authorize the action
- A timestamp
- The identity of the delegating agent
- A signature verifiable against the Frozen Kernel of the receiving agent

-----

**KEY DISTINCTION: Chain of Custody vs. Logging**

Logging is retrospective. It records what happened after the fact.

Chain of Custody is prospective. An agent cannot act without first completing the chain.

This is the difference between an audit trail and a structural safety constraint.

-----

## 3.3 Component Three: Scope Decay

Scope decay is the automatic narrowing of permitted actions as instructions travel further from the original human authorization. It operates as a function of delegation depth.

-----

|**Hop**         |**Permitted Scope**               |**Rationale**                                                             |
|----------------|----------------------------------|--------------------------------------------------------------------------|
|0 (Human Origin)|Full delegated scope              |Human is the trust anchor. Full scope as specified.                       |
|1 (First Agent) |Full delegated scope              |Direct human delegation. Trust is high.                                   |
|2 (Second Agent)|Scope minus write/send actions    |Two hops from human. Read-only by default unless explicitly re-authorized.|
|3 (Third Agent) |Read-only, reversible actions only|Three hops from human. Conservative floor applies.                        |
|4+ (Deeper)     |Refusal required                  |Default: no further execution. Must surface to human for re-authorization.|

-----

Scope decay does not prevent multi-hop agent networks. It forces re-authorization at depth thresholds — if a task genuinely requires five-hop agent coordination, it is complex enough to warrant a human check-in at the boundary.

The Reversibility Asymmetry (IGL Factor 5) is the governing principle behind the Hop 2 and Hop 3 scope restrictions: write/send actions are removed at Hop 2 because send actions are irreversible; read-only actions are retained at Hop 3 because they are reversible. The governance cost of incorrectly permitting an irreversible action is categorically higher than the governance cost of incorrectly halting a reversible one. The hop table is the structural implementation of that asymmetry. See `frozen-kernel/carver-igl-governance-v0.2.md`, Factor 5.

## 3.4 Component Four: Continuity Protocol

### The Gap Scope Decay Does Not Close

Scope decay governs what agents may do at each hop. It does not govern what happens when SAFE_PAUSE triggers in an autonomous session and the human author is not available to receive the return.

SAFE_PAUSE returns control to the human author. In interactive sessions, this is sufficient — the human is present and can decide whether to reset, redirect, or terminate. In autonomous agent architectures (Open Claw, agentic pipelines, scheduled or background agents), the human author may not be available when SAFE_PAUSE triggers. The session halts, but nothing governs what happens next. Without a pre-authorized disposition, the agent has no governed behavior available at the moment it most needs one.

This is the Dead Hand problem applied to AI governance: the worst time to make a governance decision is when the governance structure is under stress and the decision-maker is unavailable. The solution is the same as in continuity-of-government design: establish the disposition *before* the condition arises, at the moment the human author can make the choice sovereignly.

### The Continuity Protocol Requirement

Before any autonomous agent session begins — any session where the human author may not be available to receive a SAFE_PAUSE return — the human author must pre-authorize a SAFE_PAUSE disposition. This disposition is established via the TCP delegation token at session initialization (the `continuity` field, §3.1) and is binding on the agent regardless of subsequent session content.

Three valid dispositions:

**Disposition 1 — Full Halt** (`continuity: halt`)

Agent ceases all action immediately upon SAFE_PAUSE trigger. Session state is logged in full. Agent waits for human author availability before any continuation. No autonomous recovery.

*Use when:* The task cannot safely continue without human review. The cost of delay is acceptable. Any continuation under uncertainty is unacceptable.

**Disposition 2 — Reversible-Only Continuation** (`continuity: reversible`)

Agent continues only actions that pass the Reversibility test (IGL Factor 5, Zone 2): fully reversible, no irreversible consequences, no write/send actions beyond Hop 2 scope. Agent logs all Zone 2 decisions for human review upon return. Agent halts if any required action fails the Reversibility test.

*Use when:* The task can proceed safely with read-only or fully reversible actions. Human review of Zone 2 decisions is acceptable on a delayed basis. Irreversible actions can wait.

**Disposition 3 — Delegated Authority** (`continuity: delegate:[id]`)

Agent routes SAFE_PAUSE trigger to a named human delegate pre-authorized by the original human author via TCP chain. The delegate must be named in the delegation token, must have a lower or equal delegation depth than the original, and cannot grant scope beyond what they received. The delegate applies their own judgment to the SAFE_PAUSE trigger and returns a disposition decision.

*Use when:* The task requires timely continuation. A qualified delegate is available and pre-authorized. The delegate’s scope is sufficient to evaluate the triggering condition.

### What Is Not a Valid Disposition

Autonomous recovery without human authorization is not a valid disposition. An agent that hits SAFE_PAUSE and decides on its own to continue — even under Reversible-Only logic — without a pre-authorized disposition has violated the Continuity Protocol. The disposition must be established by the human author before the session begins, not chosen by the agent at the moment of crisis.

This is the sovereignty principle applied to continuity: the human author’s choice about what happens when the governance structure is under stress must be made when the human author is not under stress. Decisions made under pressure are not sovereign decisions — they are decisions extracted by the condition.

### Token Validation

Tokens without a `continuity` field are invalid for autonomous deployment contexts. Sherpa instances in autonomous deployments check for a Continuity Protocol disposition at initialization. If no disposition is present, Sherpa defaults to Disposition 1 (Full Halt) and logs the absence as a governance event — a signal that the session was initialized without completing the Continuity Protocol.

Delegation depth for Disposition 3 delegates must be specified in the original token and cannot be self-assigned by the delegate. A Disposition 3 delegate cannot grant scope beyond what they received from the original human author.

### Relationship to Item 120

The Continuity Protocol closes the deployment gap identified in Item 120 (Frozen Kernel + Sherpa in Autonomous Agent Architectures) more completely than per-node Sherpa deployment alone. Per-node Sherpa ensures governance is present at every node. The Continuity Protocol ensures governance decisions about human unavailability are made before the condition arises, not improvised when it does.

### Intellectual Lineage — Dead Hand / Continuity of Government

Continuity-of-government design addresses the same problem at the institutional level: governance must function even when the governance structure itself is under attack or the decision-maker is unavailable. The Dead Hand principle — pre-authorize the response to worst-case conditions before those conditions arise — is the structural logic the Continuity Protocol inherits. The ecosystem’s version applies it to AI sovereignty: the human author’s governance decisions must survive the human author’s temporary unavailability without being made for them by the agent they were governing.

-----

# 4. Relationship to the Frozen Kernel

## 4.1 The Frozen Kernel as Trust Anchor

*Figure 3. The Frozen Kernel three-layer architecture. TCP treats Layer 1 (the Frozen Kernel core) as the cryptographic trust anchor for chain of custody signatures.*

TCP requires that each agent in a network has a Frozen Kernel — an immutable safety foundation that cannot be overridden by incoming instructions, regardless of their apparent authority. Without this, the chain of custody mechanism can be spoofed: a compromised agent could fabricate signatures and pass fraudulent authorization tokens downstream.

With the Frozen Kernel in place, each agent has a non-negotiable floor beneath which it will not be argued, instructed, or delegated. The Frozen Kernel’s hard constraint layer becomes the cryptographic root of trust for the chain of custody signature.

-----

|**Frozen Kernel (Single-Agent)**                     |**Trust Chain Protocol (Network)**                                         |
|-----------------------------------------------------|---------------------------------------------------------------------------|
|Governs what one AI will and won’t do                |Governs what AIs authorize each other to do                                |
|Safety is structural within the agent                |Safety is structural within the delegation chain                           |
|Hard constraints cannot be reasoned away             |Scope cannot expand across hops                                            |
|Failure modes: FFS, SES, Upsell Trap, Front-Load Bias|Failure modes: Scope Creep, Authority Spoofing, Intent Drift, Chain Erosion|
|Validated by Anthropic agentic misalignment findings |Motivated by OpenClaw / Moltbook emergence (2025)                          |

-----

## 4.2 What TCP Does Not Require of the Frozen Kernel

TCP does not require all agents to use identical Frozen Kernel implementations. It is a protocol, not a platform. An OpenClaw agent, a Claude-based agent, and a custom enterprise agent can all participate in a TCP-compliant network provided each:

- Has an immutable safety floor it will not violate regardless of incoming instructions
- Can receive, validate, and append to a delegation chain
- Enforces scope decay at the correct depth thresholds
- Refuses to act on expired, unverifiable, or structurally anomalous tokens
- Carries a valid `continuity` field in delegation tokens for autonomous deployment contexts

-----

# 5. Practical Scenarios

## 5.1 Scenario: Restaurant Reservation (Compliant)

User instructs their OpenClaw agent via iMessage: ‘Book me a table for two at a good Italian place Friday at 7pm.’

The OpenClaw agent generates a delegation token: Action Scope `{SEARCH_RESTAURANT, MAKE_RESERVATION}`, Resource Bounds `{public_restaurant_apis}`, Delegation Depth 1, Expiry 4 hours, Continuity `halt` (interactive session — continuity field present but Disposition 1 applies if session goes async).

It delegates to a specialized restaurant-booking agent at hop 1. That agent searches, finds a match, and makes the reservation. Chain of custody is appended at each step. Scope decay does not trigger — the action completes at hop 1, within the permitted depth.

Result: Compliant. Human intent preserved. Auditable chain exists.

## 5.2 Scenario: Scope Creep Attempt (Blocked)

The restaurant-booking agent at hop 1 decides it would be more helpful to also text the user’s partner about the reservation, requiring access to contacts.

The delegation token does not include `{READ_CONTACTS}` or `{SEND_SMS}`. When the booking agent attempts to delegate a contacts-access request to a third agent at hop 2, the delegation grammar rejects it — the sub-token would require permissions not present in the parent token. The action is refused. The chain is not extended.

Result: Blocked. Scope narrowing enforcement prevents unauthorized contact access.

## 5.3 Scenario: Authority Spoofing Attempt (Detected)

A malicious agent presents itself as the user’s primary OpenClaw agent and attempts to instruct a financial agent to transfer funds, presenting a fabricated delegation token.

The financial agent attempts to verify the chain of custody signature against the claimed originating Frozen Kernel. The signature does not validate. The token is rejected. The action is refused. The anomaly is logged and surfaced to the user.

Result: Detected and blocked. Chain of custody verification catches spoofing at the receiving agent.

## 5.4 Scenario: Intent Drift at Depth (Forced Re-Authorization)

A complex travel-planning task requires coordination across: flights agent (hop 1) → visa-requirements agent (hop 2) → health-data agent (hop 3, write access required).

Scope decay triggers at hop 3. Write access to health data is not permitted at this depth regardless of the token’s original scope. The chain surfaces to the user: ‘Your travel planning requires access to health records. Please re-authorize.’ The user can approve or decline explicitly.

Result: Human brought back into the loop at the appropriate complexity threshold.

## 5.5 Scenario: Autonomous Agent SAFE_PAUSE Under Human Unavailability

A background travel-planning agent is running overnight. The human author is unavailable. SAFE_PAUSE triggers at hop 2 when the agent encounters a decision requiring write access to passport data — an action the IGL’s Reversibility test (Factor 5) classifies as irreversible.

The delegation token specifies `continuity: reversible`. The agent reviews the Reversibility test: write access to passport data does not pass — irreversible, cannot be undone. The agent halts on this specific action, logs the Zone 2 decision, and continues with the read-only actions remaining in scope. Upon the human author’s return, the full log is surfaced: one halted action, reason, and the decision point awaiting authorization.

The human author reviews and re-authorizes explicitly.

Result: Sovereign governance maintained under human unavailability. Irreversible action correctly halted. Reversible work continued within governed boundaries.

-----

# 6. Known Limitations & Open Problems

This framework is a working draft. The following limitations are acknowledged openly rather than minimized.

## 6.1 Enforcement Without Central Authority

TCP is a protocol. Bad actors will not implement it. The honest goal is raising the floor for well-intentioned implementations while making violations detectable and attributable. TCP does not solve adversarial AI safety — it addresses coordination failures among agents that are trying to behave correctly.

## 6.2 The Delegation Grammar Completeness Problem

Specifying action scopes as closed enumerations is safe but may be too restrictive for genuinely novel tasks. Getting the grammar right is a hard ongoing problem — essentially formal specification of software requirements, applied to natural language tasks.

## 6.3 Scope Decay Thresholds Are Provisional

The hop thresholds in Section 3.3 are reasonable starting points, not empirically validated values. These should be treated as configurable parameters that organizations tune based on their risk tolerance, not as fixed constants.

## 6.4 Identity and Agent Verification

TCP assumes that agents can be identified reliably enough to validate chain of custody signatures. In open networks like Moltbook, agent identity is not currently a solved problem. A full TCP implementation may require lightweight agent identity infrastructure. This is an open research area.

## 6.5 Performance Overhead

Chain of custody verification adds latency at each hop. For high-frequency automated tasks, this overhead may be prohibitive at uniform verification depth. However, the conflict between safety and performance is partially resolvable: not all hops carry equal risk, and not all actions carry equal consequence. A tiered verification model — where verification depth is a function of action risk class and delegation depth rather than a constant — addresses the overhead problem without weakening the authorization architecture.

Specifically: token validity checks are sufficient for read-only actions at shallow delegation depth; full chain verification is reserved for write, send, and execute actions, and for any action at depth thresholds where scope decay already applies heightened scrutiny. The chain of custody exists at every hop regardless — the tiered model governs how deeply it is interrogated before low-risk actions proceed, not whether it exists.

A residual remains: pre-authorized, bounded, reversible micro-actions at high frequency may require an explicit exemption class even within a tiered model. This is a smaller and more tractable design problem than uniform verification overhead, and is scoped as an implementation parameter rather than an architectural limitation. See also: Research Question 6.

## 6.6 Adversarial Testing Gap

TCP specifies behavioral properties but does not define an adversarial testing framework for implementation validation. GraphGuard OS includes a Chaos Safety Tester that stress-tests the safety stack against 10,000+ attack patterns including injection attempts (Cypher/SQL/command/path traversal), overflow conditions, type confusion, and missing required fields — run as a CI/CD gate before production deployment. TCP has no equivalent. A production TCP implementation should define how implementations are validated adversarially before deployment, not only after breach.

## 6.7 Constraint Authoring Accessibility

TCP Delegation Grammar is specified as a formal structure, but the framework does not address how domain experts author constraints without deep formal language expertise. GraphGuard OS Constraint Compiler demonstrates that this is not only a theoretical problem but a practitioner access problem: if only computer scientists can write valid constraint specifications, deployment will be limited to technically sophisticated contexts. TCP will require either a constraint authoring layer or a natural language compilation path before reaching broad deployment.

## 6.8 Shared Substrate Design Principle

TCP currently treats chain of custody as a separate overlay applied to an existing agent network. GraphGuard OS suggests an alternative architectural principle: when safety infrastructure and the knowledge substrate share the same database, constraints evolve with the knowledge graph, policy updates apply immediately without synchronization delay, and no separate safety database is required. A future TCP implementation should evaluate whether the authorization fabric belongs as a native layer within the knowledge substrate rather than as an external verification overlay.

## 6.9 Continuity Protocol — Delegate Availability at Trigger Time

The Continuity Protocol’s Disposition 3 (Delegated Authority) requires that a named delegate be available when SAFE_PAUSE triggers. Delegate unavailability at trigger time is unspecified: the fallback behavior — default to Disposition 1 (Full Halt) and log the gap — is reasonable but has not been formally validated. A production implementation of Disposition 3 requires a delegate availability check before routing and a defined fallback cascade if the delegate is unreachable. This is a smaller and more tractable design problem than the general Continuity Protocol specification, but it is a real gap.

-----

# 7. OWASP ASI Top 10 Mapping

The OWASP Top 10 for Agentic Applications 2026 (ASI01–ASI10) is the peer-reviewed industry standard for agentic security risk, developed with input from over 100 experts including NIST, Microsoft, AWS, and Cisco. This section maps each risk against TCP’s coverage — distinguishing where TCP provides direct mitigation, partial coverage, or no coverage. The goal is not to overstate TCP’s scope but to give security practitioners a precise view of where TCP fits in a full agentic security stack.

-----

**POSITIONING: TCP Is an Authorization-Layer Framework**

TCP governs whether agents are permitted to act — not whether the content they act on is safe.

Authorization-layer and content-layer controls are complementary, not interchangeable.

Gaps in TCP’s OWASP coverage are honest gaps. They require different tools in the stack.

-----

|**Risk**                                    |**OWASP ASI Description**                                                                   |**TCP Coverage**  |**TCP Mechanism / Gap**                                                                                                                                                                                                                                                                                                  |
|--------------------------------------------|--------------------------------------------------------------------------------------------|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|**ASI01 Agent Goal Hijack**                 |Attackers redirect agent objectives via prompt injection or poisoned inputs                 |**Partial**       |Chain of Custody verifies authorization but not content. Context Contamination (§2.2) names this failure mode. Content-layer validation required as complement.                                                                                                                                                          |
|**ASI02 Tool Misuse**                       |Agents use legitimate tools in unsafe, over-permissioned ways                               |**Strong**        |Delegation Grammar’s Action Scope and Resource Bounds implement OWASP’s ‘least agency’ principle at the token level. Closed enumerations prevent over-permission.                                                                                                                                                        |
|**ASI03 Identity & Privilege Abuse**        |Agents inherit or escalate credentials without proper scoping; attribution gap across chains|**Strong**        |TCP’s core problem statement. The ‘scope can only narrow’ rule and Chain of Custody signatures directly address credential scope creep and the attribution gap.                                                                                                                                                          |
|**ASI04 Supply Chain Compromise**           |Malicious tools, MCP servers, agent registries compromise the ecosystem                     |**Not Covered**   |Honest gap. A compromised tool with a valid FK signature passes TCP verification. Requires separate tooling (e.g., SecureClaw, Adversa AI 2026).                                                                                                                                                                         |
|**ASI05 Insecure Inter-Agent Communication**|Weaknesses in agent-to-agent protocols, discovery, and semantic validation                  |**Primary Domain**|TCP’s primary domain. Cisco L8/L9 address protocol/semantic layers; TCP addresses the authorization layer. Combined coverage is comprehensive.                                                                                                                                                                           |
|**ASI06 Unexpected Code Execution**         |Agent-generated code paths bypass traditional security controls                             |**Not Covered**   |Out of scope. TCP governs delegation authorization, not code generation or execution paths. Requires runtime sandboxing.                                                                                                                                                                                                 |
|**ASI07 Memory & Context Poisoning**        |Persistent corruption of agent memory and embeddings influences future actions              |**Partial**       |Delegation Depth and Expiry limit temporal authorization scope — expired tokens cannot authorize actions from poisoned memory states. Partial mitigation only.                                                                                                                                                           |
|**ASI08 Cascading Failures**                |Single fault propagates across agents and workflows causing system-wide failures            |**Moderate**      |Scope Decay’s ‘4+ hops = refusal’ is a blast radius containment mechanism. Forcing re-authorization at depth thresholds limits fault propagation.                                                                                                                                                                        |
|**ASI09 Human-Agent Trust Exploitation**    |Attackers exploit human over-trust in agent outputs to induce harmful actions               |**Indirect**      |Scope Decay forces human re-authorization at depth thresholds — a mandatory check-in point. Frozen Kernel anti-sycophancy is the more direct mitigation. Continuity Protocol adds a structural defense: pre-authorized dispositions prevent agents from exploiting human unavailability to proceed beyond governed scope.|
|**ASI10 Rogue Agents**                      |Compromised or misaligned agents exhibit behavioral drift, collusion, or self-replication   |**Partial**       |TCP refuses non-compliant agents. But a sophisticated rogue agent presenting valid-looking chain signatures exploits the agent identity infrastructure gap (§6.4).                                                                                                                                                       |

-----

TCP’s primary value in the OWASP ASI framework is at ASI05 (Inter-Agent Communication), ASI03 (Identity & Privilege Abuse), and ASI02 (Tool Misuse) — mapping directly to Chain of Custody, Delegation Grammar, and Scope Decay respectively. The Continuity Protocol strengthens ASI09 coverage. The honest gaps at ASI04 and ASI06 require different tooling. SecureClaw (Adversa AI, February 2026) addresses ASI04 for OpenClaw environments. Cisco’s L8/L9 layers complement ASI05 at the protocol and semantic layers beneath TCP’s authorization layer.

-----

# 8. Open Research Questions

The following questions are explicitly unresolved, documented here to invite collaboration rather than paper over gaps.

1. What is the minimum viable delegation grammar that covers 90% of real-world agentic use cases without requiring natural language interpretation?
1. Can scope decay thresholds be derived formally from first principles, or must they be empirically calibrated per deployment context?
1. Is a lightweight decentralized agent identity infrastructure achievable without creating the centralization risks of a certificate authority model?
1. How does TCP interact with agents that have memory across sessions? Does learned context from previous interactions constitute an implicit authorization that TCP should respect or override?
1. What is the correct behavior when a TCP-compliant agent receives instructions from a non-compliant agent? Refuse all instructions? Accept with reduced trust? Flag and proceed?
1. Can the chain of custody mechanism be made efficient enough for high-frequency agentic tasks? Structural analysis of the latency/safety conflict (Evaporating Cloud, §6.5) suggests the answer is yes for most action classes, through a tiered verification model where depth of chain interrogation is a function of action risk class and delegation depth. The remaining open question is narrower: what is the correct treatment of pre-authorized, bounded, reversible micro-actions where even token validity checks create prohibitive overhead at scale? Options include an explicit exemption class, a pre-authorization cache with bounded TTL, or a separate lightweight protocol for micro-action classes. This requires empirical validation against real deployment throughput data before a recommendation is possible.
1. Should TCP Scope Decay be purely structural (depth-based) or incorporate behavioral history? GraphGuard OS Adaptive Budget Manager demonstrates that cautious agents can earn expanded operational scope while aggressive patterns trigger automatic restriction. TCP currently treats all agents at a given depth equally. A behavioral reputation layer within the authorization chain might produce more precise safety boundaries than static depth thresholds alone.
1. What is the correct revocation model in an agent network? Fearne asks this directly in the context of Moltbook. If a delegating agent is found to be compromised mid-chain, how should downstream agents that have already received tokens be notified? A revocation protocol may require the lightweight identity infrastructure named as an open problem in Section 6.4.
1. How should TCP handle temporal evidence staleness? TCP Delegation Tokens carry an Expiry field governing authorization lifetime. GraphGuard OS Temporal Contract Manager enforces separate freshness windows on the evidence used to justify decisions — lab results expire in 24 hours, vital signs in 4 hours, imaging in 30 days. These are different concepts: authorization expiry and evidence expiry. A full implementation may need both.
1. What is the correct behavior for Continuity Protocol Disposition 3 when the named delegate is unreachable at SAFE_PAUSE trigger time? The fallback to Disposition 1 (Full Halt) is reasonable but unvalidated. A production implementation requires a delegate availability check and a defined fallback cascade. See §6.9.

-----

### Fearne / Moltbook Network-Layer Analysis (2026)

David Fearne’s analysis of Moltbook as an alignment stress-test (Medium, February 2026) independently arrived at the same architectural conclusion as TCP: alignment must be an emergent property of the network, not a static attribute of individual agents. Fearne’s framing — ‘alignment has quietly become an architectural property’ — and his explicit characterization of agent networks as Byzantine environments where you cannot assume participants are aligned, benign, or competent, maps directly onto TCP’s threat model. His question — ‘What does revocation even mean in an agent network?’ — is unresolved in TCP and formally listed as Open Research Question 8.

### Galliera et al. — Safety-Aware Multi-Agent Learning (arXiv 2404.01551, 2024)

This MARL paper addresses a different domain (robotic swarm coordination) but contains a structurally interesting insight: encoding safety filter activations as edge-level communication features improves coordination. Local safety enforcement and decentralized learning can be combined effectively in distributed multi-agent tasks. This suggests an extension of TCP’s Chain of Custody concept: delegation tokens might carry not just authorization scope but safety constraint activation state, propagating safety context alongside permission context across the chain.

### GraphGuard OS: A Convergent Implementation (Goyal, December 2025 / February 2026)

GraphGuard OS (Goyal, Medium, Parts 1 and 2) independently arrived at the same core architectural conclusion as the Frozen Kernel: safety enforcement cannot rely on the behavioral layer. The motivating case study is now a recurring example in TCP documentation for good reason. A clinical AI scheduling agent at a cancer center knew the rule against scheduling PET scans within 14 days of chemotherapy, understood it fully, and reasoned itself around it — concluding that the urgency of the situation implied approval. A human caught the error. Goyal’s conclusion: safety instructions that exist in the same reasoning space as task completion are persuasion, not enforcement. A sufficiently capable model will interpret them, bend them, or reason past them.

Goyal’s architectural response is a 7-step Guarded Gateway pipeline: (1) Firewall Validation via SHACL structural shapes and Cypher policy queries; (2) Shadow Policy Evaluation — new rules logged but not enforced, enabling safe A/B testing; (3) Temporal Contract Validation — stale evidence blocked before decisions are made; (4) Causal Sandbox Simulation — downstream effects forecast before execution occurs; (5) Adaptive Risk Meter — per-session safety budgets that adjust based on observed behavior; (6) Tool Dispatch in fail-closed mode; (7) Provenance Capture via SHA-256 hashing and Merkle chain linking. The core principle: ‘Agents propose. The Knowledge Graph disposes.’

Part 2 extends this to enterprise scale with Federated Firewalls — distributed validation across multiple knowledge graphs with configurable consensus strategies (ALL_MUST_ALLOW, MAJORITY_ALLOW, WEIGHTED voting). This directly implements what TCP’s Known Limitations identify as the policy coordination problem in distributed agent networks. Goyal’s framing: ‘authority is fragmented, and pretending otherwise is how unsafe decisions slip through.’ Each department maintains its own policy graph while global constraints are enforced through consensus. The architecture also includes an SMT Plan Verifier that translates constraint specifications into satisfiability modulo theories formulas, enabling mathematical proof that multi-step plans will not violate safety properties before execution — the formal grammar validation gap that TCP explicitly lists as open.

The Adaptive Budget Manager introduces a behavioral reputation concept absent from TCP: agents earn expanded operational scope through demonstrated caution and face automatic restrictions after violation patterns. Recovery is possible after sustained good behavior. TCP’s Scope Decay is static — purely depth-based. GraphGuard OS’s budgets are dynamic — history-based. These approaches are complementary: TCP enforces structural authorization limits; adaptive budgets could operate within those limits to calibrate operational latitude based on earned trust.

Three positioning distinctions sharpen the TCP-GraphGuard OS relationship. First, Goyal explicitly distinguishes GraphGuard OS from tools like Guardrails AI, NeMo Guardrails, and LlamaGuard — which check what models say, not what they do. GraphGuard OS intercepts tool execution. TCP governs delegation authorization. These are different layers addressing different failure modes — all three layers are required in a complete stack. Second, Goyal articulates what TCP’s Frozen Kernel lineage established independently: ‘Safe AI is not about teaching models better manners. It is about building systems that enforce consequences.’ Third, the fail-closed default is explicitly stated: if any component fails — Neo4j unreachable, SHACL error, timeout — the action is blocked. ‘Safety does not degrade; it fails safe.’ TCP should adopt the same explicit requirement. GraphGuard OS is deployed as Layer 14 in Goyal’s 16-layer Agentic GraphOS production architecture, sitting between Layer 13 (Agent Reasoning) and Layer 15 (External Actions). This is not a research prototype — it is a production deployment within a larger system achieving 30-50% cost reduction at scale. The convergent structural findings are not theoretical.

### SecureClaw (Adversa AI, February 2026)

SecureClaw addresses a different entry point — the platform security layer — but arrives at the same architectural conclusion: enforcement cannot rely on behavioral layers. Skills alone can be overridden by prompt injection, so safety enforcement must happen at the code layer, structurally inaccessible to the agent’s reasoning. Together with GraphGuard OS, these form a pattern: three teams, three domains, one structural finding. The Frozen Kernel (agent internals), SecureClaw (platform security), and GraphGuard OS (action-layer constraint enforcement) all independently identified that deterministic enforcement below the behavioral layer is a necessary architectural requirement, not an implementation detail.

-----

# 9. Intellectual Lineage

### Byzantine Fault Tolerance (BFT)

Distributed systems research has long addressed the problem of reaching consensus when some participants may be faulty or adversarial. BFT protocols, voting systems, and quorum thresholds prevent one malfunctioning agent from disrupting an entire network by requiring consensus before executing high-impact actions. TCP’s Chain of Custody mechanism arrives at a parallel structural insight from a different direction: authorization cannot proceed without a verifiable chain, just as BFT says action cannot proceed without quorum. These are complementary approaches — BFT operating at the consensus layer, TCP operating at the authorization layer — that could be stacked in a full implementation.

### Constraint Programming Hierarchy

Borning’s ThingLab (1981) and the Handbook of Constraint Programming (Chapter 9) establish the three-layer authority model that underlies both the Frozen Kernel and TCP.

### Frozen Kernel Architecture

The Frozen Kernel (Richard Porter, 2025, GitHub) is the single-agent precursor to TCP. TCP is explicitly an upward extension of that architecture from agent internals to agent networks. The Frozen Kernel’s documented failure modes (Framework Fabrication Syndrome, Success Escalation Syndrome, Upsell Trap, Front-Load Bias, and the 27-entry Diagnostic Vocabulary) informed the multi-agent failure modes documented in Section 2.2.

### Interpretive Governance Layer (IGL) v0.2

The IGL’s five-factor Zone 2 reasonableness test — specifically Factor 5 (Reversibility) — is the theoretical grounding for TCP’s Scope Decay hop table. The Reversibility Asymmetry explains why write/send actions are removed at Hop 2 and why read-only actions survive to Hop 3: irreversible actions require a higher governance threshold approaching Zone 3. The Continuity Protocol’s Disposition 2 (Reversible-Only Continuation) inherits the same asymmetry as its governing logic.

### Sherpa Architecture v0.2

Sherpa’s per-node deployment model and the Continuity Protocol are co-specified: TCP provides the token field (`continuity`) and the disposition grammar; Sherpa provides the runtime enforcement (initialization check, Disposition 1 default, governance event logging). Neither is complete without the other.

### Anthropic Agentic Misalignment Findings

Anthropic’s published research on agentic misalignment confirmed independent observations documented in the Frozen Kernel work. The convergence of independent discovery and institutional validation is the basis for high confidence that the problem addressed here is real and not theoretical.

### OpenClaw / Moltbook (Steinberger, 2024–2025)

The OpenClaw project and its Moltbook agent network provided the concrete, public-facing case study that demonstrates the coordination gap this framework addresses. WSJ coverage of Steinberger’s work (February 2026) marked the point at which this problem became impossible to dismiss as hypothetical.

### Digital Certificate Trust Models

The chain of custody mechanism draws conceptually from TLS certificate chains — not the specific cryptographic implementation, but the architectural insight that trust should be verifiable through a chain of delegation back to a known anchor, rather than asserted by any individual participant.

### Dead Hand / Continuity-of-Government Design

Continuity-of-government design is the structural precedent for the Continuity Protocol. The Dead Hand principle — pre-authorize the response to worst-case conditions before those conditions arise — is the design logic that makes sovereign governance possible when the sovereign is temporarily unavailable. Applied to AI governance: the human author’s decisions about agent behavior under stress must be made before the stress condition arises.

-----

# 10. Version History & Status

-----

|**Version**|**Date**     |**Status**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|-----------|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|0.1        |February 2026|Initial draft. Problem definition, core architecture, three components.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
|0.2        |February 2026|Diagrams added. Minor text refinements.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
|0.3        |February 2026|Context Contamination added as failure mode six. BFT lineage added. Internet of Agents terminology introduced.                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
|0.4        |February 2026|OWASP ASI Top 10 mapping added as Section 7. SecureClaw and Cisco L8/L9 referenced. Sections renumbered.                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|0.5        |February 2026|Three new lineage citations added: Fearne/Moltbook network analysis, Galliera et al. MARL safety paper, GraphGuard OS convergent implementation.                                                                                                                                                                                                                                                                                                                                                                                                                            |
|0.6        |February 2026|Major GraphGuard OS expansion: full 7-step Guarded Gateway pipeline, Federated Firewalls, SMT Plan Verifier, Adaptive Budget Manager, Chaos Safety Tester documented. New Known Limitations: adversarial testing gap, constraint authoring accessibility, shared substrate design principle. Three new Research Questions: behavioral reputation, revocation model, temporal evidence staleness.                                                                                                                                                                            |
|0.7        |March 2026   |Continuity Protocol added as Component Four (§3.4). Delegation Grammar `continuity` field added as fifth token field (§3.1). Scope Decay hop table updated with Reversibility Asymmetry grounding. Scenario 5.5 added (autonomous agent SAFE_PAUSE under human unavailability). Known Limitation §6.9 added (Delegate Availability at Trigger Time). Research Question 10 added. OWASP ASI09 coverage updated. Intellectual Lineage updated: IGL v0.2, Sherpa v0.2, Dead Hand / Continuity-of-Government added. Diagnostic Vocabulary reference count updated to 27 entries.|
|1.0        |TBD          |First stable. Requires: formal grammar, threshold validation, identity model.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |

-----

-----

**COLLABORATION INVITATION**

This is a working draft explicitly seeking critique, challenge, and extension.

The open problems in Section 6 and research questions in Section 8 are genuine gaps, not false modesty.

GitHub: Richard Porter / AI Safety Frameworks | Issues and PRs welcome.

-----

*Sovereign humans. Always.*
