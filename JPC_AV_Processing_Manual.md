# JOHNSON PUBLISHING COMPANY ARCHIVE
### Audiovisual Materials: Arrangement and Description in ArchivesSpace

Version 2026-03-30  
Smithsonian National Museum of African American History and Culture (NMAAHC) / Getty Research Institute (GRI)

---

## Introduction

This manual serves as a guide for archival staff describing audiovisual materials from the Johnson Publishing Company (JPC) Archive in ArchivesSpace. It covers the intellectual arrangement of the Audiovisual Materials series and the practical steps for creating compliant finding aid records at each level of description. The procedures and conventions defined here complement the broader JPC Archival Processing Manual, which governs the photographic and textual portions of the collection; readers should consult both documents as needed.

These guidelines are rooted in EAD2002 and DACS standards as implemented at the Smithsonian Institution, and draw on the Archives of American Art (AAA) guidelines for processing collections with audiovisual material. Where JPC practice departs from AAA practice, this manual takes precedence.

This is a living document and will be revised as the audiovisual processing project develops.

> **N.B.** The still image processing manual (JPC Archival Processing Manual, Version xxxx.xx.xx) remains the governing document for photographic and textual portions of the collection. This manual addresses only the Audiovisual Materials series.

---

## Acknowledgements

These guidelines draw on the Archives of American Art's *Processing Collections With Audiovisual Material* (Chapters 4 and 5), and the existing JPC Archival Processing Manual.

Additional resources consulted:

- PB Core Audiovisual Metadata Standard
- NMAAHC Media Archivist digitization documentation and embedded metadata specifications

---

# Chapter 0. ArchivesSpace Fields Reference

This chapter documents every ArchivesSpace field used in JPCA AV item-level records: what data goes in it, how it gets there, whether manual adjustment is needed after import, and the archival rationale for the mapping.

Fields are grouped by the stage at which they are populated.

> **N.B.** Where "manual adjustment" is noted, this may mean editing the record directly in the ArchivesSpace staff interface or re-importing data for that record via the API. The import process supports interleaving — records can be updated by re-running the import script with `--update-existing` at any point.

---

## 0.A. Fields Populated via CSV Import

*Script: `aspace_csv_import.py`*

These fields are written to ArchivesSpace by the CSV import script. The source for each is a column in the import CSV.

---

### Title

| | |
|---|---|
| **ArchivesSpace field** | `title` |
| **CSV column** | `TITLE` |
| **MKV embedded tag** | `TITLE` — should match the ArchivesSpace title exactly if already cataloged; leave blank otherwise |
| **Data** | The descriptive title for the tape, following the conventions in Chapter 4. |
| **How imported** | Written directly by the import script. |
| **Manual adjustment** | Sometimes. Titles may be refined after import as more information becomes available. |
| **Why here** | DACS requires a title at every level of description. |

---

### Component Unique Identifier

| | |
|---|---|
| **ArchivesSpace field** | `component_id` |
| **CSV column** | `CATALOG_NUMBER` |
| **MKV embedded tag** | `CATALOG_NUMBER` |
| **Data** | The JPC_AV_##### identifier assigned to the tape and its case. |
| **How imported** | Written directly by the import script. |
| **Manual adjustment** | No. Must not be changed after creation. |
| **Why here** | Serves as the stable, unique identifier linking the physical object to its ArchivesSpace record and its digitized file. Duplicate values are a fatal error in the import script. |

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
| **Data** | The ref_id of the file-level record (Edited, Raw, or Promo) under which this item sits. |
| **How imported** | The script looks up the archival object with this ref_id and links the new item as its child. |
| **Manual adjustment** | No. If the parent ref_id is wrong the record must be moved manually. |
| **Why here** | Establishes the item's position in the hierarchy. Required — the import script will fail if the parent does not exist. |

---

### Resource

| | |
|---|---|
| **ArchivesSpace field** | `resource.ref` |
| **CSV column** | — (hardcoded via `creds.py`) |
| **MKV embedded tag** | No |
| **Data** | `/repositories/2/resources/7` |
| **How imported** | Hardcoded from `creds.py` configuration. |
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
| **Manual adjustment** | Occasionally — records may be suppressed individually in ArchivesSpace if needed. |
| **Why here** | All JPCA AV records are published to public interfaces by default. |

---

### Dates

| | |
|---|---|
| **ArchivesSpace field** | `dates[]` |
| **CSV columns** | `Creation or Recording Date`, `Edit Date`, `Broadcast Date` |
| **MKV embedded tag** | No |
| **Data** | YYYY-MM-DD. Date type: Single. Labels: `creation`, `Edited`, `broadcast` respectively. |
| **How imported** | The script converts M/D/YYYY input to YYYY-MM-DD and creates a separate date object for each non-empty column. |
| **Manual adjustment** | Sometimes — dates may be corrected or added after import. |
| **Why here** | DACS requires dates at the item level. Three separate date labels reflect the distinct production stages of AV material. |

