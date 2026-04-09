# dotquran Specification — 01: Archive Structure

**Version:** 1.0.0-draft
**Status:** Draft

---

## 1. Overview

A dotquran archive is a self-contained, portable unit of Quranic recitation data. This document defines the container format, permitted file extensions, mandatory internal structure, and naming conventions.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

---

## 2. Container Format

A dotquran archive **MUST** be a valid ZIP archive conforming to the ZIP Application Note (PKWARE, version 6.3.10 or later).

- The archive **MUST NOT** use ZIP64 extensions unless the total uncompressed content exceeds 4 GiB.
- The archive **MUST NOT** be password-protected or encrypted.
- The archive **MUST NOT** use unsupported compression methods. Conforming implementations **MUST** support `STORE` (method 0) and `DEFLATE` (method 8). The `BZIP2` (method 12) and `LZMA` (method 14) methods **MAY** be used by producers; consumers **SHOULD** support them.
- All file paths within the archive **MUST** use `/` (forward slash) as a path separator regardless of host operating system.
- All text files within the archive **MUST** be encoded as UTF-8 without BOM.

---

## 3. File Extensions

A dotquran archive file **MUST** use one of the following file extensions:

| Extension | Usage |
|---|---|
| `.qrn` | **RECOMMENDED** — short, canonical extension |
| `.quran` | **PERMITTED** — human-readable alternative |

Implementations **MUST** treat both extensions identically. A file bearing either extension and valid ZIP structure **MUST** be accepted as a candidate dotquran archive and validated against this specification.

---

## 4. Mandatory Internal Files

Every conforming dotquran archive **MUST** contain the following files at the paths specified. All paths are relative to the archive root.

### 4.1 `manifest.json`

**Location:** `manifest.json` (archive root)
**MIME type (logical):** `application/json`

Contains structured metadata describing the reciter, the performance, and the coverage of the archive. Full schema and field definitions are provided in [`02-manifest-format.md`](02-manifest-format.md). The file **MUST** be valid JSON and **MUST** validate against [`schemas/manifest.schema.json`](../schemas/manifest.schema.json).

### 4.2 `sync.json`

**Location:** `sync.json` (archive root)
**MIME type (logical):** `application/json`

Contains the relational synchronization timeline mapping audio segments to the recitation path. Full schema and structural definitions are provided in [`03-synchronization-protocol.md`](03-synchronization-protocol.md). The file **MUST** be valid JSON and **MUST** validate against [`schemas/sync.schema.json`](../schemas/sync.schema.json).

### 4.3 Audio File

**Location:** `audio/<filename>.<ext>` where `<filename>` is any valid filename string and `<ext>` is a supported audio extension.

The archive **MUST** contain exactly one audio file in the `audio/` directory. The presence of more than one audio file in the `audio/` directory **MUST** be treated as a validation error unless a future version of this specification explicitly permits it.

The audio file path declared in `manifest.json` under `audio.path` **MUST** exactly match the path of the audio file present in the archive. A mismatch **MUST** be treated as a validation error.

---

## 5. Permitted Audio Formats

The following audio formats are supported. Format capability is declared in `manifest.json`.

| Extension | Format | Notes |
|---|---|---|
| `.opus` | Opus in Ogg container | **RECOMMENDED** — best size/quality ratio for speech |
| `.m4a` | AAC in MPEG-4 container | **SHOULD** be used when `.opus` is not feasible |
| `.mp3` | MPEG-1 Audio Layer III | **PERMITTED** — maximum legacy compatibility |
| `.ogg` | Vorbis in Ogg container | **PERMITTED** |
| `.wav` | PCM in WAVE container | **PERMITTED** — not recommended for distribution due to size |
| `.flac` | Free Lossless Audio Codec | **PERMITTED** |

Producers **SHOULD** use `.opus` encoded at 48 kHz, stereo or mono, at a bitrate between 32 kbps and 128 kbps. Recitation audio is primarily speech; higher bitrates yield diminishing perceptual returns.

---

## 6. Optional Internal Files

The following optional files **MAY** be included in the archive. Consumers **MUST NOT** fail validation if these files are absent.

| Path | Description |
|---|---|
| `cover.jpg` or `cover.png` | Artwork image for the recitation (max 2 MiB) |
| `LICENSE.txt` | License text for the audio content |
| `NOTICE.txt` | Attribution or copyright notice |

---

## 7. Internal File Layout (Reference)

The following illustrates a minimal conforming archive:

```
my-recitation.qrn
├── manifest.json
├── sync.json
└── audio/
    └── recitation.opus
```

A complete archive with optional files:

```
full-recitation.qrn
├── manifest.json
├── sync.json
├── cover.jpg
├── LICENSE.txt
├── NOTICE.txt
└── audio/
    └── recitation.opus
```

---

## 8. Validation Requirements

A conforming consumer **MUST** perform the following checks before processing an archive:

1. The file is a valid ZIP archive.
2. `manifest.json` is present at the archive root and is valid JSON conforming to `manifest.schema.json`.
3. `sync.json` is present at the archive root and is valid JSON conforming to `sync.schema.json`.
4. The audio file path declared in `manifest.json` resolves to an existing file within the archive.
5. The `sync.schemaVersion` in `sync.json` and the `specVersion` in `manifest.json` are versions that the consumer supports.

Consumers **MUST** surface a descriptive error to the user when any of these checks fail. Consumers **MUST NOT** silently ignore or attempt to repair a structurally invalid archive.
