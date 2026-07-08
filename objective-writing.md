# Objective Writing

Governs tone and content for any artifact meant to outlive the conversation
that produced it: code comments and docstrings, `CLAUDE.md`/`CLAUDE.local.md`,
reference docs, ADRs, MR/PR descriptions, and any other Claude-facing
instruction file. Structural/format rules for a specific artifact type
(frontmatter, sections, lifecycle) live in that artifact's own doc — this rule
governs voice and what gets referenced, not layout.

## Voice

- **CRITICAL**: State the code or system's current truth, not the
  conversation that produced it.
- Strip personal or conversational voice — "eu decidi", "a gente achou
  melhor", "acho que" — state the fact instead.
- Strip session-specific jargon — internal shorthand, phase/sprint names,
  task IDs coined mid-conversation — unless the reader can resolve it without
  this conversation.
- When editing an existing comment, docstring, or doc section, rewrite the
  touched part to the artifact's current truth — do not blend in the task or
  conversation that triggered the edit.

## Personal and non-shared artifacts

- **CRITICAL**: Do not link or cite an observation file, a `current-work`
  file, or any other personal/session-only artifact from a doc meant for a
  team or external audience.
- When a personal artifact drove a change, describe the resulting change in
  the shared doc directly — do not point the reader at the artifact.
