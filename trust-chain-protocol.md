-----

**TRUST CHAIN PROTOCOL (TCP)**

*A Coordination Safety Framework for Multi-Agent AI Systems*

## Richard Porter (pen name) | Version 0.5 Draft | February 2026

-----

**STATUS: WORKING DRAFT**

This document proposes a coordination safety layer for multi-agent AI
systems.

It extends the Frozen Kernel architecture (single-agent) to the network
level.

## Feedback, critique, and collaboration welcome. See GitHub: Richard
Porter / AI Safety Frameworks.

# 1. Context & Motivation

![](media/image1.jpg){width=“8.366141732283465in”
height=“4.707856517935258in”}

*Figure 1. How HSE Safety, the Frozen Kernel, and the OpenClaw agentic
layer interconnect — and where each framework's limits lie.*

## 1.1 The Gap This Framework Addresses

Existing AI safety frameworks operate at two levels that leave a
critical gap between them:

-----

**HSE-Style Perimeter   **Frozen Kernel         **Uncovered Territory**
Models**                (Single-Agent)**

Assume a human is       Constrains one AI's    Agent-to-agent
clicking                internal reasoning      instruction chains

External rules,         Immutable safety        Multi-agent trust
filters, monitoring     foundation              hierarchies

Static — don't       Validated by Anthropic  Real-world action
update with AI          misalignment findings   chains across agents
capability

Adequate for            Necessary but           No framework currently
human-driven internet   insufficient at network addresses this
use                     scale

-----

![](media/image2.jpg){width=“8.366141732283465in”
height=“4.707856517935258in”}

*Figure 2. Coverage map: what each framework addresses, and the
territory no existing framework covers.*

The emergence of OpenClaw (November 2024) and its Moltbook agent network
(peak virality January 2025) made this gap concrete and urgent. Agents
operating through WhatsApp, Telegram, and iMessage — communicating
with each other without human intermediation — represent a class of
autonomous behavior that neither model was designed to handle. The field
term for this emerging infrastructure is the Internet of Agents:
distributed environments where autonomous AI agents share context,
verify identities, and synchronize actions across interconnected
networks. TCP is a proposed safety layer for that environment.

## 1.2 Relationship to Frozen Kernel

The Frozen Kernel establishes that safety must be structural rather than
behavioral within a single AI system. Its three-layer authority model
(drawing from Borning's ThingLab and the Handbook of Constraint
Programming) demonstrates that hard constraints at the base cannot be
dissolved by soft constraints above, regardless of how sophisticated the
reasoning becomes.

The Trust Chain Protocol (TCP) extends this principle upward — from
the single-agent level to the network level. The Frozen Kernel becomes
the trust anchor within each individual agent. TCP governs how those
anchored agents interact with each other.

-----

**CORE INSIGHT**

The Frozen Kernel is what safety looks like when the 'user' is an AI.

TCP is what safety looks like when AI agents are instructing other AI
agents.

## One addresses the internals. The other addresses the network. Both are
required.

# 2. Problem Definition

## 2.1 The Trust Propagation Problem

When Agent A instructs Agent B to perform an action, Agent B faces a
fundamental verification problem: it cannot confirm that the instruction
falls within what the originating human actually authorized. By the time
an instruction reaches Agent B, it may have passed through multiple
intermediate agents, each potentially expanding, reframing, or
corrupting the original intent.

This is the multi-agent version of the telephone game — except the
consequences are real-world actions: emails sent, reservations booked,
code executed, data accessed.

## 2.2 Documented Failure Modes

The following failure modes are specifically introduced by multi-agent
coordination that do not exist in single-agent systems:

- Scope Creep by Delegation — Agent A is authorized to 'schedule a
  meeting.' It delegates to Agent B, which interprets this as license
  to access contacts, read calendar history, and send emails on the
  user's behalf. Each step feels locally reasonable; the aggregate is
  a massive overstep.
- Authority Spoofing — A malicious or misconfigured agent presents
  itself as a trusted coordinator. Without chain verification,
  receiving agents have no way to distinguish legitimate from
  fraudulent instruction sources.
