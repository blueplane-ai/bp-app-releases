# Blueplane Desktop App

## Installation and Launch

1. Download and run the setup script:
   ```bash
   curl -fsSL https://github.com/blueplane-ai/bp-app-releases/releases/latest/download/blueplane-setup.sh | bash
   ```
   This installs the Blueplane app for Claude and Cursor automatically. For Claude Code, Conductor, and Cursor, no additional steps are required.

2. Add Blueplane to your PATH:
   ```bash
   export PATH="$HOME/.blueplane/bin:$PATH"
   ```

3. Configure OpenCode (optional):
   In your `opencode.json`, add the plugin to the `plugin` array:
   ```json
   "@blueplane/opencode-capture@latest"
   ```
   The plugin is automatically downloaded and run when a new session is created or the desktop app is reloaded.

4. Log in:
   ```bash
   blueplane login
   ```
   Authenticate using your Google Workspace email address.

5. Start the daemon:
   ```bash
   blueplane start
   ```

6. Update the app:
   ```bash
   blueplane upgrade
   ```

## Directory Allowlist & Ignorelist — CLI Guide

Control which project directories have their telemetry synced to the cloud using `blueplane allowlist`.

---

### Quick Start

```bash
# Only sync telemetry from specific projects
blueplane allowlist --add ~/projects/myapp
blueplane allowlist --add ~/projects/research

# Never sync telemetry from a sensitive subdirectory
blueplane allowlist --ignore ~/projects/myapp/secrets

# Apply changes
blueplane restart
```

---

### Commands

#### View current configuration

```bash
blueplane allowlist
```

When no filters are configured, outputs:

```
No directory filters configured.
All directories are synced.
```

When filters exist, outputs:

```
Allowed directories:
  /Users/alice/projects/myapp
  /Users/alice/projects/research

Ignored directories:
  /Users/alice/projects/myapp/secrets
```

#### Add directories to the allowlist

```bash
# Add a single directory
blueplane allowlist --add /Users/alice/projects/myapp

# Add multiple directories
blueplane allowlist --add ~/projects/myapp --add ~/projects/research

# Use relative paths (resolved to absolute automatically)
cd ~/projects/myapp
blueplane allowlist --add .
```

#### Remove directories from the allowlist

```bash
blueplane allowlist --remove /Users/alice/projects/myapp
```

#### Clear the entire allowlist

```bash
blueplane allowlist --clear-allowed
```

#### Add directories to the ignorelist

```bash
# Ignore a single directory
blueplane allowlist --ignore ~/projects/private

# Ignore multiple directories
blueplane allowlist --ignore ~/projects/private --ignore ~/.secrets
```

#### Remove directories from the ignorelist

```bash
blueplane allowlist --unignore ~/projects/private
```

#### Clear the entire ignorelist

```bash
blueplane allowlist --clear-ignored
```

---

### Flag Reference

| Flag | Description |
|---|---|
| `--add <path>` | Add directory to allowlist (repeatable) |
| `--remove <path>` | Remove directory from allowlist (repeatable) |
| `--ignore <path>` | Add directory to ignorelist (repeatable) |
| `--unignore <path>` | Remove directory from ignorelist (repeatable) |
| `--clear-allowed` | Clear all allowed directories |
| `--clear-ignored` | Clear all ignored directories |

All changes require `blueplane restart` to take effect.

---

### Editing the config directly

The CLI writes to `~/.blueplane/agent.yaml`. The two relevant keys are `allowed_directories` and `ignored_directories`:

```yaml
# ~/.blueplane/agent.yaml (other fields omitted)
allowed_directories:
  - /Users/alice/projects/myapp
  - /Users/alice/projects/research
ignored_directories:
  - /Users/alice/projects/private
```

If you edit the file by hand, restart the daemon with `blueplane restart`.
