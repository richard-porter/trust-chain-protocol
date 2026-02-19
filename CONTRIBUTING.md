# Contributing to the Trust Chain Protocol

This is a working draft. Version 1.0 is not yet stable — it requires formal grammar definition, threshold validation, and an agent identity model.

That means contributions are both welcome and genuinely useful right now.

## What’s Most Needed

The open research questions in the README are the best starting points. Specifically:

- **Agent identity infrastructure** — TCP assumes agents can be identified well enough to validate signatures. In open networks, this is unsolved. Lightweight approaches that avoid central authority are the priority.
- **Delegation grammar completeness** — Can a closed enumeration cover 90% of real-world agentic use cases without requiring natural language interpretation?
- **Scope decay threshold validation** — The hop thresholds are reasonable starting points, not empirically validated. If you have data or a principled argument for different thresholds, open an issue.
- **Non-compliant agent handling** — What is the correct behavior when a TCP-compliant agent receives instructions from a non-compliant one?

## How to Contribute

- **Open an Issue** to raise a question, flag a gap, propose an alternative, or point to related work TCP should reference.
- **Open a Pull Request** to propose specific changes to the protocol text or the OWASP ASI mapping.
- **Point to independent implementations** — if you’ve built something that addresses the same problems TCP addresses, a reference is valuable even if the approaches differ.

## What This Is Not Looking For

- Feature requests that expand TCP’s scope beyond authorization-layer governance
- Suggestions to replace the Frozen Kernel dependency with probabilistic alternatives
- Implementations that require central certificate authority infrastructure

## License

Released for public benefit under Creative Commons Attribution 4.0. Attribution appreciated but not required.

The only ask: **keep humans in the authorization chain.**
