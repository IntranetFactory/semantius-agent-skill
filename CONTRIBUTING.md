# CONTRIBUTING.md

Thank you for contributing to Semantius Agent Skills! Here's how to get started:

[1. Getting Started](#getting-started) | [2. Issues](#issues) |
[3. Pull Requests](#pull-requests) | [4. Contributing New References](#contributing-new-references) |
[5. Creating a New Skill](#creating-a-new-skill)

## Getting Started

To ensure a positive and inclusive environment, please read our code of conduct before contributing.

## Issues

If you find a typo, have a suggestion for a new skill/reference, or want to improve
existing skills/references, please create an Issue.

- Please search existing Issues before creating a new one.
- Please include a clear description of the problem or suggestion.
- Tag your issue appropriately (e.g., `bug`, `question`, `enhancement`,
  `new-reference`, `new-skill`, `documentation`).

## Pull Requests

We actively welcome your Pull Requests! Here's what to keep in mind:

- If you're fixing an Issue, make sure someone else hasn't already created a PR
  for it. Link your PR to the related Issue(s).
- We will always try to accept the first viable PR that resolves the Issue.
- If you're new, we encourage you to take a look at issues tagged with `good first issue`.
- If you're proposing a significant new skill or major changes, please open a
  Discussion first to gather feedback before investing time in implementation.

### Pre-Flight Checks

Before submitting your PR, please run these checks:

```bash
npm run build  # Regenerate AGENTS.md files
npm run check  # Format and lint
```

## Contributing New References

To add a reference to an existing skill:

1. Navigate to `skills/{skill-name}/references/`
2. Create a new markdown file for your reference
3. Write explanation and examples
4. Run validation and update navigation:

```bash
npm run build
npm run check
```

## Creating a New Skill

Skills follow the [Agent Skills Open Standard](https://agentskills.io/).

### 1. Create the directory structure

```bash
mkdir -p skills/my-skill/references
```

### 2. Create SKILL.md

```yaml
---
name: my-skill
description: Brief description of what this skill does and when to use it.
license: MIT
metadata:
  author: your-org
  version: "1.0.0"
  organization: Your Org
  date: January 2026
  abstract: Detailed description of this skill.
---

# My Skill

Instructions for agents using this skill.

## References

- https://example.com/docs
```

### 3. Create reference files

Add markdown files to the `references/` directory with detailed documentation.

### 4. Validate and build

```bash
npm run build
npm run check
```

## Questions or Feedback?

- Open an Issue for bugs or suggestions
- Start a Discussion for broader topics or proposals
- Check existing Issues and Discussions before creating new ones

## License

By contributing to this repository, you agree that your contributions will be
licensed under the MIT License.
