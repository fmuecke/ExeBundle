# ExeBundle  
**Ship applications as single executables**

ExeBundle packages applications for distribution into **single, self-contained Windows executables** that run **without installers or external files**.

This repository serves as the **public release, documentation, and community hub** for ExeBundle.  
The core source code is developed and maintained in a separate private repository.


## What ExeBundle Does

ExeBundle creates an *executable bundle*:  
a single `.exe` that encapsulates everything required to run an application.

The resulting executable can be:
- copied
- distributed
- deployed
- executed

without relying on installers, runtimes, or additional files on the target system.


## Scope and Terminology

In this context, *applications* include:
- GUI applications
- command-line tools
- utilities
- internal tools
- support or diagnostic binaries

ExeBundle focuses on **packaging and distribution**.  
Execution is self-contained and follows a deterministic model.


## What ExeBundle Is Not

ExeBundle is deliberately focused. It is:

- not a container runtime
- not a sandbox or virtual machine
- not an installer framework
- not a mandatory obfuscation or protection layer

ExeBundle operates entirely in **user mode** and relies on **standard Windows execution semantics**.


## Key Characteristics

- **Single-file distribution**  
  One executable represents the complete application.

- **Self-contained execution**  
  No installers, no external files, no runtime dependencies.

- **Windows-first**  
  Designed specifically for Windows environments.

- **Predictable behavior**  
  Controlled unpacking and execution.

- **Focused tooling**  
  ExeBundle does not prescribe how applications are built or structured.


## Packaging Capabilities

ExeBundle can bundle:
- executables
- DLLs
- scripts (e.g. PowerShell, CMD)
- configuration files
- assets and resource files

Advanced characteristics include:
- deterministic extraction behavior
- optional local caching for repeated execution
- support for different compression strategies (e.g. startup speed vs. bundle size)
- compatibility with Authenticode-signed binaries


## How It Works (High Level)

1. You define which files and entry points should be bundled
2. ExeBundle packages them into a single executable
3. At runtime:
   - the bundle is prepared locally (side-by-side or cached)
   - integrity is verified
   - the selected entry point is launched
4. No global installation is performed
5. No system state is modified unless the bundled application does so


## Typical Use Cases

- Distributing internal tools
- Shipping utilities to customers
- Delivering applications into restricted environments
- Packaging tools with dependencies
- Providing portable diagnostics or helpers
- Simplifying deployment pipelines


## Releases

All public releases are published via **GitHub Releases**.

Each release includes:
- prebuilt binaries
- checksums
- release notes

The source code is not part of this repository.


## Documentation

This repository contains:
- usage instructions
- command-line reference
- packaging concepts
- limitations and known constraints

Documentation evolves together with the releases.


## Technology

- Platform: Windows
- Language: C++
- No runtime dependencies
- No kernel drivers
- No executable rewriting at runtime


## Licensing & Usage

ExeBundle is proprietary software.

Free usage is permitted for:
- Private, non-commercial use
- Educational use
- Commercial evaluation and testing purposes (PoC, compatibility checks)

A **commercial license** is required for:
- Production use
- Operational or revenue-generating workflows
- Distribution to customers

Details are described in the [LICENSE.txt](LICENSE.txt) file.  
For commercial licensing, redistribution, or enterprise use, just contact the author.


## Issues & Discussions

This repository is the **public interaction point** for ExeBundle.

- Use **Issues** for bugs or concrete problems.
- Use **Discussions** for questions, ideas, and feedback.

Feature requests are welcome, but evaluated deliberately.


## Roadmap

Development priorities:
- stability
- predictability
- minimalism
- long-term maintainability

ExeBundle intentionally avoids feature bloat.


## Status

ExeBundle is actively developed.  
Interfaces and behavior may evolve between releases.

Breaking changes are documented explicitly.

---

© 2025 – Florian Mücke // ExeBundle
 
