# dotquran Specification — 03: Synchronization Protocol

**Version:** 1.0.0-draft
**Status:** Draft

---

## 1. Overview

`sync.json` defines the **Relational Timeline**: a formal mapping between spans of audio time and spans of Quranic text. This document defines the data model, the two granularity levels (Ayah and Word), and the mechanism for representing non-linear recitation events such as verse repetitions and partial repetitions.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

The normative schema is [`schemas/sync.schema.json`](../schemas/sync.schema.json).

---

## 2. The Relational Timeline Model

### 2.1 Motivation: Why a Flat List Fails

A naive synchronization model looks like this:

```json
[
  { "surah": 2, "ayah": 1, "start": 0.0, "end": 3.2 },
  { "surah": 2, "ayah": 2, "start": 3.2, "end": 9.7 }
]
```

This model assumes:
1. Each Ayah appears exactly once.
2. Audio time and recitation progress are both monotonically increasing.
3. The start of the next entry immediately follows the end of the previous.

None of these assumptions hold for live or mujawwad recitation. A reciter may:
- Repeat Ayah 5 in full before continuing to Ayah 6.
- Repeat only the final clause of Ayah 12.
- Pause silently mid-Ayah and then resume.
- Recite the same basmala from a single audio segment that appears as both the opening of Surah 1 and the inter-Surah marker before Surah 2.

### 2.2 The Two-Domain Model

The Relational Timeline separates two independent domains:

**Domain A: Audio Segments (`segments`)**
: A catalog of named, non-overlapping spans of audio time. Each segment has a unique `id`, a `startMs` and `endMs` in milliseconds, and an optional `label`.

**Domain B: Timeline Path (`timeline`)**
: An ordered list of **events** representing the chronological sequence of text as the reciter performs it. Each event references one or more audio segments and one Ayah reference (and optionally, a word range within that Ayah).

The mapping between domains is a **many-to-many relation**: a single audio segment **MAY** be referenced by multiple timeline events (repetition), and a single timeline event **MAY** reference multiple audio segments (a single Ayah span across a spliced audio edit).

---

## 3. `sync.json` Top-Level Structure

