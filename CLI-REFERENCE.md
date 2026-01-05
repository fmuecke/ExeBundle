# ExeBundle - Command-Line Reference

**Version**: See `ExeBundle.exe license` for current version
**License**: Free for non-commercial use. See LICENSE.txt for details.


## Quick Reference

```bash
# Build with auto-discovery (recommended for most cases)
ExeBundle.exe build --exe MyApp.exe --out MyApp-Bundled.exe --auto

# Build from file list
ExeBundle.exe build --exe MyApp.exe --files filelist.txt --out MyApp-Bundled.exe

# Build with manual file selection
ExeBundle.exe build --exe MyApp.exe --out MyApp-Bundled.exe -- lib1.dll lib2.dll

# Build with custom command line and working directory
ExeBundle.exe build --exe MyApp.exe --out MyApp-Bundled.exe --auto \
  --cmdline "{exe} --config {bundledir}\app.cfg" \
  --workdir "{bundledir}"

# Show license information
ExeBundle.exe license
```


## Commands

### `build`
Create a bundled executable from an application and its dependencies.

**Syntax**:
```bash
ExeBundle.exe build [options]
```

### `license`
Display ExeBundle license terms and third-party library licenses (LZ4, Zstandard).

**Syntax**:
```bash
ExeBundle.exe license
```


## Build Options

### Required Options

#### `-o, --out <path>`
**Output bundled executable path** (required)

Specifies the path where the bundled executable will be created.

**Example**:
```bash
--out MyApp-Portable.exe
```


### Input Selection

You must choose **exactly one** of these three file selection modes:

#### `-a, --auto`
**Auto-discover files recursively from exe directory**

Automatically discovers and bundles all files in the same directory as the main executable, including subdirectories. This is the recommended mode for most applications.

**How it works**:
1. Starts from the directory containing the specified `--exe` file
2. Recursively scans all subdirectories
3. Includes all regular files found
4. Excludes the main executable itself (added separately)
5. Maintains directory structure in the bundle
6. Creates a `.log` file listing all bundled files

**Default Limits** (can be overridden with `--limits`):
- Maximum 500 files (safe preset)
- Maximum 500 MB total size (safe preset)

**Example**:
```bash
ExeBundle.exe build --exe MyApp.exe --out Bundled.exe --auto
```

**When to use**:
- Bundling complete applications with DLLs and assets
- When you want everything in the app directory included
- For portable application distribution

**Output**:
Creates a `.log` file named after the output file (e.g., `Bundled.exe.log` for `--out Bundled.exe`) containing the list of all bundled files.


#### `--files <path>`
**Read file list from text file**

Reads the list of files to bundle from a text file. Each line specifies a file to include, as a path relative to the main executable directory.

**File format**:
```text
lib1.dll
lib2.dll
assets\icon.png
config\settings.ini
```

**Example**:
```bash
ExeBundle.exe build --exe MyApp.exe --files filelist.txt --out Bundled.exe
```

**Note**: The `--exe` option is required when using `--files`. The file list contains only additional files to bundle (not the exe itself).

**When to use**:
- Selective bundling (only specific files)
- Repeated builds with consistent file sets
- When you need precise control over included files


#### `-- <files...>`
**Manual file list (positional arguments after --)**

Manually specify files to bundle as command-line arguments after the `--` separator.

**Path resolution**: Paths are resolved **relative to the directory containing `--exe`**, not the current working directory.

**Example**:
```bash
ExeBundle.exe build --exe MyApp.exe --out Bundled.exe -- lib1.dll lib2.dll config.json
```

If `MyApp.exe` is located at `C:\MyApp\bin\MyApp.exe`, then `lib1.dll` must be at `C:\MyApp\bin\lib1.dll`.

**When to use**:
- Quick ad-hoc bundling
- Scripting scenarios where file list is dynamically generated


### Main Executable

#### `-e, --exe <path>`
**Main executable to bundle** (required)

Specifies the primary executable that will be launched when the bundled executable runs. This executable must be a valid Windows PE file (.exe).

This option is **always required** for all file selection modes (`--auto`, `--files`, or manual `--`).

**Example**:
```bash
--exe MyApp.exe
```


### Compression Options

#### `-c, --compression <mode>`
**Compression strategy**

