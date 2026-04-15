# Blueplane Package Manager (BPM)

The **Blueplane Package Manager** works like an internal app store for AI coding tool extensions. It lets organizations create custom skills, commands, and hooks for their AI tools, package them up, and distribute them to team members.

---

## Why BPM Exists

Without BPM, sharing custom AI tool configurations across a team requires manual file copying, inconsistent setups, and no way to track versions. BPM solves this by providing a standard way to publish, discover, install, and update packages.

---

## What a Package Contains

A BPM package is a bundle that can include:

- **Skills** -- reusable capabilities that extend what Claude Code or Cursor can do (for example, a skill that knows how to query your company's internal APIs).
- **Commands** -- slash commands that developers can invoke inside their AI tools.
- **Hooks** -- scripts that run automatically in response to events (for example, logging every time an AI tool edits a file).

Each package includes a manifest file that describes its name, version, author, and which platforms it supports.

---

## How It Works

**Scaffolding** -- Running `bpm init` creates a new package directory with a template manifest, empty folders for skills, commands, and hooks, and a starter README. This gives package authors a ready-made structure to begin building.

**Publishing** -- A package author creates a directory with their skills, commands, and hooks, along with a manifest file. Running `bpm publish` validates the package, bundles it into an archive, and uploads it to the organization's private registry in the cloud.

**Discovering** -- Team members can browse available packages with `bpm list --remote` or search by keyword with `bpm search`. Each organization has its own isolated registry, so packages are only visible within the organization.

**Installing** -- Running `bpm install <package-name>` downloads the package, places files in the correct locations for each AI tool, and registers any hooks. The developer's tools immediately pick up the new capabilities.

**Updating and Removing** -- `bpm update` pulls the latest versions of installed packages. `bpm uninstall` cleanly removes a package and its hook registrations.

---

## Key Design Choices

- **Organization isolation** -- Each organization has its own package registry. Packages from one organization are never visible to another.
- **Integrity verification** -- Every package is checksummed so that what gets installed is exactly what was published.
- **Cross-platform** -- Packages can include scripts for both macOS/Linux and Windows, and the correct version is selected automatically at install time.
- **Offline support** -- Packages can be installed from a local file for environments without internet access.
