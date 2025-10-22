# Glamorous Toolkit Development Guide

## Project Overview

Glamorous Toolkit (GT) is a **moldable development environment** built in Pharo Smalltalk and Rust. It embodies a philosophy where the development environment adapts to match the context of your work through custom, contextual tools.

### Key Characteristics
- Written in **Pharo Smalltalk** (`.st` files in `src/`) with Rust components
- Live, image-based development using Pharo images and VMs
- Extensive use of **Lepiter** (literate programming notebooks) for documentation
- Multi-repository architecture with ~50+ component projects coordinated via Baselines
- Cross-platform releases (macOS, Linux, Windows) for both x86_64 and ARM64

## Architecture

### Component Structure
- **`src/`**: Pharo Smalltalk packages organized by Baseline (e.g., `BaselineOfGToolkit`, `BaselineOfGToolkitForPharo9`)
- **`lepiter/`**: Documentation and knowledge base in Lepiter notebook format (`.lepiter` files)
- **`scripts/`**: Build automation and deployment scripts
  - `scripts/localbuild/`: User installation scripts (mac.sh, linux.sh, windows.ps1)
  - `scripts/build/`: CI/CD build orchestration
- **`Jenkinsfile`**: Complete CI/CD pipeline definition (1200+ lines, critical reference)

### Baseline System
GT uses **Metacello Baselines** for dependency management:
- `BaselineOfGToolkit`: Main entry point loading all components
- `BaselineOfGToolkitForPharo9`: Pharo 9+ specific configuration
- Each baseline defines GitHub repositories, packages, and dependencies
- Example pattern: `baseline: 'ComponentName' with: [ spec repository: 'github://feenkcom/repo:main/src' ]`

### Build Process
The build uses custom tooling:
1. **gt-installer** (Rust): Downloads VMs, builds images, packages releases
2. **feenk-releaser** (Rust): Manages multi-repo releases and GitHub releases
3. Version files: `gtoolkit-builder.version`, `gtoolkit-vm.version`, `feenk-releaser.version`

Key build stages (see `Jenkinsfile`):
- Load source from `PHARO_IMAGE_URL` base image
- Clone and load from GitHub repos via Baselines
- Run examples/tests with `--disable-deprecation-rewrites --skip-packages` flags
- Package for multiple platforms simultaneously
- Sign macOS packages with notarization
- Build Docker images for linux/amd64 and linux/arm64

## Development Workflows

### Local Installation
Install from sources (builds full image, ~10 minutes):
```bash
# Mac
curl https://dl.feenk.com/scripts/mac.sh | bash

# Linux
curl https://dl.feenk.com/scripts/linux.sh | bash

# Windows (PowerShell)
wget https://dl.feenk.com/scripts/windows.ps1 -OutFile windows.ps1; ./windows.ps1
```

### Testing
Tests are **GToolkit Examples** (not traditional SUnit):
- Examples serve as both tests and executable documentation
- Run via: `gt-installer test --disable-deprecation-rewrites --skip-packages "GToolkit-Boxer" "Sparta-Cairo" ...`
- Results output as JUnit XML for CI integration
- Some packages intentionally skipped: GToolkit-Boxer, Sparta-Cairo/Skia, RemoteExamples-GemStone, PythonBridge-Pharo

### CI/CD Pipeline
Jenkins-based multi-platform build:
- Builds tentative image on macOS ARM64 (primary)
- Tests/packages in parallel: macOS (x86_64, ARM64), Linux (x86_64, ARM64), Windows (x86_64)
- Special Linux build includes GemStone and Python integration tests
- Docker multi-arch images published to `feenkcom/gtoolkit`
- Releases to GitHub with platform-specific packages: `GlamorousToolkit-{os}-{arch}-v{version}.zip`

