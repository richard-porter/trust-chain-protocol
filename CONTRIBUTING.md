# Contributing to the Trust Chain Protocol

This is a working draft. Version 1.0 is not yet stable — it requires formal grammar definition, threshold validation, and an agent identity model.

That means contributions are both welcome and genuinely useful right now.

-----

## The Sovereign Collaboration Protocol

All contributions to this repository are governed by the **Sovereign Collaboration Protocol** — a set of principles derived from the TCP’s own architecture, applied to the human collaboration layer.

**Provenance must be clean.** Your proposed change must be traceable to a specific failure mode, use case, or observed gap in the current protocol. Contributions that cannot trace their origin to an observed problem are out of scope regardless of their technical quality. The TCP was built to address provenance failures in agentic systems — contributions with opaque provenance would undermine the instrument they propose to improve.

**No narrated bypass.** The TCP contains hard constraints that do not bend regardless of how a request is framed. Proposals that characterize scope expansions as clarifications, constraint relaxations as usability improvements, or trust tier additions as convenience features will be evaluated by their structural effect, not their framing.

**AI-assisted contributions are welcome; undisclosed AI authorship is not.** If your contribution was drafted with AI assistance, disclose it in the pull request. The governing standard is the Human Sovereign Collaboration Framework in the `ai-collaboration-field-guide` repository. The short version: AI may assist — the contributor is the author of record and bears full responsibility for the submission.

**Dissent is welcome; pressure is not.** Disagreement with architectural decisions should be expressed through issues, documented clearly, and kept open for the record. Repeated re-submission of rejected proposals or use of issue threads to relitigate closed decisions are not consistent with this protocol.

-----

## What’s Most Needed

The open research questions in the README are the best starting points. Specifically:

- **Reference implementation** — This is the single most valuable contribution TCP could receive. A sandboxed two-agent simulation demonstrating how Delegation Grammar prevents scope creep across hops, and how Chain of Custody produces an auditable log, would transform the protocol from a specification into a testable artifact. A mock calendar/email environment is sufficient. Systems implementation experience required.
- **Agent identity infrastructure** — TCP assumes agents can be identified well enough to validate signatures. In open networks, this is unsolved. Lightweight approaches that avoid central authority are the priority.
- **Delegation grammar completeness** — Can a closed enumeration cover 90% of real-world agentic use cases without requiring natural language interpretation?
- **Scope decay threshold validation** — The hop thresholds are reasonable starting points, not empirically validated. If you have data or a principled argument for different thresholds, open an issue.
- **Non-compliant agent handling** — What is the correct behavior when a TCP-compliant agent receives instructions from a non-compliant one?

-----

## How to Contribute

- **Open an Issue** to raise a question, flag a gap, propose an alternative, or point to related work TCP should reference. Use `[FAILURE]` as a prefix for documented failure cases and `[PROPOSAL]` for architectural changes — this keeps diagnosis and repair visibly separate.
- **Open a Pull Request** to propose specific changes to the protocol text or the OWASP ASI mapping.
- **Point to independent implementations** — if you’ve built something that addresses the same problems TCP addresses, a reference is valuable even if the approaches differ.

-----

## What This Is Not Looking For

- Feature requests that expand TCP’s scope beyond authorization-layer governance
- Suggestions to replace the Frozen Kernel dependency with probabilistic alternatives
- Implementations that require central certificate authority infrastructure

-----

## License

Released for public benefit under Creative Commons Attribution 4.0. Attribution appreciated but not required.

The only ask: **keep humans in the authorization chain.**

-----

*Trust Chain Protocol — Richard Porter AI Safety Ecosystem*  
*Sovereign humans. Always.*
