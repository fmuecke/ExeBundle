# Changelog

All notable changes to ExeBundle will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.92] - 2026-01-01

### Breaking Changes

- **Bundle format change**: The internal header structure has been extended from ~136 bytes to ~520 bytes to support new runtime execution features. **Bundles created with v0.91 or earlier will not work with v0.92 loaders, and vice versa.** All bundles must be rebuilt with v0.92.

- **Default command line behavior change**: The bundled application now receives the executable path as `argv[0]` (proper Windows convention), whereas previously `argv[0]` was missing or incorrect. This fixes a long-standing bug but may affect applications that relied on the incorrect behavior.

### Added

- **Custom command line templates** (`--cmdline <template>`): Specify a custom command line for the bundled application with variable substitution support
  - Supports `{exe}` variable (full path to extracted executable)
  - Supports `{bundledir}` variable (extraction directory path)
  - When specified, runtime arguments are completely ignored (replace mode)
  - Example: `--cmdline "{exe} --preset=production --config {bundledir}\config.ini"`

- **Custom working directory** (`--workdir <template>`): Specify a custom working directory for the bundled application
  - Supports same `{exe}` and `{bundledir}` variable substitution
  - Default behavior (no template): working directory is the extraction folder
  - Example: `--workdir "{bundledir}"`

- **Diagnostic output for templates**: `/exebundle:diag` now displays command line and working directory templates

### Fixed

- **argv[0] now contains executable path**: The default behavior now properly passes the bundled executable's full path as `argv[0]`, fixing compatibility with applications that rely on this standard Windows convention

### Technical Details

- Header struct size: 136 → 520 bytes (adds 256-byte CmdLineTemplate and 128-byte WorkDirTemplate fields)
- Variable substitution performed at runtime via simple string replacement
- Template validation enforces length limits (255 chars for cmdline, 127 for workdir)

### Examples

```bash
# Custom command line with configuration path
ExeBundle.exe build --exe MyApp.exe --out Bundle.exe --auto \
  --cmdline "{exe} --config {bundledir}\settings.ini"

# Custom working directory
ExeBundle.exe build --exe MyApp.exe --out Bundle.exe --auto \
  --workdir "{bundledir}"

# Both options combined
ExeBundle.exe build --exe MyApp.exe --out Bundle.exe --auto \
  --cmdline "{exe} --verbose" \
  --workdir "{bundledir}"
```

---

## [0.91] - 2025-12-31

### Breaking Changes

- **`--exe` is now required in all cases**: The `--exe` option must now be explicitly specified for all file selection modes (`--auto`, `--files`, and manual `--`). Previously, when using `--files`, the exe could be implicitly specified as the first line in the file list. This implicit behavior has been removed for clarity and consistency.

### Changed

- Simplified `--files` mode: File lists now contain only additional files to bundle (not the exe itself)
- CLI help text now marks `--exe` as `(required)` to reflect this requirement
- Improved error messages when `--exe` is missing

### Migration Guide

**Before (0.90)**:
```bash
# This worked - exe in file list
ExeBundle.exe build --files filelist.txt --out bundle.exe
```

**After (0.91)**:
```bash
# Now required - explicit --exe
ExeBundle.exe build --exe MyApp.exe --files filelist.txt --out bundle.exe
```

**File list format change**:
```diff
  Before (0.90):          After (0.91):
- MyApp.exe              (removed - use --exe instead)
  lib1.dll                lib1.dll
  lib2.dll                lib2.dll
  assets\icon.png         assets\icon.png
```

### Rationale

This change improves:
- **Clarity**: Single source of truth for the main executable
- **Consistency**: All modes now work the same way
- **Scriptability**: No ambiguity about which exe is being bundled
- **Maintainability**: Simpler code with less conditional logic

---

## [0.90] - 2025-01-02

### Changed
- Updated public README to better align with keyword topics
- Tweaked loader output
- Renamed "side-by-side" strategy to "subdir"
- Using canonical description as slogan

---

*For releases prior to 0.90, see git commit history.*
