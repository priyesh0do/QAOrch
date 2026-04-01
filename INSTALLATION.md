# Installation Guide

Complete step-by-step installation guide for the QA Test Orchestrator.

## Table of Contents

- [System Requirements](#system-requirements)
- [Quick Installation](#quick-installation)
- [Detailed Installation](#detailed-installation)
- [MCP Server Setup](#mcp-server-setup)
- [Verification](#verification)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)

---

## System Requirements

### Minimum Requirements

- **Node.js**: 18.0.0 or higher
- **RAM**: 4GB
- **Disk Space**: 500MB for installation + space for generated artifacts
- **OS**: Windows 10/11, macOS 12+, or Linux (Ubuntu 20.04+)

### Recommended Requirements

- **Node.js**: 20.0.0 or higher
- **RAM**: 8GB or more
- **Disk Space**: 2GB
- **OS**: Windows 11, macOS 13+, or Ubuntu 22.04+

### Peer Dependencies

- **@playwright/test**: ^1.40.0 (optional, for running generated tests)

---

## Quick Installation

### For Linux/Mac

```bash
# Clone the repository
git clone https://github.com/your-org/qa-test-orchestrator.git
cd qa-test-orchestrator

# Run installation script
bash scripts/install.sh

# Configure your project
# Edit orchestrator-config.json with your paths

# Verify installation
npm run verify
```

### For Windows

```powershell
# Clone the repository
git clone https://github.com/your-org/qa-test-orchestrator.git
cd qa-test-orchestrator

# Run installation script
powershell -ExecutionPolicy Bypass -File scripts/install.ps1

# Configure your project
# Edit orchestrator-config.json with your paths

# Verify installation
npm run verify
```

---

## Detailed Installation

### Step 1: Check Node.js Version

Ensure you have Node.js 18.0.0 or higher installed:

```bash
node --version
```

If Node.js is not installed or version is too old:

**Windows/Mac**:
- Download from [https://nodejs.org](https://nodejs.org)
- Install the LTS version

**Linux (Ubuntu/Debian)**:
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### Step 2: Clone the Repository

```bash
git clone https://github.com/your-org/qa-test-orchestrator.git
cd qa-test-orchestrator
```

### Step 3: Install Dependencies

```bash
npm install
```

This installs:
- `commander` - CLI framework

No other runtime dependencies are required! The orchestrator uses only Node.js built-in modules.

### Step 4: Create Required Directories

The installation scripts create these directories automatically. If installing manually:

```bash
mkdir -p .agent-state
mkdir -p logs
mkdir -p test-designs
mkdir -p pages
mkdir -p tests/functional
mkdir -p tests/integration
mkdir -p tests/api
mkdir -p tests/e2e
```

### Step 5: Configure the Orchestrator

Copy and edit the configuration file:

```bash
cp orchestrator-config.json orchestrator-config.local.json
```

Edit `orchestrator-config.json` with your project paths:

```json
{
  "version": "3.0.0",
  "architecture": "agent-driven",
  "projectPath": "/path/to/your/test-project",
  "requirementPath": "/path/to/your/requirements",
  "frontendProjectPath": "/path/to/your/frontend",
  "referenceProjectPath": "/path/to/reference/project",
  "baseUrl": "http://localhost:5173",
  "framework": "Playwright",
  "language": "JavaScript"
}
```

**Path Configuration Tips**:
- Use absolute paths for clarity
- Windows users: Use forward slashes `/` or escaped backslashes `\\\\`
- Verify paths exist before running agents

---

## MCP Server Setup

The orchestrator requires MCP (Model Context Protocol) servers to function optimally.

### Required MCP Servers

1. **MDS MCP** - MDS Design System documentation
2. **Platform UI MCP** - Platform UI Shell Helpers documentation

### Optional MCP Servers

3. **Chrome DevTools MCP** - DOM inspection for locator healing

### Setup Instructions

#### Automated Setup (Linux/Mac)

```bash
bash scripts/setup-mcp.sh
```

#### Manual Setup

1. **Open Claude Code Settings**
   - Launch Claude Code
   - Open Settings/Preferences
   - Navigate to MCP Servers section

2. **Add MDS MCP Server**
   ```json
   {
     "name": "mds-mcp",
     "command": "npx",
     "args": ["-y", "@your-org/mds-mcp"]
   }
   ```

3. **Add Platform UI MCP Server**
   ```json
   {
     "name": "platform-ui",
     "command": "npx",
     "args": ["-y", "@your-org/platform-ui-mcp"]
   }
   ```

4. **Restart Claude Code**

5. **Verify MCP Connection**
   In Claude Code, try:
   ```
   List all MDS components
   ```

   If successful, you should see a list of MDS components.

For detailed MCP setup, see [docs/guides/mcp-integration.md](docs/guides/mcp-integration.md).

---

## Verification

### Run Verification Script

```bash
npm run verify
```

Expected output:

```
╔════════════════════════════════════════════════════════════╗
║         QA Test Orchestrator - Verification                ║
╚════════════════════════════════════════════════════════════╝

[1/10] Checking project structure...
  ✓ agents/ directory exists
  ✓ agents/base/ directory exists
  ✓ agents/specialized/ directory exists
  ...

[2/10] Checking configuration file...
  ✓ orchestrator-config.json exists
  ✓ Version is 3.0.0
  ✓ Architecture is agent-driven
  ...

[3/10] Checking agent registration...
  ✓ FunctionalTestAutomationAgent registered
  ✓ IntegrationTestAgent registered
  ✓ APITestAgent registered
  ...

[10/10] Summary
  ✅ ALL CHECKS PASSED (46/46)

The agent-driven architecture is fully implemented and ready for use.
```

### Manual Verification

Test each component individually:

#### 1. Verify Agent System

```bash
node -e "const agents = require('./agents'); console.log(Object.keys(agents));"
```

Expected output:
```javascript
[
  'AgentOrchestrator',
  'StateManager',
  'Agent',
  'ApprovalUI',
  'FunctionalTestAutomationAgent',
  'IntegrationTestAgent',
  'APITestAgent',
  'LocatorHealingAgent',
  'EndToEndTestAgent',
  'ProjectSetupAgent'
]
```

#### 2. Verify Configuration

```bash
npx qa-orchestrator config
```

Should display your configuration without errors.

#### 3. Verify CLI

```bash
npx qa-orchestrator list-agents
```

Should list all 6 agents with their status.

---

## Configuration

### Essential Configuration

Minimum required configuration:

```json
{
  "version": "3.0.0",
  "architecture": "agent-driven",
  "projectPath": "./test-project",
  "agents": {
    "enabled": true
  }
}
```

### Recommended Configuration

For best results:

```json
{
  "version": "3.0.0",
  "architecture": "agent-driven",
  "projectPath": "./test-project",
  "requirementPath": "./requirements",
  "frontendProjectPath": "./frontend",
  "referenceProjectPath": "./reference-tests",
  "baseUrl": "http://localhost:5173",
  "framework": "Playwright",
  "language": "JavaScript",
  "agents": {
    "enabled": true,
    "parallelExecution": true,
    "maxConcurrentAgents": 3
  },
  "mcpServers": {
    "mds": {
      "enabled": true,
      "required": true
    },
    "platformUI": {
      "enabled": true,
      "required": true
    }
  }
}
```

See [docs/guides/configuration.md](docs/guides/configuration.md) for complete configuration reference.

---

## Troubleshooting

### Issue: npm install fails

**Error**: `npm ERR! code EACCES`

**Solution** (Linux/Mac):
```bash
sudo chown -R $USER:$USER ~/.npm
npm install
```

**Solution** (Windows):
Run PowerShell as Administrator.

---

### Issue: Node.js version too old

**Error**: `Node.js version 16.x.x is too old`

**Solution**:
```bash
# Linux/Mac
nvm install 20
nvm use 20

# Windows
# Download and install from nodejs.org
```

---

### Issue: Verification fails

**Error**: `❌ Configuration file not found`

**Solution**:
Ensure `orchestrator-config.json` exists in the project root:
```bash
cp orchestrator-config.example.json orchestrator-config.json
```

---

### Issue: MCP servers not connected

**Error**: `⚠ MCP server 'mds-mcp' not available`

**Solution**:
1. Check Claude Code MCP configuration
2. Run `bash scripts/setup-mcp.sh`
3. Restart Claude Code
4. Verify connection: `List all MDS components`

---

### Issue: Permission denied (Linux/Mac)

**Error**: `bash: scripts/install.sh: Permission denied`

**Solution**:
```bash
chmod +x scripts/install.sh
bash scripts/install.sh
```

---

### Issue: PowerShell execution policy (Windows)

**Error**: `Cannot be loaded because running scripts is disabled`

**Solution**:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

---

## Post-Installation

### Next Steps

1. **Configure Your Project**
   - Edit `orchestrator-config.json`
   - Set correct paths
   - Enable required agents

2. **Prepare Requirements**
   - Create requirements with REQ-XXX format
   - Add acceptance criteria (AC-XXX)
   - Store in `requirementPath` directory

3. **Set Up Reference Project** (Optional)
   - Point `referenceProjectPath` to existing tests
   - Agents will analyze patterns and naming conventions

4. **Run Your First Agent**
   ```bash
   npx qa-orchestrator start functional --module YourModule
   ```

   Or with Claude Code:
   ```
   Start FunctionalTestAutomationAgent for module YourModule
   ```

5. **Read Documentation**
   - [Getting Started Guide](docs/getting-started.md)
   - [Agent Architecture](docs/architecture.md)
   - [Configuration Guide](docs/guides/configuration.md)

---

## Getting Help

- **Documentation**: [docs/](docs/)
- **Examples**: [examples/](examples/)
- **Issues**: [GitHub Issues](https://github.com/your-org/qa-test-orchestrator/issues)
- **Discussions**: [GitHub Discussions](https://github.com/your-org/qa-test-orchestrator/discussions)

---

## Upgrading

### From v2.0 to v3.0

See [Migration Guide](docs/migration/v2-to-v3.md) for detailed upgrade instructions.

Quick steps:
1. Update `orchestrator-config.json`:
   - Change `version` to `"3.0.0"`
   - Change `architecture` to `"agent-driven"`
   - Set `workflows.enabled` to `false`
   - Add `agents` section

2. Run verification:
   ```bash
   npm run verify
   ```

3. Update commands:
   - Old: "Run the functional testing workflow"
   - New: "Start FunctionalTestAutomationAgent"

---

## Uninstallation

To remove the orchestrator:

```bash
# Remove node_modules
rm -rf node_modules

# Remove generated artifacts (optional)
rm -rf .agent-state test-designs pages tests

# Remove the repository
cd ..
rm -rf qa-test-orchestrator
```

---

**Installation Complete! 🎉**

You're now ready to use the QA Test Orchestrator. Start with the [Getting Started Guide](docs/getting-started.md).
