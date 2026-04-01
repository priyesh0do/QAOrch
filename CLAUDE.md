# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**QA Test Orchestrator** is a Claude Code plugin that provides 6 specialized AI agents for automated Playwright test generation. It's a **pure markdown-based plugin** with no JavaScript runtime - all agent definitions, commands, and skills are markdown files that Claude Code interprets.

**Version**: 3.0.0 (agent-driven architecture)
**Framework**: Playwright
**Language**: JavaScript
**Architecture**: Agent-driven with low autonomy (4 approval checkpoints per agent)

### Organizational Context & Adoption Strategy

**Current State**: The organization uses 3 different UI automation frameworks (Selenium with C#, Selenium with Java, Cypress), leading to fragmented knowledge, higher maintenance costs, and inconsistent practices.

**QA CoE Recommendation**:
- **Existing automation**: Keep in current tools - no migration, no disruption
- **New automation**: Use QA Orchestrator (Playwright + JavaScript) for all new work

### Three User Journeys

1. **Journey 1: New Project Teams** - Teams starting new modules/greenfield projects
   - Full QA Orchestrator adoption from day one
   - Hands-on QA CoE support for first test suite
   - Path to full independence

2. **Journey 2: Mixed Teams** - Teams with existing automation AND new features
   - Keep existing tests in Selenium/Cypress unchanged
   - Use QA Orchestrator for ALL new features
   - Parallel execution of old + new tools
   - Natural transition over time

3. **Journey 3: Maintenance-Mode Teams** - Stable products with few new features
   - Keep existing tests in current tools
   - Use QA Orchestrator when new features arrive
   - Slower adoption pace
   - Quarterly refresher sessions with QA CoE

**All journeys support teams with or without MDS/Platform UI** - the orchestrator adapts by analyzing frontend code directly when MCP servers aren't available.

## Architecture

### Core Components

```
.claude-plugin/plugin.json    # Plugin metadata and registration
orchestrator-config.json       # Runtime configuration (required)
agents/                        # 6 specialized agent definitions (markdown)
commands/                      # Slash command definitions (markdown)
skills/                        # Reusable testing patterns (markdown)
docs/                         # Architecture and guides
examples/                     # Usage examples
```

### The 6 Agents

1. **QATestOrchestrator** (orchestrator.md) - Master coordinator with preflight checks
2. **FunctionalTestAutomationAgent** - Generate functional tests with REQ→AC→TC traceability
3. **IntegrationTestAgent** - Generate integration tests with database validation
4. **APITestAgent** - Generate API tests from OpenAPI/Swagger specs
5. **LocatorHealingAgent** - Auto-heal broken locators with 5 alternatives
6. **EndToEndTestAgent** - Generate E2E tests for complete user journeys
7. **ProjectSetupAgent** - Initialize project structure and base classes

### Agent Dependencies

```
FunctionalTestAutomationAgent (no dependencies, runs first)
  ├── IntegrationTestAgent (depends on Functional)
  └── EndToEndTestAgent (depends on Functional + Integration)

APITestAgent (independent, can run in parallel)
LocatorHealingAgent (independent, triggered by test failures)
ProjectSetupAgent (independent, setup only)
```

## Configuration

### Required File: orchestrator-config.json

This file **must exist** in the project root and contains:

```json
{
  "version": "3.0.0",
  "architecture": "agent-driven",
  "projectPath": "path/to/test-project",
  "requirementPath": "path/to/requirements",
  "frontendProjectPath": "path/to/frontend",
  "baseUrl": "http://localhost:5173",
  "framework": "Playwright",
  "language": "JavaScript"
}
```

**Critical paths to validate**:
- `projectPath`: Where tests will be generated
- `requirementPath`: Where requirements documents are stored (format: {moduleName}.md)
- `frontendProjectPath`: Source code for locator analysis

## Working with Agents

### Starting an Agent

Users can invoke agents naturally:
```
Start FunctionalTestAutomationAgent for module AbsenceCode
```

Or via slash commands:
```
/start-functional-tests AbsenceCode
/orchestrate
/heal-locators Dashboard
```

### The 4-Phase Approval Model with Interactive Review

**Every agent execution requires user approval at 4 checkpoints**:

1. **Analysis Phase** → [A]pprove - Requirements parsed, components identified
2. **Design Phase (Interactive)** → Multiple interactions → [A]pprove All - Test cases reviewed and approved
3. **Generation Phase** → [A]pprove - Code artifacts generated
4. **Validation Phase** → [A]pprove - Syntax, locators, coverage validated

#### Phase 2: Interactive Design Review (NEW)

Phase 2 includes an **interactive review loop** where users can refine test cases before approving:

**Workflow**:
1. Agent generates test cases and displays them in formatted console output
2. User reviews all test cases with full details (steps, data, expected results)
3. User chooses from interactive options:
   - **[A]pprove All** - Export to CSV and continue (exits loop)
   - **[M]odify** - Modify specific test case (regenerate with changes)
   - **[R]egenerate** - Regenerate all test cases with different approach
   - **[S]pecific** - Show individual test case details
   - **[R]emove** - Remove test case (with coverage impact warning)
   - **[A]dd** - Add new test case for uncovered scenario
   - **[Q]uestion** - Ask questions about any test case
4. Loop continues until user approves all test cases
5. Agent exports approved test cases to CSV (Jira-compatible format)

**Benefits**:
- User sees exactly what will be generated before code creation
- Iterative refinement without regenerating code
- CSV export for Jira test management integration
- Full control over test case design

**IMPORTANT**: Always wait for explicit user approval before proceeding to the next phase.

### Agent Execution Flow

When a user requests an agent:

1. Read and validate `orchestrator-config.json`
2. Verify MCP server connections (if required)
3. Read the agent definition from `agents/{agent-name}.md`
4. **Phase 1: Analysis** - Parse requirements and identify components
5. **Phase 2: Design (Interactive)**:
   - Generate test cases
   - Display in formatted console
   - Interactive review loop (user modifies/adds/removes)
   - Export approved test cases to CSV
6. **Phase 3: Generation** - Read CSV and generate code for approved test cases only
7. **Phase 4: Validation** - Validate generated artifacts
8. Save state to allow resuming

**Key Change**: Phase 2 now includes interactive review before CSV export. Phase 3 generates code ONLY for test cases approved and exported in Phase 2.

## MCP Server Integration

### MCP Servers for MDS-Based Projects

1. **mds-mcp** (MDS Design System) - **Optional Enhancement**
   - Used by: FunctionalTestAutomationAgent, IntegrationTestAgent
   - Tools: `listAllComponents`, `getComponentDocs`, `getComponentExamples`
   - Purpose: Accurate MDS component locators and props
   - **When to use**: Projects using MDS Design System components

2. **platform-ui** (Platform UI Shell Helpers) - **Optional Enhancement**
   - Used by: FunctionalTestAutomationAgent, IntegrationTestAgent, APITestAgent
   - Tools: `list_shell_helpers`, `get_shell_helper_docs`
   - Purpose: Shell helper API integration
   - **When to use**: Projects using Platform UI shell helpers

3. **chrome-devtools** (DOM inspection) - **Optional**
   - Used by: LocatorHealingAgent
   - Purpose: Real-time DOM analysis for locator healing
   - **When to use**: Enhanced locator healing with live DOM inspection

### Working WITHOUT MDS/Platform UI MCP Servers

**The orchestrator fully supports non-MDS projects!** If your team doesn't use MDS or Platform UI:

1. **Set `frontendProjectPath` in orchestrator-config.json**
   ```json
   {
     "frontendProjectPath": "C:\\path\\to\\your\\react-app\\src",
     "mdsIntegration": { "enabled": false },
     "platformUIHelpers": { "enabled": false }
   }
   ```

2. **Orchestrator will analyze your frontend code directly**
   - Reads React/Vue/Angular/vanilla JS components
   - Identifies UI patterns (Bootstrap, Material-UI, custom components)
   - Generates locators based on actual code structure
   - Creates page objects matching your UI framework

3. **Locator strategies adapt to your framework**
   - **React**: Looks for `data-testid`, component props, className patterns
   - **Vue**: Analyzes template structure, v-bind attributes
   - **Angular**: Uses component selectors, ng-* attributes
   - **Custom**: Identifies unique attributes in your codebase

4. **Full functionality maintained**
   - ✅ Requirements parsing (REQ → AC → TC traceability)
   - ✅ Test design documents
   - ✅ Page object generation
   - ✅ Test specification generation
   - ✅ Locator healing (using frontend code as reference)

### Checking MCP Availability

Before starting agents:
1. Check if MCP tools are available: `mcp__mds-mcp__listAllComponents`
2. If not available AND project doesn't use MDS:
   - ✅ Continue with frontend code analysis
   - ✅ Verify `frontendProjectPath` is configured
   - ✅ Inform user: "MCP servers not detected. Using frontend code analysis mode."
3. If not available AND project DOES use MDS:
   - ❌ Warn user to configure MCP servers for best results
   - ✅ Offer fallback: analyze frontend code instead

## Requirements Format

Requirements documents must follow this format:

```markdown
## REQ-001: Requirement Title
Description of the requirement

### AC-001: Acceptance Criteria Title
Description of the acceptance criteria

### AC-002: Another Acceptance Criteria
Description
```

**Key patterns**:
- Requirements: `## REQ-XXX: Title`
- Acceptance Criteria: `### AC-XXX: Title` (under requirements)
- Each AC must link to a parent REQ

## Generated Artifacts

### Functional Tests
```
test-designs/{module}/functional-design.md          # Test design document
test-designs/{module}/test-cases-{module}.csv       # User-approved test cases (Jira-ready)
test-designs/{module}/test-cases-summary.md         # Human-readable test case summary
test-designs/{module}/traceability-matrix.md        # REQ→AC→TC mapping
pages/{module}/*.page.js                            # Page object models
tests/functional/{module}/*.spec.js                 # Test specifications
```

### Integration Tests
```
test-designs/{module}/integration-design.md         # Integration design
test-designs/{module}/test-cases-{module}.csv       # Integration scenarios (Jira-ready)
test-designs/{module}/test-cases-summary.md         # Human-readable summary
test-designs/{module}/data-flow-diagram.md          # Data flow mapping
tests/integration/{module}/*.spec.js                # Integration tests
```

### E2E Tests
```
test-designs/{module}/e2e-design.md                 # E2E test design
test-designs/{module}/test-cases-{module}.csv       # E2E scenarios (Jira-ready)
test-designs/{module}/test-cases-summary.md         # Human-readable summary
test-designs/{module}/user-journey-map.md           # User journey diagrams
tests/e2e/{module}/*.spec.js                        # E2E test specifications
```

### API Tests
```
test-designs/{module}/api-design.md                 # API test design
test-designs/{module}/test-cases-{module}.csv       # API scenarios (Jira-ready)
test-designs/{module}/test-cases-summary.md         # Human-readable summary
test-designs/{module}/api-endpoints.md              # Endpoint documentation
tests/api/{module}/*.spec.js                        # API test specs
tests/api/services/apiClient.js                     # API client utility
tests/api/services/schemaValidator.js               # Schema validator
```

### CSV Export for Jira Integration

**All test generation agents** (Functional, Integration, E2E, API) now export test cases to CSV format during Phase 2:

**File Location**: `test-designs/{module}/test-cases-{module}.csv`

**CSV Format** (Jira-compatible):
```csv
Test Case ID,Test Case Title,Description,Test Type,Priority,Requirement ID,Acceptance Criteria ID,Preconditions,Test Steps,Expected Result,Test Data,Labels,Module
```

**Test Case ID Formats**:
- Functional: `TC-001`, `TC-002`, etc.
- Integration: `IS-001`, `IS-002`, etc.
- E2E: `E2E-001`, `E2E-002`, etc.
- API: `API-001`, `API-002`, etc.

**Usage**:
1. CSV is created AFTER user approves test cases in Phase 2 interactive review
2. CSV contains ONLY user-approved test cases
3. Phase 3 generates code from the CSV (not from internal state)
4. CSV can be imported to Jira Test Management (Xray, Zephyr, or native)
5. Keep CSV in version control for traceability

## CSV Export and Jira Integration

### Overview

All 4 test generation agents (Functional, Integration, E2E, API) export user-approved test cases to CSV format during Phase 2. This enables:
- **Jira Test Management** integration (Xray, Zephyr, or native test cases)
- **Manual review** of test cases before code generation
- **Team collaboration** on test design
- **Version control** of test specifications
- **Traceability** outside the codebase

### CSV Export Workflow

```
Phase 2: Design
   ↓
Agent generates test cases
   ↓
Display in console (formatted, detailed)
   ↓
User reviews interactively:
   • Modify specific test cases
   • Add new test cases
   • Remove test cases
   • Regenerate all
   ↓
User approves all test cases
   ↓
CSV exported: test-designs/{module}/test-cases-{module}.csv
   ↓
Phase 3: Code generated from CSV
```

### CSV Format Specification

**Columns** (Jira-compatible):
```
Test Case ID, Test Case Title, Description, Test Type, Priority,
Requirement ID, Acceptance Criteria ID, Preconditions, Test Steps,
Expected Result, Test Data, Labels, Module
```

**Test Case ID Conventions**:
- **Functional**: TC-001, TC-002, TC-003, ...
- **Integration**: IS-001, IS-002, IS-003, ...
- **E2E**: E2E-001, E2E-002, E2E-003, ...
- **API**: API-001, API-002, API-003, ...

**Field Formatting**:
- **Test Steps**: Semicolon-separated (e.g., `"1. Step one; 2. Step two; 3. Step three"`)
- **Test Data**: JSON format wrapped in quotes (e.g., `"{""field"": ""value""}"`)
- **Preconditions**: Semicolon-separated list
- **Expected Result**: Semicolon-separated outcomes
- **Labels**: Comma-separated tags (e.g., `"automated,functional,smoke"`)

**CSV Formatting Rules** (RFC 4180):
- Fields with commas/semicolons wrapped in double quotes
- Double quotes escaped by doubling (e.g., `"He said ""Hello"""`)
- Newlines within fields preserved in quotes

### Importing to Jira

**Steps to import CSV to Jira**:

1. **Open Jira Project** → Navigate to Test Cases section

2. **Start Import** → Click "Import" → "CSV Import"

3. **Upload CSV** → Select `test-cases-{module}.csv`

4. **Map Columns**:
   ```
   Test Case ID        → Issue Key or ID
   Test Case Title     → Summary
   Description         → Description
   Test Type           → Test Type (custom field)
   Priority            → Priority
   Requirement ID      → Requirement Link (custom field)
   Acceptance Criteria → Custom field
   Preconditions       → Preconditions (custom field)
   Test Steps          → Test Steps (custom field)
   Expected Result     → Expected Result (custom field)
   Test Data           → Test Data (custom field)
   Labels              → Labels
   Module              → Component or Custom field
   ```

5. **Validate** → Review mapping and preview

6. **Import** → Confirm import

**Note**: Column mapping depends on your Jira configuration. Some fields may require Xray or Zephyr plugins.

### Using CSV for Team Collaboration

**Scenario 1: QA Lead Reviews Test Cases**
1. Agent generates test cases → exports CSV
2. QA Lead opens CSV in Excel/Google Sheets
3. QA Lead reviews, adds comments, modifies test cases
4. QA Lead imports to Jira for team discussion
5. Team approves test cases in Jira
6. Export approved tests from Jira back to CSV (same format)
7. Replace orchestrator CSV with approved version
8. Continue to Phase 3 (code generation)

**Scenario 2: Parallel Test Design and Automation**
1. Manual QA team designs test cases in Jira
2. Export test cases from Jira to CSV (matching orchestrator format)
3. Place CSV in `test-designs/{module}/` directory
4. Run agent: "Generate code from CSV for module {ModuleName}"
5. Agent reads CSV and generates page objects + test specs

**Scenario 3: Version Control**
1. Commit CSV files to Git along with generated code
2. CSV acts as single source of truth for test specifications
3. Code reviews include both CSV and generated code
4. Changes to test cases = update CSV + regenerate code

### Additional Files Generated

Along with the CSV, agents create:

1. **test-cases-summary.md** - Human-readable markdown format
   - Same content as CSV but formatted for readability
   - Includes code examples, locator strategies
   - Easier to review in pull requests

2. **traceability-matrix.md** - REQ → AC → TC mapping
   - Visual matrix showing coverage
   - Identifies gaps in test coverage
   - Links requirements to test cases

3. **Agent-specific files**:
   - Functional: `functional-design.md`
   - Integration: `data-flow-diagram.md`
   - E2E: `user-journey-map.md`
   - API: `api-endpoints.md`

### Best Practices

1. **Keep CSV in version control** - Track test case changes over time
2. **Review CSV before Phase 3** - Catch issues before code generation
3. **Use Jira for team collaboration** - Import CSV for discussions
4. **Don't edit CSV after Phase 3** - Regenerate code if test cases change
5. **Maintain traceability** - Keep Requirement ID and AC ID columns accurate
6. **Use consistent naming** - Follow TC-XXX, IS-XXX, E2E-XXX, API-XXX conventions

## Coding Standards

### Page Object Pattern

```javascript
class PageName {
  constructor(page) {
    this.page = page;
    // Locators as properties, not methods
    this.submitButton = page.locator('[data-testid="submit"]');
  }

  // Action methods (async)
  async clickSubmit() {
    await this.submitButton.click();
  }

  // Assertion helpers (return boolean or throw)
  async isSubmitVisible() {
    return await this.submitButton.isVisible();
  }
}
```

### Locator Priority (Best to Worst)

1. `[data-testid="unique-id"]` - Most stable
2. `mds-component[unique-prop="value"]` - MDS components
3. `[role="button"][aria-label="Submit"]` - Accessibility attributes
4. `.css-class` - Last resort, fragile

### Test Structure

```javascript
test.describe('Feature Name', () => {
  test.beforeEach(async ({ page }) => {
    // Setup
  });

  test('TC-001: Test case description', async ({ page }) => {
    // Arrange
    // Act
    // Assert
  });
});
```

## Common Development Tasks

### Verify Configuration
```
/verify
```
Validates orchestrator-config.json and checks all paths exist.

### Generate Functional Tests
```
Start FunctionalTestAutomationAgent for module {ModuleName}
```
Requirements must exist at: `{requirementPath}/{ModuleName}.md`

**What happens**:
1. **Phase 1**: Requirements parsed, components identified → User approves
2. **Phase 2 (Interactive)**:
   - Agent generates test cases (TC-001, TC-002, etc.)
   - Displays ALL test cases in formatted console with full details
   - User reviews and interacts:
     - Type `[M]odify` to change TC-003's test steps
     - Type `[A]dd` to add a new edge case test
     - Type `[R]emove` to remove TC-007 (not needed)
     - Type `[S]pecific` to see details of TC-005
     - Type `[Q]uestion` to ask about test case design
     - Type `[R]egenerate` to regenerate all with more detail
   - Loop continues until user types `[A]pprove All`
   - CSV exported: `test-designs/{ModuleName}/test-cases-{ModuleName}.csv`
3. **Phase 3**: Code generated from approved CSV (only approved test cases) → User approves
4. **Phase 4**: Validation runs → User approves

**Output**: Page objects, test specs, CSV file (Jira-ready), traceability matrix, summary.md

**Example Interactive Session**:
```
Agent: [Displays TC-001, TC-002, TC-003 in console]
Agent: What would you like to do? [A]pprove All, [M]odify, [R]emove, [A]dd...

User: Modify

Agent: Which test case? (e.g., TC-002)

User: TC-002

Agent: What would you like to change in TC-002?

User: Add a step to verify the success notification timeout

Agent: [Regenerates TC-002 with new step, displays updated test case]
Agent: Test case TC-002 updated. What would you like to do?

User: Approve All

Agent: [Exports CSV with all test cases including updated TC-002]
Agent: CSV exported. Continue to Phase 3? [A]pprove, [M]odify, [R]eject
```

### Heal Broken Locators
```
/heal-locators {ModuleName}
```
Analyzes test failures and generates up to 5 alternative locators per failure.

### Setup New Project
```
/setup-project
```
Creates directory structure, base classes, and rules.md.

### Run Multiple Agents in Parallel
```
Start FunctionalTestAutomationAgent and APITestAgent for module {ModuleName}
```
Agents run concurrently with independent approval checkpoints.

## Important Patterns

### Traceability Mapping

**Always maintain the chain**:
```
REQ-001 → AC-001 → TC-001, TC-002
         AC-002 → TC-003
REQ-002 → AC-003 → TC-004, TC-005
```

Every test case must trace back to an acceptance criterion, which traces to a requirement.

### State Management

Agents save state to `./.agent-state/` directory:
- Enables resuming interrupted executions
- Allows inter-agent communication
- Persists artifacts between runs

### Parallel Execution

Independent agents can run concurrently:
- **Max concurrent**: 3 agents (configurable)
- **Dependency resolution**: Automatic (orchestrator handles)
- **Approval independence**: Each agent waits at its own checkpoints

## Key Files to Reference

### Core Documentation
- **docs/architecture.md** - Detailed architecture explanation
- **docs/getting-started.md** - 10-minute quickstart guide
- **agents/*.md** - Agent-specific implementation details
- **examples/*.md** - Working examples and walkthroughs
- **skills/*.md** - Reusable testing patterns

### Sprint Planning & Scope Management
- **docs/guides/qa-scope-and-metrics.md** - QA scope definition and performance measurement for one person in a sprint
- **docs/guides/qa-story-points-guide.md** - Story point estimation and capacity planning for QA engineers
- **docs/guides/manager-qa-scope-measurement.md** - Engineering manager's guide to measuring and validating QA scope
- **docs/guides/manager-quick-reference.md** - 1-page quick reference card for managers (print and keep at desk)
- **docs/guides/10-day-sprint-planning.md** - Complete 10-day sprint plan with QA Orchestrator
- **docs/guides/sprint-planning.md** - 15-day sprint plan with quality gates
- **docs/guides/sprint-quick-reference.md** - Sprint command quick reference

## Migration Notes (v2.0 → v3.0)

**Breaking changes**:
- Workflows deprecated (workflow markdown files no longer executed)
- Commands changed: Use agent names, not workflow names
- All agents now require 4 approval checkpoints
- Configuration format updated (new `agents` section)

**Do NOT reference** `workflows/` directory - it's deprecated documentation only.

## Troubleshooting

### "Plugin not loading"
Check `.claude-plugin/plugin.json` exists and is valid JSON.

### "Agent not starting"
1. Verify `orchestrator-config.json` exists
2. Run `/verify` to check configuration
3. Ensure all paths in config are valid

### "MCP server not connected"
1. Check Claude Code settings → MCP Servers
2. Verify required servers are configured
3. Restart Claude Code after adding servers

### "Locator failures in tests"
Run `/heal-locators {ModuleName}` to auto-generate alternatives.

## Design Philosophy

1. **Pure Markdown** - No JavaScript runtime, no dependencies
2. **Low Autonomy** - User approval required at every checkpoint
3. **Full Traceability** - Every test traces to requirements
4. **MDS Integration** - Component-aware locator strategies
5. **Agent Specialization** - Each agent is expert in its domain
6. **State Management** - Agents coordinate via shared state

## When Working on This Plugin

### Adding a New Agent
1. Create `agents/new-agent.md` with frontmatter and phases
2. Register in `.claude-plugin/plugin.json` under `agents.definitions`
3. Create corresponding command in `commands/`
4. Update docs/architecture.md with agent description

### Adding a New Command
1. Create `commands/new-command.md` with frontmatter
2. Register in `.claude-plugin/plugin.json` under `commands.definitions`
3. Document in README.md

### Adding a New Skill
1. Create `skills/skill-name/SKILL.md` with patterns
2. Register in `.claude-plugin/plugin.json` under `skills.definitions`
3. Reference from agent definitions where applicable

### Modifying Agent Behavior
Edit the agent's markdown file directly - no build step required. Changes are immediately effective.

## Critical Rules

1. **Never skip approval checkpoints** - Users must explicitly approve each phase
2. **Always validate orchestrator-config.json** - Fail fast if invalid
3. **Maintain traceability** - Every TC must link to AC, every AC to REQ
4. **Check MCP servers** - Verify availability before using MCP tools
5. **Generate complete artifacts** - Don't create partial files
6. **Follow naming conventions** - `*.page.js` for pages, `*.spec.js` for tests
7. **Preserve state** - Save to `.agent-state/` for coordination
8. **Respect dependencies** - Don't run dependent agents before their dependencies
9. **Interactive Phase 2 review** - Always enter interactive loop, wait for user to approve all test cases before CSV export
10. **CSV-driven Phase 3** - Generate code ONLY from test cases in the CSV file (not from internal state)
11. **CSV format compliance** - Follow RFC 4180 format, escape special characters properly
12. **Test case ID conventions** - Use TC-XXX (Functional), IS-XXX (Integration), E2E-XXX (E2E), API-XXX (API)
