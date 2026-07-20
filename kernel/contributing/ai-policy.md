# AI Policy

Kyronix accepts contributions that are generated or assisted by AI tools, subject to the following conditions.

## Requirements

1. **Disclosure:** All AI-generated or AI-assisted contributions must be clearly disclosed in the pull request description. State which AI tool was used and how it was used.

2. **Human Review:** AI-generated code must be reviewed by a human before submission. The human reviewer is responsible for the correctness, safety, and quality of the code.

3. **Testing:** AI-generated code must pass the full test suite. The contributor must verify that the code does not introduce regressions.

4. **Attribution:** If the AI tool generates substantial portions of the code, the contribution should note this in the commit message footer (e.g., `AI-assisted: Copilot`).

## Prohibited Practices

- Submitting AI-generated code without human review or testing.
- Using AI to generate commits that intentionally obscure the nature of the change.
- Using AI to bypass code review processes.

## Recommended Use Cases

AI tools are particularly useful for:

- Generating boilerplate code (e.g., syscall dispatch entries, struct definitions)
- Writing documentation (subject to human review for accuracy)
- Suggesting fixes for bugs identified by static analysis
- Translating code between languages or formats

> **Note:** The quality standards for AI-generated code are identical to those for human-written code. There is no separate standard or exemption for AI contributions.