---

### Extent Type

| | |
|---|---|
| **ArchivesSpace field** | `extents[].extent_type` |
| **CSV column** | `Original Format` |
| **MKV embedded tag** | `ORIGINAL_MEDIA_TYPE` — note this tag uses a different format: Format (PB Core), manufacturer, model (e.g., `1-inch type C, Sony, V1-K`). The CSV `Original Format` column uses only the PB Core format term to match the ArchivesSpace controlled vocabulary. |
| **Data** | The physical format of the tape, e.g., `1 inch videotape`, `VHS`. Must match the ArchivesSpace `extent_extent_type` controlled vocabulary exactly. |
| **How imported** | Written by the import script. Validated against the live ArchivesSpace enumeration before import. |
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
| **Manual adjustment** | Occasionally — Number may be updated manually when multiple exact duplicate copies are retained together. |
| **Why here** | Each item record describes a single tape (Portion = whole, Number = 1) per JPCA AV description policy. |

---

### Scope and Content Note

| | |
|---|---|
| **ArchivesSpace field** | `notes[]` — type: `scopecontent`, multipart, text subnote |
| **CSV column** | `DESCRIPTION` |
| **MKV embedded tag** | `DESCRIPTION` — note this tag is intentionally brief and may differ from the fuller ArchivesSpace scope note; it is not a direct source for the ArchivesSpace field. |
| **Data** | A brief description of the tape's intellectual content. |
| **How imported** | Written by the import script as a multipart note with a text subnote. Only created when the column contains content. |
| **Manual adjustment** | Sometimes — content may be refined after import. |
| **Why here** | Provides intellectual content description at the item level. |

> **N.B.** A new CSV column — likely `ASpace Scope and Contents Note` or `ASpace Description` (name TBD) — will replace `DESCRIPTION` as the source for the ArchivesSpace import. Like `ASpace PhysTech Note`, this column will be staff-assembled, informed by but not identical to the `DESCRIPTION` MKV tag, and reviewed before import. The `DESCRIPTION` field will remain as the MKV tag embedding source. See Appendix B.

---

### Physical Characteristics and Technical Requirements Note

| | |
|---|---|
| **ArchivesSpace field** | `notes[]` — type: `phystech`, multipart, text subnote |
| **CSV column** | `ASpace PhysTech Note` |
| **MKV embedded tags** | `_TRANSFER_NOTES` and/or `_PRE_TRANSFER_NOTES` — the CSV column may draw from either or both, or may contain staff-written content not found in either tag. |
| **Data** | Technical and physical observations about the tape. May draw from `_TRANSFER_NOTES`, `_PRE_TRANSFER_NOTES`, both, or neither — assembled and edited by staff before import. |
| **How imported** | Written by the import script as a multipart note with a text subnote. Only created when the column contains content. |
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
| **How imported** | The import script creates a new top container for each item using the CATALOG_NUMBER as the indicator. |
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
| **How imported** | Hardcoded in the import script. |
| **Manual adjustment** | No. |
| **Why here** | Required by ArchivesSpace to type the instance. |

---

## 0.B. Fields Populated via Directory Processing Script

*Script: `aspace-rename-directories.py`*

These fields are written to ArchivesSpace by the directory processing script, which runs after the digitized .mkv file is available.

---

### Duration

| | |
|---|---|
| **ArchivesSpace field** | `notes[]` — type: `phystech`, Defined List subnote, label: `Duration` |
| **Source** | Extracted from the .mkv file via `mediainfo` |
| **MKV embedded tag** | No — extracted directly from the file's stream metadata, not from a tag. |
| **Data** | Runtime in hh:mm:ss format, e.g., `01:23:45`. |
| **How imported** | `aspace-rename-directories.py` fetches the existing ArchivesSpace record, appends a Defined List subnote to the Physical Characteristics and Technical Requirements note (preserving any existing text subnotes), and writes it back. Idempotent — re-running removes and rewrites the Duration entry without duplicating it. |
| **Manual adjustment** | No. Must not be edited by hand. |
| **Why here** | Duration is the most important descriptive datum for a video recording after its title. Extracting it from the actual digitized file guarantees accuracy over any estimate in the source documentation. |

---

### Extent Physical Details