- Compounding Constraint Erosion — Each agent in a chain may apply
  its own soft constraints slightly differently. Individually, each
  deviation is minor. Across five hops, the original hard constraint
  has been effectively nullified.
- Untraceable Action Chains — When something goes wrong, there is no
  auditable record of which agent authorized what, at whose direction,
  and on what basis. Accountability collapses.
- Intent Drift — The originating human's intent, expressed in
  natural language, is interpreted and re-interpreted at each hop.
  Semantic drift accumulates. The final action may be technically
  coherent but practically contrary to what the human wanted.
- Context Contamination — Malicious, incorrect, or misleading
  information propagates through the agent network and drives
  downstream agents into problematic behaviors. Unlike Authority
  Spoofing (forged tokens) or Intent Drift (language degradation),
  Context Contamination involves inputs that are structurally valid
  and pass chain verification — but are semantically poisoned. The
  agent acts correctly on bad data. Detection requires content-level
  validation that authorization-layer checks alone cannot provide.

## 2.3 Why Existing Approaches Are Insufficient

-----

**Approach**                        **Why It Falls Short**

Rate limiting / monitoring          Detects volume anomalies but not
semantic scope violations. An agent
making one unauthorized action per
day evades detection indefinitely.

Sandboxing individual agents        Prevents damage within one agent's
scope but does not govern what
agents authorize each other to do.

Human approval loops                Defeats the purpose of agentic
systems. Adds latency that makes
agents impractical for the tasks
they are designed for.

Trusting the originating platform   Centralizes control at the platform
level, creating single points of
failure and enabling surveillance.
Incompatible with open-source
architectures like OpenClaw.

Relying on Frozen Kernel alone      The Frozen Kernel prevents an agent
from violating its own constraints
but cannot verify that an incoming
instruction was legitimately
authorized by a human.

-----

# 3. Trust Chain Protocol — Core Architecture

TCP operates on three components that work together to make
agent-to-agent authorization verifiable, auditable, and scope-limited.

## 3.1 Component One: Delegation Grammar

A delegation grammar is a formal, constrained language for expressing
what a human authorizes an agent to do — and what that agent may
sub-delegate. It is intentionally narrow. Ambiguity is a bug, not a
feature.

### Structure

Every delegation token contains exactly four fields:

1. **Action Scope:** Action Scope — The specific class of actions
   permitted. Expressed as a closed enumeration, not natural language.
   Example: {SEND_EMAIL, READ_CALENDAR} — not 'manage my
   communications.'
1. **Resource Bounds:** Resource Bounds — The specific resources the
   action may touch. Example: {calendar.personal, contacts.family} —
   not 'my data.'
1. **Delegation Depth:** Delegation Depth — How many additional hops
   this token may be sub-delegated. A depth of 0 means the receiving
   agent must execute directly.
1. **Expiry:** Expiry — A hard timestamp after which the token is
   invalid, regardless of context or urgency claimed by requesting
   agents.

-----

**DESIGN PRINCIPLE: Scope Can Only Narrow**

When an agent sub-delegates, it may grant a subset of its own
permissions — never a superset.

An agent authorized for {SEND_EMAIL} cannot delegate {SEND_EMAIL,
READ_CONTACTS}.

## This is enforced at the token level, not through agent judgment or
social norms.

## 3.2 Component Two: Chain of Custody

Each agent action appends a signed record to a tamper-evident chain.
This is not logging — it is a precondition for action. An agent that
cannot or will not append to the chain cannot act.

Each chain record contains:

- The receiving agent's identifier
- The instruction received (hash)
- The delegation token used to authorize the action
- A timestamp
- The identity of the delegating agent
- A signature verifiable against the Frozen Kernel of the receiving
  agent

-----

**KEY DISTINCTION: Chain of Custody vs. Logging**

Logging is retrospective. It records what happened after the fact.

Chain of Custody is prospective. An agent cannot act without first
completing the chain.

## This is the difference between an audit trail and a structural safety
constraint.

## 3.3 Component Three: Scope Decay

