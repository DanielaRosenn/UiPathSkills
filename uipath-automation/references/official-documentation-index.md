# Official UiPath documentation index (Studio & activities)

**Last reviewed (skill pack):** 2026-03-21 — spot-check links against your installed Studio version.

Use these **canonical** pages when verifying behavior, property names, or compatibility. Replace `2024.10` in Studio URLs with your installed Studio version if you pin docs to a release.

**Skill-internal deep dives (merged master pack):** Full map — **[UNIFIED-INDEX.md](UNIFIED-INDEX.md)**. Compact router — **[orchestration/README.md](orchestration/README.md)**.

## Studio (automation projects & designer)

| Topic | URL |
|--------|-----|
| About automation projects | https://docs.uipath.com/studio/standalone/2024.10/user-guide/about-automation-projects |
| About `project.json` | https://docs.uipath.com/studio/standalone/2024.10/user-guide/about-the-projectjson-file |
| Windows – Legacy compatibility | https://docs.uipath.com/studio/standalone/2024.10/user-guide/about-the-windows-legacy-compatibility |
| Modern Design Experience (MDE) | https://docs.uipath.com/studio/standalone/2024.10/user-guide/modern-design-experience |
| Modern vs Classic experience | https://docs.uipath.com/studio/standalone/2024.10/user-guide/differences-between-modern-experience-and-classic-experience |
| Project templates (REFramework, Background, etc.) | https://docs.uipath.com/studio/standalone/2024.10/user-guide/project-templates |

## Workflow types (Studio)

| Topic | URL |
|--------|-----|
| Sequences | https://docs.uipath.com/studio/standalone/2024.10/user-guide/sequences |
| Flowcharts | https://docs.uipath.com/studio/standalone/2024.10/user-guide/flowcharts |
| State machines | https://docs.uipath.com/studio/standalone/2024.10/user-guide/state-machines |
| Forms workflow | https://docs.uipath.com/studio/standalone/2024.10/user-guide/form-type-of-workflow |
| Coded workflow | https://docs.uipath.com/studio/standalone/2024.10/user-guide/coded-workflow |
| Code source file | https://docs.uipath.com/studio/standalone/2024.10/user-guide/code-source-file |
| Invoking coded source file | https://docs.uipath.com/studio/standalone/2024.10/user-guide/invoking-coded-source-file |
| Global Exception Handler | https://docs.uipath.com/studio/standalone/2024.10/user-guide/global-exception-handler |

## Control flow (Studio user guide)

| Topic | URL |
|--------|-----|
| Control Flow activities (overview) | https://docs.uipath.com/studio/standalone/2024.10/user-guide/control-flow-activities |
| If / Switch / While / For Each / etc. (with examples) | Same section; follow links under *Control Flow* in the Studio TOC |

## Activities reference (`UiPath.*` activity packs)

| Topic | URL |
|--------|-----|
| Workflow activities (pack index) | https://docs.uipath.com/activities/other/latest/workflow/workflow-activities |
| **Invoke Code** | https://docs.uipath.com/activities/other/latest/workflow/invoke-code |
| RestSharp + Invoke Code example | https://docs.uipath.com/activities/other/latest/workflow/example-using-the-restsharp-library-in-the-invoke-code-activity |
| Workflow Foundation activities (Sequence, Try Catch, State Machine, …) | https://docs.uipath.com/activities/other/latest/workflow/workflow-foundation-activities |

## Integration Service & Data Service (connectors)

| Topic | URL |
|--------|-----|
| Integration Service-based activities (overview) | https://docs.uipath.com/activities/other/latest/workflow/integration-service-based-activities |
| Data Service activities (classic pack) | https://docs.uipath.com/activities/other/latest/workflow/data-service-activities |

For **connector setup, triggers, and authentication** in Cloud, use Automation Cloud / Integration Service product docs (search the UiPath docs site for *Integration Service*).

## Best practices & analyzer (Studio)

| Topic | URL |
|--------|-----|
| Workflow design | https://docs.uipath.com/studio/standalone/2024.10/user-guide/workflow-design |
| UI automation | https://docs.uipath.com/studio/standalone/2024.10/user-guide/ui-automation |
| Project organization | https://docs.uipath.com/studio/standalone/2024.10/user-guide/project-organization |
| Naming rules (Workflow Analyzer) | https://docs.uipath.com/studio/standalone/2024.10/user-guide/naming-rules |

## Imports / namespaces

| Topic | URL |
|--------|-----|
| Importing namespaces (Studio) | https://docs.uipath.com/studio/standalone/2024.10/user-guide/importing-namespaces |

---

**Note:** Activity class IDs in XAML (e.g. `ui:`, `excel:`) map to packages in `project.json`. Always confirm **package name + version** in Manage Packages after adding activities.