| | |
|---|---|
| **ArchivesSpace field** | `extents[].physical_details` |
| **Source** | Hardcoded in `aspace-rename-directories.py` |
| **MKV embedded tag** | No |
| **Data** | `SD video, color, sound` (standard value for most JPCA AV). |
| **How imported** | Set programmatically on all extents when the script updates the record. |
| **Manual adjustment** | Yes — tapes that deviate from the standard (BW, silent, HD) must be corrected manually after the automated update. |
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
| **Why here** | Provides named access points for researchers. Agent inheritance from parent episode records to individual raw footage tapes requires manual review — see Chapter 7. |

---

## 0.D. Complete Example: CSV Row → ArchivesSpace Record

The following example shows what a single item-level archival object looks like as JSON at each stage of the workflow. This is the actual data structure sent to and stored by the ArchivesSpace API.

### Sample CSV Row

```
CATALOG_NUMBER:             JPC_AV_00012
TITLE:                      Ebony/Jet Celebrity Showcase, episode 22, promo
Creation or Recording Date: 8/1/1982
Edit Date:                  [empty]
Broadcast Date:             [empty]
Original Format:            2 inch videotape
ASpace Parent RefID:        abc123def456
DESCRIPTION:                Promotional clip for episode 22 of the Ebony/Jet Celebrity Showcase series.
ASpace PhysTech Note:       Slight ringing present throughout. Hue is inconsistent; skin tones are redder in some sections.
```

---

### Stage 1: After `aspace_csv_import.py`

The import script creates the archival object and its top container.

**Archival Object:**

