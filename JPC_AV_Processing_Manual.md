# JOHNSON PUBLISHING COMPANY ARCHIVE
### Audiovisual Materials: Arrangement and Description in ArchivesSpace

Version 2026-09-12

Smithsonian National Museum of African American History and Culture (NMAAHC) / Getty Research Institute (GRI)

---

## Introduction

This manual serves as a guide for archival staff describing audiovisual materials from the Johnson Publishing Company (JPC) Archive in ArchivesSpace. It covers the intellectual arrangement of the Audiovisual Materials series and the practical steps for creating compliant finding aid records at each level of description. The procedures and conventions defined here complement the broader JPC Archival Processing Manual, which governs the photographic and textual portions of the collection; readers should consult both documents as needed.

These guidelines are rooted in EAD2002 and DACS standards as implemented at the Smithsonian Institution, and draw on the Archives of American Art (AAA) guidelines for processing collections with audiovisual material. Where JPC practice departs from AAA practice, this manual takes precedence.

This is a living document and will be revised as the audiovisual processing project develops.

> **N.B.** The still image processing manual (JPC Archival Processing Manual, current version) remains the governing document for photographic and textual portions of the collection. This manual addresses only the Audiovisual Materials series.

---

## Acknowledgements

These guidelines draw on the Archives of American Art's *Processing Collections With Audiovisual Material* (Chapters 4 and 5), and the existing JPC Archival Processing Manual.

Additional resources consulted:

- PB Core Audiovisual Metadata Standard
- NMAAHC TBM preservation documentation, archival practices, and embedded metadata specifications

These guidelines were developed with input from staff at the Getty Research Institute (GRI) and across the Smithsonian Institution.

---

# Chapter 0. ArchivesSpace Fields Reference

This chapter documents every ArchivesSpace field used in JPCA AV item-level records: what data goes in it, how it gets there, whether manual adjustment is needed after import, and the archival rationale for the mapping.

Fields are grouped by the stage at which they are populated. Two kinds of statement appear side by side and are labeled where they differ: **policy** (what JPCA AV description requires) and **automation** (what the scripts in the `aspace_jpc_av` repository enforce or write). The scripts accept some input the policy would not, and automate less than the policy covers; the policy governs.

> **N.B.** Where "manual adjustment" is noted, this may mean editing the record directly in the ArchivesSpace staff interface or re-importing data for that record. Records can be updated at any point by exporting them with `aspace_csv_export.py`, editing the sheet, and re-running the import script with `--update-only` — see 0.E.

---

## 0.A. Fields Populated via CSV Import

*Script: `aspace_csv_import.py`*

These fields are written to ArchivesSpace by the CSV import script. The source for each is a column in the import CSV. A create sheet must carry every column below; an update-only sheet needs `CATALOG_NUMBER` plus whichever columns are being changed.

---

### Title

| | |
|---|---|
| **ArchivesSpace field** | `title` |
| **CSV column** | `ASpace Title` |
| **MKV embedded tag** | `TITLE` — should match the ArchivesSpace title exactly if already cataloged; leave blank otherwise |
| **Data** | The descriptive title for the tape, following the conventions in Chapter 5. |
| **How imported** | Written directly by the import script. A blank cell falls back to the catalog number, so an untitled import shows its identifier as a placeholder title until cataloged. |
| **Manual adjustment** | Sometimes. Titles may be refined after import as more information becomes available. Under `--update-only` a blank cell leaves the stored title alone. |
| **Why here** | DACS requires a title at every level of description. |

---

### Component Unique Identifier

| | |
|---|---|
| **ArchivesSpace field** | `component_id` |
| **CSV column** | `CATALOG_NUMBER` |
| **MKV embedded tag** | `CATALOG_NUMBER` |
| **Data** | The JPC_AV_##### identifier assigned to the tape, its case, and its digitized file. Policy: five digits. Automation: the scripts accept `JPC_AV_` followed by any run of digits and refuse anything else (a letter O for a zero, a stray suffix). The identifier form for ephemera child records is under discussion — see Appendix B. |
| **How imported** | Written directly by the import script. It is also the matching key for every later operation: `--update-only` finds the record by it, and the directory processing script finds the record for a folder by it. |
| **Manual adjustment** | No. Must not be changed after creation. |
| **Why here** | Serves as the stable, unique identifier linking the physical object to its ArchivesSpace record and its digitized file. A `--create-records` run stops before writing anything if any row's identifier already exists in ArchivesSpace (with `--skip-duplicates`, that row is skipped instead); the validator also flags an identifier that appears twice in the sheet. |

---

### Level of Description

| | |
|---|---|
| **ArchivesSpace field** | `level` |
| **CSV column** | — (hardcoded) |
| **MKV embedded tag** | No |
| **Data** | `item` |
| **How imported** | Hardcoded in the import script. |
| **Manual adjustment** | No. |
| **Why here** | All tapes are described at the item level per JPCA AV arrangement policy. |

---

### Parent

| | |
|---|---|
| **ArchivesSpace field** | `parent.ref` |
| **CSV column** | `ASpace Parent RefID` |
| **MKV embedded tag** | No |
| **Data** | The ref_id of the item's *immediate* file-level parent: the production-stage record (Edited, Raw, or Promo) for an independent tape, or the unit's own file record beneath Raw for a tape that is part of a set (see 3.E, 4.C). The importer accepts any parent in the resource, and `--update-only` never re-parents, so a wrong ref_id is fixed by moving the record in ArchivesSpace. |
| **How imported** | The script looks up the archival object with this ref_id — exactly one must match within the AV resource — and links the new item as its child. Used only when creating: `--update-only` ignores the column and never re-parents. |
| **Manual adjustment** | No. If the parent ref_id is wrong the record must be moved manually. |
| **Why here** | Establishes the item's position in the hierarchy. Required for creation — a `--create-records` run checks every parent first and stops before writing anything if one does not exist. |

---

### Resource

| | |
|---|---|
| **ArchivesSpace field** | `resource.ref` |
| **CSV column** | — (from `creds.py`) |
| **MKV embedded tag** | No |
| **Data** | `/repositories/{repo_id}/resources/{resource_id}` — `/repositories/2/resources/7` in both current instances |
| **How imported** | Taken from the environment selected in `creds.py` (sandbox or production, chosen with `--env` when more than one is configured). Every script also refuses to write to a record outside this resource. |
| **Manual adjustment** | No. |
| **Why here** | Links the item to the JPCA resource record. |

---

### Publish

| | |
|---|---|
| **ArchivesSpace field** | `publish` |
| **CSV column** | — (hardcoded) |
| **MKV embedded tag** | No |
| **Data** | `true` |
| **How imported** | Hardcoded in the import script. |
| **Manual adjustment** | Occasionally — records may be suppressed individually in ArchivesSpace if needed. `--update-only` never touches the flag. |
| **Why here** | All JPCA AV records are published to public interfaces by default. |

---

### Dates

| | |
|---|---|
| **ArchivesSpace field** | `dates[]` |
| **CSV columns** | `Creation or Recording Date`, `Edit Date`, `Broadcast Date` |
| **MKV embedded tag** | No |
| **Data** | YYYY-MM-DD (or the partial forms YYYY-MM and YYYY). Date type: Single. Labels: `creation`, `Edited`, `broadcast` respectively. |
| **How imported** | One date object per non-empty column. Accepted input: M/D/YYYY, M/D/YY, YYYY-MM-DD, YYYY/MM/DD, and the partial forms YYYY-MM and YYYY (stored as written). Two-digit years resolve within the collection's span. Any date set or changed must fall within 1940–2020; anything outside is rejected as a typo. The Date Expression is written equal to the Begin value. Under `--update-only` a supplied date replaces only the same-label date and keeps that date's other fields; a stored date outside the range round-trips as long as the sheet leaves it unchanged. A row that would *change* a label is refused if the stored date under that label is anything but a plain single date (it has an End, its type is inclusive or bulk, or its type is missing) or if several dates share the label, since one cell cannot express that change; an untouched date of any kind coexists with edits to other fields. |
| **Manual adjustment** | Sometimes — dates may be corrected or added after import. Policy expressions such as `undated` or *approximately 1986–1987* (see 4.D.i) are entered by hand; a hand-edited expression survives `--update-only` unless that date's Begin is changed in the sheet. |
| **Why here** | DACS requires dates at the item level. Three separate date labels reflect the distinct production stages of AV material. |

---

### Extent Type

