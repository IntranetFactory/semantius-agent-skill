AGENTS.md

## Pre-Commit Requirements

IMPORTANT: Before committing ANY changes, you MUST run the CI checks locally to verify they pass:

```bash
npm install  # if node_modules doesn't exist
npm run ci:check
```

Only commit after `ci:check` passes. If it fails, fix the issues first (usually `npm run format` or `npm run lint` will auto-fix).
