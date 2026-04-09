# dotquran Specification — 02: Manifest Format

**Version:** 1.0.0-draft
**Status:** Draft

---

## 1. Overview

`manifest.json` is the metadata file of the dotquran archive. It identifies the reciter, characterizes the performance (style, qira'at, maqam), declares the audio asset, and states the coverage range of the archive. It is the entry point for any application loading a dotquran file.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

The normative schema is [`schemas/manifest.schema.json`](../schemas/manifest.schema.json). This document describes the semantics of each field. In case of conflict, the JSON Schema is authoritative for structural validation and this document is authoritative for semantic interpretation.

---

## 2. Top-Level Structure

```json
{
  "specVersion": "1.0",
  "id": "urn:dotquran:rec:...",
  "reciter": { ... },
  "performance": { ... },
  "audio": { ... },
  "coverage": { ... },
  "created": "2026-04-10T00:00:00Z",
  "generator": "..."
}
```

---

## 3. Field Definitions

### 3.1 `specVersion`

**Type:** `string`
**Required:** MUST be present.

The version of the dotquran specification this manifest conforms to, in `MAJOR.MINOR` format (e.g., `"1.0"`). Consumers **MUST** reject archives with a `specVersion` whose major component is greater than the highest major version they support. Consumers **SHOULD** warn on minor version mismatches but **MAY** continue processing.

---

### 3.2 `id`

**Type:** `string`
**Required:** MUST be present.

A globally unique identifier for this specific archive. The value **MUST** be a URN in the form:

```
urn:dotquran:rec:<reciter-slug>:<coverage-slug>:<timestamp>
```

- `<reciter-slug>`: A URL-safe, lowercase, hyphenated representation of the reciter's name (e.g., `abdulbasit-abdussamad`).
- `<coverage-slug>`: A short descriptor of coverage (e.g., `surah-2` or `full-quran`).
- `<timestamp>`: An ISO 8601 date string with no separators (e.g., `20260410`).

Example: `urn:dotquran:rec:abdulbasit-abdussamad:surah-2:20260410`

Producers **SHOULD** generate this value deterministically. Consumers **MUST NOT** depend on any particular structure within the URN beyond it being unique.

---

### 3.3 `reciter`

**Type:** `object`
**Required:** MUST be present.

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | `string` | MUST | Full reciter name in the archive's primary language. |
| `nameAr` | `string` | SHOULD | Full reciter name in Arabic script. |
| `nameEn` | `string` | SHOULD | Full reciter name transliterated in Latin script. |
| `bio` | `string` | MAY | Short biographical note. Maximum 500 characters. |
| `nationality` | `string` | MAY | ISO 3166-1 alpha-2 country code (e.g., `"EG"`). |

---

### 3.4 `performance`

**Type:** `object`
**Required:** MUST be present.

Characterizes the style of this specific tilawa performance.

| Field | Type | Required | Description |
|---|---|---|---|
| `tilawa` | `string` (enum) | MUST | Recitation style. See §3.4.1. |
| `qiraat` | `string` (enum) | MUST | Qira'at (reading tradition). See §3.4.2. |
| `riwaya` | `string` | SHOULD | Riwaya (transmission chain) within the qira'at (e.g., `"Hafs"`, `"Warsh"`). |
| `maqam` | `string` | MAY | Primary melodic mode (maqam) used, if applicable (e.g., `"Bayati"`, `"Rast"`, `"Hijaz"`). |
| `notes` | `string` | MAY | Free-text performance notes. Maximum 1000 characters. |

#### 3.4.1 `tilawa` Enumerated Values

| Value | Meaning |
|---|---|
| `murattal` | Slow, measured recitation for learning and listening |
| `mujawwad` | Highly melodic, ornamented recitation |
| `muallim` | Teaching/instructional recitation, often with pauses |
| `other` | Any style not covered by the above; `notes` SHOULD clarify |

#### 3.4.2 `qiraat` Enumerated Values

| Value | Qira'at |
|---|---|
| `hafs-an-asim` | Hafs ʿan ʿĀṣim (most widely used) |
| `warsh-an-nafi` | Warsh ʿan Nāfiʿ |
| `qalun-an-nafi` | Qālūn ʿan Nāfiʿ |
| `al-duri-an-abi-amr` | Al-Dūrī ʿan Abī ʿAmr |
| `ibn-kathir-al-makki` | Ibn Kathīr al-Makkī |
| `ibn-amir-al-shami` | Ibn ʿĀmir al-Shāmī |
| `shu-ba-an-asim` | Shuʿba ʿan ʿĀṣim |
| `hamza-al-kufi` | Ḥamza al-Kūfī |
| `al-kisai` | Al-Kisāʾī |
| `khalaf-an-hamza` | Khalaf ʿan Ḥamza |
| `other` | Any other; `notes` SHOULD clarify |

---

### 3.5 `audio`

**Type:** `object`
**Required:** MUST be present.

| Field | Type | Required | Description |
|---|---|---|---|
| `path` | `string` | MUST | Path to the audio file within the archive, relative to the archive root (e.g., `"audio/recitation.opus"`). |
| `format` | `string` (enum) | MUST | Audio format identifier. One of: `"opus"`, `"m4a"`, `"mp3"`, `"ogg"`, `"wav"`, `"flac"`. |
| `durationSeconds` | `number` | MUST | Total audio duration in seconds, as a positive number with up to 3 decimal places. |
| `sampleRateHz` | `integer` | SHOULD | Audio sample rate in Hz (e.g., `48000`). |
| `channels` | `integer` | SHOULD | Number of audio channels. `1` for mono, `2` for stereo. |
| `bitrateKbps` | `number` | MAY | Nominal bitrate in kbps for lossy formats. |
| `sha256` | `string` | SHOULD | Lowercase hex-encoded SHA-256 digest of the audio file bytes. Used for integrity verification. |

---

### 3.6 `coverage`

**Type:** `object`
**Required:** MUST be present.

Describes the Quranic text range covered by this archive.

| Field | Type | Required | Description |
|---|---|---|---|
| `type` | `string` (enum) | MUST | One of: `"full"` (complete Quran), `"juz"`, `"surah"`, `"range"`. |
| `from` | `object` | MUST if `type` is `"surah"` or `"range"` | Starting position. Contains `surah` (integer 1–114) and `ayah` (integer ≥ 1). |
| `to` | `object` | MUST if `type` is `"range"` | Ending position. Same structure as `from`. |
| `juzNumber` | `integer` | MUST if `type` is `"juz"` | Juz number (1–30). |
| `surahNumber` | `integer` | MUST if `type` is `"surah"` | Surah number (1–114). |

---

### 3.7 `created`

**Type:** `string` (ISO 8601 datetime)
**Required:** SHOULD be present.

The UTC datetime at which this archive was produced (e.g., `"2026-04-10T14:30:00Z"`). Consumers **MAY** use this for display and sorting purposes.

---

### 3.8 `generator`

**Type:** `string`
**Required:** MAY be present.

A free-text string identifying the software that produced this archive (e.g., `"dotquran-tools/1.2.0"`). Consumers **MUST NOT** alter behavior based on this field.

---

## 4. Example `manifest.json`

```json
{
  "specVersion": "1.0",
  "id": "urn:dotquran:rec:abdulbasit-abdussamad:surah-2:20260410",
  "reciter": {
    "name": "Abdul Basit Abdul Samad",
    "nameAr": "عبد الباسط عبد الصمد",
    "nameEn": "Abdul Basit Abdul Samad",
    "nationality": "EG"
  },
  "performance": {
    "tilawa": "mujawwad",
    "qiraat": "hafs-an-asim",
    "riwaya": "Hafs",
    "maqam": "Bayati",
    "notes": "Live studio recording, 1960s. Contains repeated verses as performed."
  },
  "audio": {
    "path": "audio/recitation.opus",
    "format": "opus",
    "durationSeconds": 7241.553,
    "sampleRateHz": 48000,
    "channels": 1,
    "bitrateKbps": 64,
    "sha256": "a3f5b2c1d4e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2"
  },
  "coverage": {
    "type": "surah",
    "surahNumber": 2,
    "from": { "surah": 2, "ayah": 1 }
  },
  "created": "2026-04-10T00:00:00Z",
  "generator": "dotquran-tools/1.0.0"
}
```