```json
{
  "jsonmodel_type": "archival_object",
  "resource": {"ref": "/repositories/2/resources/7"},
  "parent": {"ref": "/repositories/2/archival_objects/12345"},
  "level": "item",
  "publish": true,
  "title": "Ebony/Jet Celebrity Showcase, episode 22, promo",
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

**Top Container:**

```json
{
  "indicator": "JPC_AV_00012",
  "type": "AV Case",
  "repository": {"ref": "/repositories/2"}
}
```

---

### Stage 2: After `aspace-rename-directories.py`

The directory processing script updates the record with Duration and Physical Details.

**Updated Physical Characteristics and Technical Requirements note** (Duration added as a Defined List subnote):

```json
{
  "jsonmodel_type": "note_multipart",
  "type": "phystech",
  "publish": true,
  "subnotes": [
    {
      "jsonmodel_type": "note_text",
      "content": "Slight ringing present throughout. Hue is inconsistent; skin tones are redder in some sections."
    },
    {
      "jsonmodel_type": "note_definedlist",
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

**Updated Extent** (Physical Details added):

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

### How It Looks in the ArchivesSpace Staff Interface

**Basic Information**
- Level: Item
- Title: Ebony/Jet Celebrity Showcase, episode 22, promo
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

# Chapter 1. The Audiovisual Materials Series

## 1.A. Scope and History of the Audiovisual Collection

The Johnson Publishing Company Archive comprises approximately 10,527 audio and video recordings documenting JPC productions from the late 1970s through the early 2000s. The audiovisual material was previously housed in the basement studio of the JPC office building at 820 South Michigan Avenue, Chicago, which also served as a working production studio for the Ebony/Jet Showcase and related programs. The tapes were boxed by Armstrong-Johnston in approximately 525 boxes, organized primarily by physical format rather than content, with no discernible pre-existing intellectual arrangement.

Because the tapes arrived without a meaningful archival arrangement, and because substantial portions are unlabeled or ambiguously labeled, no attempt is made to retain the original physical organization. The arrangement described in this manual reflects a newly imposed intellectual arrangement based on content, production history, and the relationships between items.

## 1.B. Position Within the Overall Collection

The Audiovisual Materials series is one of ten series in the JPC Archive:

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

## 1.C. Sub-Series

The Audiovisual Materials series is divided into the following sub-series. These reflect the primary program categories identified through inventory work and digitization. Additional sub-series may be added as content discoveries arise.

| Sub-Series | Description |
|------------|-------------|
| Ebony/Jet Showcase, TV Program | The flagship sub-series. Ran 1984–1993; constitutes the largest body of tape in the collection. Further divided into sub-series by season and then by episode. |
| Ebony/Jet Celebrity Showcase | One season (1982–83), preceding the main EJS run. Treated as a separate sub-series. Episodes numbered with 2-digit identifiers. |
| Ebony/Jet Fashion Fair | Content related to the Ebony Fashion Fair and associated cosmetics products. The most varied sub-series in terms of production sources and date range. |
| Johnson Family Media Appearances | News programs and promotional broadcasts featuring members of the Johnson family or JPC executives, where the media appearance itself is the primary focus. Also includes internally produced programs about JPC. |
| American Black Achievement Awards | Broadcasts of the annual American Black Achievement Awards, 1978–1993. |
| Journalist Field Recordings | Approximately 500 audio recordings, likely made by JPC journalists recording conversations with subjects in connection with editorial work for JPC publications. Recording quality varies widely. |

> **N.B.** Do not create a new sub-series without prior consultation with McDowell, Blake.

---

# Chapter 2. Hierarchy and Levels of Description

## 2.A. Overview

Archival objects in ArchivesSpace correspond to the following levels, reflecting a whole-to-part relationship from the series down to the individual tape or recording:

| Level | When to Use |
|-------|-------------|
| Series | The Audiovisual Materials series as a whole. Pre-existing in ArchivesSpace; not created by processing archivists. |
| Sub-Series | A major subdivision of the series, such as Ebony/Jet Showcase or Journalist Field Recordings. Also used for seasons and individual episodes within EJS. Do not create without prior consultation. |
| File | A logical grouping of multiple related items within a sub-series or episode. Used when several tapes together constitute a single intellectual unit — e.g., all broadcast masters for a given episode, or all raw footage tapes for a single interview. |
| Item | A single physical object: one videotape, one audiocassette, one reel. The primary level at which AV description occurs. |

## 2.B. The EJS Hierarchy in Detail

The Ebony/Jet Showcase sub-series has the deepest nesting structure currently in use. The full hierarchy for an EJS episode is:

```
Ebony/Jet Showcase, TV series, 1984–1993  (Series)
> Season 1, 1985–1986  (Sub-Series)
> > Episode 1001, 1985-09-12  (Sub-Series)
> > > Edited  (File)
> > > > [Title of tape — often from inventory], JPC_AV_01548, 1985-08-24  (Item)
> > > > > Ephemera, JPC_AV_01549  (Item — child of tape, if applicable)
> > > Raw  (File)
> > > > Ebony/Jet Showcase, episode 1001, James Brown interview tape 1 of 3, 1985-08-03  (Item)
> > > > Ebony/Jet Showcase, episode 1001, James Brown interview tape 2 of 3, 1985-08-03  (Item)
> > > > Ebony/Jet Showcase, episode 1001, James Brown interview tape 3 of 3, 1985-08-03  (Item)
> > > Promo  (File)
> > > > [Title of tape], JPC_AV_01550, 1985-08-24  (Item)
```

The three file-level groupings — **Edited**, **Raw**, and **Promo** — are pre-created in ArchivesSpace for each episode sub-series. Items are linked to one of these three parents via the `ASpace Parent RefID` column in the import CSV. These grouping names are also defined as a constant in `aspace_csv_import.py` (see `EJS_FILE_GROUPS`).

The Scope and Content note describing the episode's intellectual content, and the Production Crew note, sit at the **episode sub-series level**. The Scope and Content note with a short description of the tape's content, and the Physical Characteristics and Technical Requirements note, sit at the **item level**.

### Ephemera

If a tape has ephemera associated with it — inserts, notes, paperwork found in the tape case — that ephemera is described as an item-level child record of the tape. The ephemera will be photographed and described separately. Its JPC_AV_##### identifier serves as both the Component Unique Identifier and the basis for its title (e.g., *Ephemera, JPC_AV_01549*).

### Episode vs. Season Sub-Series

Seasons are created as sub-series directly beneath the Ebony/Jet Showcase sub-series. Individual episodes are created as sub-series beneath the appropriate season. File and item levels are created beneath each episode sub-series as needed based on the variety of tape types present.

> **N.B.** Not every episode requires all file-level groupings. Create only the file-level records that reflect the material actually present.

## 2.C. Arranging Production Materials

The EJS sub-series and, to a lesser extent, the Ebony/Jet Fashion Fair sub-series are production archives. For any given episode or program, the collection may contain camera originals, rough edits, finished masters, safety copies, promos, and raw interview footage — all artifacts of different stages of the production process.

**Do not arrange production material by format.** Artifacts from different stages of production may exist in the same format, and different stages with quite different intellectual content should be described separately even if they share a format. Conversely, multiple copies of the finished program in different formats represent the same intellectual content and should generally be described together in a single component with multiple extents.

The basic stages of video production, and the types of materials typically associated with each, are:

- **Shooting:** Unedited camera footage; unedited audio recordings
- **Editing:** Rough edits; work tapes; outtakes
- **Finishing:** Final edited master; safety master; other finished versions for different distribution outlets
- **Distribution/Reference:** Broadcast copies; dubs; VHS reference copies; internal JPC library copies; alternate cuts or edits; clips

The file-level groupings within an episode sub-series — **Edited**, **Raw**, and **Promo** — reflect these stages. Not every episode will have all three.

## 2.D. Multiple Copies, Duplicates, and Originals

It is common in production archives to find multiple copies and generations of the same content. JPC policy on handling duplicates is as follows.

**Do not discard duplicates without authorization.** Different copies may be valuable for different purposes — restoration, new production, online access, or research. Consult McDowell, Blake before weeding any AV material.

**Copies that pre-date the media format:** Occasionally a tape will contain content that pre-dates the format (e.g., a VHS dub of footage recorded on a format no longer accessible). Such tapes are the archival original held by JPC even if they are not the generation of original recording. Note the copy status clearly in Physical Detail so researchers understand what they are accessing.

> **N.B.** See the NMAAHC Media Archivist for help identifying originals, duplicates, and production element types.

## 2.E. When to Use File vs. Item Level

Use the **file level** when multiple tapes constitute a single intellectual grouping, such as the three tapes of a raw interview. In this case:

- Create a file-level record to represent the group as a whole, with an extent reflecting the total number of tapes.
- Create child item-level records for each individual tape.

An exception applies when multiple exact copies of a tape on the same format are retained together. Describe them at the item level with Portion = Whole and Number = the count of copies retained.

---

# Chapter 3. Description at Each Level

## 3.A. Collection-Level Description

The Audiovisual Materials series is a sub-component of the collection-level resource record describing the JPC Archive as a whole. The following elements must be present or updated as AV processing proceeds.

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

## 3.B. Series-Level Description

The Audiovisual Materials series record, and its sub-series records, carry the primary narrative description of each program category. This is where content-specific description belongs.

### Required Series/Sub-Series Elements

| Element | Required? | Notes |
|---------|-----------|-------|
| Title | **Yes** | See Chapter 4. |
| Date | **Yes** | Span date or single date appropriate to the series or sub-series. |
| Level of Description | **Yes** | sub-series |
| Publish? | **Yes** | Always checked. |
| Scope and Content | **Yes** | Describe the intellectual content of the series or sub-series. At the episode sub-series level, this is where the description of the episode's content goes. Use general material designations (video recordings, sound recordings) rather than format-specific terms (VHS, U-matic). Reserve format terms for the item level. |
| Production Crew note (episode sub-series, if credits known) | **Yes** (if known) | A labeled Scope and Content note containing a defined list of production credits. See 3.B.i. |

### Optional Series/Sub-Series Elements

| Element | Required? | Notes |
|---------|-----------|-------|
| Arrangement Note | Optional | Use when the arrangement of the sub-series warrants explanation, e.g., *Episodes are arranged chronologically by broadcast date.* Can also be used to cross-reference related documentation in other series. |
| Conditions Governing Access | Optional | Use at the sub-series level only if AV material in that sub-series has a specific access condition distinct from the collection level. |
| Conditions Governing Reproduction and Use | Optional | |
| Existence and Location of Copies | Optional | Use at the sub-series level if all or most material in that sub-series has been digitized. |

### 3.B.i Production Crew Note (Episode Sub-Series)

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

## 3.C. File-Level Description

File-level records group related items within an episode sub-series. They carry minimal description; the substantive notes belong at the item level.

For EJS and Ebony/Jet Celebrity Showcase, the three pre-defined file-level groupings are **Edited**, **Raw**, and **Promo**. These records are pre-created in ArchivesSpace for each episode sub-series; processing archivists do not create them. Items are linked to one of these three parents via the `ASpace Parent RefID` in the import CSV.

| Element | Required? | Notes |
|---------|-----------|-------|
| Title | **Yes** | One of: *Edited*, *Raw*, *Promo*. |
| Level of Description | **Yes** | file |
| Extent | **Yes** | Aggregate extent for the group (Portion = whole, Number = total tape count). |
| Publish? | **Yes** | Always checked. |
| Scope and Content | Optional | Use only when the file group requires clarification beyond the title, or to cross-reference related material elsewhere in the finding aid. |

## 3.D. Item-Level Description

Item-level records represent individual physical objects — a single videotape or audiocassette. This is where the detailed descriptive and physical work occurs.

### Required Item-Level Elements

| Element | Required? | Notes |
|---------|-----------|-------|
| Title | **Yes** | See Chapter 4. |
| Level of Description | **Yes** | item |
| Component Unique Identifier | **Yes** | The JPC_AV_##### identifier assigned to the tape. If a tape has not been assigned a JPC_AV_##### identifier, stop and contact McDowell, Blake before proceeding. |
| Date | **Yes** (if known) | Label: creation, Edited, or broadcast. Type: Single. Enter in the Begin field in YYYY-MM-DD format. See 3.D.i. |
| Extent | **Yes** | Portion: whole. Number: 1 (or higher only for retained exact duplicate copies). Type: from controlled vocabulary (see Appendix A). Physical Details: SD video, color, sound, or the applicable combination. |
| Duration | **Yes** | hh:mm:ss. Added programmatically to the Physical Characteristics and Technical Requirements note as a Defined List subnote by `aspace-rename-directories.py`. Not entered manually. |
| Instance | **Yes** | Instance type: Moving Images (Video) or Audio. Top container type: AV Case. Top container indicator: JPC_AV_#####. See Chapter 5. |
| Publish? | **Yes** | Always checked. |

### Optional Item-Level Elements

| Element | Required? | Notes |
|---------|-----------|-------|
| Scope and Content note | Optional | A brief description of the tape's intellectual content. Use when meaningful content information is available. See 3.D.iii. |
| Physical Characteristics and Technical Requirements note | Optional | Use when there is something to report about transfer quality, playback issues, or tape condition. See 3.D.iv. |
| Conditions Governing Access | Optional | Only when an individual item has a specific access condition. |
| Conditions Governing Reproduction and Use | Optional | |
| Existence and Location of Copies | Optional | |

---

### 3.D.i Date

Use the following labels for AV item-level dates:

| Label | Usage |
|-------|-------|
| creation | The date on which the recording was originally made. |
| Edited | The date on which an edited version was produced. |
| broadcast | The date on which the recording was broadcast. |

Date type will usually be **Single**. Enter the date in the Begin field in YYYY-MM-DD format. The Date Expression field should contain the same date, or a human-readable approximation if the exact date is uncertain.

If a date cannot be determined from the tape label, video slate, or context, use `undated` as the Date Expression and leave the Begin/End fields blank. If a date range can be approximated from context, provide the approximation in the Date Expression (e.g., *approximately 1986–1987*) and populate Begin/End with the broadest applicable range.

---

### 3.D.ii Extent

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

> **N.B.** The Physical Details field is updated programmatically by `aspace-rename-directories.py`, which sets it to `SD video, color, sound` on all extents. For tapes that deviate from this standard (BW, silent, HD), update the Physical Details field manually after the automated update.

**Multiple extents:** If a single item contains multiple formats or you need to note copy status separately for different pieces of media, use multiple extents, each with Portion = Part. Common cases for JPC:

- An interview that exists as both the original 1-inch reel and a VHS reference dub
- A finished master and a safety copy on different formats

In the Physical Details field, note copy status where relevant:

> *Original*  
> *Duplicate*  
> *Reference copy*  
> *VHS dub; location of 1-inch original unknown*

---

### 3.D.iii Scope and Content Note

The Scope and Content note at the item level provides a brief description of the tape's intellectual content. It is created as a multipart note with one or more text subnotes.

Sources for content description include:

- Label or writing on the tape case or tape itself
- Text in the video slate or leader
- Episode inventories and production documentation
- The `DESCRIPTION` field in the import CSV, if the record was created through the automated import workflow

Keep descriptions brief. A single sentence is acceptable. It is also acceptable to leave this note out entirely if no content information is available — do not fabricate or speculate about content.

> **⟶ AAA CH. 5 SUGGESTION:** The AAA guidelines identify additional specific triggers for a Scope and Content note. Consider adopting any of the following that are useful for JPCA:
>
> - **Transcripts or paper records found with the tape:** *Includes transcript.* / *Paper note found in original tape case.*
> - **Content that does not match the label:** Note both what the label says and what is actually on the tape.
> - **Blank or unrelated content** at the start of a recording: *First 18 minutes is color bars. Program content begins at 00:18:00.*
> - **Shared physical tapes:** When one tape contains parts of two separately described items, add a cross-reference in the Scope and Content note of each. Example: *Interview concludes at 00:34:12 on tape 2; remainder of tape 2 contains the opening of the Stevie Wonder segment (see EJS episode 1001, Stevie Wonder interview, tape 1 of 3).*
> - **Unclear tape sequence:** *Sequence of original recordings appears to be: tape marked "PM" first, tape marked "Eve 1" second.*

---

### 3.D.iv Physical Characteristics and Technical Requirements Note

The Physical Characteristics and Technical Requirements (phystech) note captures playback and technical observations about the tape and its recording. It is created as a multipart note with a text subnote. Use this note only when there is something to report — it is not required if there are no observations to record.

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
- Item value: hh:mm:ss (extracted from the .mkv file via mediainfo)

> **N.B.** Duration is not entered manually. It is added programmatically from the digitized file and should not be edited by hand.

---

# Chapter 4. Title

## 4.A. General Title Rules

Use the official title if one exists. If no official title exists, construct one by combining relevant elements such as the series name, program type, episode number, subject matter, creator name, or qualifying descriptors.

| Rule | Detail |
|------|--------|
| No format terms | Never use a specific format term (VHS, U-matic, Betacam) as a title or part of a title. Titles should convey intellectual content, not physical format. |
| No ambiguous labels | Do not transcribe ambiguous media labels directly as unit titles. If you do not understand what a label means, researchers likely will not either. Consult the NMAAHC Media Archivist for unlabeled or poorly labeled media. |
| Qualifiers only when needed | Do not force a qualifier if one is not needed. Qualifiers should reflect markings on the tape itself or text visible in the video, not content derived by the archivist from watching the recording. |
| Qualifier consistency | Qualifiers must be consistent across tapes. We use **Safety Master** as the standard form — not *master (safety)*, *safety copy*, or *safety*. |
| No "copy" | Avoid the word *copy* in qualifiers. Use *master*, *safety*, or *dub*; not *master copy* or *safety copy*. |
| Spelling | Retain original or colloquial spelling found on the tape unless it is an obvious typographic error that would cause confusion. Correct *mastre* to *master*, but do not change *trax* to *tracks*. |

### Unidentified or Poorly Labeled Tapes

When a tape is unlabeled or its label cannot be interpreted, do not use the label as the unit title and do not use a specific format term as the title. If the type of content can be inferred from context (e.g., it is in a box with other EJS material), use that context to construct a title. If content truly cannot be determined, use the word "unidentified" with the general material designation:

> *Unidentified Video Recording*  
> *Unidentified Sound Recording*

## 4.B. Ebony/Jet Showcase Title Pattern

For all EJS and Ebony/Jet Celebrity Showcase material, use the following pattern:

```
<Series Name>, <episode number>[, <qualifier if needed>]
```

Examples:

> *Ebony/Jet Showcase, episode 1001*  
> *Ebony/Jet Showcase, episode 1004, safety master*  
> *Ebony/Jet Showcase, episode 1005, studio footage*  
> *Ebony/Jet Celebrity Showcase, episode 09, Grace Jones makeup and fashion sequences*

Episode numbering conventions differ between the two programs:

| Program | Episode Number Format |
|---------|-----------------------|
| Ebony/Jet Showcase (1984–1993) | 4-digit numbers spanning all seasons: 1001, 7024, 9008, etc. |
| Ebony/Jet Celebrity Showcase (1982–83) | 2-digit numbers: 07, 13, 22, etc. |

> **N.B.** We follow normalized titling wherever possible. For episodes that have a specific broadcast title (e.g., *The San Francisco Show*), the normalized pattern takes precedence — use the episode number format and omit the broadcast title from the ArchivesSpace title field.

## 4.C. Raw Footage and Interview Tape Titles

For raw footage tapes, include enough identifying information in the title to distinguish the tape within its group. Where multiple tapes cover the same interview or shoot, use a part indicator:

> *Ebony/Jet Celebrity Showcase, episode 11, James Brown interview, tape 1 of 3, 1985-08-03*  
> *Ebony/Jet Celebrity Showcase, episode 11, James Brown interview, tape 2 of 3, 1985-08-03*  
> *Ebony/Jet Celebrity Showcase, episode 11, James Brown interview, tape 3 of 3, 1985-08-03*

## 4.D. Non-EJS Title Patterns

For sub-series other than EJS, use the same general principles: formal title if available, otherwise a constructed title using content, creator, date, or genre. Apply qualifiers as needed but avoid format terms and the word *copy*.

> *American Black Achievement Awards, 1982 broadcast*  
> *[Subject name] interview, [publication context if known], [date]*  
> *Ebony Fashion Fair, 1979, promotional footage*

## 4.E. Series and Sub-Series Titles

Use one of the following for a series or sub-series title, in order of preference:

1. The formal title of the series, if one exists, along with a form or genre term if needed for clarity (e.g., *Ebony/Jet Showcase, TV Program*).
2. The form or genre of the content, if the series contains one or two types (e.g., *Television Programs*, *Radio Broadcasts*).
3. A general material designation when the series contains varied content (e.g., *Video Recordings*, *Sound Recordings*). Avoid the term *Audiovisual Material* as a series title.

---

# Chapter 5. Instances and Containers

## 5.A. Overview

Instances in ArchivesSpace link an archival object to its physical container. Every item-level AV record must have exactly one instance.

## 5.B. Creating an Instance

1. In the Instances section, click **Add Container Instance**.
2. For **Type**, select `Moving Images (Video)` for video recordings, or the applicable type for audio.
3. In the **Top Container** field, search for the existing top container using the JPC_AV_##### identifier. If the top container does not yet exist, create it (see 5.C).

## 5.C. Top Containers for AV Material

The top container represents the outermost physical housing of the tape — in most cases, the original tape case.

| Field | Value |
|-------|-------|
| Container Type | AV Case |
| Indicator | The JPC_AV_##### identifier assigned to the tape (e.g., JPC_AV_01548). Serves as the primary container identifier in the absence of barcodes. |
| Barcode | Not currently in use for AV material. Leave blank. |

> **N.B.** Top containers for AV material are not shared between multiple archival objects. Each tape has its own AV Case top container, identified by its unique JPC_AV_##### indicator. The JPC_AV_##### identifier serves dual purposes: it appears as both the Component Unique Identifier on the archival object and as the indicator on its top container.

## 5.D. Instance Type Controlled Vocabulary

| Media Type | Instance Type Value |
|------------|---------------------|
| Video recordings | Moving Images (Video) |
| Audio recordings | Audio |

---

# Chapter 6. Controlled Vocabulary

## 6.A. Extent Type (Original Format)

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

## 6.B. Date Labels

| Label | When to Use |
|-------|-------------|
| creation | The date on which the recording was originally made or recorded. |
| Edited | The date on which an edited version was produced. |
| broadcast | The date on which the recording was broadcast or publicly transmitted. |

## 6.C. Physical Details for Extent

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

## 6.D. Note Types and Structures

| Note Type | Level | Purpose and Structure |
|-----------|-------|-----------------------|
| Scope and Content (`scopecontent`) | Item | Brief description of intellectual content. Text subnote. |
| Physical Characteristics and Technical Requirements (`phystech`) | Item | Technical and physical observations about the tape and its recording. Text subnote. After digitization, also contains a Defined List subnote with label "Duration" and value in hh:mm:ss. |
| Scope and Content — Production Crew (`scopecontent`, label: "Production Crew") | Episode sub-series | Labeled note containing a Defined List of production credits. Role and name for each crew member. Entered manually in ArchivesSpace from tape labels, slates, or production documentation. |

---

# Chapter 7. Agents and Subjects

## 7.A. Agents

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

## 7.B. Subjects

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
| Title | **Yes** | See Chapter 4 |
| Component Unique Identifier | **Yes** | JPC_AV_##### |
| Date Label | **Yes** (if known) | creation \| Edited \| broadcast |
| Date Type | **Yes** | Single |
| Date Expression | **Yes** | YYYY-MM-DD or approximation |
| Extent Portion | **Yes** | whole |
| Extent Number | **Yes** | 1 (or count of identical retained copies) |
| Extent Type | **Yes** | From controlled vocabulary; must match ArchivesSpace dropdown exactly |
| Extent Physical Details | **Yes** | SD video, color, sound (or applicable combination) |
| Duration | **Yes** | Added programmatically to Physical Characteristics and Technical Requirements note as a Defined List subnote by `aspace-rename-directories.py`. Not entered manually. |
| Instance Type | **Yes** | Moving Images (Video) or Audio |
| Top Container Type | **Yes** | AV Case |
| Top Container Indicator | **Yes** | JPC_AV_##### |
| Publish? | **Yes** | Always checked |

## A.3 Optional Item-Level Fields: Quick Reference

| Element | Required? | Value / Notes |
|---------|-----------|---------------|
| Scope and Content note | Optional | Multipart note, text subnote. Brief intellectual content description. |
| Physical Characteristics & Technical Requirements note | Optional | Multipart note, text subnote. Use when there is something to report about transfer quality, playback issues, or tape condition. Defined list subnote added programmatically for Duration. |
| Conditions Governing Access | Optional | Only when item has a specific access condition |
| Conditions Governing Reproduction and Use | Optional | |
| Existence and Location of Copies | Optional | |
| Agents | Optional | Subject, Creator. See Chapter 7. |
| Subjects | Optional | See Chapter 7. |

---

# Appendix B. Open Questions and Items Under Discussion

The following items are pending final decisions. Consult McDowell, Blake for current status before proceeding on any of these points.

| Item | Status |
|------|--------|
| New CSV column needed — `ASpace Scope and Contents Note` or `ASpace Description` (name TBD) — to serve as the ArchivesSpace import source for the scope note, parallel to `ASpace PhysTech Note`. Will be staff-assembled, informed by but not identical to the `DESCRIPTION` MKV tag. `DESCRIPTION` remains as the MKV embedding source. | Pending |
| Conditions Governing Access note language specific to JPCA AV | Language TBD |
| Exact language for the Processing Note documenting digitization actions | TBD |
| Agent inheritance for raw footage tapes: how to prevent inappropriate propagation of episode-level agents to individual item records | Manual review workflow in progress |
| Journalist Field Recordings sub-series: whether to subdivide and on what basis as content is better understood | TBD |
| Rights statement — language, applicability, and whether required at item level for JPCA AV | TBD |

---

*JPC Archive | Audiovisual Materials: Arrangement and Description in ArchivesSpace | Version 2026-03-30 | NMAAHC / GRI*