# Firefight Modpack Format

Version 0.1.0

## 1. Introduction

### 1.1 Purpose

This document defines a package format for distributing Firefight mods: one archive that carries a
game-ready payload, machine-readable metadata, and optional human documentation.

### 1.2 Scope

The format covers what a mod package contains and how a consumer reads it. It does not define the
content formats of the game itself; a payload consists of files the game already understands.

The format is versioned independently of any game release. A package declares the game release it
targets in its metadata, so one format serves every release.

### 1.3 Audience

Producers write packages: the mod editor, build scripts, and authors who assemble archives by hand.
Consumers read packages: the in-game mod manager and any tool that inspects or installs mods.

## 2. Conformance

### 2.1 Requirement keywords

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY and
OPTIONAL are to be interpreted as described in RFC 2119.

### 2.2 Conforming package

A package is conforming when it satisfies every MUST and MUST NOT in sections 3 to 8.

### 2.3 Conforming consumer

A consumer is conforming when it satisfies every MUST and MUST NOT in section 9, and classifies every
condition listed in section 10 with the result stated there.

Producers and consumers are independent; a package can be conforming while no conforming consumer
exists.

## 3. Terminology

| Term | Definition |
| --- | --- |
| Package | One archive that carries one mod. |
| Payload | The file tree a consumer installs. |
| Payload root | The directory inside the archive that contains the payload. |
| Consumer | Software that reads packages. |
| Producer | Software or person that writes packages. |
| Legacy package | A package without a metadata file, as produced before this document. |
| Format version | The integer `spec_version` declared inside a package. |
| Document version | The semantic version of this document, recorded in `CHANGELOG.md`. |

## 4. Package

### 4.1 Archive

A package is a ZIP archive.

- Producers SHOULD name the file with the `.ffmod` extension.
- Consumers MUST accept both `.ffmod` and `.zip` regardless of the extension present.
- Compression MAY be deflate or store.

### 4.2 Entry names

Entry names inside the archive are relative paths.

- Producers MUST separate path segments with the forward slash `/`.
- A consumer MUST treat the backslash `\` as an equivalent separator, because archives written on
  some platforms contain it.
- Entry names MUST NOT be absolute, MUST NOT contain a `..` segment, and MUST NOT contain a drive
  letter.
- A single path segment MUST NOT exceed 255 bytes.
- A consumer MUST reject the archive when an entry violates these rules, rather than skipping that
  entry.

### 4.3 Top-level entries

| Path | Requirement | Role |
| --- | --- | --- |
| `Mod/` | REQUIRED | Payload; see section 5. |
| `Mod_Information.toml` | REQUIRED | Metadata; see section 6. |
| `README.md` | RECOMMENDED | Readme for a person who unpacks the archive; see section 8. |
| `LICENSE` | RECOMMENDED; REQUIRED when the package redistributes third-party assets | Licence text; see section 8. |
| `DOCS/` | OPTIONAL | Documentation pages; see section 7. |

A producer MUST NOT place a second payload tree at the archive root. A consumer MUST ignore any
top-level entry outside this table and MUST NOT treat its presence as an error.

### 4.4 Example

```
<name>.ffmod
├── Mod/
│   ├── Data/
│   ├── Images/
│   ├── Sounds/
│   └── mod.txt
├── README.md
├── Mod_Information.toml
├── LICENSE
└── DOCS/
    ├── About.md
    └── About_zh.md
```

## 5. Payload

The payload root is the entry `Mod/` at the archive root. A consumer installs the payload so that
its content becomes the content of the game's mod directory.

- `Mod/mod.txt` MUST exist and MUST be a regular file.
- `Mod/mod.txt` keeps the format the game reads. This document does not redefine it. A consumer MAY
  read its `<name>` tag for display and MUST fall back to metadata when the tag is absent.
- `Mod/Data/`, `Mod/Images/` and `Mod/Sounds/` MAY be present in any combination.
- Paths inside the payload are relative to the payload root.
- The payload MUST NOT contain a nested `Mod/` directory.

A payload carries files in the game's own formats. Working files of a producer, for example
configuration sources that are compiled into a payload, do not belong in the payload.

## 6. Metadata

### 6.1 Location and encoding

The metadata file is TOML v1.0 encoded as UTF-8 without a byte order mark. It lives at the archive
root, beside the payload.

### 6.2 File name

- A producer MUST name the file `Mod_Information.toml`.
- A consumer MUST also accept `ModInformation.toml`, written without the underscore, and MUST prefer
  the underscored name when both are present.

### 6.3 Fields

| Field | Type | Requirement | Meaning |
| --- | --- | --- | --- |
| `spec_version` | integer | REQUIRED | Format version the package conforms to. This document defines version `1`. |
| `name` | string | REQUIRED | Display name, 1 to 64 characters. |
| `version_name` | string | REQUIRED | Mod version as `X.Y.Z`, decimal digits, no leading zeros. |
| `version_code` | integer | REQUIRED | Non-negative integer that orders releases of one mod. |
| `game_version` | string | REQUIRED | Game release the payload targets, for example `"13.2.0"`. |
| `authors` | array of strings | REQUIRED | One or more author names. |
| `id` | string | OPTIONAL | Stable identifier matching `[a-z0-9._-]{1,64}`. |
| `update_code` | integer | OPTIONAL | Producer-defined integer that signals an update. Absent means `0`. |
| `license` | string | OPTIONAL | SPDX licence identifier, for example `"MIT"`. |
| `description` | string | OPTIONAL | One line of plain text, at most 200 characters. |
| `homepage` | string | OPTIONAL | Absolute `https` URL. |
| `docs` | array of tables | OPTIONAL | Index of documentation pages; see section 7.3. |

A consumer MUST ignore fields it does not define. A consumer MUST NOT reject a package because an
OPTIONAL field is absent.

When `id` is absent, a consumer identifies the mod by `name` together with the archive file name.

### 6.4 Example

```toml
spec_version = 1
id = "example-overhaul"
name = "Example Overhaul"
version_name = "1.0.0"
version_code = 1
update_code = 0
game_version = "13.2.0"
authors = ["Example Author"]
license = "MIT"
description = "Replaces unit equipment tables."
homepage = "https://example.invalid/overhaul"

