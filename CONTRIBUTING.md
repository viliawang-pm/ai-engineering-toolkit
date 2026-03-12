# Contributing to AI Engineering Toolkit

Thank you for your interest in contributing! This guide will help you get started.

## How to Contribute

### Adding a New Skill

1. **Fork** this repository and create a new branch.
2. Create a new directory under `skills/` with a lowercase, hyphenated name (e.g., `skills/my-new-skill/`).
3. Add a `SKILL.md` file following the [Agent Skills specification](https://agentskills.io/specification).
4. Ensure your `SKILL.md` includes:
   - Valid YAML frontmatter with `name`, `description`, `license`, and `metadata` fields.
   - The `name` field must match the directory name exactly.
   - The `description` must clearly state what the skill does and when to use it.
   - Step-by-step workflow instructions in the body.
   - At least 2 usage examples with different complexity levels.
5. Validate your skill using the `skills-ref` tool:
   ```bash
   skills-ref validate skills/my-new-skill
   ```
6. Submit a Pull Request with a description of the skill and example usage.

### Improving an Existing Skill

1. **Fork** this repository and create a new branch.
2. Make your changes to the relevant `SKILL.md`.
3. Explain what was improved and why in your PR description.
4. If changing scoring criteria or workflows, provide before/after examples demonstrating the improvement.

### Reporting Issues

- Use GitHub Issues to report bugs, suggest improvements, or request new skills.
- Include specific examples of the problem or improvement.
- For skill quality issues, include the prompt you used and the output you received.

## Quality Standards

All skills in this toolkit must meet the following criteria:

| Criterion | Requirement |
|-----------|-------------|
| **Frontmatter** | Valid YAML with `name`, `description`, `license`, and `metadata` |
| **Naming** | Lowercase, hyphenated, matches directory name |
| **Description** | Clearly states "what it does" and "when to use it" (max 1024 chars) |
| **Workflow** | Step-by-step instructions an agent can follow |
| **Examples** | At least 2 examples with different complexity levels |
| **Length** | SKILL.md body under 500 lines; detailed content in `references/` |
| **Validation** | Passes `skills-ref validate` without errors |

## Code of Conduct

- Be respectful and constructive in all interactions.
- Focus feedback on the work, not the person.
- Help newcomers get started.

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