```json
{
  "schemaVersion": "1.0",
  "granularity": "ayah",
  "segments": [ ... ],
  "timeline": [ ... ]
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `schemaVersion` | `string` | MUST | Version of this sync schema, in `MAJOR.MINOR` format. |
| `granularity` | `string` (enum) | MUST | Declares the finest granularity level present. One of `"ayah"` or `"word"`. |
| `segments` | `array` | MUST | The audio segment catalog. See §4. |
| `timeline` | `array` | MUST | The ordered recitation event sequence. See §5. |

---

## 4. Audio Segments (`segments`)

Each entry in the `segments` array defines a named span of audio time.

```json
{
  "id": "seg-001",
  "startMs": 0,
  "endMs": 3200,
  "label": "Opening silence + Basmala"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | `string` | MUST | A string that is unique across all entries in the `segments` array. **MUST** match the pattern `[a-zA-Z0-9_-]+`. |
| `startMs` | `integer` | MUST | Start time of the segment in milliseconds, inclusive. **MUST** be ≥ 0. |
| `endMs` | `integer` | MUST | End time of the segment in milliseconds, exclusive. **MUST** be strictly greater than `startMs`. |
| `label` | `string` | MAY | Human-readable description for tooling and debugging. |

### Rules for Segments

- Segment IDs **MUST** be unique within the `segments` array.
- Segments **MUST NOT** overlap in their time ranges. An overlap is defined as any two segments `A` and `B` where `A.startMs < B.endMs` and `B.startMs < A.endMs`.
- Segments **SHOULD** be sorted in ascending order of `startMs`. Consumers **MUST NOT** require sorted order.
- Gaps between segments (audio spans not covered by any segment) are **PERMITTED**. They represent audio content not considered part of the synchronization timeline (e.g., pre-recitation commentary, inter-surah silence).
- The `endMs` of any segment **MUST NOT** exceed the `durationSeconds * 1000` of the audio file declared in `manifest.json`.

---

## 5. Timeline Events (`timeline`)

The `timeline` array is an ordered sequence of **events**. The order of events in this array defines the recitation sequence as experienced by the listener. Event index 0 is the first thing recited; the last entry is the last.

```json
{
  "eventId": "evt-001",
  "ref": { "surah": 1, "ayah": 1 },
  "segmentIds": ["seg-001"],
  "wordRange": null
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `eventId` | `string` | MUST | A string unique across all entries in the `timeline` array. **MUST** match `[a-zA-Z0-9_-]+`. |
| `ref` | `object` | MUST | The Quranic text reference for this event. See §5.1. |
| `segmentIds` | `array of string` | MUST | Ordered list of segment IDs whose audio, played in sequence, constitutes this event's audio. **MUST** contain at least one entry. Each ID **MUST** correspond to an entry in `segments`. |
| `wordRange` | `object` or `null` | MUST | Word-level granularity. See §5.2. Set to `null` if this archive uses `"ayah"` granularity only. |

### 5.1 The `ref` Object (Polymorphic Recitation Reference)

The `ref` field is a **polymorphic object** discriminated by the optional `type` field. It identifies *what* was recited in a given timeline event — either a specific Ayah, the opening Isti'adha formula, or the closing Tasdiq formula.

Three types are defined:

| `type` value | Meaning | `surah` / `ayah` |
|---|---|---|
| `"ayah"` (or omitted) | A specific Ayah or unnumbered Basmalah | REQUIRED |
| `"istiadha"` | أعوذ بالله من الشيطان الرجيم | MUST NOT be present |
| `"tasdiq"` | صدق الله العظيم | MUST NOT be present |

#### Type: `"ayah"` (default)

The `type` field **MAY** be omitted; absence is equivalent to `"ayah"`. Both `surah` and `ayah` **MUST** be present.

```json
{ "surah": 2, "ayah": 255 }
{ "type": "ayah", "surah": 2, "ayah": 255 }
```

| Field | Type | Required | Description |
|---|---|---|---|
| `type` | `"ayah"` | MAY | Discriminator. If omitted, `"ayah"` is implied. |
| `surah` | `integer` | MUST | Surah number. **MUST** be in the range [1, 114]. |
| `ayah` | `integer` | MUST | Ayah number within the surah. **MUST** be ≥ 0. See Basmalah rules below. |

**Basmalah (`ayah: 0`) Rules:**

The value `ayah: 0` is reserved exclusively for the unnumbered Basmalah (بِسْمِ اللَّهِ الرَّحْمَٰنِ الرَّحِيمِ). The following rules apply:

- **Surah 1 (Al-Fatihah):** The Basmalah is the first numbered verse. It **MUST** be encoded as `{ "surah": 1, "ayah": 1 }`. Using `ayah: 0` for Surah 1 is **invalid**.
- **Surahs 2–114 (except Surah 9):** The Basmalah precedes the Surah but is not assigned a verse number in the Uthmani text. It **MUST** be encoded as `{ "surah": N, "ayah": 0 }`.
- **Surah 9 (At-Tawbah):** Has no Basmalah. The first recited verse is `{ "surah": 9, "ayah": 1 }`. Producers **MUST NOT** include an `ayah: 0` event for Surah 9.

#### Type: `"istiadha"`

The Isti'adha (أعوذ بالله من الشيطان الرجيم) is a liturgical opening formula. It is not a numbered Ayah. When a producer wishes to synchronize the Isti'adha audio span, they **MUST** use this type. No `surah` or `ayah` fields are permitted.

```json
{ "type": "istiadha" }
```

#### Type: `"tasdiq"`

The Tasdiq (صدق الله العظيم) is the closing affirmation recited at the end of a recitation session. It is not a numbered Ayah. No `surah` or `ayah` fields are permitted.

```json
{ "type": "tasdiq" }
```

#### Combined Example

The following `timeline` excerpt shows a complete recitation opening (Isti'adha → Basmalah of Surah 2 → first Ayah) followed by a closing Tasdiq:

```json
{
  "segments": [
    { "id": "seg-istiadha",  "startMs": 0,     "endMs": 4100  },
    { "id": "seg-basmala-2", "startMs": 4100,  "endMs": 7400  },
    { "id": "seg-2-1",       "startMs": 7400,  "endMs": 14200 },
    { "id": "seg-tasdiq",    "startMs": 14200, "endMs": 18000 }
  ],
  "timeline": [
    { "eventId": "evt-istiadha",  "ref": { "type": "istiadha" },                       "segmentIds": ["seg-istiadha"],  "wordRange": null },
    { "eventId": "evt-basmala-2", "ref": { "type": "ayah", "surah": 2, "ayah": 0 },   "segmentIds": ["seg-basmala-2"], "wordRange": null },
    { "eventId": "evt-2-1",       "ref": { "type": "ayah", "surah": 2, "ayah": 1 },   "segmentIds": ["seg-2-1"],       "wordRange": null },
    { "eventId": "evt-tasdiq",    "ref": { "type": "tasdiq" },                         "segmentIds": ["seg-tasdiq"],    "wordRange": null }
  ]
}
```

### 5.2 The `wordRange` Object (Optional Word-Level Granularity)

When `granularity` is `"word"`, timeline events **SHOULD** include a `wordRange` object. When `granularity` is `"ayah"`, `wordRange` **MUST** be `null`.

```json
{ "wordIndexStart": 0, "wordIndexEnd": 3 }
```

| Field | Type | Required | Description |
|---|---|---|---|
| `wordIndexStart` | `integer` | MUST | Zero-based index of the first word of this event within its Ayah. |
| `wordIndexEnd` | `integer` | MUST | Zero-based index of the last word of this event within its Ayah, **inclusive**. **MUST** be ≥ `wordIndexStart`. |

Word indices **MUST** correspond to the standard Uthmani word boundaries as defined by the Tanzil corpus or an equivalent authoritative source. Producers **SHOULD** document the word-boundary corpus they used in the `performance.notes` field of `manifest.json`.

---

## 6. Representing Repetition

Repetition is the primary reason this model exists. The following subsections show how to encode the most common non-linear recitation patterns.

### 6.1 Full Ayah Repetition

A reciter recites Ayah 5, then repeats it entirely before moving to Ayah 6. The audio of the repetition is a new, distinct audio segment.

```json
{
  "segments": [
    { "id": "seg-ayah5-first",  "startMs": 40000, "endMs": 48500 },
    { "id": "seg-ayah5-repeat", "startMs": 48500, "endMs": 57100 },
    { "id": "seg-ayah6",        "startMs": 57100, "endMs": 65000 }
  ],
  "timeline": [
    { "eventId": "evt-1", "ref": { "surah": 2, "ayah": 5 }, "segmentIds": ["seg-ayah5-first"],  "wordRange": null },
    { "eventId": "evt-2", "ref": { "surah": 2, "ayah": 5 }, "segmentIds": ["seg-ayah5-repeat"], "wordRange": null },
    { "eventId": "evt-3", "ref": { "surah": 2, "ayah": 6 }, "segmentIds": ["seg-ayah6"],        "wordRange": null }
  ]
}
```

`evt-1` and `evt-2` both reference Ayah 2:5, but each references a distinct audio segment. The timeline order declares that the first performance of Ayah 5 happened before the second.

### 6.2 Partial (Word-Level) Repetition

A reciter recites words 0–4 of Ayah 10, then repeats words 3–4 before continuing with words 5–9. This requires word-level granularity (`"granularity": "word"`).

```json
{
  "segments": [
    { "id": "seg-a10-full",   "startMs": 100000, "endMs": 112000 },
    { "id": "seg-a10-repeat", "startMs": 112000, "endMs": 116500 },
    { "id": "seg-a10-rest",   "startMs": 116500, "endMs": 125000 }
  ],
  "timeline": [
    { "eventId": "evt-1", "ref": { "surah": 2, "ayah": 10 }, "segmentIds": ["seg-a10-full"],   "wordRange": { "wordIndexStart": 0, "wordIndexEnd": 4 } },
    { "eventId": "evt-2", "ref": { "surah": 2, "ayah": 10 }, "segmentIds": ["seg-a10-repeat"], "wordRange": { "wordIndexStart": 3, "wordIndexEnd": 4 } },
    { "eventId": "evt-3", "ref": { "surah": 2, "ayah": 10 }, "segmentIds": ["seg-a10-rest"],   "wordRange": { "wordIndexStart": 5, "wordIndexEnd": 9 } }
  ]
}
```

### 6.3 Audio Reuse for Legacy/Studio Optimization

In a live performance, phrases recited multiple times (such as the opening Basmalah vs. an inter-surah Basmalah) are distinct audio events and **MUST** be mapped to separate, unique segments. 

However, legacy studio datasets often optimize file size by artificially reusing a single audio recording of the Basmalah before every Surah. The protocol supports importing these stitched datasets by allowing the same segment ID to appear in multiple distinct timeline events.

```json
{
  "segments": [
    { "id": "seg-studio-basmala", "startMs": 0, "endMs": 3200, "label": "Reused Studio Basmala" },
    { "id": "seg-s1-a2", "startMs": 3200, "endMs": 8000 }
  ],
  "timeline": [
    { "eventId": "evt-s1-b", "ref": { "type": "ayah", "surah": 1, "ayah": 1 }, "segmentIds": ["seg-studio-basmala"], "wordRange": null },
    { "eventId": "evt-s1-a2", "ref": { "type": "ayah", "surah": 1, "ayah": 2 }, "segmentIds": ["seg-s1-a2"], "wordRange": null },
    { "eventId": "evt-s2-b", "ref": { "type": "ayah", "surah": 2, "ayah": 0 }, "segmentIds": ["seg-studio-basmala"], "wordRange": null }
  ]
}
```

> **Note:** Sharing a segment ID across multiple timeline events is permitted by this specification. Consumers **MUST** handle this case. A consumer that highlights audio during playback **SHOULD** highlight all text references that share the currently playing segment.

---

## 7. Consumer Rendering Requirements

### 7.1 Determining Current Text from Audio Position

Given a playback position `P` (in milliseconds), a consumer **MUST** determine the active timeline events as follows:

1. Find all segments `s` where `s.startMs <= P < s.endMs`.
2. Find all timeline events `e` where `s.id` is in `e.segmentIds` for the matched segment(s).
3. The set of matched events is the **active set** at position `P`.

If the active set contains more than one event (because a segment is shared), the consumer **MAY** highlight all of them simultaneously or **MAY** use the event ordering in `timeline` to prefer the most recently started event. Consumers **SHOULD** document which strategy they implement.

### 7.2 Seeking to a Text Reference

Given a user request to seek to `(surah S, ayah A)`, a consumer **MUST**:

1. Find all timeline events `e` where `e.ref.surah == S` and `e.ref.ayah == A`.
2. Select the first such event in timeline order (lowest index).
3. Seek to `segments[e.segmentIds[0]].startMs`.

If no matching event exists, the consumer **MUST** surface an error or a "not found" state rather than silently seeking to an incorrect position.

---

## 8. Minimal Conforming `sync.json` Example

```json
{
  "schemaVersion": "1.0",
  "granularity": "ayah",
  "segments": [
    { "id": "s001", "startMs": 0,    "endMs": 3200  },
    { "id": "s002", "startMs": 3200, "endMs": 9700  },
    { "id": "s003", "startMs": 9700, "endMs": 16400 }
  ],
  "timeline": [
    { "eventId": "e001", "ref": { "surah": 1, "ayah": 1 }, "segmentIds": ["s001"], "wordRange": null },
    { "eventId": "e002", "ref": { "surah": 1, "ayah": 2 }, "segmentIds": ["s002"], "wordRange": null },
    { "eventId": "e003", "ref": { "surah": 1, "ayah": 3 }, "segmentIds": ["s003"], "wordRange": null }
  ]
}
```
