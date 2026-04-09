# dotquran specification

**Version:** 1.0.0-draft
**Status:** Draft
**License:** CC BY 4.0

---

## Abstract

The **dotquran** initiative defines an open, portable, self-contained file format for Quranic audio recitation. A dotquran archive (`.qrn` or `.quran`) bundles a recitation audio file, structured metadata, and a **relational, non-linear synchronization timeline** into a single transferable unit.

The format is designed to be implementation-agnostic: any conforming application can render a fully synchronized Quran recitation experience without reliance on external APIs, proprietary databases, or network connectivity.

---

## The Problem

Existing synchronized Quran recitation solutions share three fundamental limitations:

### 1. Fragmentation and Vendor Lock-in

Synchronization data is either locked behind proprietary application caches, distributed via non-standard REST APIs, or embedded in app-specific binary formats. There is no portable unit that a user can download once and use across any player or platform.

### 2. Assumed Linear Recitation

All known public sync formats model recitation as a simple ordered list of `[ayah_number, start_time, end_time]` triples. This model is structurally incapable of representing real Tajweed performances, which commonly include:

- **Repetition of complete verses** — a reciter may recite Ayah 5 twice before continuing.
- **Partial repetitions** — a reciter may repeat only the last phrase of an Ayah.
- **Pauses and restarts** — the audio timeline is not monotonically consumed by the text timeline.

Forcing such a performance into a linear model requires either data loss (dropping the repetitions) or data corruption (splitting an Ayah reference across discontinuous segments with no formal mechanism to express the relationship).

### 3. Offline Inaccessibility

Because sync data and audio are distributed separately, a user who downloads a recitation MP3 cannot portably carry the synchronization data alongside it. The audio and its timing context are structurally decoupled.

---

## The Solution

A dotquran archive is a single ZIP file containing:

| Component | Description |
|---|---|
| `manifest.json` | Structured metadata about the reciter, tilawa style, qira'at, and coverage |
| `sync.json` | The relational synchronization timeline |
| Audio file | The recitation audio (`.opus` recommended, others supported) |

The critical architectural innovation is in `sync.json`. Rather than a flat list, it separates the **audio domain** (what spans of audio exist) from the **recitation domain** (what sequence of text was recited) and defines a formal mapping between them. This allows a single audio span to be referenced multiple times in the recitation sequence, and allows a single Ayah reference to be mapped to multiple non-contiguous audio spans — without duplicating audio data or breaking the timeline.

---

## Archive Structure Overview

```
my-recitation.qrn  (ZIP archive)
├── manifest.json          # Reciter and performance metadata
├── sync.json              # Relational synchronization data
└── audio/
    └── recitation.opus    # The recitation audio
```

See [`specs/01-archive-structure.md`](specs/01-archive-structure.md) for the full definition.

---

## Specification Index

| Document | Contents |
|---|---|
| [`specs/01-archive-structure.md`](specs/01-archive-structure.md) | Container format, file extensions, mandatory files |
| [`specs/02-manifest-format.md`](specs/02-manifest-format.md) | Metadata schema, audio format recommendations |
| [`specs/03-synchronization-protocol.md`](specs/03-synchronization-protocol.md) | Relational timeline, granularity levels, repetition handling |

## Schema Index

| File | Validates |
|---|---|
| [`schemas/manifest.schema.json`](schemas/manifest.schema.json) | `manifest.json` inside the archive |
| [`schemas/sync.schema.json`](schemas/sync.schema.json) | `sync.json` inside the archive |

---

## Key Words

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in all specification documents are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

---

## Contributing

This specification is in active draft status. Issues and pull requests are welcome. Please open an issue before submitting a significant structural change to allow design discussion.