Controls the compression strategy applied to bundled files. Each mode uses a different algorithm optimized for the specified goal. Affects bundle size, creation time, and extraction speed.

**Values**:

| Mode | Algorithm | Best For | Compression | Speed |
|------|-----------|----------|-------------|-------|
| `none` | No compression | Already compressed files (images, videos) | None | Instant |
| `fast` | Xpress Huffman | Quick builds, faster extraction | Good | Fast |
| `balanced` | MSZIP (deflate) | General use (default) | Better | Medium |
| `ultra` | Zstandard | Smallest bundles, network distribution | Best | Slower |

**Beware that algorithms might change in future versions**

**Default**: `balanced`

**Examples**:
```bash
# Fastest extraction
ExeBundle.exe build --exe App.exe --out Bundle.exe --auto --compression fast

# Smallest bundle size
ExeBundle.exe build --exe App.exe --out Bundle.exe --auto --compression ultra

# No compression
ExeBundle.exe build --exe App.exe --out Bundle.exe --auto --compression none
```

**Recommendations**:
- Use `balanced` for most cases (good size reduction, reasonable speed)
- Use `ultra` for network distribution (minimize download size)
- Use `fast` for frequent testing/iteration cycles
- Use `none` if files are already compressed (images, videos, archives)


### Auto-Discovery Limits

#### `--limits <preset>`
**Safety limits for auto-discovery mode**

Prevents accidental bundling of excessively large directory trees when using `--auto`.

**Values**:

| Preset | Max Files | Max Size | Use Case |
|--------|-----------|----------|----------|
| `safe` (default) | 500 | 500 MB | Normal applications |
| `relaxed` | 2,000 | 2 GB | Large applications, game assets |
| `off` | Unlimited | Unlimited | CI/controlled environments only |

**Default**: `safe`

**Example**:
```bash
# Allow larger bundles
ExeBundle.exe build --exe App.exe --out Bundle.exe --auto --limits relaxed

# No limits (controlled environments only)
ExeBundle.exe build --exe App.exe --out Bundle.exe --auto --limits off
```

**⚠ Warning about `--limits off`**:
- Disables all safeguards against accidental bundling of large directory trees
- **Intended for CI/CD pipelines or controlled build environments only**
- In interactive use, prefer `--limits relaxed` for large applications
- Risk: May bundle unexpected files (node_modules, .git, build artifacts, etc.)

**Notes**:
- Only applies to `--auto` mode
- Limits are checked before bundling begins
- Error message shows actual count/size if limits exceeded


## Runtime Behavior (Embedded into Bundle)

These **build-time options** control the runtime behavior of the bundled executable. They are **permanently embedded into the bundle** at build time and affect how it behaves when users run it.

### `--extract <mode>`
**File extraction strategy**

Controls where and how files are extracted at runtime.

**Values**:

| Mode | Behavior | Use Case |
|------|----------|----------|
| `auto` (default) | Smart selection based on bundle characteristics | Recommended for most cases |
| `cache` | Extract to user's temp directory with caching (survives runs) | Faster repeated launches |
| `subdir` | Extract to subdirectory next to executable | Debugging, inspection |
| `temp` | Extract to unique temp directory (cleaned after run) | Maximum isolation, single-run |

**Default**: `auto`

**Examples**:
```bash
# Enable persistent caching for faster repeated runs
ExeBundle.exe build --exe App.exe --out Bundle.exe --auto --extract cache

# Extract to local subdirectory (useful for debugging)
ExeBundle.exe build --exe App.exe --out Bundle.exe --auto --extract subdir
```

**Cache behavior**:
- Cache directory: `%TEMP%\dlb-<guid>`
- Survives across runs
- Files verified for integrity on each run
- Can be cleared manually by user or with `/exebundle:cleanup` switch


### `--cleanup <policy>`
**Temporary file cleanup policy**

Controls when extracted files are deleted.

**Values**:

| Policy | Behavior | Use Case |
|--------|----------|----------|
| `auto` (default) | Smart cleanup based on extraction mode | Recommended |
| `always` | Always delete files after app exits | Maximum disk cleanliness |
| `never` | Never delete extracted files | Debugging, persistence |

**Default**: `auto`

**Examples**:
```bash
# Force cleanup even in cache mode
ExeBundle.exe build --exe App.exe --out Bundle.exe --auto --cleanup always

# Leave files for inspection
ExeBundle.exe build --exe App.exe --out Bundle.exe --auto --cleanup never
```


