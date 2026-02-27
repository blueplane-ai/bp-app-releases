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
   Authenticate using your `@superfiliate.com` email address.

5. Start the daemon:
   ```bash
   blueplane start
   ```

6. Update the app:
   ```bash
   blueplane upgrade
   ```
