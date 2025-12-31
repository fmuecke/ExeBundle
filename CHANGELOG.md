# Changelog

All notable changes to ExeBundle will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.91] - 2025-01-03

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
