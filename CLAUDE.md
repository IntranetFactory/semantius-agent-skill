AGENTS.md

## Pre-Commit Requirements

IMPORTANT: Before committing ANY changes to files in `skills/semantius/`, you MUST:

1. **Run the build** to regenerate the skill bundle:
```bash
npm install  # if node_modules doesn't exist
npm install --prefix packages/skills-build  # install build dependencies
npm run build
```

2. **Run CI checks** to verify formatting/linting passes:
```bash
npm run ci:check
```

3. **Commit BOTH** the source files AND the generated `skills/semantius.skill` file.

The `semantius.skill` file is a ZIP archive containing all skill files. It MUST be rebuilt and committed whenever any file in `skills/semantius/` changes.

Only commit after both `build` and `ci:check` pass. If `ci:check` fails, fix the issues first (usually `npm run format` or `npm run lint` will auto-fix).
