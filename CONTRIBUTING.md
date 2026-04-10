# Contributing to the dotquran Specification

Thank you for your interest in improving the dotquran specification. This document describes how to participate effectively.

---

## Table of Contents

- [What This Repository Contains](#what-this-repository-contains)
- [Kinds of Contributions](#kinds-of-contributions)
- [Before You Start](#before-you-start)
- [Opening an Issue](#opening-an-issue)
- [Submitting a Pull Request](#submitting-a-pull-request)
- [Writing Style Guidelines](#writing-style-guidelines)
- [Schema Changes](#schema-changes)
- [Versioning](#versioning)

---

## What This Repository Contains

This repository holds the **dotquran format specification** only — prose documents and JSON Schemas. It does not contain any implementation code. Contributions affect the definition of the format, not any runtime behavior.

---

## Kinds of Contributions

| Type | Description |
|---|---|
| **Clarification** | Fix ambiguous or contradictory normative language |
| **Editorial** | Fix typos, grammar, broken links, formatting |
| **Bug fix** | Correct a schema or spec rule that produces an incorrect or unimplementable result |
| **Extension proposal** | Add new fields, new granularity levels, or new format capabilities |
| **Schema update** | Adjust a JSON Schema to match the prose specification |

---

## Before You Start

**For editorial and clarification fixes:** Feel free to open a pull request directly.

**For bug fixes:** Open an issue first (use the *Specification Bug* template) so the problem can be reproduced and agreed upon before a fix is committed.

**For extension proposals or structural changes:** Open an issue first (use the *Extension Proposal* template). Major changes to the data model or archive structure require design discussion before implementation work begins. A pull request submitted without a preceding issue for a significant change is likely to be held pending that discussion.

---

## Opening an Issue

Use the issue templates provided:

- **Specification Bug** — for normative language that is incorrect, contradictory, or unimplementable.
- **Ambiguity Report** — for language that is technically valid but unclear enough to produce divergent implementations.
- **Extension Proposal** — for new fields, new features, or changes to the data model.

When describing a problem, cite the specific section and document (e.g., *`specs/03-synchronization-protocol.md` §4.2*) and quote the exact normative statement you believe is incorrect or unclear.

---

## Submitting a Pull Request

1. **Fork** the repository and create a branch from `main`.
2. Make your changes. Keep each pull request focused on a single concern.
3. **Reference the issue** your PR addresses in the description (e.g., `Closes #12`).
4. Ensure your changes comply with the [Writing Style Guidelines](#writing-style-guidelines) below.
5. If your change affects a JSON Schema, update both the schema file and the corresponding prose specification.
6. Open the PR against the `main` branch.

Pull requests may be reviewed by maintainers or community members. Be responsive to review feedback; PRs with no activity for 30 days may be closed.

---

## Writing Style Guidelines

All specification prose must follow these rules:

### RFC 2119 Keywords

Normative requirements **MUST** use the key words defined in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt):

> **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, **OPTIONAL**

These words **MUST** appear in **ALL CAPS** when used in their normative sense. Do not use them in lowercase for normative requirements, and do not use them in ALL CAPS for non-normative observations.

### Formatting

- Section headings use Markdown ATX style (`##`, `###`).
- Normative JSON examples must be valid against the corresponding schema.
- Tables use the `|---|` separator row format consistent with existing documents.
- File paths are wrapped in backticks (e.g., `` `manifest.json` ``).
- Cross-references to other spec documents use relative Markdown links.

### Tone

Write in a precise, technical register consistent with IETF/W3C specification style. Avoid colloquial language in normative sections.

---

## Schema Changes

The JSON Schema files in `schemas/` are normative. When prose and schema conflict, the schema governs structural validation and the prose governs semantic interpretation (see `specs/02-manifest-format.md` §1 for the stated rule).

If you change a schema:

- Ensure it validates all examples given in the corresponding spec document.
- Note in your PR description which spec section the schema change corresponds to.
- Do not introduce breaking changes to the schema without a corresponding version bump discussion.

---

## Versioning

The specification is currently at **1.0.0-draft**. There is no formal release process yet. Version bumps will be discussed and decided by maintainers when the specification stabilizes.

Do not change the `"Version:"` header in spec documents in a PR unless the PR is explicitly a version bump.
