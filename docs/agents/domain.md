# Domain Docs

How engineering skills should consume this repository's domain documentation.

## Before exploring, read these

- `CONTEXT.md` at the repository root.
- `CONTEXT-MAP.md` if it exists; read each context relevant to the task.
- Relevant decisions under `docs/adr/`.

If these files do not exist, proceed silently. The domain-modeling skill creates them when terminology or architectural decisions are established.

## File structure

This repository currently uses a single-context layout:

```
/
├── CONTEXT.md
├── docs/adr/
└── docs/agents/
```

## Use the glossary's vocabulary

Use domain terms as defined in `CONTEXT.md`. Avoid synonyms that its glossary explicitly excludes.

If a required concept is missing, reconsider whether the term belongs to the project or record the gap for domain modeling.

## Flag ADR conflicts

Explicitly identify output that conflicts with an existing ADR rather than silently overriding it.