| | |
|---|---|
| **ArchivesSpace field** | `extents[].extent_type` |
| **CSV column** | `Original Format` |
| **MKV embedded tag** | `ORIGINAL_MEDIA_TYPE` — note this tag uses a different format: Format (PB Core), manufacturer, model (e.g., `1-inch type C, Sony, V1-K`). The CSV `Original Format` column uses only the PB Core format term to match the ArchivesSpace controlled vocabulary. |
| **Data** | The physical format of the tape, e.g., `1 inch videotape`, `VHS`. Must match the ArchivesSpace `extent_extent_type` controlled vocabulary exactly. |
| **How imported** | Written by the import script and validated against the live ArchivesSpace enumeration before any write. Policy: every item has an extent. Automation: a blank cell creates the record with no extent rather than refusing it. Under `--update-only` only the type changes on an existing extent — every other extent field is kept — while a record with no extent gets a complete new one (portion whole, number 1, the type); an unchanged stored term round-trips even if it has since been retired from the vocabulary. A record that has acquired two or more extents (not policy — see 3.E) is left alone unless the cell would change the first extent's type, which is refused. |
| **Manual adjustment** | If the value is wrong it must be corrected — see note above on correction methods. |
| **Why here** | DACS requires an extent at the item level. Format type is the primary physical characteristic of an AV item. |

---

### Extent Portion and Number

| | |
|---|---|
| **ArchivesSpace field** | `extents[].portion`, `extents[].number` |
| **CSV column** | — (hardcoded) |
| **MKV embedded tag** | No |
| **Data** | Portion: `whole`. Number: `1`. |
| **How imported** | Hardcoded in the import script. |
| **Manual adjustment** | No. Number stays 1: an exact duplicate is its own item (see 3.E). |
| **Why here** | Each item record describes a single tape (Portion = whole, Number = 1) per JPCA AV description policy. |

---

### Scope and Content Note

| | |
|---|---|
| **ArchivesSpace field** | `notes[]` — type: `scopecontent`, multipart, text subnote |
| **CSV column** | `ASpace Scope and Contents Note` |
| **MKV embedded tag** | `DESCRIPTION` — note this tag is intentionally brief and may differ from the fuller ArchivesSpace scope note; it is not a direct source for the ArchivesSpace field. |
| **Data** | A brief description of the tape's intellectual content, staff-assembled and reviewed before import. |
| **How imported** | Written by the import script as a multipart note (label empty, published) with one text subnote. Only created when the column contains content. Under `--update-only` a changed cell replaces the first text paragraph of the first note of this type that carries text; if no note of the type carries text, the text is added to the first note of the type ahead of its existing subnotes (a Duration list included); if the record has no note of the type at all, a new one is appended. The targeted note's other paragraphs, its metadata, and any additional notes of the type are preserved. A blank cell changes nothing. |
| **Manual adjustment** | Sometimes — content may be refined after import. |
| **Why here** | Provides intellectual content description at the item level. |

---

### Physical Characteristics and Technical Requirements Note

| | |
|---|---|
| **ArchivesSpace field** | `notes[]` — type: `phystech`, multipart, text subnote |
| **CSV column** | `ASpace PhysTech Note` |
| **MKV embedded tags** | `_TRANSFER_NOTES` and/or `_PRE_TRANSFER_NOTES` — the CSV column may draw from either or both, or may contain staff-written content not found in either tag. |
| **Data** | Technical and physical observations about the tape. May draw from `_TRANSFER_NOTES`, `_PRE_TRANSFER_NOTES`, both, or neither — assembled and edited by staff before import. |
| **How imported** | Written by the import script as a multipart note (label empty, published) with one text subnote. Only created when the column contains content. Updated the same way as the Scope and Content note; the Duration defined list that directory processing adds (0.B) is preserved. |
| **Manual adjustment** | Sometimes — content may be added or corrected after import. |
| **Why here** | Documents tape condition and transfer quality at the item level. Essential for researchers and future preservation decision-making. |

---

### Top Container

| | |
|---|---|
| **ArchivesSpace field** | `instances[].sub_container.top_container` — type: `AV Case`, indicator: `JPC_AV_#####` |
| **CSV column** | `CATALOG_NUMBER` (used as indicator) |
| **MKV embedded tag** | No |
| **Data** | Container type: `AV Case`. Indicator: the JPC_AV_##### identifier. |
| **How imported** | The import script looks for an AV Case top container with this indicator first: if exactly one exists it is reused, if none exists one is created, and if several share the indicator the row is refused until the duplicates are cleaned up in ArchivesSpace. |
| **Manual adjustment** | No. |
| **Why here** | Links the intellectual record to its physical housing. The JPC_AV_##### identifier on the container matches the one on the tape and case, creating a chain from the physical object to the ArchivesSpace record to the digitized file. |

---

### Instance Type

| | |
|---|---|
| **ArchivesSpace field** | `instances[].instance_type` |
| **CSV column** | — (hardcoded) |
| **MKV embedded tag** | No |
| **Data** | `Moving Images (Video)` for video; `Audio` for audio recordings. |
| **How imported** | The import script hardcodes `Moving Images (Video)`. Policy covers audio; automation does not yet — no script creates or directory-processes audio records, which are created manually (export and the MADS check can still read them; see Chapter 6 and Appendix B). |
| **Manual adjustment** | No. |
| **Why here** | Required by ArchivesSpace to type the instance. |

---

## 0.B. Fields Populated via Directory Processing Script

*Script: `aspace-rename-directories.py`*

These fields are written to ArchivesSpace by the directory processing script, which runs on the digitized folders after the media file is available (`.mkv` by default; optical-disc `.mp4` packages with `--mp4`). For each `JPC_AV_#####` folder it finds the record by Component Unique Identifier, re-verifies it before writing, updates the two fields below, and then renames the folder (see 1.F).

---

### Duration

| | |
|---|---|
| **ArchivesSpace field** | `notes[]` — type: `phystech`, Defined List subnote, item label: `Duration` |
| **Source** | Extracted from the media file via `mediainfo` |
| **MKV embedded tag** | No — extracted directly from the file's stream metadata, not from a tag. |
| **Data** | Runtime in hh:mm:ss format, e.g., `01:23:45`. |
| **How imported** | The script fetches the record and, depending on what it finds: updates every existing Duration item in place (across all phystech notes, so no stale value is left behind); or, if the record has a phystech note but no Duration, appends a Defined List to that note after its existing text; or, if the record has no phystech note, creates one (label empty, published) holding only the Duration list. Re-running never duplicates the entry. |
| **Manual adjustment** | No. Must not be edited by hand. |
| **Why here** | Runtime extracted from the digitized file is more accurate than any estimate from the source documentation, and is not available until digitization is complete. |

---

### Extent Physical Details

| | |
|---|---|
| **ArchivesSpace field** | `extents[].physical_details` |
| **Source** | Default value in `aspace-rename-directories.py` |
| **MKV embedded tag** | No |
| **Data** | `SD video, color, sound` (standard value for most JPCA AV). |
| **How imported** | Filled only when the field is blank on a record with exactly one extent. An existing value is never overwritten; on a record with no extent, or one that has acquired several (not policy — see 3.E), this step is skipped (Duration processing and the folder rename continue as usual) and the extents are left for staff to correct. |
| **Manual adjustment** | Yes — tapes that deviate from the standard (BW, silent, HD) must have the value set manually. This can be done before or after the automated run; the script will not replace it. |
| **Why here** | Documents the technical characteristics of the video signal. Set by the directory processing script because the digitization process confirms these properties. |

---

## 0.C. Fields Entered Manually in ArchivesSpace

These fields are not populated by any script and must be entered directly in the ArchivesSpace staff interface.

---

### Production Crew Note

| | |
|---|---|
| **ArchivesSpace field** | `notes[]` — type: `scopecontent`, label: `Production Crew`, Defined List subnote |
| **Source** | Tape label, video slate, production documentation |
| **MKV embedded tag** | No |
| **Data** | Role and name for each crew member, e.g., *Camera — Jane Doe*. |
| **How entered** | Manually in ArchivesSpace at the episode sub-series level. |
| **Why here** | Production credits are an important form of access point for researchers and are associated with the episode as a whole, not with individual tapes. |

---

### Agents

| | |
|---|---|
| **ArchivesSpace field** | `linked_agents[]` — roles: `subject`, `creator` |
| **Source** | Tape label, video slate, episode documentation |
| **MKV embedded tag** | No |
| **Data** | Names of individuals, groups, or organizations featured in or responsible for the recording. |
| **How entered** | Manually in ArchivesSpace, primarily at the episode sub-series level. Applied to individual item records only when the agent is specific to that tape and not already covered at a higher level. |
| **Why here** | Provides named access points for researchers. Agent inheritance from parent episode records to individual raw footage tapes requires manual review — see Chapter 8. |

