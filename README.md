# ExeBundle  
**Ship applications as single executables**

ExeBundle packages applications for distribution into **single, self-contained Windows executables** that run **without installers or external files**.

This repository is the **public release, documentation, and community hub** for ExeBundle.  
The core source code is developed in a separate private repository.

## Core Concept

ExeBundle creates an **executable bundle**:  
a single `.exe` that encapsulates everything required to run an application.

In this context, *applications* include:
- GUI applications
- command-line tools
- utilities and internal tools
- support or diagnostic binaries

ExeBundle focuses on **packaging and distribution** with **deterministic, self-contained execution**.


## Capabilities

ExeBundle can bundle:
- executables and DLLs
- scripts (e.g. PowerShell, CMD)
- configuration files
- assets and resource files

Key characteristics:
- **single-file distribution**
- **no installers or external dependencies**
- **predictable unpacking and execution**
- optional local caching for repeated runs
- support for different compression strategies
- compatible with Authenticode-signed binaries


## How It Works (High Level)

1. Define which files and main executable should be bundled
2. ExeBundle packages everything into a single executable
3. At runtime:
   - the bundle is extracted locally (cached, subdir, or temporary)
   - integrity is verified
   - the main executable is launched
4. No global installation is performed
5. No system state is modified unless the bundled application does so


## Typical Use Cases

- Distributing internal tools
- Shipping utilities to customers
- Delivering applications into restricted environments
- Packaging tools with dependencies
- Providing portable diagnostics or helpers
- Simplifying deployment pipelines


## Releases & Documentation

All public releases are published via **GitHub Releases** and include:
- prebuilt binaries
- checksums
- release notes

This repository also contains:
- usage instructions
- command-line reference
- packaging concepts
- known limitations

Source code is not part of this repository.


## Technology, Licensing & Community

**Technology**
- Platform: Windows
- Language: C++
- No runtime dependencies
- No kernel drivers
- No executable rewriting at runtime
- Operates entirely in **user mode** and relies on **standard Windows execution semantics**


**Licensing**
- Free for non-commercial use only
- Commercial usage requires a separate license (available on request)
- Details described in [LICENSE.txt](LICENSE.txt), [LICENSE-SUMMARY.md](LICENSE-SUMMARY.md) and [LICENSE-FAQ.md](LICENSE-FAQ.md)


**Community**
- Use **Issues** for bugs or concrete problems
- Use **Discussions** for questions, ideas, and feedback



## Status

ExeBundle is actively developed.  
Interfaces and behavior may evolve between releases.

Breaking changes are documented explicitly.

---

© 2025 – Florian Mücke // ExeBundle