[[docs]]
file = "DOCS/About.md"
lang = "en"
title = "About"

[[docs]]
file = "DOCS/About_zh.md"
lang = "zh"
title = "关于"
```

## 7. Documentation directory

`DOCS/` holds Markdown pages for readers. Pages are UTF-8 encoded and use the `.md` extension.

### 7.1 File names

| Pattern | Role |
| --- | --- |
| `<stem>.md` | Page without a language tag. Treated as the default page. |
| `<stem>_<lang>.md` | Page localised to `<lang>`. |

`<lang>` is a lower case ISO 639-1 code, optionally followed by `-` and an upper case ISO 3166-1
alpha-2 region code, for example `zh` or `zh-CN`. `<stem>` matches `[A-Za-z0-9_-]{1,64}`.

### 7.2 Resolution order

A consumer that needs a page in a requested language MUST try these names in order and MUST use the
first that exists:

1. `<stem>_<lang>-<REGION>.md`
2. `<stem>_<lang>.md`
3. `<stem>.md`

If none exists, the page is absent. A consumer MUST NOT fall back to a page with a different stem.

### 7.3 Index

The `docs` array lists the pages a consumer presents first. Each entry accepts:

| Field | Type | Requirement | Meaning |
| --- | --- | --- | --- |
| `file` | string | REQUIRED | Path relative to the archive root, for example `DOCS/About.md`. |
| `lang` | string | RECOMMENDED | Language tag of the page. |
| `title` | string | OPTIONAL | Display title. Absent means the file stem. |

A page present in `DOCS/` but absent from the index remains valid and readable.

## 8. Readme and licence

`README.md` is Markdown written for a person who unpacks the archive outside the game. It SHOULD
state what the mod changes, which game release it targets, and how to install it. It MUST NOT be the
only place where information a consumer needs is recorded; metadata belongs in the metadata file.

`LICENSE` carries the licence text that applies to the package content. When the `license` field is
present, its SPDX identifier MUST agree with the text in `LICENSE`.

## 9. Import procedure

A conforming consumer performs these steps in order.

1. Open the archive. Reject it when an entry name violates section 4.2.
2. Read `Mod_Information.toml`, or `ModInformation.toml` as described in section 6.2.
3. If neither exists, treat the package as legacy: locate the shallowest directory that directly
   contains `mod.txt` and use it as the payload root.
4. Determine the payload root: the entry `Mod/` when present; otherwise the shallowest directory
   that directly contains `mod.txt`. If two directories at the same depth both contain `mod.txt`,
   stop and report an ambiguous package.
5. Validate the metadata as described in section 10. Reject the package on an error; import it and
   record the condition on a warning.
6. Copy the payload only. A consumer MUST NOT copy `Mod_Information.toml`, `README.md`, `LICENSE` or
   `DOCS/` into the game's mod directory.
7. Retain the metadata and the documentation in the consumer's own library, keyed by `id` when
   present and by the archive file name otherwise. Silently discarding them is not permitted; a
   consumer that cannot store them MUST report that.
8. Record the installed `version_name` and `version_code` so that a later package can be compared
   against them.

A consumer MUST NOT modify the payload during import. A consumer that can disable an installed mod
MUST keep the payload unchanged and represent the disabled state outside it.

## 10. Validation

| Condition | Result |
| --- | --- |
| Archive is not a readable ZIP | error |
| Entry name violates section 4.2 | error |
| `Mod/mod.txt` missing | error |
| Metadata file missing | warning; import as a legacy package |
| `spec_version` missing | warning; treated as `1` |
| `spec_version` greater than the consumer's supported version | warning; import continues |
| A REQUIRED field is missing or has the wrong type | error |
| `version_name` does not match `X.Y.Z` | error |
| `version_code` negative | error |
| `license` present but `LICENSE` missing | warning |
| `docs[].file` names an entry that is absent | warning; the entry is ignored |
| Nested `Mod/Mod/` | error |
| Two directories at the same depth contain `mod.txt` | error |

An error rejects the package. A warning imports it and records the condition for display.

## 11. Versioning

`spec_version` identifies the format. This document is versioned separately, following semantic
versioning, and every version is recorded in `CHANGELOG.md`.

- Adding an OPTIONAL field, or changing wording without changing behaviour, keeps the format version.
- Adding a REQUIRED field, removing a field, or changing the meaning of a field increments the format
  version.
- A consumer declares the highest format version it supports. A package that declares a higher value
  is imported with a warning, as stated in section 10.

Packages written before this document remain importable through the legacy path of section 9 and the
`mod.txt` name fallback of section 5.

## 12. Out of scope

This document does not define:

- the internal format of `mod.txt`, which belongs to the game;
- map data, including `CustomMapCodeNames.txt`;
- producer project directories and their working files;
- asset trimming or other redistribution optimisation;
- distribution channels, signatures and update feeds.
