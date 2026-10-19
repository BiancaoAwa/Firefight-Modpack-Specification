# Repository structure, publishing and pending work

## Files

```mermaid
flowchart TD
    subgraph files["Repository files"]
        README["README.md — overview, document index, versioning rules"]
        SPEC["docs/spec.md — normative specification, English"]
        SPECZH["docs/spec.zh.md — normative specification, Chinese"]
        FLOW["docs/flow.md — this diagram"]
        CHANGELOG["CHANGELOG.md — document versions and entries"]
        LICENSE["LICENSE — CC BY-SA 4.0 text"]
        ATTR[".gitattributes — LF endings for every text file"]
    end

    subgraph artifacts["Build artifacts"]
        NOBUILD["No build step and no generated artifacts"]
        PUBLISH["Publishing: merge a pull request into main, then tag vX.Y.Z on the merge commit"]
        NOBUILD --- PUBLISH
    end

    subgraph pending["Pending"]
        P1["Manager: installZip discards archive-root metadata, so section 9 step 7 of the specification is not implemented"]
        P2["Editor: writes ModInformation.toml, while producers must write Mod_Information.toml per section 6.2"]
        P3["Manager: no Markdown renderer, so pages under DOCS cannot be displayed inside the manager"]
        P4["No package validator exists, so the table in section 10 is not enforced by tooling"]
        P5["No sample package is kept in this repository"]
    end

    SPEC --> CHANGELOG
    SPEC --> FLOW
    CHANGELOG --> PUBLISH
    SPEC --> P1
    SPEC --> P2
    SPEC --> P3
    SPEC --> P4
    CHANGELOG --> P5
```

## Build artifacts

No build step exists. Every file is written and committed as is; nothing is generated, so nothing
has to be kept out of version control beyond editor backups.

Publishing a document version: merge a pull request into `main`, then create a tag `vX.Y.Z` on the
merge commit with the matching `CHANGELOG.md` entry.

## Pending work

| Item | Scope | Reference |
| --- | --- | --- |
| Retain archive-root metadata on import | Mod manager | `docs/spec.md` section 9, step 7 |
| Write `Mod_Information.toml` with the underscored name | Mod editor | `docs/spec.md` section 6.2 |
| Render pages under `DOCS` | Mod manager | `docs/spec.md` section 7 |
| Validate packages against the table | Tooling | `docs/spec.md` section 10 |
| Ship a sample package | Both consumers | `docs/spec.md` section 4.4 |