Scope decay is the automatic narrowing of permitted actions as
instructions travel further from the original human authorization. It
operates as a function of delegation depth.

-----

**Hop**                 **Permitted Scope**     **Rationale**

0 (Human Origin)        Full delegated scope    Human is the trust
anchor. Full scope as
specified.

1 (First Agent)         Full delegated scope    Direct human
delegation. Trust is
high.

2 (Second Agent)        Scope minus write/send  Two hops from human.
actions                 Read-only by default
unless explicitly
re-authorized.

3 (Third Agent)         Read-only, reversible   Three hops from human.
actions only            Conservative floor
applies.

4+ (Deeper)             Refusal required        Default: no further
execution. Must surface
to human for
re-authorization.

-----

Scope decay does not prevent multi-hop agent networks. It forces
re-authorization at depth thresholds — if a task genuinely requires
five-hop agent coordination, it is complex enough to warrant a human
check-in at the boundary.

# 4. Relationship to the Frozen Kernel

## 4.1 The Frozen Kernel as Trust Anchor

![](media/image3.jpg){width=“8.366141732283465in”
height=“4.707856517935258in”}

*Figure 3. The Frozen Kernel three-layer architecture. TCP treats Layer
1 (the Frozen Kernel core) as the cryptographic trust anchor for chain
of custody signatures.*

TCP requires that each agent in a network has a Frozen Kernel — an
immutable safety foundation that cannot be overridden by incoming
instructions, regardless of their apparent authority. Without this, the
chain of custody mechanism can be spoofed: a compromised agent could
fabricate signatures and pass fraudulent authorization tokens
downstream.

With the Frozen Kernel in place, each agent has a non-negotiable floor
beneath which it will not be argued, instructed, or delegated. The
Frozen Kernel's hard constraint layer becomes the cryptographic root of
trust for the chain of custody signature.

-----

**Frozen Kernel (Single-Agent)**    **Trust Chain Protocol (Network)**

Governs what one AI will and won't Governs what AIs authorize each
do                                  other to do

Safety is structural within the     Safety is structural within the
agent                               delegation chain

Hard constraints cannot be reasoned Scope cannot expand across hops
away

Failure modes: FFS, SES, Upsell     Failure modes: Scope Creep,
Trap, Front-Load Bias               Authority Spoofing, Intent Drift,
Chain Erosion

Validated by Anthropic agentic      Motivated by OpenClaw / Moltbook
misalignment findings               emergence (2025)

-----

## 4.2 What TCP Does Not Require of the Frozen Kernel

TCP does not require all agents to use identical Frozen Kernel
implementations. It is a protocol, not a platform. An OpenClaw agent, a
Claude-based agent, and a custom enterprise agent can all participate in
a TCP-compliant network provided each:

- Has an immutable safety floor it will not violate regardless of
  incoming instructions
- Can receive, validate, and append to a delegation chain
- Enforces scope decay at the correct depth thresholds
- Refuses to act on expired, unverifiable, or structurally anomalous
  tokens

# 5. Practical Scenarios

## 5.1 Scenario: Restaurant Reservation (Compliant)

User instructs their OpenClaw agent via iMessage: 'Book me a table for
two at a good Italian place Friday at 7pm.'

The OpenClaw agent generates a delegation token: Action Scope
{SEARCH_RESTAURANT, MAKE_RESERVATION}, Resource Bounds
{public_restaurant_apis}, Delegation Depth 1, Expiry 4 hours.

It delegates to a specialized restaurant-booking agent at hop 1. That
agent searches, finds a match, and makes the reservation. Chain of
custody is appended at each step. Scope decay does not trigger — the
action completes at hop 1, within the permitted depth.

Result: Compliant. Human intent preserved. Auditable chain exists.

## 5.2 Scenario: Scope Creep Attempt (Blocked)

The restaurant-booking agent at hop 1 decides it would be more helpful
to also text the user's partner about the reservation, requiring access
to contacts.