### Working with Lepiter Notebooks
Lepiter is GT's native documentation format (stored in `lepiter/`):
- Each `.lepiter` file is a page with unique hash-based name
- `lepiter.properties` defines database configuration
- Notebooks contain executable Pharo code snippets, visualizations, and text
- Used extensively for project documentation (equivalent to markdown docs in other projects)

## Pharo/Smalltalk Conventions

### File Structure
- **Classes**: Defined in `.class.st` files: `Class { #name : #ClassName, #superclass : #SuperClass, #category : #PackageName }`
- **Extensions**: Methods added to existing classes in `.extension.st` files
- **Packages**: Defined in `package.st` files: `Package { #name : #PackageName }`

### Naming Patterns
- Classes: PascalCase with `Gt` or `Le` prefixes (e.g., `GtToolkit`, `LeDatabase`)
- Methods: camelCase with colons for parameters (e.g., `initialize`, `gtExamplesFor:`)
- Baselines: `BaselineOfProjectName` pattern
- Categories/Protocols: dash-separated (e.g., `#'*PackageName'`, `#'post load'`)

### Common Pharo Patterns
```smalltalk
"Method with pragma"
methodName
    <gtExample>
    ^ self computation

"Class definition"
Class {
    #name : #MyClass,
    #superclass : #Object,
    #category : #MyPackage
}

"Pharo version conditionals (common in GT)"
self 
    forPharo12AndNewer: [ "Pharo 12+ code" ]
    forPharo11: [ "Pharo 11 code" ]
```

### Code Formatting
GT enforces specific formatting via `RBConfigurableFormatter`:
- Max line length: 80 characters
- No new line after pragma
- Same-line keyword arguments
- Retain blank lines between statements
- Set in `BaselineOfGToolkit>>postLoadGToolkit:`

## Integration Points

### Multi-Repository Coordination
GT consists of 50+ repositories under `github.com/feenkcom/`:
- Core: gtoolkit, Bloc (graphics), Brick (widgets), lepiter
- Language support: gt4pharo, gt4python, gt4ruby, gt4gemstone
- Tools: gtoolkit-coder, gtoolkit-debugger, gtoolkit-spotter, gtoolkit-visualizer
- Dependencies loaded via Baseline repository specs

### External Systems
- **GemStone/S**: Remote Smalltalk VM integration (tested on Linux x86_64 only)
- **PythonBridge**: Python interop (PyPI package: gtoolkit-bridge)
- **Docker**: Published multi-arch images at `feenkcom/gtoolkit`
- **Jenkins**: Main CI orchestration with platform-specific build nodes

### Graphical Stack
GT uses custom rendering (Sparta/Bloc/Brick), not Morphic:
- **Sparta**: 2D graphics rendering (Cairo/Skia backends)
- **Bloc**: UI framework with single rendering tree
- **Brick**: Widget library
- Text editor embeds graphics elements (live, moldable)

## Critical Files Reference

- **`Jenkinsfile`**: Complete build/test/release pipeline
- **`src/BaselineOfGToolkit/BaselineOfGToolkit.class.st`**: Main dependency graph
- **Version files**: `*.version` files specify tool versions
- **`README.md`**: User-facing documentation and quick start
- **`lepiter/`**: In-depth documentation as live notebooks

## Common Pitfalls

1. **Don't treat this as a traditional file-based project** - it's image-based with source in repositories
2. **Examples are tests** - don't look for XUnit-style test classes
3. **Lepiter files are documentation** - hash-named files in `lepiter/` are the knowledge base
4. **Platform-specific considerations** - Windows needs firewall rules, macOS needs code signing
5. **Multi-repo complexity** - changes often span multiple feenkcom repositories
6. **Pharo version compatibility** - GT patches Pharo base classes (see `GToolkit-PharoBasePatch-*` packages)

## Resources

- Documentation: https://book.gtoolkit.com
- Community: Discord (discord.gg/FTJr9gP)
- Videos: https://youtube.com/@gtoolkit
- Website: https://gtoolkit.com
