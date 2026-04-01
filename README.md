# QA Test Orchestrator

> **Claude Code plugin with 6 specialized agents for intelligent Playwright test generation**

[![Version](https://img.shields.io/badge/version-3.0.0-blue.svg)](https://github.com/your-org/qa-test-orchestrator/releases)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/claude--code-plugin-purple.svg)](https://claude.ai/claude-code)
[![Playwright](https://img.shields.io/badge/playwright-1.40+-orange.svg)](https://playwright.dev)

---

## ⚡ What is QA Test Orchestrator?

A **Claude Code plugin** that provides 6 specialized AI agents to automatically generate comprehensive Playwright test suites with full traceability, intelligent locator healing, and MDS design system integration.

### 🎯 Key Features

- 🤖 **6 Specialized Agents** - Each expert in their testing domain
- 📊 **Full Traceability** - Automatic REQ → AC → TC mapping
- 🔍 **Smart Locator Healing** - Auto-fix broken selectors with 5 alternatives
- 🛡️ **User-Controlled** - Low autonomy with 4 approval checkpoints
- 🔗 **MCP Integration** - Leverages MDS Design System & Platform UI
- 🎯 **Pure Markdown** - No dependencies, no runtime, just instructions for Claude

---

## 📦 Installation

### Quick Install

In Claude Code:
```
# Step 1: Create the plugins directory
  New-Item -Path "$env:USERPROFILE\.claude-code\plugins" -ItemType Directory -Force

# Step 2: Now create the symlink
  New-Item -ItemType SymbolicLink `-Path "$env:USERPROFILE\.claude-code\plugins\qa-test-orchestrator" `-Target "C:\Working\test_orchestrator_workflow_v3"
 
  # Step 3: Verify it worked
  Test-Path "$env:USERPROFILE\.claude-code\plugins\qa-test-orchestrator\.claude-plugin\plugin.json"
 
  If you still get errors, try this alternative:
 
  # Check if .claude-code directory exists at all
  Test-Path "$env:USERPROFILE\.claude-code"
 
  # If False, create the entire structure
  New-Item -Path "$env:USERPROFILE\.claude-code" -ItemType Directory -Force
  New-Item -Path "$env:USERPROFILE\.claude-code\plugins" -ItemType Directory -Force
 
  # Then create symlink
  New-Item -ItemType SymbolicLink `
    -Path "$env:USERPROFILE\.claude-code\plugins\qa-test-orchestrator" `
    -Target "C:\Working\test_orchestrator_workflow_v3"
 
  If symlinks still don't work (some Windows versions have restrictions), use the copy method instead:
 
  # Create directory structure
  New-Item -Path "$env:USERPROFILE\.claude-code\plugins" -ItemType Directory -Force
 
  # Copy the entire plugin
  Copy-Item -Path "C:\Working\test_orchestrator_workflow_v3" `
    -Destination "$env:USERPROFILE\.claude-code\plugins\qa-test-orchestrator" `
    -Recurse
 
  # Verify
  Get-ChildItem "$env:USERPROFILE\.claude-code\plugins\qa-test-orchestrator"
 
  Note: With the copy method, you'll need to re-copy whenever you make changes to your plugin. With symlink, changes are immediately
  reflected.
 
  After either method, restart Claude Code completely and try:
  Start FunctionalTestAutomationAgent for module TestModule
```

That's it! The plugin is now available.

### Manual Install (Development)

```bash
# Clone the repository
git clone https://github.com/your-org/qa-test-orchestrator.git
cd qa-test-orchestrator

# Symlink to Claude Code plugins directory
ln -s $(pwd) ~/.claude-code/plugins/qa-test-orchestrator

# Restart Claude Code
```

---

## 🚀 Quick Start

### 1. Create Configuration

Create `orchestrator-config.json` in your project:

```json
{
  "version": "3.0.0",
  "architecture": "agent-driven",
  "projectPath": "./test-project",
  "requirementPath": "./requirements",
  "frontendProjectPath": "./frontend",
  "baseUrl": "http://localhost:5173",
  "framework": "Playwright",
  "language": "JavaScript"
}
```

### 2. Prepare Requirements

Create a requirements document at `requirements/YourModule.md`:

```markdown
## REQ-001: Add New Record
Users shall be able to add new records.

### AC-001: Display Add Button
The system shall display an "Add" button.

### AC-002: Open Form
When clicked, a form shall open with required fields.
```

### 3. Generate Tests

In Claude Code:
```
Start FunctionalTestAutomationAgent for module YourModule
```

Or use the slash command:
```
/start-functional-tests YourModule
```

### 4. Review and Approve

The agent will guide you through 4 phases:
1. **Analysis** - Review parsed requirements → [A]pprove
2. **Design** - Review traceability matrix → [A]pprove
3. **Generation** - Review generated files → [A]pprove
4. **Validation** - Review coverage report → [A]pprove

### 5. Generated Artifacts

```
test-designs/YourModule/functional-design.md
test-designs/YourModule/traceability-matrix.md
pages/YourModule/*.page.js
tests/functional/YourModule/*.spec.js
```

---

## 🤖 The 6 Agents

### 1. FunctionalTestAutomationAgent
**Generate functional tests with full traceability**

**Command**: `/start-functional-tests [module]`

**Generates**:
- Test design documents
- Traceability matrices (REQ → AC → TC)
- MDS-aware page objects
- Playwright test specifications

[View Agent Documentation](agents/functional-test-agent.md)

---

### 2. IntegrationTestAgent
**Generate integration tests with database validation**

**Command**: `/start-integration-tests [module]`

**Generates**:
- Integration test scenarios
- Data flow diagrams
- Database validation tests

**Requires**: FunctionalTestAutomationAgent

[View Agent Documentation](agents/integration-test-agent.md)

---

### 3. APITestAgent
**Generate API tests from OpenAPI/Swagger specs**

**Command**: `/start-api-tests [module]`

**Generates**:
- API test specifications
- Schema validators
- API client utilities

[View Agent Documentation](agents/api-test-agent.md)

---

### 4. LocatorHealingAgent
**Auto-heal broken locators**

**Command**: `/heal-locators [module]`

**Generates**:
- Up to 5 alternative locators per failure
- Updated page objects
- Healing reports with recommendations

[View Agent Documentation](agents/locator-healing-agent.md)

---

### 5. EndToEndTestAgent
**Generate E2E tests for complete user journeys**

**Command**: `/start-e2e-tests [module]`

**Generates**:
- End-to-end test scenarios
- User journey maps
- Navigation flow tests

**Requires**: FunctionalTestAutomationAgent, IntegrationTestAgent

[View Agent Documentation](agents/e2e-test-agent.md)

---

### 6. ProjectSetupAgent
**Initialize project structure**

**Command**: `/setup-project`

**Generates**:
- Project directory structure
- rules.md document
- Base classes and configurations

[View Agent Documentation](agents/project-setup-agent.md)

---

## 📚 Available Commands

| Command | Description |
|---------|-------------|
| `/start-functional-tests [module]` | Generate functional tests |
| `/start-integration-tests [module]` | Generate integration tests |
| `/start-api-tests [module]` | Generate API tests |
| `/heal-locators [module]` | Heal broken locators |
| `/start-e2e-tests [module]` | Generate E2E tests |
| `/setup-project` | Initialize project structure |
| `/verify` | Verify configuration |

You can also just tell Claude naturally:
```
Start FunctionalTestAutomationAgent for module AbsenceCode
Heal broken locators in the UserManagement module
Verify my orchestrator configuration
```

---

## ⚙️ Configuration

### Required Settings

```json
{
  "version": "3.0.0",
  "architecture": "agent-driven",
  "projectPath": "./test-project",
  "requirementPath": "./requirements",
  "baseUrl": "http://localhost:5173"
}
```

### Optional Settings

```json
{
  "frontendProjectPath": "./frontend",
  "referenceProjectPath": "./reference-tests",
  "framework": "Playwright",
  "language": "JavaScript"
}
```

[View Complete Configuration Reference](docs/guides/configuration.md)

---

## 🔗 MCP Server Setup

The orchestrator requires MCP servers for enhanced capabilities:

### Required MCP Servers

1. **MDS MCP** - MDS Design System documentation
   ```json
   {
     "name": "mds-mcp",
     "command": "npx -y @your-org/mds-mcp"
   }
   ```

2. **Platform UI MCP** - Platform UI Shell Helpers
   ```json
   {
     "name": "platform-ui",
     "command": "npx -y @your-org/platform-ui-mcp"
   }
   ```

### Optional MCP Servers

3. **Chrome DevTools MCP** - DOM inspection for locator healing

[View Complete MCP Setup Guide](docs/guides/mcp-integration.md)

---

## 💡 Usage Examples

### Example 1: Generate Complete Test Suite

```
Start FunctionalTestAutomationAgent, IntegrationTestAgent, and APITestAgent for module PayrollCalculation
```

Claude will run all 3 agents, respecting dependencies, and generate complete test coverage.

---

### Example 2: Heal Broken Tests

When tests fail with locator errors:

```
Heal broken locators in the Dashboard module
```

The LocatorHealingAgent will:
1. Analyze failed locators
2. Generate 5 alternative strategies
3. Update page objects with the best option
4. Create backups before modifying

---

### Example 3: New Project Setup

```
Setup a new test project
```

The ProjectSetupAgent will:
1. Create directory structure
2. Generate base classes
3. Create rules.md with standards
4. Set up Playwright configuration

---

## 🛡️ Low Autonomy Model

Every agent requires user approval at **4 checkpoints**:

```
1. Analysis Phase    → [A]pprove to continue
2. Design Phase      → [A]pprove to continue
3. Generation Phase  → [A]pprove to continue
4. Validation Phase  → [A]pprove to complete
```

**At each checkpoint, you can**:
- **[A]pprove** - Continue to next phase
- **[M]odify** - Provide corrections and retry
- **[R]eject** - Stop execution

This ensures full transparency and control over all generated artifacts.

---

## 📖 Documentation

### Core Guides

| Guide | Description |
|-------|-------------|
| [Getting Started](docs/getting-started.md) | 10-minute quick start |
| [Architecture](docs/architecture.md) | Agent architecture overview |
| [Configuration](docs/guides/configuration.md) | Complete config reference |
| [MCP Integration](docs/guides/mcp-integration.md) | MCP server setup |

### Agent Reference

Each agent has comprehensive documentation:
- [FunctionalTestAutomationAgent](agents/functional-test-agent.md)
- [IntegrationTestAgent](agents/integration-test-agent.md)
- [APITestAgent](agents/api-test-agent.md)
- [LocatorHealingAgent](agents/locator-healing-agent.md)
- [EndToEndTestAgent](agents/e2e-test-agent.md)
- [ProjectSetupAgent](agents/project-setup-agent.md)

### Usage Examples

Detailed walkthroughs:
- [Basic Functional Test Generation](examples/basic-functional-test.md)
- [Parallel Execution](examples/parallel-execution.md)

---

## 🏗️ Plugin Architecture

```
qa-test-orchestrator/
├── .claude-plugin/
│   └── plugin.json              # Plugin metadata
├── agents/
│   ├── functional-test-agent.md # Agent definitions (markdown)
│   ├── integration-test-agent.md
│   ├── api-test-agent.md
│   ├── locator-healing-agent.md
│   ├── e2e-test-agent.md
│   └── project-setup-agent.md
├── commands/
│   ├── start-functional.md      # Command definitions
│   ├── heal-locators.md
│   └── verify.md
├── skills/
│   ├── playwright-testing/      # Reusable skills
│   ├── traceability-mapping/
│   └── mds-locators/
├── docs/                        # Documentation
├── examples/                    # Usage examples
└── README.md                    # This file
```

### Pure Markdown

This plugin uses **no JavaScript, no dependencies**. Everything is markdown-based instructions for Claude Code.

---

## 🤝 Contributing

We welcome contributions! See our [Contributing Guide](.github/CONTRIBUTING.md) for details.

### Quick Contribution Steps

1. Fork the repository
2. Create a feature branch
3. Make your changes (edit markdown files)
4. Submit a pull request

**No build step, no dependencies, just edit markdown!**

---

## 🐛 Troubleshooting

### Issue: Plugin not loading

**Solution**:
1. Check plugin is in `~/.claude-code/plugins/`
2. Restart Claude Code
3. Verify `.claude-plugin/plugin.json` is valid JSON

### Issue: Agent not starting

**Solution**:
1. Run `/verify` to check configuration
2. Ensure `orchestrator-config.json` exists
3. Check all paths are valid

### Issue: MCP servers not connected

**Solution**:
1. Open Claude Code settings → MCP Servers
2. Add required MCP servers
3. Restart Claude Code
4. Test with: "List all MDS components"

[View Full Troubleshooting Guide](docs/troubleshooting.md)

---

## 📜 License

MIT License - See [LICENSE](LICENSE) file

---

## 🙏 Acknowledgments

Built for Claude Code by the QA Automation Team

**Technologies**:
- [Claude Code](https://claude.ai/claude-code) - AI-powered development
- [Playwright](https://playwright.dev) - End-to-end testing
- **MCP Servers**: MDS Design System, Platform UI Shell Helpers

---

## 📞 Support

- 📖 **Documentation**: [docs/](docs/)
- 🐛 **Bug Reports**: [GitHub Issues](https://github.com/your-org/qa-test-orchestrator/issues)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/your-org/qa-test-orchestrator/discussions)
- ✅ **Verify Setup**: `/verify`

---

## 🎯 Success Metrics

✅ **6 specialized agents** covering all test types
✅ **Pure markdown** - No runtime dependencies
✅ **Full traceability** - REQ → AC → TC
✅ **MCP integration** - MDS & Platform UI
✅ **User-controlled** - 4 approval checkpoints
✅ **Comprehensive docs** - Agent reference, guides, examples

---

## 📈 Version History

### v3.0.0 (2026-02-11) - Claude Code Plugin

- 🎉 **Converted to Claude Code plugin** (pure markdown)
- 🤖 **6 specialized agents** with markdown definitions
- 📝 **Commands** for easy agent invocation
- 🎓 **Skills** for reusable patterns
- 📖 **Comprehensive documentation**
- 🗂️ **Deprecated JavaScript** implementation (still available in `deprecated/`)

---

<div align="center">

**⭐ Star us on GitHub — it helps!**

[Install Plugin](https://claude.ai/claude-code) | [View Docs](docs/) | [Report Issue](https://github.com/your-org/qa-test-orchestrator/issues)

</div>
