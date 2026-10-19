# Firefight Modpack Format

Defines the package format for Firefight mods: one archive holding a game-ready payload under
`Mod/`, the metadata file `Mod_Information.toml`, and optional documentation under `DOCS/`.

Version 0.1.0. Format version `spec_version = 1`.

## Documents

| Path | Content |
| --- | --- |
| `docs/spec.md` | Normative specification, English. |
| `docs/spec.zh.md` | Normative specification, Chinese. |
| `docs/flow.md` | Repository structure, publishing, pending work. |
| `CHANGELOG.md` | Document versions and their entries. |
| `LICENSE` | Licence text, CC BY-SA 4.0. |

## Package layout

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

## Status

The format is defined; the consumers are not yet conforming.

- The mod manager imports packages through the legacy path and discards archive-root metadata.
- The mod editor writes `ModInformation.toml` without the underscore, which consumers must accept
  while producers must write the underscored name.

The gap list is maintained in `docs/flow.md`.

## Two version numbers

- `spec_version`, an integer in `Mod_Information.toml`, identifies the format. Adding an optional
  field keeps it; a new required field or a change of meaning increments it.
- The document version follows semantic versioning and is recorded in `CHANGELOG.md`. A tag `vX.Y.Z`
  marks a published document version.

## Related repositories

- `BiancaoAwa/Firefight-Mod-Manager-Android` — the in-game Android mod manager.
- `BiancaoAwa/firefight-mod-editor-open` — the desktop mod editor.

## Licence

The specification text is licensed under Creative Commons Attribution-ShareAlike 4.0 International.
See `LICENSE`.
