# Intercom-managed content

This folder is the source of truth for your Intercom workspace's
**Articles**, **Internal Articles**, and **Content Snippets** —
managed by the Intercom Sync GitHub app.

## How it works

- **Edits in Intercom** flow here automatically as rolling sync PRs
  (one per ~15 minutes of activity).
- **Edits in git** apply to your Intercom workspace when a PR against
  `main` that touches files in this folder is merged.
- Files outside `intercom/` are ignored by the sync.

## Editing

Open a normal pull request against `main` and change the files under
this folder. When the PR merges, your changes apply to Intercom.

See [CLAUDE.md](./CLAUDE.md) for the full directive vocabulary,
frontmatter schema, and cross-reference syntax.

## Don't edit `intercom/docs-sync/*` branches directly

Those are rolling sync branches — Intercom force-pushes them on every
sync. Direct edits are overwritten. Always open a fresh PR against
`main`.
