# Archer Instructions For Claude-Compatible Agents

The canonical project guidance is in [AGENTS.md](AGENTS.md). Read it before changing this workspace.

Use the following authority split:

- [SPEC.md](SPEC.md) defines product features, user workflows, scope, and acceptance criteria.
- [AGENTS.md](AGENTS.md) defines repository boundaries, engineering invariants, change rules, and validation expectations.
- `archer-api/README.md` and `archer-app/README.md` define setup and developer usage for their respective projects.

Do not create alternate copies of the specification. Keep changes within the project that owns the behavior, preserve the API boundary, and run the narrowest relevant validation before handoff.