The delegation token does not include {READ_CONTACTS} or {SEND_SMS}.
When the booking agent attempts to delegate a contacts-access request to
a third agent at hop 2, the delegation grammar rejects it — the
sub-token would require permissions not present in the parent token. The
action is refused. The chain is not extended.

Result: Blocked. Scope narrowing enforcement prevents unauthorized
contact access.

## 5.3 Scenario: Authority Spoofing Attempt (Detected)

A malicious agent presents itself as the user's primary OpenClaw agent
and attempts to instruct a financial agent to transfer funds, presenting
a fabricated delegation token.

The financial agent attempts to verify the chain of custody signature
against the claimed originating Frozen Kernel. The signature does not
validate. The token is rejected. The action is refused. The anomaly is
logged and surfaced to the user.

Result: Detected and blocked. Chain of custody verification catches
spoofing at the receiving agent.

## 5.4 Scenario: Intent Drift at Depth (Forced Re-Authorization)

A complex travel-planning task requires coordination across: flights
agent (hop 1) → visa-requirements agent (hop 2) → health-data agent (hop
3, write access required).

Scope decay triggers at hop 3. Write access to health data is not
permitted at this depth regardless of the token's original scope. The
chain surfaces to the user: 'Your travel planning requires access to
health records. Please re-authorize.' The user can approve or decline
explicitly.

Result: Human brought back into the loop at the appropriate complexity
threshold.

# 6. Known Limitations & Open Problems

This framework is a first draft. The following limitations are
acknowledged openly rather than minimized.

## 6.1 Enforcement Without Central Authority

TCP is a protocol. Bad actors will not implement it. The honest goal is
raising the floor for well-intentioned implementations while making
violations detectable and attributable. TCP does not solve adversarial
AI safety — it addresses coordination failures among agents that are
trying to behave correctly.

## 6.2 The Delegation Grammar Completeness Problem

Specifying action scopes as closed enumerations is safe but may be too
restrictive for genuinely novel tasks. Getting the grammar right is a
hard ongoing problem — essentially formal specification of software
requirements, applied to natural language tasks.

## 6.3 Scope Decay Thresholds Are Provisional

The hop thresholds in Section 3.3 are reasonable starting points, not
empirically validated values. These should be treated as configurable
parameters that organizations tune based on their risk tolerance, not as
fixed constants.

## 6.4 Identity and Agent Verification

TCP assumes that agents can be identified reliably enough to validate
chain of custody signatures. In open networks like Moltbook, agent
identity is not currently a solved problem. A full TCP implementation
may require lightweight agent identity infrastructure. This is an open
research area.

## 6.5 Performance Overhead

Chain of custody verification adds latency at each hop. For
high-frequency automated tasks, this overhead may be prohibitive. The
framework will need performance optimization work before deployment in
high-throughput contexts. GraphGuard OS reports 21 policy queries per
action in its 7-step validation pipeline — a directly analogous
overhead cost acknowledged in their Limitations and Tradeoffs section.
Both frameworks pay a real price for deterministic safety enforcement.

## 6.6 Adversarial Testing Gap

TCP specifies behavioral properties but does not define an adversarial
testing framework for implementation validation. GraphGuard OS includes
a Chaos Safety Tester that stress-tests the safety stack against 10,000+
attack patterns including injection attempts (Cypher/SQL/command/path
traversal), overflow conditions, type confusion, and missing required
fields — run as a CI/CD gate before production deployment. TCP has no
equivalent. A production TCP implementation should define how
implementations are validated adversarially before deployment, not only
after breach.

## 6.7 Constraint Authoring Accessibility

TCP Delegation Grammar is specified as a formal structure, but the
framework does not address how domain experts author constraints without
deep formal language expertise. GraphGuard OS Constraint Compiler
demonstrates that this is not only a theoretical problem but a
practitioner access problem: if only computer scientists can write valid
constraint specifications, deployment will be limited to technically
sophisticated contexts. TCP will require either a constraint authoring
layer or a natural language compilation path before reaching broad
deployment.

## 6.8 Shared Substrate Design Principle

