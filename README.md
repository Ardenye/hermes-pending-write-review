# Pending Write Review

A Hermes Skill for reviewing staged Skill and Memory writes in bounded, auditable batches. It is designed for users who want to inspect proposals before they affect persistent Skills or Memory; it does not bypass Hermes approval gates.

## Repository layout

```text
skills/
└── pending-write-review/
    └── SKILL.md
```

## Install from a published GitHub repository

Replace `<owner>/<repo>` with the repository path after it is published:

```bash
hermes skills tap add <owner>/<repo>
hermes skills search pending-write-review --source <owner>/<repo>
hermes skills install <owner>/<repo>/pending-write-review
```

## Before publication

- Public author attribution: `Arden (Ardenye), Hermes Agent`.
- License: MIT (see `LICENSE`).
- Review the Skill against the current Hermes CLI and approval-handler behavior; this draft intentionally requires runtime verification for implementation details that may change.
