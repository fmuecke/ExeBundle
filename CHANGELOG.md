# Changelog

All notable changes to ExeBundle will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0/0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.94] - 2026-01-18

### Breaking Changes

- **CLI option renaming for better Windows compatibility**:
  - `--` (double dash) renamed to `--files` for specifying manual file lists
  - `--files` renamed to `--filelist` for reading file list from text file
  - These changes improve compatibility with Windows command-line conventions where `--` token can be problematic

**Migration guide**:

```bash
# Before (0.92): Manual file list with --
ExeBundle.exe build --exe app.exe --out bundle.exe -- lib1.dll lib2.dll data.txt

# After (0.93): Use --files
ExeBundle.exe --exe app.exe --out bundle.exe --files lib1.dll lib2.dll data.txt
```

```bash
# Before (0.92): File list from text file
ExeBundle.exe build --exe app.exe --out bundle.exe --files filelist.txt

# After (0.93): Use --filelist
ExeBundle.exe --exe app.exe --out bundle.exe --filelist filelist.txt
```

### Added

- **Script bundling support**: Bundle and execute PowerShell, batch, and other script files directly
  - Non-PE files (scripts, .NET assemblies) can now be specified as `--exe` parameter
  - Use custom command line templates to specify interpreter (e.g., `--cmdline "powershell -File {exe}"`)
  - Enables portable script distribution without separate executable wrapper
  - Example: `ExeBundle.exe build -e script.ps1 -o bundle.exe --cmdline "powershell -ExecutionPolicy Bypass -File {exe}" -- script.ps1 helpers.psm1`

- **Automatic path quoting**: Template variables `{exe}` and `{bundledir}` are now automatically quoted
  - Handles paths with spaces correctly without manual quoting
  - `{exe}` expands to `"C:\Program Files\App\app.exe"` (quoted)
  - Prevents command-line parsing issues

- **Enhanced non-PE file support**: Builder now gracefully handles non-executable files
  - Warning displayed when non-.exe file is specified
  - Resource extraction skipped for non-PE files
  - Default 32-bit console loader used for scripts

### Changed

- **CLI: `build` command now optional (default)**: The `build` command keyword can be omitted for shorter syntax
  - `ExeBundle.exe --exe app.exe --out bundle.exe --auto` (new, shorter)
  - `ExeBundle.exe build --exe app.exe --out bundle.exe --auto` (still works)
  - Running with no arguments now shows help instead of error
  - Fully backward compatible - explicit `build` still works

- **Command line execution model**: `CreateProcessA` now parses full command line instead of fixed executable
  - Enables arbitrary process execution (e.g., `cmd.exe`, `powershell.exe`)
  - Template is now the complete command, not just arguments
  - More flexible than previous implementation

- **Increased template limits**: Command line and working directory templates expanded to 4096 characters (from 255/127)
  - Supports complex command lines and long paths
  - Header serialization updated to use `uint16_t` length prefix (max 65535, limited to 4096 for safety)

- **Compression system overhaul**: Simplified and unified compression using Zstandard
  - All compression modes now use Zstandard (zstd) instead of Windows internal algorithms
  - Windows Compression API (xpress, xpress_huff, lzms, mszip) disabled due to poor performance
  - Compression levels: `none`, `fast`, `balanced` (default), `ultra`
  - Builder now displays compressed size after bundling

### Fixed

- **Version info compatibility**: Fixed bundling of executables without version information resources
  - Builder now adds default version information to loader when source exe has none
  - Prevents bundle creation failures with certain executables

- **Compression level handling**: Fixed handling of edge-case compression levels
  - Improved validation and error handling for compression modes
  - More robust compression configuration

### Performance

- **Loader size reduced by ~60KB**: Extensive optimizations to minimize bundle overhead
  - Replaced STL filesystem and chrono with lightweight Windows API wrappers
  - Disabled unnecessary runtime checks and features
  - Smaller bundles, faster downloads

- **Builder memory optimization**: Reduced memory footprint during bundle creation
  - Eliminated intermediate storage vectors in build pipeline
  - Early deallocation of file entry data after overlay creation
  - In-place file entry processing instead of creating new vectors
  - Significant memory savings for large bundles

---

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