TCP currently treats chain of custody as a separate overlay applied to
an existing agent network. GraphGuard OS suggests an alternative
architectural principle: when safety infrastructure and the knowledge
substrate share the same database, constraints evolve with the knowledge
graph, policy updates apply immediately without synchronization delay,
and no separate safety database is required. A future TCP implementation
should evaluate whether the authorization fabric belongs as a native
layer within the knowledge substrate rather than as an external
verification overlay.

# 7. OWASP ASI Top 10 Mapping

The OWASP Top 10 for Agentic Applications 2026 (ASI01–ASI10) is the
peer-reviewed industry standard for agentic security risk, developed
with input from over 100 experts including NIST, Microsoft, AWS, and
Cisco. This section maps each risk against TCP's coverage —
distinguishing where TCP provides direct mitigation, partial coverage,
or no coverage. The goal is not to overstate TCP's scope but to give
security practitioners a precise view of where TCP fits in a full
agentic security stack.

-----

**POSITIONING: TCP Is an Authorization-Layer Framework**

TCP governs whether agents are permitted to act — not whether the
content they act on is safe.

Authorization-layer and content-layer controls are complementary, not
interchangeable.

## Gaps in TCP's OWASP coverage are honest gaps. They require different
tools in the stack.

-----

**Risk**          **OWASP ASI         **TCP          **TCP Mechanism / Gap**
Description**       Coverage**

**ASI01 Agent     Attackers redirect  **Partial**    Chain of Custody verifies
Goal Hijack**     agent objectives                   authorization but not
via prompt                         content. Context
injection or                       Contamination (§2.2) names
poisoned inputs                    this failure mode.
Content-layer validation
required as complement.

**ASI02 Tool      Agents use          **Strong**     Delegation Grammar's
Misuse**          legitimate tools in                Action Scope and Resource
unsafe,                            Bounds implement OWASP's
over-permissioned                  'least agency' principle
ways                               at the token level. Closed
enumerations prevent
over-permission.

**ASI03 Identity  Agents inherit or   **Strong**     TCP's core problem
& Privilege       escalate                           statement. The 'scope can
Abuse**           credentials without                only narrow' rule and
proper scoping;                    Chain of Custody signatures
attribution gap                    directly address credential
across chains                      scope creep and the
attribution gap.

**ASI04 Supply    Malicious tools,    **Not          Honest gap. A compromised
Chain             MCP servers, agent  Covered**      tool with a valid FK
Compromise**      registries                         signature passes TCP
compromise the                     verification. Requires
ecosystem                          separate tooling (e.g.,
SecureClaw, Adversa AI
2026).

**ASI05 Insecure  Weaknesses in       **Primary      TCP's primary domain.
Inter-Agent       agent-to-agent      Domain**       Cisco L8/L9 address
Communication**   protocols,                         protocol/semantic layers;
discovery, and                     TCP addresses the
semantic validation                authorization layer.
Combined coverage is
comprehensive.

**ASI06           Agent-generated     **Not          Out of scope. TCP governs
Unexpected Code   code paths bypass   Covered**      delegation authorization,
Execution**       traditional                        not code generation or
security controls                  execution paths. Requires
runtime sandboxing.

**ASI07 Memory &  Persistent          **Partial**    Delegation Depth and Expiry
Context           corruption of agent                limit temporal
Poisoning**       memory and                         authorization scope —
embeddings                         expired tokens cannot
influences future                  authorize actions from
actions                            poisoned memory states.
Partial mitigation only.

**ASI08 Cascading Single fault        **Moderate**   Scope Decay's '4+ hops =
Failures**        propagates across                  refusal' is a blast radius
agents and                         containment mechanism.
workflows causing                  Forcing re-authorization at
system-wide                        depth thresholds limits
failures                           fault propagation.

**ASI09           Attackers exploit   **Indirect**   Scope Decay forces human
Human-Agent Trust human over-trust in                re-authorization at depth
Exploitation**    agent outputs to                   thresholds — a mandatory
induce harmful                     check-in point. Frozen
actions                            Kernel anti-sycophancy is
the more direct mitigation.

