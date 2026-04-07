# BPM — Blueplane Package Manager

**Create, publish, and install reusable AI coding extensions across your team.**

BPM is a package management system for AI coding tool extensions. It gives teams a proper registry for sharing **skills**, **commands**, and **hooks** that work across Claude Code and Cursor — with Codex and Opencode support releasing shortly.

---

## Table of Contents

- [Quick Start](#quick-start)
- [Concepts](#concepts)
- [CLI Reference](#cli-reference)
- [Package Authoring Guide](#package-authoring-guide)
- [The Package Manifest](#the-package-manifest)
- [Package Contents](#package-contents)
- [Publishing](#publishing)
- [Installing](#installing)
- [File Placement](#file-placement)
- [Lockfile](#lockfile)
- [Organization-Scoped Releases](#organization-scoped-releases)
- [Security and Integrity](#security-and-integrity)
- [For the Sierra Team: `bp-team`](#for-the-sierra-team-bp-team)
- [Troubleshooting](#troubleshooting)

---

## Quick Start

```bash
# Authenticate
blueplane login

# Browse available packages in your org
blueplane bpm list --remote

# Install a package
blueplane bpm install my-package

# Create your own
blueplane bpm init --name my-package --os unix
# ... add your skills, commands, hooks ...
blueplane bpm validate --dir .
blueplane bpm publish --dir .
```

---

## Concepts

BPM packages contain three types of content that extend AI coding assistants:

| Type | What It Is | Example |
|------|-----------|---------|
| **Skills** | Markdown-based guides and domain knowledge that AI assistants reference when working | Deployment procedures, architecture context, API documentation |
| **Commands** | Slash commands exposed in the assistant interface | `/deploy`, `/review`, `/test-plan` |
| **Hooks** | Shell scripts that run in response to platform events | Pre-commit validation, post-task cleanup |

Packages declare which **platforms** they support (Claude Code, Cursor) and which **operating systems** they target (Unix, Windows). BPM handles placing files into the correct platform-specific directories automatically.

---

## CLI Reference

### `blueplane bpm init`

Scaffold a new package directory.

```bash
blueplane bpm init --name my-package --os unix
blueplane bpm init --name my-package --os unix,windows
```

Creates:
```
my-package/
  bpm.yaml          # Package manifest
  skills/           # Skill markdown files
  commands/         # Command markdown files
  hooks/            # Hook shell scripts
```

**Flags:**

| Flag | Required | Description |
|------|----------|-------------|
| `--name` | Yes | Package name (lowercase, alphanumeric + hyphens, 2-64 chars) |
| `--os` | Yes | Target operating systems: `unix`, `windows`, or `unix,windows` |

### `blueplane bpm validate`

Validate a package directory against all manifest and content rules.

```bash
blueplane bpm validate --dir .
blueplane bpm validate --dir ./my-package
```

Checks:
- Manifest schema (name, version, platforms, OS, author email)
- Content declarations match actual files on disk
- Hook files exist for declared platforms (`.sh` for Unix, `.ps1` for Windows)
- No hidden files or symlinks

**Flags:**

| Flag | Required | Description |
|------|----------|-------------|
| `--dir` | No | Package directory (defaults to current directory) |

### `blueplane bpm publish`

Build an archive and publish to your organization's registry.

```bash
blueplane bpm publish --dir .
blueplane bpm publish --dir . --dry-run
blueplane bpm publish --dir . --force
```

Workflow:
1. Validates the package
2. Builds a `.tar.gz` archive
3. Computes SHA-256 checksums (archive-level and per-file)
4. Requests a presigned S3 upload URL from the cloud API
5. Uploads the archive directly to S3
6. Confirms publication — version transitions from `pending` to `published`

**Flags:**

| Flag | Required | Description |
|------|----------|-------------|
| `--dir` | No | Package directory (defaults to current directory) |
| `--dry-run` | No | Validate and build archive without uploading |
| `--force` | No | Overwrite an existing version |

### `blueplane bpm install`

Install a package from your org registry or a local file.

```bash
blueplane bpm install my-package
blueplane bpm install my-package@1.2.0
blueplane bpm install --file ./my-package.tar.gz --platforms claude
```

Workflow:
1. Resolves the package and version from the registry (or reads a local file)
2. Downloads the archive via presigned S3 URL
3. Verifies the SHA-256 checksum
4. Extracts to the local package cache (`~/.blueplane/packages/{name}/`)
5. Places files into platform-specific directories
6. Updates the lockfile (`~/.blueplane/bpm-lock.json`)

**Flags:**

| Flag | Required | Description |
|------|----------|-------------|
| `--platforms` | No | Target platforms to install for (e.g., `claude`, `cursor`) |
| `--file` | No | Install from a local `.tar.gz` file instead of the registry |

### `blueplane bpm uninstall`

Remove an installed package and all its placed files.

```bash
blueplane bpm uninstall my-package
```

Removes all skills, commands, and hooks placed by the package and cleans up the lockfile entry.

### `blueplane bpm list`

List packages — locally installed or available in your org registry.

```bash
blueplane bpm list                  # Locally installed packages
blueplane bpm list --remote         # All packages in your org registry
blueplane bpm list --json           # JSON output
blueplane bpm list --remote --json  # JSON output from registry
```

**Flags:**

| Flag | Description |
|------|-------------|
| `--remote` | Query the org registry instead of local installs |
| `--json` | Output as JSON |

### `blueplane bpm search`

Search your organization's package registry.

```bash
blueplane bpm search deploy
blueplane bpm search "test framework"
```

Searches package names and descriptions. Results are scoped to your organization.

### `blueplane bpm update`

Update installed packages to their latest published versions.

```bash
blueplane bpm update               # Update all packages
blueplane bpm update my-package    # Update a specific package
blueplane bpm update --dry-run     # Preview what would be updated
```

**Flags:**

| Flag | Description |
|------|-------------|
| `--dry-run` | Show what would be updated without making changes |

---

## Package Authoring Guide

### 1. Initialize

```bash
blueplane bpm init --name my-deploy-tools --os unix
cd my-deploy-tools
```

### 2. Edit the Manifest

Open `bpm.yaml` and fill in your package metadata:

```yaml
name: "my-deploy-tools"
version: "1.0.0"
description: "Production deployment workflows for the backend team"
author: "alice@yourcompany.com"

platforms:
  - claude
  - cursor

os:
  - unix

contents:
  skills:
    - name: "deploy-guide"
      description: "Step-by-step production deployment procedure"
    - name: "rollback-guide"
      description: "How to safely roll back a failed deploy"
  commands:
    - name: "deploy"
      description: "Run the deployment workflow"
  hooks:
    claude:
      - "pre-tool-use"
```

### 3. Add Content

**Skills** — Create a markdown file for each declared skill:

```
skills/
  deploy-guide/
    SKILL.md        # The skill content
  rollback-guide/
    SKILL.md
```

**Commands** — Create a markdown file for each declared command:

```
commands/
  deploy.md         # Slash command content
```

**Hooks** — Create shell scripts for each declared hook:

```
hooks/
  pre-tool-use.sh   # Unix hook (required when os includes unix)
  pre-tool-use.ps1  # Windows hook (required when os includes windows)
```

### 4. Validate

```bash
blueplane bpm validate --dir .
```

BPM reports all validation errors at once (not fail-fast), so you can fix everything in one pass.

### 5. Publish

```bash
# Preview first
blueplane bpm publish --dir . --dry-run

# Publish for real
blueplane bpm publish --dir .
```

Your package is now available to everyone in your organization.

---

## The Package Manifest

The `bpm.yaml` file is the heart of every package. Here is the full schema:

```yaml
# Required fields
name: "my-package"              # Package name
version: "1.0.0"                # Semantic version
description: "What it does"     # Human-readable description
author: "user@company.com"      # Author email (must match org domain)

# Required — at least one of each
platforms:                       # Supported AI coding platforms
  - claude                       #   Claude Code
  - cursor                       #   Cursor
os:                              # Supported operating systems
  - unix
  - windows

# Package contents
contents:
  skills:                        # List of skills
    - name: "skill-name"
      description: "Optional description"
  commands:                      # List of commands
    - name: "command-name"
      description: "Optional description"
  hooks:                         # Hooks keyed by platform
    claude:
      - "hook-name"
    cursor:
      - "hook-name"

# Optional
min_blueplane_version: "1.0.0"  # Minimum CLI version required
```

### Validation Rules

| Field | Rule |
|-------|------|
| `name` | Regex `^[a-z][a-z0-9-]{1,63}$` — 2-64 characters, starts with a lowercase letter, alphanumeric and hyphens only |
| `version` | Strict semantic versioning (`major.minor.patch`). A leading `v` prefix is stripped automatically. Prerelease and build metadata are allowed (e.g., `1.0.0-beta.1`) |
| `description` | Required, non-empty |
| `author` | Valid email address (RFC 5322). Domain must match your organization's domain at publish time |
| `platforms` | At least one. Valid values: `claude`, `cursor` |
| `os` | At least one. Valid values: `unix`, `windows` |
| `contents.hooks` | Hook platform keys must be present in the `platforms` list |
| `min_blueplane_version` | Optional. If present, must be valid semver |

---

## Package Contents

### Skills

Skills are markdown documents that provide domain knowledge to AI assistants. Each skill lives in its own directory under `skills/` and must contain a `SKILL.md` file.

```
skills/
  my-skill/
    SKILL.md            # Primary skill document (required)
    additional-docs.md  # Supporting files (optional)
```

When installed, the AI assistant can reference these skills during conversations to provide informed, context-aware assistance.

### Commands

Commands are markdown files that define slash commands users can invoke in their AI coding tool. Each command is a single `.md` file under `commands/`.

```
commands/
  deploy.md             # Becomes /deploy (or /package-name:deploy)
  test-plan.md          # Becomes /test-plan
```

### Hooks

Hooks are shell scripts that execute in response to platform events. Each declared hook needs a script file for each target OS.

```
hooks/
  pre-commit.sh         # Unix version
  pre-commit.ps1        # Windows version (if os includes windows)
```

Hook files must be executable. BPM selects the correct file extension (`.sh` or `.ps1`) based on the target platform at install time.

---

## Publishing

### Archive Format

When you publish, BPM builds a `.tar.gz` archive from your package directory with the following rules:

- **Hidden files and directories** (names starting with `.`) are excluded
- **Symlinks** are skipped — all entries must be regular files or directories
- **File permissions** are preserved in tar headers
- **Paths** are normalized to forward slashes, relative to the package root (no leading `./` or `/`)

### Version Lifecycle

Published versions go through a status lifecycle:

```
pending  →  published  →  yanked (optional)
```

- **Pending**: Version record created, archive uploaded to S3, not yet confirmed
- **Published**: Archive verified and confirmed — available for installation
- **Yanked**: Withdrawn from the registry — no longer installable

Only `published` versions appear in `list` and `search` results.

### Republishing

By default, publishing a version that already exists will fail. Use `--force` to overwrite an existing version. Use this with caution — consumers who have already installed the version will not be automatically updated.

---

## Installing

### From the Registry

```bash
blueplane bpm install my-package           # Latest version
blueplane bpm install my-package@1.2.0     # Specific version
```

### From a Local File

```bash
blueplane bpm install --file ./package.tar.gz --platforms claude
```

Local file installs skip the registry entirely — useful for testing packages before publishing.

### Checksum Verification

Every install verifies the archive's SHA-256 checksum against the value stored in the registry. If the checksum doesn't match, the install is aborted and no files are placed. Per-file hashes are also written to `hashes.json` in the package cache for future verification.

---

## File Placement

When a package is installed, BPM places files into platform-specific directories:

| Content Type | Placement Path |
|-------------|---------------|
| Skills | `~/.claude/skills/{package-name}/{skill-name}/` |
| Commands | `~/.claude/commands/{package-name}/{command-name}.md` |
| Hooks | `~/.claude/hooks/bpm/{package-name}/{hook-name}.sh` |

For Cursor, substitute `~/.cursor/` for `~/.claude/`.

The package cache (full extracted archive) is stored at:
```
~/.blueplane/packages/{package-name}/
```

Uninstalling removes both the placed files and the cache.

---

## Lockfile

BPM tracks installed packages in `~/.blueplane/bpm-lock.json`. This file records what is installed, when it was installed, and where files were placed.

```json
{
  "version": 1,
  "packages": {
    "my-package": {
      "version": "1.0.0",
      "installed_at": "2026-04-07T15:00:00Z",
      "checksum": "a1b2c3d4...",
      "platforms": ["claude"],
      "os": ["unix"],
      "files": [
        "~/.claude/skills/my-package/deploy-guide/SKILL.md",
        "~/.claude/commands/my-package/deploy.md",
        "~/.claude/hooks/bpm/my-package/pre-commit.sh"
      ]
    }
  }
}
```

The lockfile is written atomically (write to `.tmp`, then rename) to prevent corruption. Hook entries store base names without OS-specific extensions — the correct extension is resolved at runtime.

---

## Organization-Scoped Releases

Package registries are **scoped at a per-organization level**. Your organization is determined by your email domain — all users with the same domain share a registry.

- **Superfiliate packages and Sierra packages are not cross-accessible** at this time
- Each organization maintains its own independent, private registry
- Authentication is handled automatically via your Blueplane login (Google or Microsoft OAuth)
- The author email in your manifest must match your organization's domain

**Public releases are coming soon.** We are actively working on a shared registry that will allow packages to be published and discovered across organizations.

---

## Security and Integrity

BPM is designed with defense-in-depth:

- **SHA-256 checksums** computed at both the archive and per-file level
- **Checksum verification on every install** — any mismatch aborts the operation before files are placed
- **No hidden files or symlinks** in published archives — reduces attack surface
- **Presigned URLs** for direct S3 uploads and downloads — archives never pass through the API server
- **Path traversal protection** during extraction — archive entries cannot escape the target directory
- **Version lifecycle** — pending versions cannot be installed; only explicitly confirmed versions are available
- **Atomic lockfile writes** — prevents corruption from interrupted operations
- **Audit trail** — every install, uninstall, and update event is recorded in the database

---

## For the Sierra Team: `bp-team`

The **`bp-team`** package contains the complete set of skills and commands that were used to build the BPM product itself — including structured planning, TDD workflows, investigation frameworks, and red-green-refactor bug fixing. These are the same methodologies that powered BPM's own development, now packaged for your team.

```bash
blueplane bpm install bp-team
```

---

## Troubleshooting

### "package name invalid"

Package names must be 2-64 characters, start with a lowercase letter, and contain only lowercase letters, numbers, and hyphens. Regex: `^[a-z][a-z0-9-]{1,63}$`.

### "author domain does not match organization"

The email in your `bpm.yaml` author field must use your organization's domain. If you're `alice@acme.com`, your org is `acme.com`, and the author field must be an `@acme.com` address.

### "checksum mismatch"

The downloaded archive doesn't match the checksum stored in the registry. This could indicate a corrupted download or a tampered archive. Try installing again — if the error persists, contact your Blueplane administrator.

### "version already exists"

You're trying to publish a version that's already been published. Either bump your version in `bpm.yaml` or use `--force` to overwrite.

### Validate reports multiple errors

This is by design. BPM collects all validation errors in a single pass so you can fix everything at once rather than playing whack-a-mole.

### Hooks not triggering

Ensure hook files are executable (`chmod +x hooks/my-hook.sh`) and that the hook platform key in your manifest matches your declared platforms.
