# Nodefusion ITOps Dev — Codespaces Reference

## GitHub Repository Structure

## How It Works — Automatic Setup

### postCreateCommand
Runs **once** on first Codespace creation. Takes 5–10 minutes.
- Installs Claude CLI
- Generates HTTPS dev certificate
- Installs Playwright CLI and Chromium browser

### postStartCommand
Runs on **every start**. Takes 2–5 minutes.
- Waits for ADO auth helper to be ready
- Clones ADO repositories into /workspaces/
- Copies secrets.json into projects
- Runs dotnet restore for all projects
- Copies Claude permissions and crun script
- Sets up crun alias

Repositories cloned automatically:
- NodefusionITOps — ITOps project
- Aexum — Aexum project
- Mock.IDP — Mock IDP for testing
- Agents.Skills — Claude config and scripts

---

## Source Control — Branch, Commit, Push

Each project is a separate git repository. Use the VS Code Source Control panel in the left sidebar (branch icon).
- **Change branch** — click the branch name in the bottom-left status bar or in the Source Control panel
- **Commit** — enter commit message → click Commit
- **Push** — click ... → Push
- **Create new branch** — click branch name in status bar → Create new branch

Switch branch before starting work:
```
cd /workspaces/NodefusionITOps
git checkout develop
# or new branch
git checkout -b feature/my-feature
```

- Check the active branch in the bottom-left status bar, an asterisk (develop\*) means there are uncommitted changes
- Verify that appsettings.Development.json on your branch contains correct values before running the project

---

## Run Projects

### From Terminal

NodefusionITOps:
```
cd /workspaces/NodefusionITOps/src/Nodefusion.NodefusionITOps
dotnet run
```

Aexum WebApi:
```
cd /workspaces/Aexum/src/Nodefusion.Aexum.WebApi
dotnet run
```

Mock.IDP:
```
cd /workspaces/Nodefusion.Mock.Idp/src/Nodefusion.Mock.Idp
dotnet run
```

Verify Mock.IDP is running:
```
curl -k https://localhost:5099/.well-known/openid-configuration
```

### Playwright Tests
```
cd /workspaces/NodefusionITOps
npx playwright test
# or if tests are in .NET project
dotnet test --filter "Category=Playwright"
```
- Start Mock.IDP in a separate terminal before running Playwright tests if the tests depend on it

### Ports

View forwarded ports in the **Ports panel** in the bottom right. All listed ports are public:
- 5114 — ITOps HTTP
- 7246 — ITOps HTTPS
- 44377 — Aexum WebAssembly
- 44363 — Aexum WebApi HTTPS
- 5246 — Aexum WebApi HTTP
- 5099 — Mock.IDP
- The port used for login (e.g. 7246 or 44363) must be added as a redirect URI in the ADO/Entra app registration. Contact admin on first setup

---

## Claude CLI — Permissions

- Claude is configured with global permissions allowing all actions without confirmation prompts
- Configuration is in `~/.claude/settings.json`
- Source is `Nodefusion.Agents.Skills/config/claude/settings.json`

---

## crun — How It Works

`crun` is an alias for `/workspaces/claude-run.sh`. It runs Claude in autonomous mode and automatically shuts down the Codespace when Claude finishes the task or on timeout.

Timeout rules:
- **16 h** — hard limit, shuts down regardless
- **45 min** — inactivity timeout, no output

---

## Troubleshooting

### Terminal fails to launch
- **Symptom**: The terminal process failed to launch: Starting directory (cwd) does not exist
- **Cause**: ADO repositories not yet cloned when the terminal tried to open
- **Fix**:
  * Ctrl+Shift+P → Terminal: Create New Terminal in Editor Area
  * Manually run `external-git clone && external-git config`

### Recovery Mode
- **Symptom**: screen shows "Your codespace is in recovery mode"
- **Cause**: postStartCommand or postCreateCommand failed
- **Fix**:
  1. Click View Creation Log and check the last error
  2. Open terminal in recovery mode
  3. Contact admin with the log output

### Run and Debug fails — Project file does not exist
- **Symptom**: MSBUILD : error MSB1009: Project file does not exist
- **Cause**: launch.json or tasks.json has an incorrect path, missing `src/`
- **Fix**: verify `.vscode/launch.json` and `.vscode/tasks.json` use `${workspaceFolder}/src/...` in all paths

### crun not available
```
source ~/.bashrc
# or directly
/workspaces/claude-run.sh "your task"
```

### dotnet not available
```
export PATH=$PATH:/usr/share/dotnet
echo 'export PATH=$PATH:/usr/share/dotnet' >> ~/.bashrc
source ~/.bashrc
```

### Claude permissions prompt appearing
```
# verify settings.json is in place
cat ~/.claude/settings.json
# if missing or empty
cp /workspaces/Nodefusion.Agents.Skills/config/claude/settings.json ~/.claude/settings.json
```

### Project loads wrong config (e.g. prod clientId)
- **Symptom**: project runs from develop branch but uses production values
- **Cause**: appsettings.Development.json not committed on that branch or missing values
- **Fix**:
  1. Check active branch in the bottom-left status bar
  2. Verify appsettings.Development.json content in the project
  3. If development values are missing, add and commit appsettings.Development.json on the branch

### Login fails on first Codespace creation
- **Cause**: Codespace redirect URI not registered in the ADO app registration
- **Fix**: Contact admin — the redirect URI needs to be added

### Rebuild Codespace
- Required only when devcontainer.json changes
- Ctrl+Shift+P → **Codespaces: Rebuild Container**
- Rebuild takes 10–15 minutes, filesystem resets but all committed code remains

---

## Notes

- **Billing** — charged only while the Codespace is running; a stopped Codespace incurs storage cost only
- **Inactivity timeout** — GitHub automatically stops the Codespace after 30 minutes of inactivity, per organization settings
- **Rebuild vs Stop/Start** — rebuild is only needed when devcontainer.json changes; normal Stop → Start preserves all installed tools and ADO login
- **Secrets** — never commit secrets.json to git; it is automatically generated from repository Codespace secrets on every start
- **Branch awareness** — Codespace loads main by default for all repositories; always verify the active branch in the status bar before starting work
- **ASPNETCORE\_ENVIRONMENT** — automatically set to Development for all processes in the Codespace; no manual configuration needed
- **Mock.IDP** — internal only, accessible only within the container, no external exposure needed