### `--protect <policy>`
**Extracted directory protection**

Controls write-protection of the extraction directory to prevent tampering.

**Values**:

| Policy | Behavior | Use Case |
|--------|----------|----------|
| `auto` (default) | Enable protection where appropriate | Recommended |
| `on` | Always protect extraction directory | Enhanced security |
| `off` | Never protect (files remain writable) | Apps that modify their own files |

**Default**: `auto`

**Examples**:
```bash
# Disable protection for apps that modify their files
ExeBundle.exe build --exe App.exe --out Bundle.exe --auto --protect off
```


### `--cmdline <template>`
**Custom command line template**

Specifies a custom command line with variable substitution. 
The template defines the complete command to execute, including which process to run.

**Important:** 
The template specifies the complete command line, including which process to execute.
  - For bundled executables: Start with `{exe}` (e.g., `"{exe} --arg"`)
  - For scripts/interpreters: Specify the process explicitly (e.g., `"powershell {exe}"`)
  - Other runtime arguments are **completely ignored** when using custom templates

**Variable Substitution**:

| Variable      | Expansion                              | Example                  |
|---------------|----------------------------------------|--------------------------|
| `{exe}`       | Quoted path to main bundled executable | `"C:\...\cache\app.exe"` |
| `{bundledir}` | Quoted path to extraction directory    | `"C:\...\cache"`         |

**Examples**:
```bash
# Hardcode application arguments
ExeBundle.exe build --exe App.exe --out Bundle.exe --auto \
  --cmdline "{exe} --preset=production"

# Reference configuration file in bundle
ExeBundle.exe build --exe App.exe --out Bundle.exe --auto \
  --cmdline "{exe} --config {bundledir}\settings.ini"

# Multiple variables and arguments
ExeBundle.exe build --exe App.exe --out Bundle.exe --auto \
  --cmdline "{exe} --verbose --data {bundledir}\data --log {bundledir}\app.log"

# Bundle PowerShell script with dependencies
ExeBundle.exe build -e script.ps1 -o bundle.exe --cleanup always \
    --cmdline "powershell -ExecutionPolicy Bypass {exe}" \
    -- script.ps1 module.psm1
  
# Bundle with subdirectory structure
  ExeBundle.exe build -e launcher.bat -o bundle.exe \
    --cmdline "cmd.exe /c {exe}" \
    -- launcher.bat data\config.json
```

**Variable Quoting**:
  - Variables are automatically quoted to handle spaces in paths
  - Do not add extra quotes: use {exe}, not "{exe}"  

**Important Notes**:
  - Template length limited to 4096 characters
  - Variables are case-sensitive (`{exe}` works, `{EXE}` does not)
  - Unknown variables (e.g., `{foo}`) are left as-is
  - Templates are fixed at build time - bundle creator controls execution

**Security Note**: 
Command templates are evaluated at bundle creation time, not runtime. 
The bundle creator has full control over what executes. Use caution when bundling untrusted scripts.

### `--workdir <template>`
**Custom working directory**

Specifies a custom working directory for the bundled application with variable substitution support. The application's current working directory is set to the expanded template path before execution.

**Variable Substitution**:

Same variables as `--cmdline`: `{exe}` and `{bundledir}`.

**Default**: When not specified, the working directory is set to the extraction folder.

**Examples**:
```bash
# Set working directory to extraction folder (same as default)
ExeBundle.exe build --exe App.exe --out Bundle.exe --auto \
  --workdir "{bundledir}"

# Set to subdirectory within bundle
ExeBundle.exe build --exe App.exe --out Bundle.exe --auto \
  --workdir "{bundledir}\data"

# Use absolute path (advanced)
ExeBundle.exe build --exe App.exe --out Bundle.exe --auto \
  --workdir "C:\ProgramData\MyApp"
```

**Important Notes**:
- Template length limited to 127 characters
- Directory must exist when the application starts or process creation will fail
- Variables are case-sensitive
- Unknown variables are left as-is

**Use Cases**:
- Applications that expect to run from a specific directory
- Setting working directory to a subdirectory containing data files
- Compatibility with applications that use relative paths for file access


## Runtime Switches

These switches are used when **running the bundled executable**, not when building it.

