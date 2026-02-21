# ExeBundle
![Chocolatey Version](https://img.shields.io/chocolatey/v/exebundle)
![WinGet Package Version](https://img.shields.io/winget/v/fmuecke.ExeBundle)


**Ship one executable. Nothing else.**

ExeBundle is a **Windows application bundler** that packages tools and applications into a
**single, self-contained executable (.exe)** — **no installer**, no MSI, no external files.

If your application needs more than one file to run, ExeBundle turns it into **exactly one executable**.

## Why ExeBundle Exists

Shipping software on Windows often means one of these compromises:

* installers with complex setup logic
* ZIP archives with “extract everything first” instructions
* missing DLLs, broken paths, or version conflicts
* administrative privileges where none should be required

ExeBundle exists to solve **one specific problem**:

> **How do I ship an application as a single executable that just runs?**

This makes ExeBundle especially useful for **portable Windows tools** and
**self-contained command-line utilities** that should run without setup.

## What ExeBundle Is (and Is Not)

**ExeBundle is:**

* a **Windows application bundler**
* focused on **distribution**, not installation
* produces **single-executable**, **self-contained** applications
* suitable for tools, utilities, and internal applications

**ExeBundle is not:**

* an installer framework
* a replacement for MSI, MSIX, or setup wizards
* a packaging solution for system-wide deployment

## Typical Use Cases

* Shipping internal tools to colleagues
* Delivering utilities to customers
* Providing portable diagnostics or helpers
* **DLL bundling** for Windows applications
* **Script distribution** - ship PowerShell/batch scripts as standalone executables
* Running tools in restricted or locked-down environments
* Simplifying CI/CD delivery pipelines
* Shipping **portable Windows tools** without installers
* Distributing automation scripts without requiring interpreter setup

If you want users to **download one file and run it**, ExeBundle fits.

## Quick Start (30 seconds)

### Installation

**Direct download:** `ExeBundle.exe` (verify with `.sha256`)

**Package managers:**
```powershell
# Winget
winget install fmuecke.ExeBundle

# Scoop
scoop bucket add fmuecke https://github.com/fmuecke/scoop-bucket
scoop install exebundle

# Chocolatey
choco install exebundle
```

### Create an ExeBundle

**Bundle a Windows executable:**

```text
ExeBundle.exe -e MyApp.exe -o MyApp-Bundled.exe -a
```

**Bundle a PowerShell script:**

```text
ExeBundle.exe -e script.ps1 -o script-bundled.exe --cmdline "powershell -ExecutionPolicy Bypass -File {exe}" --files script.ps1
```

_Note: The `build` command is optional and can be omitted (it's the default)._

Result:

* Single executable file
* No installer required
* No external files needed
* Runs on any compatible Windows system
* Scripts execute without separate interpreter installation

_For detailed examples on how to bundle PowerShell, batch, and other scripts directly, see [Script Bundling Guide](CLI-REFERENCE.md#script-bundling-guide)._


## How It Works (High Level)

1. You define a main executable and additional files
2. ExeBundle packages everything into a single `.exe`
3. At runtime:
   * files are extracted locally (cache, subdir, or temp)
   * integrity is verified
   * the main executable is launched
   * the files are cleaned up (unless cached)
4. No global installation is performed
5. No system state is modified unless your application does so

## What Can Be Bundled

* Windows executables and **DLL dependencies**
* command-line tools
* **scripts (PowerShell, batch, Python)** - bundle scripts for direct execution
* configuration files
* assets and resource files like images etc.

Key characteristics:

* **single-executable** distribution
* **self-contained** runtime behavior
* predictable unpacking and execution
* optional local caching for repeated runs
* selectable compression strategies

## Releases & Documentation

All public releases are published via **GitHub Releases** and include:

* prebuilt binaries
* checksums
* release notes

### Documentation

* **[CLI-REFERENCE.md](CLI-REFERENCE.md)** - Complete command-line reference with detailed option explanations
* **[CHANGELOG.md](CHANGELOG.md)** - Version history and breaking changes
* **[LICENSE-FAQ.md](LICENSE-FAQ.md)** - Common licensing questions

The core source code is developed in a separate private repository.

## Licensing

ExeBundle is **free for non-commercial use**, including:

* private use
* education
* open-source projects
* non-profit organizations
* evaluation and testing

Commercial usage requires a separate license.

See:

* [LICENSE.txt](LICENSE.txt)
* [LICENSE-SUMMARY.md](LICENSE-SUMMARY.md)
* [LICENSE-FAQ.md](LICENSE-FAQ.md)

## Technology

* Platform: Windows
* Language: C++
* No runtime dependencies
* No kernel drivers
* No executable rewriting at runtime
* Runs entirely in user mode using standard Windows execution semantics

## Community & Feedback

* **Issues** → bugs or concrete problems
* **Discussions** → questions, ideas, feedback

Real-world usage feedback is explicitly welcome.


## Keywords

Windows · single executable · application bundler · portable apps ·
self-contained · no installer · DLL bundling · command-line tools


## Status

ExeBundle is actively developed.
Interfaces and behavior may evolve between releases.

Breaking changes are documented explicitly.

© 2016–2025 Florian Mücke
