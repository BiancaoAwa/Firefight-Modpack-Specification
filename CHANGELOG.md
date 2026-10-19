# Changelog

## [Unreleased]

## [0.1.0] - 2026-10-20

### Added

- Package layout with the game-native payload under `Mod/`, the metadata file
  `Mod_Information.toml`, `README.md`, `LICENSE` and `DOCS/`.
- Field table for `Mod_Information.toml`, including the format version `spec_version = 1`.
- Rules for localised documentation file names and their resolution order.
- Import procedure for consumers, including the legacy path for packages without metadata.
- Validation table that separates errors from warnings.
- Compatibility rules for future changes to the format.
- Chinese translation of the specification at `docs/spec.zh.md`.

### Changed

- Normative documents are placed under `docs/`; the repository root holds `README.md`,
  `CHANGELOG.md`, `LICENSE` and `.gitattributes`.
