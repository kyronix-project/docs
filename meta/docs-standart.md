# Technical Documentation Standard for LLM

> **Purpose:** This standard defines the structure and style of technical documentation optimized for perception and processing by Large Language Models (LLMs), while preserving the stylistic and informational density adopted in the Managarm project.

---

## 1. General Principles

### 1.1. Hierarchical Organization
- Documentation follows the **"context pyramid"** principle: from a general table of contents to narrowly focused instructions.
- The root file (`SUMMARY.md` or `index.md`) serves as a **system prompt**, containing only section titles and links. This allows the LLM to instantly determine the location of any topic.

### 1.2. Single Responsibility Principle
- Each file is responsible for **exactly one topic**, as indicated by its name and path.
- **Bad:** `building.md`, mixing Docker, manual builds, and package updates.
- **Good:** `building/cbuildrt.md`, `building/with-docker.md`, `building/updating.md`.

### 1.3. Predictable Navigation
- All transitions between sections are made through **explicit links** of the form `[text](path/to/file.md#anchor)`.
- It is forbidden to use phrases like "as mentioned earlier" without a link — this breaks the LLM's context.

---

## 2. Document Structure

### 2.1. Heading (H1)
- Must clearly and unambiguously define the content of the section.
- **Format:** `# Topic Name`
- **Example:** `# Building Managarm with cbuildrt`

### 2.2. Introduction (Preamble)
- The first 1–2 sentences after the heading must answer:
  - *What is this document about?*
  - *Where are we in the overall structure?*
- **Format:** A direct statement that introduces the context.
- **Example:** `This section explains how to build Managarm in a Docker environment.`

### 2.3. Main Body
- **Step-by-step instructions** — formatted as numbered lists (`1.`, `2.`, `3.`).
- **Enumerations of options/rules** — formatted as bulleted lists (`-`, `*`).
- **Commands and code** — placed in code blocks with language specification (e.g., ```bash, ```yaml, ```c).
- **Important notes** — highlighted using admonition blocks (see Section 3).

### 2.4. Conclusion (Optional)
- May contain links to related sections or a final verification of the result.

---

## 3. Callouts / Admonitions

Standardized markers are used to highlight critically important information:

| Marker | Purpose | Example |
|--------|---------|---------|
| `> **Note:**` | Additional information that does not affect the main process | `> **Note:** we recommend using cbuildrt as it is faster.` |
| `> **Important:**` | A mandatory condition critical to the success of the operation | `> **Important:** the src_mount and build_mount paths cannot be changed after starting the build.` |
| `> **Warning:**` | A warning about risks, errors, or non-recommended actions | `> **Warning:** support for building without containers is not a priority for the project.` |
| `> **Tip:**` | Helpful advice that simplifies the developer's life | `> **Tip:** use git commit --fixup to amend early commits.` |

---

## 4. Terminology and Glossary

### 4.1. Introducing New Terms
- Each technical term (e.g., `xbstrap`, `cbuildrt`, `bragi`, `hel`) must:
  1. Be **defined** at its first mention in the document.
  2. Be **linked** to a separate reference file in the table of contents (e.g., `sys-arch/bragi/index.md`).

### 4.2. Naming Style
- **Package/command names:** formatted in monospace (`` `xbstrap` ``).
- **File and path names:** formatted in monospace (`` `bootstrap-site.yml` ``).
- **System module names:** written in lowercase in the text, but capitalized in section headings (e.g., `thor` in text, but `# Thor` in the heading).

---

## 5. Code and Command Style

### 5.1. Terminal Commands
- All commands must be ready for copy-pasting.
- Environment variables or placeholders are used, explicitly indicated as `~/managarm`, `/path/to/rootfs`.
- **Example:**
  ```bash
  mkdir ~/managarm && cd ~/managarm
  git clone https://github.com/managarm/bootstrap-managarm.git src
  ```

### 5.2. Configuration Files
- Complete config examples are always placed in code blocks with format specification (`yaml`, `json`, `toml`).
- Critical parameters are highlighted with comments inside the block or explanations outside it.
- **Example:**
  ```yaml
  container:
    runtime: cbuildrt
    rootfs: /path/to/your/rootfs  # Replace with the actual path
  ```

---

## 6. Reference System

- **Internal links:** always specified relative to the documentation root.
- **External links:** only allowed for official sources (GitHub, official websites).
- **Anchors:** for navigation within large documents, anchors of the form `#section-name` are used.
- **Example:** `[Building](building/index.md#building)`

---

## 7. Prohibited Practices

- ❌ **Information duplication** — instead of copying, use links to the original source.
- ❌ **Mixing abstraction levels** — do not mix architectural concepts and step-by-step commands in the same file.
- ❌ **Implicit assumptions** — do not write "obviously..." or "as everyone knows...". The LLM does not know what is "obvious" to a human.
- ❌ **Lack of context** — each file must be self-contained when read together with its heading and first paragraph.

---

## 8. Document Validation Checklist

Before publication, verify:

- [ ] The table of contents (`SUMMARY.md`) has been updated and contains a link to the new file.
- [ ] The file begins with an H1 heading and an introductory paragraph.
- [ ] All instructions are numbered and step-by-step.
- [ ] All commands have been tested for correctness.
- [ ] All terms are defined or have a link to the glossary.
- [ ] Appropriate attention blocks (`Note`, `Warning`, `Important`) have been used.
- [ ] There is no duplication — links to related sections are present.

---

**This standard is mandatory for all technical documents intended for integration with LLM systems, and guarantees uniformity of style, high informativeness, and predictability of perception.**