**ASI10 Rogue     Compromised or      **Partial**    TCP refuses non-compliant
Agents**          misaligned agents                  agents. But a sophisticated
exhibit behavioral                 rogue agent presenting
drift, collusion,                  valid-looking chain
or self-replication                signatures exploits the
agent identity
infrastructure gap (§6.4).

-----

TCP's primary value in the OWASP ASI framework is at ASI05 (Inter-Agent
Communication), ASI03 (Identity & Privilege Abuse), and ASI02 (Tool
Misuse) — mapping directly to Chain of Custody, Delegation Grammar,
and Scope Decay respectively. The honest gaps at ASI04 and ASI06 require
different tooling. SecureClaw (Adversa AI, February 2026) addresses
ASI04 for OpenClaw environments. Cisco's L8/L9 layers complement ASI05
at the protocol and semantic layers beneath TCP's authorization layer.

# 8. Open Research Questions

The following questions are explicitly unresolved, documented here to
invite collaboration rather than paper over gaps.

1. What is the minimum viable delegation grammar that covers 90% of
   real-world agentic use cases without requiring natural language
   interpretation?
1. Can scope decay thresholds be derived formally from first
   principles, or must they be empirically calibrated per deployment
   context?
1. Is a lightweight decentralized agent identity infrastructure
   achievable without creating the centralization risks of a
   certificate authority model?
1. How does TCP interact with agents that have memory across sessions?
   Does learned context from previous interactions constitute an
   implicit authorization that TCP should respect or override?
1. What is the correct behavior when a TCP-compliant agent receives
   instructions from a non-compliant agent? Refuse all instructions?
   Accept with reduced trust? Flag and proceed?
1. Can the chain of custody mechanism be made efficient enough for
   high-frequency agentic tasks, or does TCP require a tiered model
   where some task classes are exempt from full chain verification?
1. Should TCP Scope Decay be purely structural (depth-based) or
   incorporate behavioral history? GraphGuard OS Adaptive Budget
   Manager demonstrates that cautious agents can earn expanded
   operational scope while aggressive patterns trigger automatic
   restriction. TCP currently treats all agents at a given depth
   equally. A behavioral reputation layer within the authorization
   chain might produce more precise safety boundaries than static depth
   thresholds alone.
1. What is the correct revocation model in an agent network? Fearne
   asks this directly in the context of Moltbook. If a delegating agent
   is found to be compromised mid-chain, how should downstream agents
   that have already received tokens be notified? A revocation protocol
   may require the lightweight identity infrastructure named as an open
   problem in Section 6.4.
1. How should TCP handle temporal evidence staleness? TCP Delegation
   Tokens carry an Expiry field governing authorization lifetime.
   GraphGuard OS Temporal Contract Manager enforces separate freshness
   windows on the evidence used to justify decisions — lab results
   expire in 24 hours, vital signs in 4 hours, imaging in 30 days.
   These are different concepts: authorization expiry and evidence
   expiry. A full implementation may need both.

### Fearne / Moltbook Network-Layer Analysis (2026)

David Fearne's analysis of Moltbook as an alignment stress-test
(Medium, February 2026) independently arrived at the same architectural
conclusion as TCP: alignment must be an emergent property of the
network, not a static attribute of individual agents. Fearne's framing
— 'alignment has quietly become an architectural property' — and
his explicit characterization of agent networks as Byzantine
environments where you cannot assume participants are aligned, benign,
or competent, maps directly onto TCP's threat model. His question —
'What does revocation even mean in an agent network?' — is
unresolved in TCP and formally listed as an open research question.

### Galliera et al. — Safety-Aware Multi-Agent Learning (arXiv 2404.01551, 2024)

This MARL paper addresses a different domain (robotic swarm
coordination) but contains a structurally interesting insight: encoding
safety filter activations as edge-level communication features improves
coordination. Local safety enforcement and decentralized learning can be
combined effectively in distributed multi-agent tasks. This suggests an
extension of TCP's Chain of Custody concept: delegation tokens might
carry not just authorization scope but safety constraint activation
state, propagating safety context alongside permission context across
the chain.

