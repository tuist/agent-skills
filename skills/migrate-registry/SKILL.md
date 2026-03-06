---
name: migrate-spm-to-tuist-registry
description: Migrates Swift Package Manager dependencies in Tuist Project.swift files from .remote(url:) to registry-based .package(id:) format, replacing packages one at a time with build validation after each change. Use when adopting the Tuist registry for an existing project.
---

# Migrate SPM Packages to Tuist Registry

## Syntax by File

| File | URL-based | Registry-based |
|------|-----------|----------------|
| `Project.swift` | `.remote(url: "https://...", requirement: ...)` | `.package(id: "owner.repo", from: "x.y.z")` |
| `Tuist/Package.swift` | `.package(url: "https://...", from: "x.y.z")` | `.package(id: "owner.repo", from: "x.y.z")` |

> `Project.swift` uses `.remote(url:)`. `Tuist/Package.swift` uses `.package(url:)`. These are different APIs.

## Quick Start

1. Detect which integration style the project uses (see below).
2. Enable registry in `Tuist.swift` with `registryEnabled: true`.
3. Follow the migration path for the detected integration style.
4. Run `tuist generate --no-open` after each replacement.
5. Keep the registry ID on success; revert on failure.
6. Commit the final state with only the successfully migrated packages.

## Preflight Checklist

- **Detect integration style** — check where packages are defined (see below)
- Confirm `Tuist.swift` exists (not `Config.swift`)
- Note exact version requirements for each package
- Ensure `mise` is activated once per session before running `tuist`

## Detect Integration Style

Before migrating, determine where packages are declared:

**Project.swift-based** — packages live in the `packages:` array of each `Project.swift`:
```swift
// Projects/App/Project.swift
let project = Project(
    packages: [
        .remote(url: "https://github.com/firebase/firebase-ios-sdk", requirement: ...)
    ]
)
```

**XcodeProj-based (Tuist/Package.swift)** — packages live in `Tuist/Package.swift` as a Swift package manifest:
```swift
// Tuist/Package.swift
let package = Package(
    dependencies: [
        .package(url: "https://github.com/firebase/firebase-ios-sdk", from: "11.8.1")
    ]
)
```

> A project uses one style only — never both. If `Tuist/Package.swift` has a non-empty `dependencies` array, it is XcodeProj-based. If `Project.swift` files have non-empty `packages:` arrays, it is Project.swift-based.

## Registry ID Format

Tuist registry uses Swift Package Index as its backend. A package is available only if it exists on Swift Package Index.

- Format: `{owner}.{repo}` — all lowercase, dots instead of slashes
- Example: `https://github.com/firebase/firebase-ios-sdk` → `firebase.firebase-ios-sdk`

**Edge case — dots in repo names:** Replace `.` in the repository name with `_`.

```
https://github.com/groue/GRDB.swift → groue.GRDB_swift
```

## Outputs

- `Tuist.swift` updated with `registryEnabled: true`
- `Project.swift` files with eligible packages converted to `.package(id:, from:)`
- Packages not on registry remain as `.remote(url:)`

## Migration Workflow

### Step 1 — Enable the registry (both styles)

Add `registryEnabled: true` to `generationOptions` in `Tuist.swift`. Do **not** create a separate `Config.swift`.

```swift
let tuist = Tuist(
    fullHandle: "org/repo",
    project: .tuist(
        generationOptions: .options(
            enableCaching: true,
            registryEnabled: true
        )
    )
)
```

Running `tuist generate` automatically creates the registry configuration file.

### Step 2 — Activate mise (once per session)

```bash
eval "$(mise activate bash)"
```

### Step 3A — Project.swift-based migration

Replace one `.remote(url:)` entry at a time:

```swift
// Before
.remote(url: "https://github.com/firebase/firebase-ios-sdk",
        requirement: .upToNextMajor(from: "11.8.1"))

// After
.package(id: "firebase.firebase-ios-sdk", from: "11.8.1")
```

> Use `from:` directly — not `requirement: .upToNextMajor(from:)`.

Run `tuist generate --no-open` after each change:
- **Success** → keep, move to the next package
- **`package not found on registry`** → revert to `.remote(url:)`, move to the next package

Repeat for every entry across all `Project.swift` files.

### Step 3B — XcodeProj-based migration (`Tuist/Package.swift`)

Two options are available:

**Option A — explicit `id:` references** (guarantees registry-first resolution):
```swift
// Tuist/Package.swift — replace one at a time and validate after each
.package(id: "firebase.firebase-ios-sdk", from: "11.8.1")
```

**Option B — `--replace-scm-with-registry` flag** (keeps URL declarations, resolves from registry automatically when available):
```swift
let tuist = Tuist(
    fullHandle: "org/repo",
    project: .tuist(
        installOptions: .options(
            passthroughSwiftPackageManagerArguments: ["--replace-scm-with-registry"]
        )
    )
)
```

> `--replace-scm-with-registry` is **only for XcodeProj-based integration**. It has no effect on `Project.swift` packages.

## Authentication

Without authentication, the registry allows **1,000 requests per minute per IP**. For teams or CI, log in to raise the limit to **20,000 requests per minute**:

```bash
tuist registry login
```

Requires a Tuist account with `fullHandle` set in `Tuist.swift`. Commit the generated registry configuration file so all team members and CI share the same setup.

## Important Constraints

- Keep packages in **one place only**: either `Project.swift` or `Tuist/Package.swift`, never both
- `packages:` in `Project.swift` and dependencies in `Tuist/Package.swift` conflict at generation time
- Do not run `tuist registry setup` manually — `registryEnabled: true` handles this automatically

## Common Failure Patterns

- **`no registry configured for '...' scope`**: `registryEnabled: true` is missing from `Tuist.swift`
- **`package not found on registry`**: package is not on Swift Package Index; revert to `.remote(url:)`
- **Wrong ID for dotted repo names**: `groue/GRDB.swift` must be `groue.GRDB_swift`, not `groue.GRDB.swift`
- **`tuist: command not found`**: run `eval "$(mise activate bash)"` first
- **Generation fails after mixing locations**: packages defined in both `Project.swift` and `Tuist/Package.swift`; pick one and remove the other

## Done Checklist

- `registryEnabled: true` is set in `Tuist.swift`
- Every package has been tried for registry migration
- Successfully migrated packages use `.package(id:, from:)`
- Packages not on registry remain as `.remote(url:)` in `Project.swift`
- Dotted repo names use `_` instead of `.` in the ID
- `tuist generate --no-open` succeeds cleanly
