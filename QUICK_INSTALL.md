# Quick Install Guide

## TL;DR - Install Your Plugin NOW

### On Windows (PowerShell as Administrator)

```powershell
# Navigate to your orchestrator
cd C:\Working\test_orchestrator_workflow_v3

# Create symlink
New-Item -ItemType SymbolicLink `
  -Path "$env:USERPROFILE\.claude-code\plugins\qa-test-orchestrator" `
  -Target "C:\Working\test_orchestrator_workflow_v3"

# Verify
Test-Path "$env:USERPROFILE\.claude-code\plugins\qa-test-orchestrator\.claude-plugin\plugin.json"
```

### On Linux/Mac

```bash
# Navigate to your orchestrator
cd /path/to/test_orchestrator_workflow_v3

# Create symlink
ln -s "$(pwd)" ~/.claude-code/plugins/qa-test-orchestrator

# Verify
ls ~/.claude-code/plugins/qa-test-orchestrator/.claude-plugin/plugin.json
```

### Restart Claude Code

Close and restart Claude Code completely.

### Test It

```
# In Claude Code, test with:
Start FunctionalTestAutomationAgent for module TestModule

# Or use slash command:
/start-functional-tests TestModule
```

---

## Understanding the Structure

### Your Current Structure (Single Plugin)

```
test_orchestrator_workflow_v3/
├── .claude-plugin/
│   └── plugin.json              ← Plugin metadata (YOU HAVE THIS!)
├── agents/                      ← 6 agent definitions
├── commands/                    ← 3 commands
└── skills/                      ← 3 skills
```

**You do NOT need marketplace.json** because you're installing a single plugin, not a marketplace of plugins.

### Reference Repo Structure (Marketplace)

```
agents-main/
├── .claude-plugin/
│   └── marketplace.json         ← Lists 73 plugins
└── plugins/
    ├── plugin1/                 ← Each has plugin.json
    ├── plugin2/
    └── ...                      (73 total)
```

**The reference repo needs marketplace.json** because it's a collection of 73 different plugins.

---

## Key Difference

| Your Orchestrator | Reference Repo |
|-------------------|----------------|
| **1 plugin** with 6 agents | **73 plugins** with 112 agents |
| Needs: `plugin.json` ✅ | Needs: `marketplace.json` ✅ |
| Install: Symlink directly | Install: Marketplace, then choose plugins |

---

## Why You Were Confused

The reference repo has **TWO levels**:
1. **Marketplace level**: marketplace.json (lists all 73 plugins)
2. **Plugin level**: Each plugin has its own plugin.json

Your orchestrator is **ONE level**:
- **Plugin level**: plugin.json (you already have this!)

You're installing at the plugin level, so you only need plugin.json.

---

## Troubleshooting

### "Plugin not showing up"

1. **Check the symlink exists**:
   ```powershell
   # Windows
   Test-Path "$env:USERPROFILE\.claude-code\plugins\qa-test-orchestrator"

   # Should return: True
   ```

2. **Check plugin.json is there**:
   ```powershell
   # Windows
   Get-Content "$env:USERPROFILE\.claude-code\plugins\qa-test-orchestrator\.claude-plugin\plugin.json"

   # Should show JSON content
   ```

3. **Restart Claude Code** (completely quit and reopen)

4. **Check Claude Code plugin directory exists**:
   ```powershell
   # If it doesn't exist, create it:
   New-Item -Path "$env:USERPROFILE\.claude-code\plugins" -ItemType Directory -Force
   ```

### "Permission denied"

Run PowerShell as **Administrator** on Windows.

### "Symlink not working"

Use **copy method** instead:

```powershell
Copy-Item -Path "C:\Working\test_orchestrator_workflow_v3" `
  -Destination "$env:USERPROFILE\.claude-code\plugins\qa-test-orchestrator" `
  -Recurse
```

---

## Next Steps

1. ✅ Install plugin (command above)
2. ✅ Restart Claude Code
3. ✅ Test: `Start FunctionalTestAutomationAgent for module TestModule`
4. ✅ Read: `INSTALLATION.md` for detailed docs
5. 📖 Optional: `CREATE_MARKETPLACE.md` if you want marketplace structure later

---

## Summary

**You DON'T need marketplace.json** - you already have everything you need!

Your `plugin.json` at `.claude-plugin/plugin.json` is correct and complete.

Just symlink your directory to `~/.claude-code/plugins/qa-test-orchestrator` and restart Claude Code.

🎉 That's it!
