# ExeBundle

**Ship one executable. Nothing else.**

ExeBundle is a Windows-focused tool to **package applications into a single, self-contained `.exe`**
— no installer, no MSI, no external files.

If your application needs more than one file to run, ExeBundle turns it into **exactly one executable**.

## Why ExeBundle Exists

Shipping software on Windows often means one of these compromises:

* installers with complex setup logic
* ZIP archives with “extract everything first” instructions
* missing DLLs, broken paths, or version conflicts
* administrative privileges where none should be required

ExeBundle exists to solve **one specific problem**:

> **How do I ship an application as a single executable that just runs?**

## What ExeBundle Is (and Is Not)

**ExeBundle is:**

* a bundler for Windows executables
* focused on distribution, not installation
* deterministic and self-contained
* suitable for tools, utilities, and internal applications

**ExeBundle is not:**

* an installer framework
* a replacement for MSI, MSIX, or setup wizards
* a packaging solution for system-wide deployment

## Typical Use Cases

* Shipping internal tools to colleagues
* Delivering utilities to customers
* Providing portable diagnostics or helpers
* Packaging applications with DLL dependencies
* Running tools in restricted or locked-down environments
* Simplifying CI/CD delivery pipelines

If you want users to **download one file and run it**, ExeBundle fits.

## Quick Start (30 seconds)

```text
ExeBundle.exe build -e MyApp.exe -o MyApp-Bundled.exe -a
```

Result:

* `MyApp-Bundled.exe`
* no installer
* no external files
* all files/folders in the same folder as MyApp.Exe are automatically included
* runs on any compatible Windows system

## How It Works (High Level)

1. You define a main executable and additional files
2. ExeBundle packages everything into a single `.exe`
3. At runtime:
   * files are extracted locally (temp or cache)
   * integrity is verified
   * the main executable is launched
   * the files are cleanup (unless cached)
4. No global installation is performed
5. No system state is modified unless your application does so

## What Can Be Bundled

* executables and DLLs
* command-line tools
* scripts (PowerShell, CMD)
* configuration files
* assets and resource files like images etc.

Key characteristics:

* single-file distribution
* predictable unpacking and execution
* optional local caching for repeated runs
* selectable compression strategies

## Releases & Documentation

All public releases are published via **GitHub Releases** and include:

* prebuilt binaries
* checksums
* release notes

This repository contains:

* usage instructions
* command-line reference
* packaging concepts
* known limitations

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

## Status

ExeBundle is actively developed.
Interfaces and behavior may evolve between releases.

Breaking changes are documented explicitly.

© 2016–2025 Florian Mücke
