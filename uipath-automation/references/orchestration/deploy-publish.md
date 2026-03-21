# Module: Deploy & Publish to Orchestrator
**Covers:** Publishing from Studio UI, CLI/CI-CD, project.json settings, environment promotion, Git source control

---

## Publish from Studio UI

1. Design tab → **Publish**
2. Select feed: `Orchestrator Tenant Processes Feed` (most common) or Personal Workspace
3. Set version: `Major.Minor.Patch` — follow semver (e.g. `1.2.0`)
4. Add release notes describing what changed
5. Studio packages as `.nupkg` and uploads to Orchestrator automatically

**Version strategy:**
- `Patch` (1.0.**1**) → bug fixes, no new behavior
- `Minor` (1.**1**.0) → new features, backward compatible
- `Major` (**2**.0.0) → breaking changes to arguments/behavior

---

## Publish via CLI / CI-CD

```bash
# UiPath CLI (cross-platform, recommended for pipelines)
uipath project pack ./project.json --output ./dist/

uipath orch package publish \
  --packagePath "./dist/MyProcess.1.2.0.nupkg" \
  --orchestratorUrl "https://cloud.uipath.com/org/tenant" \
  --orchestratorTenant "MyTenant" \
  --orchestratorApplicationId "$CLIENT_ID" \
  --orchestratorApplicationSecret "$CLIENT_SECRET" \
  --orchestratorApplicationScope "OR.Execution OR.Folders" \
  --orchestratorFolder "Shared"

# Run Workflow Analyzer before packaging (fail build on errors)
uipath project analyze ./project.json --analyzerTraceLevel Error
```

**Azure DevOps / GitHub Actions pattern:**
```yaml
# .github/workflows/deploy.yml
- name: Analyze project
  run: uipath project analyze ./project.json --analyzerTraceLevel Error

- name: Pack
  run: uipath project pack ./project.json --output ./dist/ --autoVersion

- name: Publish to UAT
  run: uipath orch package publish --packagePath ./dist/*.nupkg
  env:
    CLIENT_ID: ${{ secrets.UIPATH_CLIENT_ID }}
    CLIENT_SECRET: ${{ secrets.UIPATH_CLIENT_SECRET }}
```

---

## project.json Key Settings

```json
{
  "name": "InvoiceProcessing",
  "description": "Processes invoice items from SAP via Orchestrator queue",
  "projectVersion": "1.3.0",
  "runtimeOptions": {
    "requiresUserInteraction": false,
    "supportsPersistence": false
  },
  "targetFramework": "Portable",
  "entryPointPaths": ["Main.xaml"],
  "globalHandlerFilePath": "GlobalExceptionHandler.xaml",
  "dependencies": {
    "UiPath.Excel.Activities": "[2.23.1, )",
    "UiPath.UIAutomation.Activities": "[25.10.0, )",
    "UiPath.System.Activities": "[25.10.0, )"
  },
  "analyzerOptions": {
    "analyzeLevel": "Error",
    "rules": {
      "ST-DBP-003": { "isEnabled": true, "defaultAction": "Error" },
      "ST-SEC-001": { "isEnabled": true, "defaultAction": "Error" },
      "ST-NMG-002": { "isEnabled": true, "defaultAction": "Warning" }
    }
  }
}
```

**`targetFramework` values:**
- `Portable` = Cross-platform (runs on Linux robots, recommended for new projects)
- `Windows` = Windows-only (required for full UI automation)
- `Legacy` = Windows-Legacy (old .NET Framework, avoid for new projects)

---

## Environment Promotion Pattern

```
Dev Robot → publish to Dev feed → manual test
         → Workflow Analyzer must pass (zero Errors)
         → promote package to UAT feed (copy NuGet package, don't republish)
         → UAT testing with UAT data
         → approve → promote to Prod feed
         → deploy to Prod robots (assign process to robot, enable trigger)
```

**Orchestrator promotion:** Tenant → Automation Ops → Packages → select version → Promote to folder

---

## Source Control (Git in Studio)

### Initial setup
1. Team tab → **Source Control** → Connect
2. Enter remote URL (GitHub/Azure DevOps/GitLab)
3. Studio creates local clone in project folder

### Workflow
```
main branch = always deployable (what's in production)
feature/fix branches = development work

1. Team → Pull (sync before starting)
2. Team → Manage Branches → Create Branch → "feature/invoice-retry-logic"
3. Develop and test locally
4. Team → Commit → select changed files → write message
5. Team → Push
6. Create Pull Request in GitHub/Azure DevOps
7. Code review → merge to main
8. Pipeline auto-publishes to Orchestrator
```

### .gitignore for UiPath projects
```gitignore
# UiPath generated/runtime files
.local/
.entities/
.objects/
.screenshots/
*.user
TestResults/
Logs/
project.lock.json
*.nupkg

# Object Repository caches
ObjectRepository/*.json

# Studio settings (machine-specific)
.settings/
```

### Conflict resolution strategy
- XAML conflicts are XML — hard to merge manually
- Keep feature branches short-lived (1-3 days)
- Use `git checkout --ours file.xaml` or `--theirs` for XAML conflicts
- If conflict is complex: rebuild the workflow from the merge base

---

## Debugging in Studio

### Breakpoints
- Click to the left of any activity line number → toggle breakpoint (red dot)
- F5 (Debug mode) → runs and pauses at breakpoints
- Breakpoints survive between runs

### Debug Panels
| Panel | Purpose |
|---|---|
| Watch | Add variable/expression → see live value at each step |
| Locals | Auto-shows all in-scope variables |
| Call Stack | Shows nested workflow invocation chain |
| Immediate | Evaluate any VB.NET expression on the fly |
| Output | Log messages + activity execution trace |

### Step controls
| Key | Action |
|---|---|
| F11 | Step Into (enter invoked workflow / child activity) |
| F10 | Step Over (execute, don't enter) |
| Shift+F11 | Step Out (finish current scope, return to caller) |
| F5 | Continue to next breakpoint |
| F6 | Step Over (alternate) |

### Slow Step (execution speed)
Debug tab → Slow Step → set speed: 1x, 2x, 4x — watch activities highlight in real-time

### Logging for diagnosis
```vb
' Add temporary verbose logging during development:
Log.Info($"[DEBUG] dtOrders.Rows.Count = {dtOrders.Rows.Count}")
Log.Info($"[DEBUG] currentStatus = '{currentStatus}'")
' Remove or change to Trace level before deploying
```

### Common debugging patterns
```
"Selector not found" → UI Explorer → Highlight → verify selector
"NullReferenceException" → Watch panel → find which variable is Nothing
"Format exception" → check the expression, add TryParse
"Timeout" → increase Timeout property, add WaitElementAppear before Click
"Queue empty unexpectedly" → Orchestrator → Queues → inspect item status
```