---

## 0.D. Complete Example: CSV Row → ArchivesSpace Record

The following example shows what a single item-level archival object looks like as JSON at each stage of the workflow. This is the actual data structure sent to and stored by the ArchivesSpace API.

### Sample CSV Row

```
CATALOG_NUMBER:                  JPC_AV_00012
ASpace Title:                    Ebony/Jet Celebrity Showcase, Episode 22, Promo
Creation or Recording Date:      8/1/1982
Edit Date:                       [empty]
Broadcast Date:                  [empty]
Original Format:                 2 inch videotape
ASpace Parent RefID:             3f9c2a7d0b1e4c6a8d5f7e9b2c4a6d8e
ASpace Scope and Contents Note:  Promotional clip for episode 22 of the Ebony/Jet Celebrity Showcase series.
ASpace PhysTech Note:            Slight ringing present throughout. Hue is inconsistent; skin tones are redder in some sections.
```

The parent ref_id is the 32-character value ArchivesSpace assigns to the parent archival object (here, the episode's *Promo* file-level record).

---

### Stage 1: After `aspace_csv_import.py --create-records`

Before writing, the importer confirms that no record with this Component Unique Identifier exists, that the parent resolves, that the extent type is in the live vocabulary, and that at most one AV Case container carries this indicator.

**Top Container** (reused if one with this indicator already exists; otherwise created first):

```json
{
  "indicator": "JPC_AV_00012",
  "type": "AV Case",
  "repository": {"ref": "/repositories/2"}
}
```

**Archival Object:**

```json
{
  "jsonmodel_type": "archival_object",
  "resource": {"ref": "/repositories/2/resources/7"},
  "parent": {"ref": "/repositories/2/archival_objects/12345"},
  "level": "item",
  "publish": true,
  "title": "Ebony/Jet Celebrity Showcase, Episode 22, Promo",
  "component_id": "JPC_AV_00012",
  "dates": [
    {
      "jsonmodel_type": "date",
      "date_type": "single",
      "label": "creation",
      "begin": "1982-08-01",
      "expression": "1982-08-01"
    }
  ],
  "extents": [
    {
      "jsonmodel_type": "extent",
      "portion": "whole",
      "number": "1",
      "extent_type": "2 inch videotape"
    }
  ],
  "notes": [
    {
      "jsonmodel_type": "note_multipart",
      "type": "scopecontent",
      "label": "",
      "publish": true,
      "subnotes": [
        {
          "jsonmodel_type": "note_text",
          "content": "Promotional clip for episode 22 of the Ebony/Jet Celebrity Showcase series."
        }
      ]
    },
    {
      "jsonmodel_type": "note_multipart",
      "type": "phystech",
      "label": "",
      "publish": true,
      "subnotes": [
        {
          "jsonmodel_type": "note_text",
          "content": "Slight ringing present throughout. Hue is inconsistent; skin tones are redder in some sections."
        }
      ]
    }
  ],
  "instances": [
    {
      "jsonmodel_type": "instance",
      "instance_type": "Moving Images (Video)",
      "sub_container": {
        "jsonmodel_type": "sub_container",
        "top_container": {"ref": "/repositories/2/top_containers/78901"}
      }
    }
  ]
}
```

The blank Edit Date and Broadcast Date cells produce no date objects. Had the title cell been blank, `title` would be `JPC_AV_00012`; had Original Format been blank, the record would have no extent.

---

### Stage 2: After `aspace-rename-directories.py`

The folder `JPC_AV_00012/` holds `JPC_AV_00012.mkv`. The record has a phystech note with no Duration, so a Defined List is appended after the existing text; the single extent's Physical Details is blank, so it is filled. Had a Duration already been present it would have been updated in place; had Physical Details held a value it would have been kept. The folder is then renamed `JPC_AV_00012_refid_<ref_id>/` (see 1.F).

**Updated Physical Characteristics and Technical Requirements note:**

```json
{
  "jsonmodel_type": "note_multipart",
  "type": "phystech",
  "label": "",
  "publish": true,
  "subnotes": [
    {
      "jsonmodel_type": "note_text",
      "content": "Slight ringing present throughout. Hue is inconsistent; skin tones are redder in some sections."
    },
    {
      "jsonmodel_type": "note_definedlist",
      "publish": true,
      "items": [
        {
          "jsonmodel_type": "note_definedlist_item",
          "label": "Duration",
          "value": "00:02:30"
        }
      ]
    }
  ]
}
```

**Updated Extent:**

```json
{
  "jsonmodel_type": "extent",
  "portion": "whole",
  "number": "1",
  "extent_type": "2 inch videotape",
  "physical_details": "SD video, color, sound"
}
```

---

### Stage 3: A later `--update-only` run

The record is exported, the title cell is edited in the sheet, and the export is re-imported with `--update-only`. Only `title` is written. The dates, the extent (including its Physical Details), both notes (including the Duration list), the parent, the container, and the Component Unique Identifier are left exactly as they were. An unchanged row is reported as "No changes needed" and is not written.

---

### How It Looks in the ArchivesSpace Staff Interface

**Basic Information**
- Level: Item
- Title: Ebony/Jet Celebrity Showcase, Episode 22, Promo
- Component Unique ID: JPC_AV_00012

**Dates**
- Creation: 1982-08-01

**Extents**
- Portion: Whole | Number: 1 | Type: 2 inch videotape | Physical Details: SD video, color, sound

**Notes**

*Scope and Contents:*
> Promotional clip for episode 22 of the Ebony/Jet Celebrity Showcase series.

*Physical Characteristics and Technical Requirements:*
> Slight ringing present throughout. Hue is inconsistent; skin tones are redder in some sections.
>
> Duration: 00:02:30

**Instance**
- Type: Moving Images (Video) | Top Container: AV Case JPC_AV_00012

---

## 0.E. Updating Records: The Export → Edit → Update Round Trip

*Scripts: `aspace_csv_export.py`, `aspace_csv_import.py --update-only`, `check_mads.py`*

Records that already exist are corrected in bulk by round trip rather than by re-creating them:

1. **Export.** `aspace_csv_export.py` walks the AV resource (or a list of identifiers) and writes a sheet with exactly the import columns, plus audit columns: the record's ref_id, URI, a staff-interface link, warnings, created/modified stamps, and its place in the tree (Level, Depth, and a Path of its ancestors). Rows are written in tree order, so an export of every level reads like the staff interface's tree (a `--list` export keeps the order of the list). Non-item rows are not warned — the Level column identifies them — but `--update-only` refuses them all the same; a blank Warnings cell on a series or file row does not make it re-importable. Warnings cover metadata gaps the policy cares about (an item with no title or date, and a missing Component Unique Identifier, which also blocks re-import); rows the importer will refuse on re-import (an identifier that is not `JPC_AV_` plus digits or is shared by two records, a stored date that is not a valid ISO date); and fields whose editing through the sheet is restricted (a date outside 1940–2020 may be corrected to one inside the range but not changed to another outside it; a range or non-single date, a label shared by several dates, and the type on a record with several extents cannot be changed at all). A record or ancestor with no position in ArchivesSpace also draws a warning that its tree order is approximate; that one does not affect re-import.
2. **Edit** the sheet. Blank cells mean "leave alone"; the round trip can replace a value but never clear one. Deletions are made in ArchivesSpace directly.
3. **Update.** `aspace_csv_import.py --update-only` finds each record by its identifier, re-verifies it, and writes only the fields whose cell differs from what is stored. It never creates a record, never re-parents, and never touches the publish flag, instances, containers, existing extent fields other than the type, existing note metadata, the Duration list, or any date label or note type the sheet does not manage. Where a record lacks a structure the sheet supplies (no extent, no note of that type, no date under that label), a complete new one is added.

The export can also confirm which items have reached the Smithsonian DAMS: `--mads-live` (or the standalone `check_mads.py`) checks each identifier's public MADS URL and records *Yes* / *No* / *check failed* / *invalid catalog number* in the sheet. It reads only the public MADS endpoint and writes nothing to ArchivesSpace.

The scripts that talk to ArchivesSpace (import, export, directory processing, the extent-type and parent checks) select sandbox or production from `creds.py` and require `--env NAME` whenever more than one is configured, so a forgotten flag can never write to the wrong instance; the MADS checker needs no environment. Runs that produce output leave a timestamped file in the reports folder (the export, MADS check and parent check accept `-o` for another path): the importer a log plus CSV and JSON receipts, the directory processor a log, the export and MADS checks their CSVs, the validator and parent check their reports; the extent-type checker only prints its result.

---

# Chapter 1. File Naming Conventions

This chapter defines the file naming conventions for all digital files produced in the JPC AV preservation project. The JPC AV identifier — a human-readable label affixed to each tape and its original case in the format `JPC_AV_#####` — is the basis for all file names.

## 1.A. Digitized Video Files

Digitized video files are delivered as a single MKV file per tape.

Convention:

```
JPC_AV_#####.mkv
```

Example: A tape with identifier `JPC_AV_05498` produces:

```
JPC_AV_05498.mkv
```

## 1.B. Digitized Audio Files

Digitized audio files are delivered as WAV files, one per side. The side suffix (`_s1`, `_s2`) is always appended, even when only one side exists.

Convention:

```
JPC_AV_#####_s1.wav
JPC_AV_#####_s2.wav
```

Example: A tape with identifier `JPC_AV_05499` with audio on both sides produces:

```
JPC_AV_05499_s1.wav
JPC_AV_05499_s2.wav
```

A single-sided tape with identifier `JPC_AV_05500` produces:

```
JPC_AV_05500_s1.wav
```

## 1.C. Photographs of Ephemera

Ephemera associated with a tape — inserts, notes, or paperwork found in the tape case — are scanned as sequentially numbered TIFF files using the tape's JPC AV identifier.

Convention:

```
JPC_AV_#####_E_001.TIFF
JPC_AV_#####_E_002.TIFF
```

Example: Two ephemera items associated with tape `JPC_AV_12345` produce:

```
JPC_AV_12345_E_001.TIFF
JPC_AV_12345_E_002.TIFF
```

> **N.B.** The file names are settled; the Component Unique Identifier of the ephemera's ArchivesSpace record is not — see 3.B and Appendix B.

## 1.D. Photographs of Cases and Tapes

Still image photographs documenting the physical condition of tape cases and tapes are delivered as sequentially numbered TIFF files prefixed with `REF_`.

Convention:

```
REF_JPC_AV_#####_001.TIFF
REF_JPC_AV_#####_002.TIFF
```

Images are ordered according to DigiTeam standards:

1. Case front
2. Case back
3. Case sides (as needed)
4. Inside case
5. Tape front
6. Tape back
7. Tape sides (as needed)

Example: Three reference photographs for tape `JPC_AV_05498` produce:

```
REF_JPC_AV_05498_001.TIFF
REF_JPC_AV_05498_002.TIFF
REF_JPC_AV_05498_003.TIFF
```

## 1.E. Caption and Audio Description Files

Caption and audio description (AD) files are delivered as plain text files prefixed with `REF_`.

### Video

Convention:

```
REF_JPC_AV_#####_CAPTIONS.txt
REF_JPC_AV_#####_AD.txt
```

Example:

```
REF_JPC_AV_05498_CAPTIONS.txt
REF_JPC_AV_05498_AD.txt
```

### Audio

Caption files for audio are per-side, following the same side suffix convention as the WAV files.

Convention:

```
REF_JPC_AV_#####_s1_CAPTIONS.txt
REF_JPC_AV_#####_s2_CAPTIONS.txt
```

Example:

```
REF_JPC_AV_05499_s1_CAPTIONS.txt
REF_JPC_AV_05499_s2_CAPTIONS.txt
```

## 1.F. Directory Names After ArchivesSpace Linking

Each digitized tape is delivered in a folder named for its identifier. Once the tape's ArchivesSpace record exists, `aspace-rename-directories.py` (see 0.B) links the two by renaming the folder to carry the record's ref_id:

```
JPC_AV_#####/                       before
JPC_AV_#####_refid_<ref_id>/        after
```

The ref_id is the 32-character value ArchivesSpace assigns to the record. A folder that cannot be matched to exactly one record is left untouched and counted as a failure; a folder already stamped is not selected again. Example:

```
JPC_AV_05498_refid_3f9c2a7d0b1e4c6a8d5f7e9b2c4a6d8e/
└── JPC_AV_05498.mkv
```

The media file keeps its delivered name unless the run is given `--rename-media`, which stamps it the same way (`JPC_AV_05498_refid_<ref_id>.mkv`). That option is refused for a folder whose checksum manifest names the file, or whose manifests cannot be read, since the rename would orphan the manifest entry.

Optical-disc transfers arrive as a package rather than a single file. The manifests record the inner names, so only the top-level folder is renamed and nothing inside the package changes (`--rename-media` is rejected in this mode):

```
JPC_AV_14180/                       before
├── JPC_AV_14180.iso                preservation image
├── JPC_AV_14180_manifest.json
└── access_JPC_AV_14180/
    ├── JPC_AV_14180.mp4            access copy: runtime comes from here
    └── JPC_AV_14180_access_manifest.json

JPC_AV_14180_refid_<ref_id>/        after (run with --mp4)
```

A multi-titleset disc (the converter keeps the unsuffixed main `JPC_AV_#####.mp4` and adds `JPC_AV_#####_title02.mp4`, `_title03.mp4`... for the other titlesets) has no single runtime and is refused; its record is handled by hand (see 4.D.iv). Audio deliveries (1.B) are not yet handled by the script — see Appendix B.

---

# Chapter 2. The Audiovisual Materials Series

## 2.A. Scope and History of the Audiovisual Collection

The Johnson Publishing Company Archive comprises approximately 10,527 audio and video recordings documenting JPC productions from the late 1970s through the early 2000s. The audiovisual material was previously housed in the basement studio of the JPC office building at 820 South Michigan Avenue, Chicago, which also served as a working production studio for the Ebony/Jet Showcase and related programs. The tapes were boxed by Armstrong-Johnston in approximately 525 boxes, organized primarily by physical format rather than content, with no discernible pre-existing intellectual arrangement.

Because the tapes arrived without a meaningful archival arrangement, and because substantial portions are unlabeled or ambiguously labeled, no attempt is made to retain the original physical organization. The arrangement described in this manual reflects a newly imposed intellectual arrangement based on content, production history, and the relationships between items.

## 2.B. Position Within the Overall Collection

Intellectually, the Audiovisual Materials series is one of ten series in the JPC Archive:

| Series | Notes |
|--------|-------|
| Black-and-White Photographs | |
| Color Photographs | |
| Oversize Photographs | |
| Johnson Publishing Company Records | |
| Ebony Fashion Fair Records | |
| Fashion Fair Cosmetics Records | |
| Staff and Freelance Photographers | |
| Business and Production Files | |
| Print Materials | |
| **Audiovisual Materials** | **Subject of this manual** |

In ArchivesSpace, however, the audiovisual material is described in a resource of its own — *Johnson Publishing Company records audio-visual materials* — separate from the resource for the rest of the archive. That resource record is the collection level for everything in this manual (see 4.A), and the program categories below are its top-level series (see 3.A).

## 2.C. Program Categories (ArchivesSpace Series)

The audiovisual material is divided into the following program categories, each a series in the AV resource. These reflect the primary categories identified through inventory work and digitization. Additional categories may be added as content discoveries arise.

| Program category (Series) | Description |
|------------|-------------|
| Ebony/Jet Showcase, TV Program | The flagship series. Ran 1984–1993; constitutes the largest body of tape in the collection. Divided into sub-series by season and then by episode. |
| Ebony/Jet Celebrity Showcase | One season (1982–83), preceding the main EJS run. Treated as a separate series. Episodes numbered with 2-digit identifiers. |
| Ebony/Jet Fashion Fair | Content related to the Ebony Fashion Fair and associated cosmetics products. The most varied category in terms of production sources and date range. |
| Johnson Family Media Appearances | News programs and promotional broadcasts featuring members of the Johnson family or JPC executives, where the media appearance itself is the primary focus. Also includes internally produced programs about JPC. |
| American Black Achievement Awards | Broadcasts of the annual American Black Achievement Awards, 1978–1993. |
| Journalist Field Recordings | Approximately 500 audio recordings, likely made by JPC journalists recording conversations with subjects in connection with editorial work for JPC publications. Recording quality varies widely. |

> **N.B.** Do not create a new series or sub-series without prior consultation with McDowell, Blake.

---

# Chapter 3. Hierarchy and Levels of Description

## 3.A. Overview

The audiovisual material is described in its own ArchivesSpace resource (*Johnson Publishing Company records audio-visual materials*). Within it, archival objects correspond to the following levels, reflecting a whole-to-part relationship from the program down to the individual tape or recording:

| Level | When to Use |
|-------|-------------|
| Series | A program category (see 2.C), such as *Ebony/Jet Showcase, TV series, 1984–1993* or *Journalist Field Recordings*. Pre-existing in ArchivesSpace; not created by processing archivists. |
| Sub-Series | A subdivision of a series: a season, and beneath it an individual episode (or, for a single-season program, the episode directly). Do not create without prior consultation. |
| File | A grouping of items within an episode: the three pre-created production-stage groupings (Edited, Raw, Promo), and beneath Raw, a record for each multi-tape unit such as the three tapes of one interview. See 3.E and 4.C. |
| Item | A single physical object: one videotape, one audiocassette, one reel. The primary level at which AV description occurs. |

## 3.B. The EJS Hierarchy in Detail

The Ebony/Jet Showcase series has the deepest nesting structure currently in use. The full hierarchy for an EJS episode, showing titles as entered (ArchivesSpace appends dates and identifiers in the tree display), is:

```
Ebony/Jet Showcase, TV series  (Series)
> Season 1  (Sub-Series)
> > Episode 1001  (Sub-Series)
> > > Edited  (File)
> > > > Ebony/Jet Showcase, Episode 1001  (Item)
> > > > > Ephemera  (Item — child of tape, if applicable)
> > > Raw  (File)
> > > > [interview file record — title defined by the Media Archivist]  (File)
> > > > > Ebony/Jet Showcase, Episode 1001, James Brown Interview, Tape 1 of 3  (Item)
> > > > > Ebony/Jet Showcase, Episode 1001, James Brown Interview, Tape 2 of 3  (Item)
> > > > > Ebony/Jet Showcase, Episode 1001, James Brown Interview, Tape 3 of 3  (Item)
> > > Promo  (File)
> > > > Ebony/Jet Showcase, Episode 1001, Promo  (Item)
```

In the staff interface each of these displays with its date appended (*Episode 1001, 1985-09-12*; *Ebony/Jet Showcase, Episode 1001, 1985-08-24*): that is the record's Date field, not part of the title.

Each item record carries its JPC_AV_##### identifier in the Component Unique Identifier field, never in the title (see 5.A). Records imported without a title display the identifier as a placeholder title until cataloged.

The three file-level groupings — **Edited**, **Raw**, and **Promo** — are pre-created in ArchivesSpace for each episode sub-series; a multi-tape unit beneath Raw gets a file record of its own (see 3.E). Items are linked to their *immediate* file-level parent via the `ASpace Parent RefID` column in the import CSV — one of the three groupings for an independent tape, the unit's own record for a tape in a set. The import script resolves that ref_id and has no knowledge of the grouping names themselves.

The Scope and Content note describing the episode's intellectual content, and the Production Crew note, sit at the **episode sub-series level**. The Scope and Content note with a short description of the tape's content, and the Physical Characteristics and Technical Requirements note, sit at the **item level**.

### Ephemera

If a tape has ephemera associated with it — inserts, notes, paperwork found in the tape case — that ephemera is described as an item-level child record of the tape, titled *Ephemera*. The ephemera is photographed and described separately, using the tape's JPC_AV_##### identifier in the file names (see 1.C).

The Component Unique Identifier of the ephemera record has not been settled. Three forms are under consideration (see Appendix B):

1. The parent tape's identifier with the suffix `_E` (e.g., `JPC_AV_01548_E`), matching the file names.
2. The parent tape's identifier exactly (`JPC_AV_01548`), shared with the tape's own record.
3. A separate identifier of its own.

> **N.B.** Until this is decided, consult McDowell, Blake before creating an ephemera record.

### Episode vs. Season Sub-Series

Seasons are created as sub-series directly beneath the Ebony/Jet Showcase series. Individual episodes are created as sub-series beneath the appropriate season. File and item levels are created beneath each episode sub-series as needed based on the variety of tape types present.

> **N.B.** Not every episode requires all file-level groupings. Create only the file-level records that reflect the material actually present.

## 3.C. Arranging Production Materials

The EJS series and, to a lesser extent, the Ebony/Jet Fashion Fair series are production archives. For any given episode or program, the collection may contain camera originals, rough edits, finished masters, safety copies, promos, and raw interview footage — all artifacts of different stages of the production process.

**Do not arrange production material by format.** Artifacts from different stages of production may exist in the same format, and different stages with quite different intellectual content should be described separately even if they share a format. Multiple copies of the finished program, in the same or different formats, represent the same intellectual content, but each tape is still its own item (one identifier, one container, one extent) placed under the *Edited* file record; the relationship between copies is stated in each item's Scope and Content note.

The basic stages of video production, and the types of materials typically associated with each, are:

- **Shooting:** Unedited camera footage; unedited audio recordings
- **Editing:** Rough edits; work tapes; outtakes
- **Finishing:** Final edited master; safety master; other finished versions for different distribution outlets
- **Distribution/Reference:** Broadcast copies; dubs; VHS reference copies; internal JPC library copies; alternate cuts or edits; clips

The file-level groupings within an episode sub-series — **Edited**, **Raw**, and **Promo** — reflect these stages. Not every episode will have all three.

## 3.D. Multiple Copies, Duplicates, and Originals

It is common in production archives to find multiple copies and generations of the same content. JPC policy on handling duplicates is as follows.

**Do not discard duplicates without authorization.** Different copies may be valuable for different purposes — restoration, new production, online access, or research. Consult McDowell, Blake before weeding any AV material.

**Copies that pre-date the media format:** Occasionally a tape will contain content that pre-dates the format (e.g., a VHS dub of footage recorded on a format no longer accessible). Such tapes are the archival original held by JPC even if they are not the generation of original recording. Note the copy status clearly in Physical Detail so researchers understand what they are accessing.

> **N.B.** See the NMAAHC Media Archivist for help identifying originals, duplicates, and production element types.

## 3.E. When to Use File vs. Item Level

Every tape is an item: one identifier, one container, one extent. Whether items sit directly under a production-stage file record or under a further file record of their own depends on the relationship between them:

- **Independent tapes** — the copies of a finished show, the promos — go directly under *Edited* or *Promo* as sibling items. They are related but do not depend on one another; an exact duplicate is described as such in its Scope and Content note.
- **Tapes that form one unit** — the three tapes of a raw interview (tape 1 of 3, 2 of 3, 3 of 3) — get a file-level record of their own beneath *Raw*, with an extent reflecting the total number of tapes, and the tapes as its child items.

Never describe several tapes as one item with a Number above 1 or with several extents: the second tape's identifier and container would have no record to live on, and the import, export and directory-processing scripts each assume one tape per record.

---

# Chapter 4. Description at Each Level

## 4.A. Collection-Level Description

The collection level for audiovisual material is the AV resource record itself — *Johnson Publishing Company records audio-visual materials* (see 2.B) — not the resource describing the rest of the JPC Archive. The following elements must be present or updated on that record as AV processing proceeds.

### Required Collection-Level Elements for AV Description

| Element | Required? | Notes |
|---------|-----------|-------|
| Abstract / Scope and Content | **Yes** | Must include audiovisual materials using a general material designation (sound recordings, video recordings). Do not use specific format terms (VHS, U-matic) at this level. Use form/genre terms for the types of content prominently represented (television programs, radio broadcasts). |
| Conditions Governing Access | **Yes** | Language TBD — see Appendix B. |
| Existence and Location of Copies | **Yes** | Indicate whether "all" or "some" audiovisual materials have been digitized. Do not use specific numbers. Do not mention items that have not been digitized; finding aids are not routinely updated and such notes mislead researchers. |

### Optional Collection-Level Elements

| Element | Required? | Notes |
|---------|-----------|-------|
| Conditions Governing Reproduction and Use | Optional | Use only if the donor specifically restricted reproduction rights for AV materials beyond the general collection terms. Do not make blanket statements about third-party rights without a specific donor restriction to cite. |
| Existence and Location of Originals | Optional | Use if a significant portion of AV exists only in duplicate form and the location of originals is known. |
| Physical Characteristics and Technical Requirements | Optional | |
| Processing Note | Optional | Document digitization, preservation actions, deaccessioning, or other significant interventions. Include year and funder. Language TBD — see Appendix B. |
| Sponsor Note | Optional | Acknowledge specific funding sources for digitization or processing. |

## 4.B. Series-Level Description

The series record for each program category, and its sub-series records, carry the primary narrative description of that program. This is where content-specific description belongs.

### Required Series/Sub-Series Elements

| Element | Required? | Notes |
|---------|-----------|-------|
| Title | **Yes** | See Chapter 5. |
| Date | **Yes** | Span date or single date appropriate to the series or sub-series. |
| Level of Description | **Yes** | series (program category) or sub-series (season, episode) |
| Publish? | **Yes** | Always checked. |
| Scope and Content | **Yes** | Describe the intellectual content of the series or sub-series. At the episode sub-series level, this is where the description of the episode's content goes. Use general material designations (video recordings, sound recordings) rather than format-specific terms (VHS, U-matic). Reserve format terms for the item level. |
| Production Crew note (episode sub-series, if credits known) | **Yes** (if known) | A labeled Scope and Content note containing a defined list of production credits. See 4.B.i. |

### Optional Series/Sub-Series Elements

| Element | Required? | Notes |
|---------|-----------|-------|
| Arrangement Note | Optional | Use when the arrangement of the series or sub-series warrants explanation, e.g., *Episodes are arranged chronologically by broadcast date.* Can also be used to cross-reference related documentation in other series. |
| Conditions Governing Access | Optional | Use at the series or sub-series level only if AV material beneath it has a specific access condition distinct from the collection level. |
| Conditions Governing Reproduction and Use | Optional | |
| Existence and Location of Copies | Optional | Use at the series or sub-series level if all or most material beneath it has been digitized. |

### 4.B.i Production Crew Note (Episode Sub-Series)

For episode sub-series records across all relevant sub-series, a labeled Scope and Content note is used to record production credits when they are available. This note is distinct from the general Scope and Content note describing the episode's intellectual content.

- **Note type:** Scope and Content (`scopecontent`), multipart
- **Label:** Production Crew
- **Subnote type:** Defined List
- **Source:** Entered manually in ArchivesSpace from credits on the tape label, video slate, or production documentation

Each entry in the defined list consists of a role and the name of the person who filled that role:

> *Camera — Jane Doe*  
> *Sound — John Doe*  
> *Director — [name]*

This note sits at the episode sub-series level. It applies to EJS, Ebony/Jet Celebrity Showcase, and other sub-series where production credits are known (e.g., American Black Achievement Awards).

## 4.C. File-Level Description

File-level records group related items within an episode sub-series. They carry minimal description; the substantive notes belong at the item level.

Two kinds of file-level record exist:

- **Production-stage groupings.** For EJS and Ebony/Jet Celebrity Showcase, **Edited**, **Raw**, and **Promo** are pre-created in ArchivesSpace for each episode sub-series; processing archivists do not create them.
- **Multi-tape units under Raw.** A set of tapes that forms one unit (see 3.E) gets its own file record beneath *Raw*, created by processing staff. Its title is defined by the Media Archivist.

Items are linked to their immediate file-level parent via the `ASpace Parent RefID` in the import CSV — the production-stage record for an independent tape, the unit's own file record for a tape in a set.

| Element | Required? | Notes |
|---------|-----------|-------|
| Title | **Yes** | *Edited*, *Raw*, or *Promo* for the production-stage groupings; defined by the Media Archivist for a multi-tape unit. |
| Level of Description | **Yes** | file |
| Extent | **Yes** | Aggregate extent for the group (Portion = whole, Number = total tape count beneath it). |
| Publish? | **Yes** | Always checked. |
| Scope and Content | Optional | Use only when the file group requires clarification beyond the title, or to cross-reference related material elsewhere in the finding aid. |

## 4.D. Item-Level Description

Item-level records represent individual physical objects — a single videotape or audiocassette. This is where the detailed descriptive and physical work occurs.

### Required Item-Level Elements

| Element | Required? | Notes |
|---------|-----------|-------|
| Title | **Yes** | See Chapter 5. |
| Level of Description | **Yes** | item |
| Component Unique Identifier | **Yes** | The JPC_AV_##### identifier assigned to the tape. If a tape has not been assigned a JPC_AV_##### identifier, stop and contact McDowell, Blake before proceeding. |
| Date | **Yes** (if known) | Label: creation, Edited, or broadcast. Type: Single (Inclusive for an approximated range). Enter in the Begin field in YYYY-MM-DD format. See 4.D.i. |
| Extent | **Yes** | Portion: whole. Number: 1. Type: from controlled vocabulary (see Appendix A). Physical Details: SD video, color, sound, or the applicable combination. |
| Duration | **Yes** (once digitized) | hh:mm:ss. Added programmatically to the Physical Characteristics and Technical Requirements note as a Defined List subnote by `aspace-rename-directories.py`, which creates the note if the record has none. Not entered manually, except for the two cases in 4.D.iv: audio waits for the script extension; multi-titleset discs are entered by hand. |
| Instance | **Yes** | Instance type: Moving Images (Video) or Audio. Top container type: AV Case. Top container indicator: JPC_AV_#####. See Chapter 6. |
| Publish? | **Yes** | Always checked. |

### Optional Item-Level Elements

| Element | Required? | Notes |
|---------|-----------|-------|
| Scope and Content note | Optional | A brief description of the tape's intellectual content. Use when meaningful content information is available. See 4.D.iii. |
| Physical Characteristics and Technical Requirements note | Optional | Textual observations are optional: use when there is something to report about transfer quality, playback issues, or tape condition. Directory processing creates the note when needed to hold Duration. See 4.D.iv. |
| Conditions Governing Access | Optional | Only when an individual item has a specific access condition. |
| Conditions Governing Reproduction and Use | Optional | |
| Existence and Location of Copies | Optional | |

---

### 4.D.i Date

Use the following labels for AV item-level dates:

| Label | Usage |
|-------|-------|
| creation | The date on which the recording was originally made. |
| Edited | The date on which an edited version was produced. |
| broadcast | The date on which the recording was broadcast. |

Date type will usually be **Single**. Enter the date in the Begin field in YYYY-MM-DD format. The Date Expression field should contain the same date in ISO 8601, or a human-readable approximation if the exact date is uncertain. The import script writes the expression equal to the Begin value; an approximation or `undated` is entered by hand and survives later `--update-only` runs unless that date's Begin is changed in the sheet.

If a date cannot be determined from the tape label, video slate, or context, use `undated` as the Date Expression and leave the Begin/End fields blank. If a date range can be approximated from context, use date type **Inclusive**: provide the approximation in the Date Expression (e.g., *approximately 1986–1987*) and populate Begin and End with the broadest applicable range. (A Single date with an End is not a range in ArchivesSpace — its EAD export drops the End.) The import sheet cannot express a range, so an Inclusive date is entered by hand; the importer will refuse to change it afterwards, which protects it.

---

### 4.D.ii Extent

Because AV items are described one tape at a time, Extent will almost always be:

- **Portion:** whole
- **Number:** 1
- **Type:** from controlled vocabulary (see Appendix A; must match ArchivesSpace dropdown exactly)
- **Physical Details:** see below

The Physical Details field captures the technical and physical characteristics of the recording. Use the following controlled patterns:

| Format | Physical Details Value |
|--------|----------------------|
| Video (standard def.) | SD video, color, sound |
| Video (high def.) | HD video, color, sound |
| Video (silent) | SD video, color, silent |
| Video (black and white) | SD video, BW, sound |
| Motion picture film | BW, sound, 800 feet *(adjust color/BW, sound/silent, and footage count as applicable)* |
| Audio | stereo *(or: mono, as applicable)* |

> **N.B.** `aspace-rename-directories.py` fills a *blank* Physical Details field with `SD video, color, sound` on records with a single extent; it never overwrites a value and leaves a record with several extents (not policy) alone. For tapes that deviate from the standard (BW, silent, HD), set the field manually — before or after the automated run.

**One extent per item.** An item never carries more than one extent. A tape that exists in several formats (an original 1-inch reel and its VHS reference dub; a master and its safety copy) is several tapes, each its own item under the appropriate file record, with the relationship stated in each Scope and Content note.

In the Physical Details field, note copy status where relevant:

> *Original*  
> *Duplicate*  
> *Reference copy*  
> *VHS dub; location of 1-inch original unknown*

---

### 4.D.iii Scope and Content Note

The Scope and Content note at the item level provides a brief description of the tape's intellectual content. It is created as a multipart note with one or more text subnotes.

Sources for content description include:

- Label or writing on the tape case or tape itself
- Text in the video slate or leader
- Episode inventories and production documentation
- The `ASpace Scope and Contents Note` column of the import CSV — staff-assembled, informed by the `DESCRIPTION` MKV tag — if the record was created through the automated import workflow

Keep descriptions brief. A single sentence is acceptable. It is also acceptable to leave this note out entirely if no content information is available — do not fabricate or speculate about content.

> **⟶ AAA CH. 5 SUGGESTION:** The AAA guidelines identify additional specific triggers for a Scope and Content note. Consider adopting any of the following that are useful for JPCA:
>
> - **Transcripts or paper records found with the tape:** *Includes transcript.* / *Paper note found in original tape case.*
> - **Content that does not match the label:** Note both what the label says and what is actually on the tape.
> - **Blank or unrelated content** at the start of a recording: *First 18 minutes is color bars. Program content begins at 00:18:00.*
> - **Shared physical tapes:** When one tape contains parts of two separately described items, add a cross-reference in the Scope and Content note of each. Example: *Interview concludes at 00:34:12 on tape 2; remainder of tape 2 contains the opening of the Stevie Wonder segment (see EJS episode 1001, Stevie Wonder interview, tape 1 of 3).*
> - **Unclear tape sequence:** *Sequence of original recordings appears to be: tape marked "PM" first, tape marked "Eve 1" second.*

---

### 4.D.iv Physical Characteristics and Technical Requirements Note

The Physical Characteristics and Technical Requirements (phystech) note captures playback and technical observations about the tape and its recording. Textual observations are entered as a text subnote and are optional — add them only when there is something to report. The note itself may still exist without any text: after digitization it holds the Duration list, and `aspace-rename-directories.py` creates it for that purpose when the record has none.

Sources for this note include:

- The `_TRANSFER_NOTES` embedded MKV tag — technical observations made during and after digitization (e.g., *dropout throughout picture*, *loss of tracking at 00:12:30*, *audio in left channel only*)
- The `_PRE_TRANSFER_NOTES` embedded MKV tag — physical inspection and conservation observations made before digitization (e.g., *tape baked for 20 hours at 130 degrees F*, *sticky leader removed*)
- Staff observations not captured in either MKV tag
- Visual inspection notes made by processing archivists on undigitized tapes

When records are created through the automated CSV import workflow, content is drawn from whichever of these sources are applicable, assembled into the `ASpace PhysTech Note` column of the import CSV, and reviewed and edited by staff before ingesting to ArchivesSpace.

Recording quality issues observed during or after digitization — such as poor video quality, audio distortion, or channel imbalance — are also recorded in this note, not in the Physical Detail sub-element of the Extent.

After digitization, `aspace-rename-directories.py` adds a Duration defined list subnote to this note:

- Subnote type: Defined List
- Item label: **Duration**
- Item value: hh:mm:ss (extracted from the digitized file via mediainfo)

> **N.B.** Duration is not entered manually. It is added programmatically from the digitized file and should not be edited by hand. If the record has no phystech note the script creates one to hold the list; an existing Duration is updated in place rather than duplicated.
>
> Two cases fall outside the automation for now (see Appendix B): **audio recordings**, which the script will be extended to cover, so their Duration waits for that; and **multi-titleset optical discs**, which the script refuses because there is no single runtime — these few records are completed by hand, entering the Duration list in the same form the script would write.

---

# Chapter 5. Title

## 5.A. General Title Rules

Use the official title if one exists. If no official title exists, construct one by combining relevant elements such as the series name, program type, episode number, subject matter, creator name, or qualifying descriptors.

| Rule | Detail |
|------|--------|
| No format terms | Never use a specific format term (VHS, U-matic, Betacam) as a title or part of a title. Titles should convey intellectual content, not physical format. |
| No ambiguous labels | Do not transcribe ambiguous media labels directly as unit titles. If you do not understand what a label means, researchers likely will not either. Consult the NMAAHC Media Archivist for unlabeled or poorly labeled media. |
| Qualifiers only when needed | Do not force a qualifier if one is not needed. Qualifiers should reflect markings on the tape itself or text visible in the video, not content derived by the archivist from watching the recording. |
| Qualifier consistency | Qualifiers must be consistent across tapes, and capitalized. We use **Safety Master** as the standard form — not *master (safety)*, *safety copy*, or *safety*. |
| No "copy" | Avoid the word *copy* in qualifiers. Use *Master*, *Safety*, or *Dub*; not *Master Copy* or *Safety Copy*. |
| Spelling | Retain original or colloquial spelling found on the tape unless it is an obvious typographic error that would cause confusion. Correct *mastre* to *master*, but do not change *trax* to *tracks*. |

### Unidentified or Poorly Labeled Tapes

When a tape is unlabeled or its label cannot be interpreted, do not use the label as the unit title and do not use a specific format term as the title. If the type of content can be inferred from context (e.g., it is in a box with other EJS material), use that context to construct a title. If content truly cannot be determined, use the word "unidentified" with the general material designation:

> *Unidentified Video Recording*  
> *Unidentified Sound Recording*

## 5.B. Ebony/Jet Showcase Title Pattern

For all EJS and Ebony/Jet Celebrity Showcase material, use the following pattern:

```
<Series Name>, Episode <number>[, <Qualifier if needed>]
```

*Episode* and every qualifier are capitalized (*Safety Master*, *Promo*, *Studio Footage*). The title never contains a date: the date lives in the record's Date field, and ArchivesSpace appends it to the title in the tree and in search results. What goes in the `ASpace Title` column is the title alone.

Examples:

> *Ebony/Jet Showcase, Episode 1001*\
> *Ebony/Jet Showcase, Episode 1004, Safety Master*\
> *Ebony/Jet Showcase, Episode 1005, Studio Footage*\
> *Ebony/Jet Celebrity Showcase, Episode 09, Grace Jones Makeup and Fashion Sequences*

Episode numbering conventions differ between the two programs:

| Program | Episode Number Format |
|---------|-----------------------|
| Ebony/Jet Showcase (1984–1993) | 4-digit numbers spanning all seasons: 1001, 7024, 9008, etc. |
| Ebony/Jet Celebrity Showcase (1982–83) | 2-digit numbers: 07, 13, 22, etc. |

> **N.B.** We follow normalized titling wherever possible. For episodes that have a specific broadcast title (e.g., *The San Francisco Show*), the normalized pattern takes precedence — use the episode number format and omit the broadcast title from the ArchivesSpace title field.

## 5.C. Raw Footage and Interview Tape Titles

For raw footage tapes, include enough identifying information in the title to distinguish the tape within its group. Where multiple tapes cover the same interview or shoot, use a part indicator:

> *Ebony/Jet Showcase, Episode 1001, James Brown Interview, Tape 1 of 3*\
> *Ebony/Jet Showcase, Episode 1001, James Brown Interview, Tape 2 of 3*\
> *Ebony/Jet Showcase, Episode 1001, James Brown Interview, Tape 3 of 3*

The tapes of one interview sit as items under a file record of their own beneath *Raw* (see 3.E); the file record's title is defined by the Media Archivist.

## 5.D. Non-EJS Title Patterns

For program categories other than EJS, use the same general principles: formal title if available, otherwise a constructed title using content, creator, or genre. Capitalize qualifiers, apply them only as needed, and avoid format terms and the word *copy*. As everywhere, the date belongs in the Date field, not the title.

> *American Black Achievement Awards, Broadcast*\
> *[Subject name] Interview, [publication context if known]*\
> *Ebony Fashion Fair, Promotional Footage*

## 5.E. Series and Sub-Series Titles

Use one of the following for a series or sub-series title, in order of preference:

1. The formal title of the series, if one exists, along with a form or genre term if needed for clarity (e.g., *Ebony/Jet Showcase, TV Program*).
2. The form or genre of the content, if the series contains one or two types (e.g., *Television Programs*, *Radio Broadcasts*).
3. A general material designation when the series contains varied content (e.g., *Video Recordings*, *Sound Recordings*). Avoid the term *Audiovisual Material* as a series title.

---

# Chapter 6. Instances and Containers

## 6.A. Overview

Instances in ArchivesSpace link an archival object to its physical container. Every item-level AV record must have exactly one instance.

## 6.B. Creating an Instance

1. In the Instances section, click **Add Container Instance**.
2. For **Type**, select `Moving Images (Video)` for video recordings, or the applicable type for audio.
3. In the **Top Container** field, search for the existing top container using the JPC_AV_##### identifier. If the top container does not yet exist, create it (see 6.C).

## 6.C. Top Containers for AV Material

The top container represents the outermost physical housing of the tape — in most cases, the original tape case.

| Field | Value |
|-------|-------|
| Container Type | AV Case |
| Indicator | The JPC_AV_##### identifier assigned to the tape (e.g., JPC_AV_01548). Serves as the primary container identifier in the absence of barcodes. |
| Barcode | Not currently in use for AV material. Leave blank. |

> **N.B.** Top containers for AV material are not shared between multiple archival objects. Each tape has its own AV Case top container, identified by its unique JPC_AV_##### indicator. The JPC_AV_##### identifier serves dual purposes: it appears as both the Component Unique Identifier on the archival object and as the indicator on its top container.

## 6.D. Instance Type Controlled Vocabulary

| Media Type | Instance Type Value |
|------------|---------------------|
| Video recordings | Moving Images (Video) |
| Audio recordings | Audio |

> **N.B.** The import script writes `Moving Images (Video)` only. Audio records are created manually until audio is added to the automated workflow (see Appendix B).

---

# Chapter 7. Controlled Vocabulary

## 7.A. Extent Type (Original Format)

The Extent Type field must match the ArchivesSpace `extent_extent_type` enumeration exactly. Values derive from the PB Core audiovisual metadata standard. Use `check_extent_types.py` to retrieve the current list from the live ArchivesSpace instance.

Representative values in use for JPC AV material:

| Value | Format Notes |
|-------|-------------|
| 1 inch videotape | 1-inch type C reel-to-reel. Most common format for EJS broadcast masters. |
| 2 inch videotape | Quad-head reel-to-reel. |
| 3/4 inch videotape | U-matic cassette. |
| 1/2 inch videotape | Open-reel 1/2 inch. |
| Betacam | Sony Betacam family (Betacam, Betacam SP, Digital Betacam). |
| Betamax | Consumer Betamax format. |
| VHS | Consumer VHS cassette. |
| U-matic | Alternative to "3/4 inch videotape" for U-matic cassettes. |
| MiniDV | Consumer/prosumer DV cassette. |
| videocassettes | Generic term; use only if specific format cannot be determined. |
| videoreels | Generic term for open-reel video. |
| videotapes | Generic term; use only as a last resort. |

> **N.B.** If the format of a tape cannot be identified, enter `videotapes` as the Extent Type and note the uncertainty in the Physical Characteristics and Technical Requirements note. Do not guess or estimate. Consult the NMAAHC Media Archivist if unsure.

## 7.B. Date Labels

| Label | When to Use |
|-------|-------------|
| creation | The date on which the recording was originally made or recorded. |
| Edited | The date on which an edited version was produced. |
| broadcast | The date on which the recording was broadcast or publicly transmitted. |

## 7.C. Physical Details for Extent

Combine the following terms as applicable. For most JPC video, the standard value is `SD video, color, sound`.

| Component | Options |
|-----------|---------|
| Resolution | SD video \| HD video |
| Color | color \| BW |
| Sound | sound \| silent |
| Film footage (film only) | e.g., 800 feet — always include footage count for motion picture film |

Examples:

- `SD video, color, sound`
- `SD video, BW, silent`
- `HD video, color, sound`
- `BW, sound, 800 feet`
- `stereo` (for audio)

## 7.D. Note Types and Structures

| Note Type | Level | Purpose and Structure |
|-----------|-------|-----------------------|
| Scope and Content (`scopecontent`) | Item | Brief description of intellectual content. Text subnote. |
| Physical Characteristics and Technical Requirements (`phystech`) | Item | Technical and physical observations about the tape and its recording. Text subnote. Includes pre-transfer physical inspection notes and post-transfer playback quality notes. After digitization, also contains a Defined List subnote with label "Duration" and value in hh:mm:ss. |
| Scope and Content — Production Crew (`scopecontent`, label: "Production Crew") | Episode sub-series | Labeled note containing a Defined List of production credits. Role and name for each crew member. Entered manually in ArchivesSpace from tape labels, slates, or production documentation. |

---

# Chapter 8. Agents and Subjects

## 8.A. Agents

Agent links at the AV item level are more nuanced than in the still image portion of the collection, because AV items — particularly raw footage tapes — may not feature all of the agents associated with their parent episode. Use caution when considering agent inheritance from parent records.

Link agents at the item level only when:

- The agent is directly associated with the content of that specific tape, based on tape label, video slate, or content review.
- The agent is not already represented at a higher level in a way that would apply broadly to this item.

> **N.B.** Do not apply agents from a parent episode record to individual raw footage tapes unless those agents actually appear on the specific tape. Raw footage tapes for a given episode may feature only a portion of the episode's subjects. Manual review is required before linking agents to raw footage items. Consult McDowell, Blake if unsure.

### Agent Roles for AV

| Role | Usage |
|------|-------|
| Subject | Individuals, groups, or organizations whose activities or presence are the subject of the recording. |
| Creator | Organizations or individuals responsible for producing or recording the content (e.g., Johnson Publishing Company). Use the Creator role with the Relator term Producer or Broadcaster as applicable. |

> **N.B.** Do not delete existing agent records. If an agent record is incorrect or missing, contact McDowell, Blake.

## 8.B. Subjects

Subject terms at the AV item level follow the same general approach as the still image portion of the collection. Link subject terms to item records when they are directly applicable to the intellectual content of the specific tape and are not already represented at a higher hierarchical level.

---

# Appendix A. ArchivesSpace Controlled Vocabulary for AV

## A.1 Extent Type Values (Partial List)

Fetch the full current list from ArchivesSpace using `check_extent_types.py`. The values below are representative; the authoritative source is the live ArchivesSpace enumeration.

- 1 inch videotape
- 2 inch videotape
- 3/4 inch videotape
- 1/2 inch videotape
- Betacam
- Betamax
- VHS
- U-matic
- MiniDV
- audiocassette
- audiotape reel
- videocassettes
- videoreels
- videotapes

## A.2 Required Item-Level Fields: Quick Reference

| Element | Required? | Value / Notes |
|---------|-----------|---------------|
| Level | **Yes** | item |
| Title | **Yes** | See Chapter 5 |
| Component Unique Identifier | **Yes** | JPC_AV_##### |
| Date Label | **Yes** (if known) | creation \| Edited \| broadcast |
| Date Type | **Yes** | Single (Inclusive for an approximated range) |
| Date Expression | **Yes** | YYYY-MM-DD or approximation |
| Extent Portion | **Yes** | whole |
| Extent Number | **Yes** | 1 |
| Extent Type | **Yes** | From controlled vocabulary; must match ArchivesSpace dropdown exactly |
| Extent Physical Details | **Yes** | SD video, color, sound (or applicable combination) |
| Duration | **Yes** (once digitized) | Added programmatically to the Physical Characteristics and Technical Requirements note as a Defined List subnote by `aspace-rename-directories.py`. Not entered manually, except audio (deferred) and multi-titleset discs (by hand) — see 4.D.iv. |
| Instance Type | **Yes** | Moving Images (Video) or Audio |
| Top Container Type | **Yes** | AV Case |
| Top Container Indicator | **Yes** | JPC_AV_##### |
| Publish? | **Yes** | Always checked |

## A.3 Optional Item-Level Fields: Quick Reference

| Element | Required? | Value / Notes |
|---------|-----------|---------------|
| Scope and Content note | Optional | Multipart note, text subnote. Brief intellectual content description. |
| Physical Characteristics & Technical Requirements note | Optional | Multipart note, text subnote. Use when there is something to report about transfer quality, playback issues, or tape condition. Defined list subnote added programmatically for Duration (the note is created if absent). |
| Conditions Governing Access | Optional | Only when item has a specific access condition |
| Conditions Governing Reproduction and Use | Optional | |
| Existence and Location of Copies | Optional | |
| Agents | Optional | Subject, Creator. See Chapter 8. |
| Subjects | Optional | See Chapter 8. |

---

# Appendix B. Open Questions and Items Under Discussion

The following items are pending final decisions. Consult McDowell, Blake for current status before proceeding on any of these points.

| Item | Status |
|------|--------|
| Ephemera Component Unique Identifier — three candidate forms: (1) the parent tape's identifier with `_E` appended, matching the scan file names (1.C); not accepted by the scripts' identifier check, so ephemera would stay outside the automated workflows unless the tools gain an exception. (2) The parent tape's identifier exactly; two records would share one identifier, which the scripts treat as a conflict. (3) A separate identifier of its own; accepted by the scripts, but would not match the file names. | Under discussion |
| Audio recordings: instance type `Audio` is policy, but no script imports or renames audio records (WAV deliveries, 1.B). Plan: extend `aspace-rename-directories.py` to read audio runtime; until then audio records are created manually and their Duration waits for the script. | Automation planned |
| Multi-titleset optical discs: no single runtime, so the directory-processing script refuses them. Plan: enter the Duration list by hand for the few (if any) such records. Whether to record one runtime per titleset or a single aggregate is not yet decided. | Manual procedure; runtime form open |
| Conditions Governing Access note language specific to JPCA AV | Language TBD |
| Exact language for the Processing Note documenting digitization actions | TBD |
| Agent inheritance for raw footage tapes: how to prevent inappropriate propagation of episode-level agents to individual item records | Manual review workflow in progress |
| Journalist Field Recordings series: whether to subdivide and on what basis as content is better understood | TBD |
| Rights statement — language, applicability, and whether required at item level for JPCA AV | TBD |

---

*JPC Archive | Audiovisual Materials: Arrangement and Description in ArchivesSpace | Version 2026-09-12 | NMAAHC / GRI*