All runtime switches use the `/exebundle:` prefix to avoid collisions with arguments intended for the bundled application. This ensures your application receives its arguments unchanged.

### `/exebundle:diag`
Display diagnostic information about the bundled executable.

**Example**:
```bash
MyApp-Bundled.exe /exebundle:diag
```

**Output includes**:
- Extraction strategy (cache/subdir/temp)
- Extraction directory path
- Cleanup policy
- Protection policy
- Command line template (or "(default)" if not specified)
- Working directory template (or "(default)" if not specified)
- Number of bundled files
- Compressed and expanded sizes
- Extraction time
- Compression algorithm used


### `/exebundle:license`
Display license information embedded in the bundled executable.

**Example**:
```bash
MyApp-Bundled.exe /exebundle:license
```


### `/exebundle:cleanup`
Remove cached files for this bundled executable.

**Example**:
```bash
MyApp-Bundled.exe /exebundle:cleanup
```

**Effect**:
- Deletes the cache directory for this specific bundled executable
- Does not affect other bundled executables
- Next run will extract fresh files


## Complete Examples

### Simple Console Application
Bundle a console app with its DLLs:
```bash
ExeBundle.exe build --exe mytool.exe --out mytool-portable.exe --auto
```

### GUI Application with Assets
Bundle a GUI application with maximum compression:
```bash
ExeBundle.exe build ^
    --exe MyApp.exe ^
    --out MyApp-Portable.exe ^
    --auto ^
    --compression ultra ^
    --limits relaxed
```

### Selective Bundling
Bundle only specific files:
```bash
ExeBundle.exe build ^
    --exe App.exe ^
    --out App-Bundle.exe ^
    -- lib1.dll lib2.dll assets\icon.png
```

### Fast Build for Testing
Quick bundle for development iteration:
```bash
ExeBundle.exe build ^
    --exe Debug\App.exe ^
    --out Test.exe ^
    --auto ^
    --compression fast
```

### Network Distribution
Optimize for download size:
```bash
ExeBundle.exe build ^
    --exe MyApp.exe ^
    --out MyApp-Download.exe ^
    --auto ^
    --compression ultra ^
    --extract cache
```

### Debugging Bundle
Bundle with files extracted to visible subdirectory:
```bash
ExeBundle.exe build ^
    --exe MyApp.exe ^
    --out MyApp-Debug.exe ^
    --auto ^
    --extract subdir ^
    --cleanup never
```

### Custom Runtime Execution
Bundle with hardcoded command line and working directory:
```bash
ExeBundle.exe build ^
    --exe MyApp.exe ^
    --out MyApp-Configured.exe ^
    --auto ^
    --cmdline "{exe} --config {bundledir}\app.cfg --verbose" ^
    --workdir "{bundledir}"
```

This creates a bundle that:
- Always runs with `--config {bundledir}\app.cfg --verbose` arguments (runtime args ignored)
- Sets working directory to the extraction folder
- Useful for pre-configured portable applications


## Understanding `--auto` Mode

The `--auto` flag enables **automatic recursive file discovery**. This is the recommended mode for most bundling scenarios.

### What Gets Bundled

When you use `--auto`:
1. ExeBundle starts from the directory containing `--exe`
2. Recursively walks through all subdirectories
3. Includes every regular file found (excluding the exe itself)
4. Preserves the directory structure in the bundle

### Example Directory Structure
```
MyApp/
├── MyApp.exe           ← Main exe (specified with --exe)
├── app.dll             ✓ Bundled
├── config.json         ✓ Bundled
├── assets/
│   ├── icon.png        ✓ Bundled
│   └── styles.css      ✓ Bundled
└── plugins/
    ├── plugin1.dll     ✓ Bundled
    └── plugin2.dll     ✓ Bundled
```

**Command**:
```bash
ExeBundle.exe build --exe MyApp\MyApp.exe --out MyApp-Bundled.exe --auto
```

**Result**: All 7 files bundled (MyApp.exe + 6 other files)

### Safety Limits

To prevent accidental bundling of huge directory trees:

| Scenario | Limit | Action |
|----------|-------|--------|
| Normal app directory | 500 files, 500 MB | Use default `--limits safe` |
| Large app or game | 2,000 files, 2 GB | Use `--limits relaxed` |
| CI/controlled builds | No limits | Use `--limits off` (with caution) |