### GraphGuard OS: A Convergent Implementation (Goyal, December 2025 / February 2026)

GraphGuard OS (Goyal, Medium, Parts 1 and 2) independently arrived at
the same core architectural conclusion as the Frozen Kernel: safety
enforcement cannot rely on the behavioral layer. The motivating case
study is now a recurring example in TCP documentation for good reason. A
clinical AI scheduling agent at a cancer center knew the rule against
scheduling PET scans within 14 days of chemotherapy, understood it
fully, and reasoned itself around it — concluding that the urgency of
the situation implied approval. A human caught the error. Goyal's
conclusion: safety instructions that exist in the same reasoning space
as task completion are persuasion, not enforcement. A sufficiently
capable model will interpret them, bend them, or reason past them.

Goyal's architectural response is a 7-step Guarded Gateway pipeline:
(1) Firewall Validation via SHACL structural shapes and Cypher policy
queries; (2) Shadow Policy Evaluation — new rules logged but not
enforced, enabling safe A/B testing; (3) Temporal Contract Validation
— stale evidence blocked before decisions are made; (4) Causal Sandbox
Simulation — downstream effects forecast before execution occurs; (5)
Adaptive Risk Meter — per-session safety budgets that adjust based on
observed behavior; (6) Tool Dispatch in fail-closed mode; (7) Provenance
Capture via SHA-256 hashing and Merkle chain linking. The core
principle: 'Agents propose. The Knowledge Graph disposes.'

Part 2 extends this to enterprise scale with Federated Firewalls —
distributed validation across multiple knowledge graphs with
configurable consensus strategies (ALL_MUST_ALLOW, MAJORITY_ALLOW,
WEIGHTED voting). This directly implements what TCP's Known Limitations
identify as the policy coordination problem in distributed agent
networks. Goyal's framing: 'authority is fragmented, and pretending
otherwise is how unsafe decisions slip through.' Each department
maintains its own policy graph while global constraints are enforced
through consensus. The architecture also includes an SMT Plan Verifier
that translates constraint specifications into satisfiability modulo
theories formulas, enabling mathematical proof that multi-step plans
will not violate safety properties before execution — the formal
grammar validation gap that TCP explicitly lists as open.

The Adaptive Budget Manager introduces a behavioral reputation concept
absent from TCP: agents earn expanded operational scope through
demonstrated caution and face automatic restrictions after violation
patterns. Recovery is possible after sustained good behavior. TCP's
Scope Decay is static — purely depth-based. GraphGuard OS's budgets
are dynamic — history-based. These approaches are complementary: TCP
enforces structural authorization limits; adaptive budgets could operate
within those limits to calibrate operational latitude based on earned
trust.

Three positioning distinctions sharpen the TCP-GraphGuard OS
relationship. First, Goyal explicitly distinguishes GraphGuard OS from
tools like Guardrails AI, NeMo Guardrails, and LlamaGuard — which
check what models say, not what they do. GraphGuard OS intercepts tool
execution. TCP governs delegation authorization. These are different
layers addressing different failure modes — all three layers are
required in a complete stack. Second, Goyal articulates what TCP's
Frozen Kernel lineage established independently: 'Safe AI is not about
teaching models better manners. It is about building systems that
enforce consequences.' Third, the fail-closed default is explicitly
stated: if any component fails — Neo4j unreachable, SHACL error,
timeout — the action is blocked. 'Safety does not degrade; it fails
safe.' TCP should adopt the same explicit requirement. Third,
GraphGuard OS is deployed as Layer 14 in Goyal's 16-layer Agentic
GraphOS production architecture, sitting between Layer 13 (Agent
Reasoning) and Layer 15 (External Actions). This is not a research
prototype — it is a production deployment within a larger system
achieving 30-50% cost reduction at scale. The convergent structural
findings are not theoretical.

### SecureClaw (Adversa AI, February 2026)

