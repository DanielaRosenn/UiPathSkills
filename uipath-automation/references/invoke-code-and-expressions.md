# Invoke Code, expressions, and imports (Studio)

Summarizes behavior described in UiPath documentation for **Invoke Code** and related designer concepts. For exact property lists, use the official page: [Invoke Code](https://docs.uipath.com/activities/other/latest/workflow/invoke-code).

**Extended patterns (DataTable/JSON/file ops in snippets):** [orchestration/invoke-code.md](orchestration/invoke-code.md). **Expression cookbook:** [orchestration/expressions.md](orchestration/expressions.md).

## Invoke Code activity (`UiPath.Core.Activities.InvokeCode`)

- **Purpose:** Synchronously runs **VB.NET** or **C#** in a snippet, with **In / Out / InOut** arguments bound to workflow variables or values.
- **Package:** `UiPath.System.Activities` (see doc for minimum Studio compatibility).
- **Language:** Property **Language** = `VBNet` or `CSharp` (must match the snippet syntax).
- **Arguments:** Define in **Edit Arguments**; direction and types must match the code (e.g. `Out` for values assigned inside the snippet).
- **Code field:** Supports the script text or a `String` variable reference (per doc).

### Cross-platform vs Windows

Documentation distinguishes **Cross-platform** vs **Windows / Windows – Legacy** configuration panels. Confirm **project compatibility** (`Windows`, `Windows - Legacy`, or cross-platform) before relying on APIs available only on Windows.

### Imports and assemblies

Per official doc: *The assemblies referenced by your code need to be added into the **Imports** panel* (namespaces). External libraries (e.g. NuGet) must be referenced in the project so types resolve at compile time.

**Related:** [Importing namespaces](https://docs.uipath.com/studio/standalone/2024.10/user-guide/importing-namespaces).

### Example pattern (VB.NET-style snippet)

Conceptually (adapt names to your variables):

```text
' In Arguments: inText (String), Out Arguments: outLength (Int32)
outLength = inText.Length
```

Bind `inText` / `outLength` in **Edit Arguments** to workflow variables.

### Advanced sample

Official walkthrough for third-party library usage:

- [Example: RestSharp in Invoke Code](https://docs.uipath.com/activities/other/latest/workflow/example-using-the-restsharp-library-in-the-invoke-code-activity)

### Code quality in workflows

- Prefer **small, testable** snippets; move large logic to **libraries** or **coded workflows**.
- Avoid embedding secrets; use **Orchestrator assets/credentials** or secure inputs.
- **Continue on error:** Doc notes interaction with **Try Catch** (if Continue On Error is True, errors may not surface to Catch as expected—see Invoke Code page).

## Workflow expressions (properties panel)

- Non–Invoke Code fields typically use the **expression editor** with the project’s **expression language** (VB.NET-style is common for classic projects; C# may be selected where supported).
- Keep expressions **short**; use **Assign** or **Invoke Code** for multi-step logic.

## Coded workflow vs Invoke Code

| Approach | Use when |
|----------|-----------|
| **Invoke Code** | Small snippet, reuse of WF variables, quick transformation |
| **Coded workflow / .cs** | Larger C# structure, testing, full IDE refactoring (see Studio *Coded workflow* and `references/coded-automations.md`) |
| **Code source file** | Shared C# invoked from visual workflow (see Studio *Code source file*) |

## Long-running / BPMN

Do not replace **ProcessDiagram** / persistence workflows with ad-hoc Invoke Code without following the **long-running** and **persistence** guidance. For Studio desktop LRW structure and main entry XAML, use **[`uipath-longrunning-workflow`](../../uipath-longrunning-workflow/SKILL.md)**; pair with [orchestration/long-running.md](orchestration/long-running.md) for serialization rules and [orchestration/invoke-code.md](orchestration/invoke-code.md) for LRW-safe data shaping inside snippets.
