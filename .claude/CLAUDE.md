# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Rush monorepo for Heft plugins published under the `@voitanos` npm scope. The repository uses Rush for package management, Heft for building, and GitHub Actions for automated NPM publishing.

**Key Technologies:**
- Rush 5.162.0 (monorepo management)
- pnpm 9.15.0 (package manager)
- Node.js 22.11.0+ (LTS only)
- Heft (build system)
- TypeScript 5.7+
- Jest (testing)

## Repository Structure

```
heft-plugins/
├── heft-plugins/
│   ├── heft-stylelint-plugin/     # Individual plugin packages
│   └── [future-plugins]/          # Additional plugins as added
├── common/
│   └── config/rush/               # Rush configuration
│       ├── command-line.json
│       ├── common-versions.json
│       ├── pnpm-lock.yaml
│       └── version-policies.json  # Individual versioning per plugin
├── .github/workflows/
│   └── publish.yml                # Automated NPM publishing
└── rush.json                      # Rush monorepo config
```

## Common Commands

### Rush Commands (Repository Level)

```console
# Install dependencies for all packages
rush update

# Build all packages
rush rebuild

# Build all packages (incremental)
rush build

# Run tests for all packages
rush test

# Clean all build outputs
rush purge

# Create version change files (for changelog)
rush change

# Bump versions based on change files
rush version --bump

# Bump pre-release versions
rush version --bump --override-prerelease-id beta
```

### Heft Commands (Package Level)

Navigate to a specific plugin directory first: `cd heft-plugins/[plugin-name]/`

```console
# Build package
heft build --clean

# Build package (incremental)
heft build

# Run tests
heft test --clean

# Watch mode for development
heft build-watch

# Run tests in watch mode
heft test --watch
```

### Running Single Tests

To run a specific test file:

```console
cd heft-plugins/[plugin-name]/
npx jest src/YourFile.test.ts
```

To run tests matching a pattern:

```console
cd heft-plugins/[plugin-name]/
npx jest -t "test name pattern"
```

## Architecture

### Rush Monorepo Structure

This repository uses **independent versioning** for each plugin. Each plugin:
- Has its own version number (defined in `package.json`)
- Can be published independently
- Uses a version policy defined in `common/config/rush/version-policies.json`
- Is associated with a policy via `versionPolicyName` in `rush.json`

**Example version policy:**
```json
{
  "definitionName": "individualVersion",
  "policyName": "heft-stylelint-plugin-policy",
  "lockedMajor": 0
}
```

### Heft Plugin Architecture

Each plugin in this monorepo follows the Heft plugin structure:

1. **Plugin Definition** (`heft-plugin.json`): Declares the plugin, entry point, and options schema
2. **Plugin Implementation**: TypeScript class implementing `IHeftTaskPlugin<TOptions>`
3. **Build Configuration** (`config/heft.json`): Defines build and test phases
4. **Package Structure**:
   - `src/` - TypeScript source files
   - `lib/` - Compiled JavaScript output (published)
   - `lib/schemas/` - JSON schemas for plugin options (published)
   - `heft-plugin.json` - Plugin manifest (published)

**Key plugin implementation points:**
- Plugins implement `IHeftTaskPlugin<TOptions>` interface
- The `apply()` method registers hooks with the task session
- Hooks typically tap into `taskSession.hooks.run` for build-time execution
- Use `taskSession.logger.terminal` for output
- Access build folder path via `heftConfiguration.buildFolderPath`

### Publishing Strategy

The repository uses **tag-based automated publishing** via GitHub Actions:

**Tag Format:** `<plugin-short-name>@v<version>`
- Example: `heft-stylelint-plugin@v0.1.0`
- Example: `heft-stylelint-plugin@v0.1.0-beta.1`

**NPM Dist-Tags:**
- Stable versions (e.g., `1.0.0`) → published to `@latest`
- Pre-release versions (e.g., `1.0.0-beta.1`, `1.0.0-rc.1`, `1.0.0-alpha.1`) → published to `@next`

**Publishing Workflow:**
1. Tag created and pushed to GitHub
2. GitHub Actions validates build and tests
3. Package published to NPM with appropriate dist-tag
4. GitHub Release created automatically

### NPM Package Configuration

Each plugin package must include:
- `"type": "module"` - ESM format
- `"main"` and `"typings"` - Entry points
- `"exports"` - Modern module resolution
- `"files"` - Only publish `lib/` and `heft-plugin.json`
- `"publishConfig"` - Public access and NPM registry
- `"peerDependencies"` - Require compatible Heft version

## Development Workflow

### Adding a New Plugin

1. Create plugin directory: `heft-plugins/<plugin-name>/`
2. Add plugin configuration to `rush.json` projects array
3. Create version policy in `common/config/rush/version-policies.json`
4. Set up plugin package structure (see existing plugins)
5. Run `rush update` to register the new project

### Making Changes to a Plugin

1. Navigate to plugin: `cd heft-plugins/<plugin-name>/`
2. Make code changes
3. Build: `heft build --clean`
4. Test: `heft test --clean`
5. Create change file: `rush change` (from repository root)
6. Commit changes

### Publishing a Plugin

**For Beta/RC releases:**
```console
# Update version in package.json to pre-release format
cd heft-plugins/<plugin-name>/
# Edit package.json version to X.Y.Z-beta.1 (or -rc.1, -alpha.1)

# Commit and tag
git add .
git commit -m "chore: prepare beta release"
git tag <plugin-name>@vX.Y.Z-beta.1
git push origin main
git push origin <plugin-name>@vX.Y.Z-beta.1
```

**For Stable releases:**
```console
# Update version in package.json to stable format
cd heft-plugins/<plugin-name>/
# Edit package.json version to X.Y.Z

# Commit and tag
git add .
git commit -m "chore: release vX.Y.Z"
git tag <plugin-name>@vX.Y.Z
git push origin main
git push origin <plugin-name>@vX.Y.Z
```

GitHub Actions will automatically build, test, and publish.

## Node.js Version Requirements

**Supported Version:** Node.js 22.11.0+ (LTS only)

This is enforced in `rush.json`:
```json
"nodeSupportedVersionRange": ">=22.11.0 <23.0.0"
```

Rush will refuse to run with unsupported Node.js versions.

## Important Notes

- Each plugin has its own CLAUDE.md with plugin-specific details
- Never commit changes without creating a change file via `rush change`
- The `lib/` directory is git-ignored (generated during build)
- Always run `rush update` after pulling changes that affect dependencies
- Pre-release versions are automatically published to the `@next` NPM tag
- GitHub Actions requires npm 11.5.1+ for trusted publishing (automatically installed in CI)