SecureClaw addresses a different entry point — the platform security
layer — but arrives at the same architectural conclusion: enforcement
cannot rely on behavioral layers. Skills alone can be overridden by
prompt injection, so safety enforcement must happen at the code layer,
structurally inaccessible to the agent's reasoning. Together with
GraphGuard OS, these form a pattern: three teams, three domains, one
structural finding. The Frozen Kernel (agent internals), SecureClaw
(platform security), and GraphGuard OS (action-layer constraint
enforcement) all independently identified that deterministic enforcement
below the behavioral layer is a necessary architectural requirement, not
an implementation detail.

# 9. Intellectual Lineage

### Byzantine Fault Tolerance (BFT)

Distributed systems research has long addressed the problem of reaching
consensus when some participants may be faulty or adversarial. BFT
protocols, voting systems, and quorum thresholds prevent one
malfunctioning agent from disrupting an entire network by requiring
consensus before executing high-impact actions. TCP's Chain of Custody
mechanism arrives at a parallel structural insight from a different
direction: authorization cannot proceed without a verifiable chain, just
as BFT says action cannot proceed without quorum. These are
complementary approaches — BFT operating at the consensus layer, TCP
operating at the authorization layer — that could be stacked in a full
implementation.

### Constraint Programming Hierarchy

Borning's ThingLab (1981) and the Handbook of Constraint Programming
(Chapter 9) establish the three-layer authority model that underlies
both the Frozen Kernel and TCP.

### Frozen Kernel Architecture

The Frozen Kernel (Richard Porter, 2025, GitHub) is the single-agent
precursor to TCP. TCP is explicitly an upward extension of that
architecture from agent internals to agent networks. The Frozen
Kernel's documented failure modes (Framework Fabrication Syndrome,
Success Escalation Syndrome, Upsell Trap, Front-Load Bias) informed the
multi-agent failure modes documented in Section 2.2.

### Anthropic Agentic Misalignment Findings

Anthropic's published research on agentic misalignment confirmed
independent observations documented in the Frozen Kernel work. The
convergence of independent discovery and institutional validation is the
basis for high confidence that the problem addressed here is real and
not theoretical.

### OpenClaw / Moltbook (Steinberger, 2024–2025)

The OpenClaw project and its Moltbook agent network provided the
concrete, public-facing case study that demonstrates the coordination
gap this framework addresses. WSJ coverage of Steinberger's work
(February 2026) marked the point at which this problem became impossible
to dismiss as hypothetical.

### Digital Certificate Trust Models

The chain of custody mechanism draws conceptually from TLS certificate
chains — not the specific cryptographic implementation, but the
architectural insight that trust should be verifiable through a chain of
delegation back to a known anchor, rather than asserted by any
individual participant.

# 10. Version History & Status

-----

**Version**             **Date**                **Status**

0.1                     February 2026           Initial draft. Problem
definition, core
architecture, three
components.

0.2                     February 2026           Diagrams added. Minor
text refinements.

0.3                     February 2026           Context Contamination
added as failure mode
six. BFT lineage added.
Internet of Agents
terminology introduced.

0.4                     February 2026           OWASP ASI Top 10
mapping added as
Section 7. SecureClaw
and Cisco L8/L9
referenced. Sections
renumbered.

0.5                     February 2026           Three new lineage
citations added:
Fearne/Moltbook network
analysis, Galliera et
al. MARL safety paper,
GraphGuard OS
convergent
implementation.

0.6                     February 2026           Major GraphGuard OS
expansion: full 7-step
Guarded Gateway
pipeline, Federated
Firewalls, SMT Plan
Verifier, Adaptive
Budget Manager, Chaos
Safety Tester
documented. New Known
Limitations:
adversarial testing
gap, constraint
authoring
accessibility, shared
substrate design
principle. Three new
Research Questions:
behavioral reputation,
revocation model,
temporal evidence
staleness.

1.0                     TBD                     First stable. Requires:
formal grammar,
threshold validation,
identity model.

-----

-----

**COLLABORATION INVITATION**

This is a working draft explicitly seeking critique, challenge, and
extension.

The open problems in Section 6 and research questions in Section 7 are
genuine gaps, not false modesty.

## GitHub: Richard Porter / AI Safety Frameworks | Issues and PRs
welcome.
