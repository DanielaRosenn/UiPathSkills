# Module: UI Automation Activities
**Package: UiPath.UIAutomation.Activities (Modern) | Studio 2025.10**

---

## Modern vs Classic Activities

**Always use Modern activities** (default in Windows/Cross-platform projects).
Classic activities exist under "Classic" category — legacy only, no new development.

### Modern Activity Advantages
- Auto-repair selectors
- Object Repository support
- Computer Vision fallback built-in
- Better web and desktop compatibility
- Unified API across web + desktop

---

## Core Interaction Activities

### Use Application/Browser (Container)
```
Use Application/Browser
  Application: [app path or URL]
  OpenMode: Never / IfNotOpen / Always
  CloseMode: Never / IfOpenedByMe / Always
  [All UI activities go inside this container]
```
**Rule**: always wrap UI interactions in Use Application/Browser — never interact outside a scope.

### Click
| Property | Description |
|---|---|
| Target | UI element (from Object Repository or indicated) |
| ClickType | Single, Double, Right, TripleClick |
| MouseButton | Left, Right, Middle |
| WaitForReady | None, Interactive, Complete |
| DelayBefore/After | ms delay (use sparingly — prefer WaitForElement) |
| ContinueOnError | Continue if element not found |

### Type Into
| Property | Description |
|---|---|
| Target | UI element (text field) |
| Text | String to type (use in_ argument, never hardcode values) |
| DelayBetweenKeys | ms between keystrokes (0 = instant, use for speed) |
| Activate | Click element before typing (default True) |
| EmptyField | Clear field before typing (default True) |

**Secure input**: use `TypeSecureText` for password fields (accepts SecureString).

### Get Text / Get Value
```
Get Text → Result: String (visible text of element)
Get Value → Result: String (value attribute, e.g. input field value)
Get Attribute → Result: String (any HTML/UI attribute by name)
```

### Check / Uncheck (checkbox)
- Check State → Result: Boolean (is checkbox checked?)

### Select Item (dropdowns)
| Property | Description |
|---|---|
| Target | Dropdown/listbox element |
| Item | String value to select |
| SelectMode | Single, Multiple (for multi-select lists) |

---

## Wait and Verify Activities

### Element Exists
- **Result**: Boolean — True if element found within timeout
- Use for conditional logic: `If elementExists Then Click Else...`
- **Timeout**: always from Config, never hardcoded

### Wait For Element (preferred over Delay)
- Waits until element appears or disappears
- **Timeout**: from Config
- **WaitMode**: Appear, Vanish, IsActive, IsEnabled
- Replace `Delay` activities with this wherever possible

### Find Element
- Returns the UI Element object
- Use when you need to store the element for later use or pass to other activities

### Verify Execution (Modern)
- Verifies the result of a preceding interaction
- Adds resilience — detects if click/type had no effect

---

## Selector Strategy

### Selector Types (Modern)
- **Strict selector**: exact match — fast, brittle on UI changes
- **Fuzzy selector**: partial match — more resilient
- **Image selector**: falls back to computer vision
- **Object Repository element**: centrally managed, shareable

### Good Selector Practices
```xml
<!-- Good: uses stable attributes -->
<webctrl tag='INPUT' name='username' type='text'/>

<!-- Bad: uses dynamic/index attributes -->
<webctrl tag='INPUT' idx='3'/>

<!-- Good: uses multiple stable attributes -->
<wnd app='app.exe' cls='Edit' name='Customer Name'/>
```

### Selector Rules
- Avoid `idx` (index) — breaks when UI changes
- Prefer: `name`, `id`, `automationId`, `type`, `cls`
- For dynamic text: use wildcards (`*`) or regex
- Dynamic IDs: use parent anchor + relative position
- Always test selectors with UI Explorer (Ribbon → UI Explorer)

### Anchor Base (for dynamic selectors)
```
Anchor Base
  Anchor: [stable nearby element]
  Activity: [interaction with dynamic element]
```
Use when the target element has no stable attributes but a nearby element does.

### Object Repository (recommended for large projects)
- Centrally define all UI elements
- Reuse across workflows without re-indicating
- Update once if UI changes (propagates everywhere)
- Create: Open Object Repository panel → Add UI Element
- Reference in workflow: use the OR element instead of indicating

---

## Data Scraping / Table Extraction

### Extract Table Data (Modern)
- Extracts structured tabular data from web/desktop UI
- **Output**: DataTable
- **MaxRows**: limit rows extracted
- Automatically handles pagination if configured

### Get Full Text / Get Visible Text
- Get Full Text: all text including hidden (desktop only)
- Get Visible Text: only visible text (OCR-powered for images)

### Screen Scraping Methods
1. **FullText**: fastest, for standard apps
2. **Native**: for Java/Citrix/legacy
3. **OCR (Google/Microsoft)**: for images, scanned docs, Citrix
4. **Computer Vision**: AI-powered, works on any surface

---

## Computer Vision / AI Integration

### Computer Vision Activities (UiPath.CV.Activities)
- `CV Click`, `CV Type`, `CV Get Text`, `CV Scroll`
- Use when standard selectors fail (Citrix, images, legacy apps)
- Requires AI Computer Vision server connection

---

## Application-Specific Patterns

### SAP Automation
- Install SAP Scripting on target machine
- Use `SAP Application` scope for SAP GUI
- Use `Send Hotkey` for F-key navigation
- Selector: `<sap cls='SapALVGrid' ...>`

### Java Automation
- Install Java Bridge extension
- Enable Java Accessibility settings
- Use `JavaScope` activity container
- Use `Get Attribute` with `value` attribute for field values

### Browser Automation
- Supported: Chrome, Firefox, Edge, IE
- Install browser extension (UiPath Extension)
- Navigate To: `"https://..." + Config("AppURL").ToString`
- Always use `Use Application/Browser` with URL for web
- Web Recording: Design tab → Web → Start

### Citrix / VDI
- Use Computer Vision activities
- Or install UiPath Robot on the VM itself (preferred)

---

## Common Patterns

### Login Pattern
```
Use Application/Browser (CloseMode = IfOpenedByMe)
  Element Exists (check if already logged in) → isLoggedIn
  If Not isLoggedIn
    Click (Login link)
    Type Into (Username field, in_Credentials.UserName)
    TypeSecureText (Password field, in_Credentials.Password)
    Click (Submit button)
    Wait For Element (Dashboard element, WaitMode = Appear)
    Log Message: "Logged into system successfully"
```

### Table Row Processing Pattern
```
Extract Table Data → dt_TableData
For Each Row in dt_TableData (TypeArgument = DataRow)
  Assign: currentId = row("ID").ToString
  Click (row's action button using currentId in selector)
  [process row]
```

### Screenshot on Error Pattern
```
Try
  [UI interactions]
Catch (Exception e)
  Take Screenshot → screenshotPath
  Log Message: "Error: " + e.Message + " | Screenshot: " + screenshotPath
  Throw
```

---

## Performance Tips

- Set `DelayBetweenKeys = 0` in Type Into for speed (unless app needs slow input)
- Use `WaitForReady = None` for elements that don't need DOM ready
- Minimize use of `Delay` — always prefer `Wait For Element`
- Use `ContinueOnError = True` only when failure is genuinely acceptable
- Close browser/app with `CloseMode = IfOpenedByMe` to avoid orphan processes
- Use `Parallel` activity for independent UI automation sequences (attended robots only)