If limits are exceeded, ExeBundle will:
1. Report the actual file count and size
2. Abort the build
3. Suggest using `--limits relaxed` or `--limits off`

### Log File

Auto-discovery mode creates a `.log` file listing all bundled files:

**Example**: If output is `MyApp.exe`, creates `MyApp.exe.log`:
```
app.dll
config.json
assets\icon.png
assets\styles.css
plugins\plugin1.dll
plugins\plugin2.dll
```

This log helps you verify what was bundled.


## File Path Rules

### Relative Paths
All file paths in bundles are stored **relative to the main executable**:
- ✓ `lib\helper.dll`
- ✓ `assets\icon.png`
- ✗ `C:\MyApp\lib\helper.dll` (absolute paths not allowed)
- ✗ `..\other\file.dll` (parent directory references not allowed)

### Directory Markers
Directories are indicated by a trailing backslash:
- `assets\` (directory)
- `assets\icon.png` (file in assets directory)


## Error Messages

### Common Errors

**"Missing required option: --out"**
- You must always specify `--out <bundle.exe>`

**"No input files specified"**
- You must use one of: `--auto`, `--files <list>`, or `-- <files...>`

**"Cannot combine --auto with --files"**
- Choose exactly one file selection mode

**"--exe required with --auto"**
- Auto mode needs to know which exe to start from

**"Auto-detect limits exceeded"**
- Directory contains more files/data than safety limits allow
- Solution: Use `--limits relaxed` or `--limits off`

**"Invalid executable"**
- The specified exe is not a valid Windows PE file
- Check that the path is correct and the file is not corrupted


## Platform and Subsystem Detection

ExeBundle automatically detects the target platform and subsystem from the input executable:

| Input Executable | Loader Variant |
|------------------|----------------|
| 32-bit GUI | 32-bit GUI loader |
| 32-bit Console | 32-bit Console loader |
| 64-bit GUI | 64-bit GUI loader |
| 64-bit Console | 64-bit Console loader |

The bundled executable will match the platform and subsystem of the original.


## Technical Details

### Bundle Structure
1. **Loader stub**: Embedded executable that extracts and runs the bundle
2. **File entries**: Each file stored with relative path, size, and data
3. **Header**: Metadata at the end (marker, compression info, sizes)
4. **Metadata**: Icon, version info, manifest copied from source exe

### Extraction Process
1. Bundled exe runs
2. Loader reads embedded data from its own resources (ID "555")
3. Creates extraction directory (temp, cache, or subdir)
4. Verifies integrity
5. Extracts files with directory structure
6. Launches main executable with forwarded arguments
7. Returns exit code from main application
8. Performs cleanup based on policy

### Compression
- Files compressed as a single stream (not individually)
- Decompression happens in memory during extraction
- Supported: Xpress Huffman, MSZIP (deflate), Zstandard (those may change in the future)


## Script Bundling Guide

**New in v0.93:** ExeBundle can now bundle and execute scripts directly without requiring a separate executable wrapper. This enables you to distribute PowerShell, batch, Python, and other scripts as standalone `.exe` files.

### PowerShell Scripts

Bundle PowerShell scripts with their dependencies:

```bash
# Basic PowerShell script
ExeBundle.exe build -e script.ps1 -o automation.exe \
  --cmdline "powershell -ExecutionPolicy Bypass -File {exe}" \
  -- script.ps1

# With modules and config files
ExeBundle.exe build -e main.ps1 -o tool.exe \
  --cmdline "powershell -ExecutionPolicy Bypass -File {exe}" \
  -- main.ps1 helpers.psm1 config.json data\settings.xml
```

**Key points:**
- Use `-ExecutionPolicy Bypass` to avoid execution policy restrictions
- Use `-File {exe}` to specify the script path
- `{exe}` is automatically replaced with the extracted script path
- All dependencies are bundled and extracted together

### Batch Scripts

Bundle batch files with their dependencies:

```bash
# Basic batch script
ExeBundle.exe build -e launcher.bat -o tool.exe \
  --cmdline "cmd.exe /c {exe}" \
  -- launcher.bat

# With configuration and data files
ExeBundle.exe build -e setup.bat -o installer.exe \
  --cmdline "cmd.exe /c {exe}" \
  -- setup.bat config.ini data\files.txt
