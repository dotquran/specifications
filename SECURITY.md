# Security Policy

## Scope

This repository contains only the **dotquran format specification** — prose documents and JSON Schemas. It does not contain any executable code, server software, or cryptographic implementation.

Security reports against this repository are therefore scoped to:

- **Specification-induced vulnerabilities** — normative rules in the spec that, if implemented as written, would create security weaknesses in conforming applications (e.g., rules that permit directory traversal paths, require processing of unbounded input, or mandate behaviors that conflict with safe parsing practices).
- **Schema weaknesses** — JSON Schema definitions that permit values which a conforming implementation would need to handle in an unsafe way.

If you have found a vulnerability in a **specific implementation** of the dotquran format (a player, validator, converter, etc.), please report it to the maintainers of that implementation directly. This repository is not the right place for such reports.

---

## Reporting a Vulnerability

If you have identified a specification-level security concern, please **do not open a public issue**. Instead:

1. Open a [GitHub Security Advisory](https://github.com/dotquran/specifications/security/advisories/new) on this repository (preferred).
2. Or email the maintainers privately using the contact information available in the repository profile.

Include in your report:

- The specific document and section (e.g., `specs/01-archive-structure.md` §4.3).
- A description of the normative rule or schema property you believe is problematic.
- A concrete description of how a conforming implementation following that rule would be made vulnerable.
- If possible, a suggested remediation or amended normative text.

---

## Response Process

| Step | Target timeframe |
|---|---|
| Acknowledgement of report | 5 business days |
| Initial assessment | 14 days |
| Resolution (spec errata or amendment) | Dependent on severity and complexity |

Reports will be kept confidential until a fix is published, at which point the reporter will be credited in the changelog unless they request otherwise.

---

## Known Considerations for Implementors

The following points are not vulnerabilities in this specification but are advisory notes for implementors of conforming applications:

- **Archive extraction:** A dotquran archive is a ZIP file. Implementations **MUST** validate that all internal paths (in particular `audio/<filename>`) do not contain path traversal sequences (e.g., `../`) before extracting to disk.
- **JSON parsing:** `manifest.json` and `sync.json` may be large. Implementations should apply reasonable size limits before parsing.
- **`id` field:** The `id` URN field in `manifest.json` is user-supplied. Implementations **MUST NOT** use it as a filesystem path or SQL identifier without sanitization.
- **Audio path:** The `audio.path` field in `manifest.json` is a relative path within the archive. Implementations **MUST** reject values that resolve outside the archive root.