```

**Key points:**
- Use `cmd.exe /c {exe}` to execute the batch script
- Batch scripts can reference other bundled files using relative paths
- Working directory is set to extraction directory by default

### Python Scripts

Bundle Python scripts (requires Python installed on target system):

```bash
# Python script with libraries
ExeBundle.exe build -e main.py -o app.exe \
  --cmdline "python {exe}" \
  -- main.py lib\helpers.py requirements.txt

# With specific Python version
ExeBundle.exe build -e app.py -o application.exe \
  --cmdline "python3 {exe}" \
  -- app.py modules\utils.py
```

**Note:** Python must be installed and in PATH on the target system.

### How It Works

1. **Build time:**
   - You specify a script file as `--exe` (instead of an executable)
   - ExeBundle detects it's not a PE file and uses a default 32-bit console loader
   - All dependencies are bundled together

2. **Runtime:**
   - The loader extracts all files to a local directory
   - The `--cmdline` template is expanded:
     - `{exe}` → full path to extracted script (automatically quoted)
     - `{bundledir}` → extraction directory path (automatically quoted)
   - The specified interpreter is launched with the expanded command line
   - Exit code from the script is returned

3. **Variables are automatically quoted:**
   - `{exe}` becomes `"C:\path\to\script.ps1"` (handles spaces)
   - No need to manually add quotes in your template

### Common Patterns

**PowerShell with arguments:**
```bash
--cmdline "powershell -ExecutionPolicy Bypass -File {exe} -ConfigPath {bundledir}\config.json"
```

**Batch with environment:**
```bash
--cmdline "cmd.exe /c set DATA_DIR={bundledir}\data && {exe}"
```

**Python with modules in bundledir:**
```bash
--cmdline "python {exe}"
--workdir "{bundledir}"
```

### Why Bundle Scripts?

- **Single-file distribution:** No "extract first" instructions
- **Bypass execution policies:** PowerShell scripts run without policy issues
- **Include dependencies:** Bundle modules, config files, data together
- **Corporate environments:** Simplify deployment in locked-down systems
- **Version isolation:** Each bundle carries its exact dependencies

### Limitations

- **Interpreter required:** Target system must have the interpreter installed (PowerShell, Python, etc.)
- **32-bit loader:** Non-PE files use the 32-bit console loader by default
- **No PE metadata:** Icons and version info from scripts are not copied (no PE resources to extract)

### Security Considerations

- Templates are fixed at **build time**, not modifiable at runtime
- Bundle creator controls what executes (already has full control over bundled files)
- Runtime arguments are **ignored** when using custom `--cmdline`
- Use `--protect on` to prevent tampering with extracted scripts


## Best Practices

### For Distribution
- Use `--compression ultra` to minimize download size
- Use `--extract cache` for faster repeated launches
- Test the bundled executable on a clean system
- Run with `/exebundle:diag` to verify bundled contents

### For Development
- Use `--compression fast` for quick iteration
- Use `--extract subdir --cleanup never` for debugging
- Keep the `.log` file to verify bundled files

### For Security
- Use `--protect on` to prevent tampering with extracted files
- Verify bundled executable with antivirus before distribution
- Test on Windows Defender (may flag unknown executables)

### For Large Applications
- Use `--limits relaxed` for apps with many files
- Consider selective bundling with `--files` instead of `--auto`
- Test extraction time with `/exebundle:diag`

### For Custom Execution Control
- Use `--cmdline` when you need to force specific arguments regardless of how users invoke the bundle
- Use `--workdir` for applications that require a specific working directory
- Reference bundled assets with `{bundledir}` in templates (e.g., config files, data directories)
- Test template expansion with `/exebundle:diag` to verify substitution
- Keep templates under length limits (4k chars for cmdline, 4k for workdir)


## Getting Help

```bash
# General help
ExeBundle.exe --help

# Build command help
ExeBundle.exe build --help

# License information
ExeBundle.exe license
```


## See Also

- **README.md** - Overview and project description
- **LICENSE.txt** - Full license terms
- **LICENSE-FAQ.md** - Common licensing questions
- **EXAMPLES.md** - Real-world bundling scenarios (if available)

---

**Copyright** © 2016–2025 Florian Mücke
**Status**: Actively developed. Interfaces may evolve between releases.
