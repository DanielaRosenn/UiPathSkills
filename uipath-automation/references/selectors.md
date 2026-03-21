# UiPath Selectors Reference

Complete guide to UiPath UI selectors for automation targeting.

## Table of Contents
1. [Selector Fundamentals](#selector-fundamentals)
2. [Selector Syntax](#selector-syntax)
3. [Desktop Selectors](#desktop-selectors)
4. [Web Selectors](#web-selectors)
5. [Dynamic Selectors](#dynamic-selectors)
6. [Best Practices](#best-practices)

---

## Selector Fundamentals

### What is a Selector?
A selector is an XML string that identifies a UI element. It contains a hierarchy of tags representing the element's path from the window to the target.

### Selector Structure
```xml
<wnd app='notepad.exe' title='Untitled - Notepad'/>
<wnd cls='Edit' />
```

Each tag represents a level in the UI hierarchy, with attributes narrowing down the specific element.

---

## Selector Syntax

### Basic Structure
```xml
<tag attribute1='value1' attribute2='value2' ... />
```

### Common Tag Types

| Tag | Description | Used For |
|-----|-------------|----------|
| `<wnd>` | Window element | Desktop windows |
| `<ctrl>` | Control element | Desktop controls |
| `<html>` | Browser window | Web application root |
| `<webctrl>` | Web control | HTML elements |
| `<java>` | Java element | Java applications |
| `<sap>` | SAP element | SAP GUI |

### Common Attributes

| Attribute | Description | Example |
|-----------|-------------|---------|
| `app` | Application name | `app='chrome.exe'` |
| `title` | Window title | `title='Dashboard'` |
| `cls` | Class name | `cls='Button'` |
| `name` | Control name | `name='btnSubmit'` |
| `id` | Element ID | `id='username'` |
| `tag` | HTML tag | `tag='INPUT'` |
| `class` | CSS class | `class='form-control'` |
| `innertext` | Element text | `innertext='Login'` |
| `idx` | Index (position) | `idx='2'` |

### Attribute Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `=` | Exact match | `id='submit'` |
| `*` | Wildcard (any) | `title='*Report*'` |
| `?` | Single char wildcard | `name='btn?'` |

---

## Desktop Selectors

### Windows Application Selector
```xml
<wnd app='notepad.exe' cls='Notepad' title='*Notepad' />
<wnd cls='Edit' />
```

### With Control Hierarchy
```xml
<wnd app='calculator.exe' cls='ApplicationFrameWindow' title='Calculator' />
<wnd cls='Windows.UI.Core.CoreWindow' />
<ctrl name='Seven' role='button' />
```

### Dialog Box Selector
```xml
<wnd app='myapp.exe' title='Main Window' />
<wnd cls='#32770' title='Save As' />
<ctrl name='File name:' role='combobox' />
```

### Citrix/Virtual Desktop
```xml
<wnd app='wfcrun32.exe' title='*Citrix*' />
<wnd cls='Transparent Windows Client' />
<ctrl name='Username' role='editable text' />
```

---

## Web Selectors

### Basic Web Selector
```xml
<html app='chrome.exe' title='Login Page' />
<webctrl tag='INPUT' id='username' />
```

### With CSS Class
```xml
<html app='chrome.exe' />
<webctrl tag='BUTTON' class='btn btn-primary' innertext='Submit' />
```

### Nested Elements
```xml
<html app='chrome.exe' title='Dashboard' />
<webctrl tag='DIV' id='main-container' />
<webctrl tag='TABLE' class='data-table' />
<webctrl tag='TR' />
<webctrl tag='TD' idx='2' />
```

### Form Elements
```xml
<!-- Text Input -->
<html app='chrome.exe' />
<webctrl tag='INPUT' type='text' name='email' />

<!-- Dropdown -->
<html app='chrome.exe' />
<webctrl tag='SELECT' id='country' />

<!-- Checkbox -->
<html app='chrome.exe' />
<webctrl tag='INPUT' type='checkbox' id='agree' />

<!-- Radio Button -->
<html app='chrome.exe' />
<webctrl tag='INPUT' type='radio' name='payment' value='credit' />
```

### Tables
```xml
<!-- Entire table -->
<html app='chrome.exe' />
<webctrl tag='TABLE' id='dataTable' />

<!-- Specific cell by column header -->
<html app='chrome.exe' />
<webctrl tag='TABLE' id='dataTable' />
<webctrl tag='TR' tableRow='3' />
<webctrl tag='TD' tableCol='Status' />

<!-- Cell by index -->
<html app='chrome.exe' />
<webctrl tag='TABLE' id='dataTable' />
<webctrl tag='TBODY' />
<webctrl tag='TR' idx='2' />
<webctrl tag='TD' idx='4' />
```

### iFrame Handling
```xml
<html app='chrome.exe' title='Main Page' />
<webctrl tag='IFRAME' id='content-frame' />
<webctrl tag='INPUT' id='fieldInsideFrame' />
```

### Shadow DOM (requires UiPath 2021.10+)
```xml
<html app='chrome.exe' />
<webctrl tag='MY-COMPONENT' />
<webctrl tag='INPUT' parentid='my-component' />
```

---

## Dynamic Selectors

### Using Variables
Variables in selectors are wrapped in double curly braces or brackets:

```xml
<!-- Using variable placeholder -->
<webctrl tag='TR' innertext='{{orderNumber}}' />

<!-- In XAML with variable binding -->
<ui:Click>
  <ui:Click.Target>
    <ui:Target>
      <ui:Target.Selector>
        <InArgument x:TypeArguments="x:String">["<html app='chrome.exe'/><webctrl tag='TR' innertext='" + strOrderNumber + "'/>"]</InArgument>
      </ui:Target.Selector>
    </ui:Target>
  </ui:Click.Target>
</ui:Click>
```

### Wildcard Patterns
```xml
<!-- Match any text containing "Invoice" -->
<webctrl tag='A' innertext='*Invoice*' />

<!-- Match varying window titles -->
<wnd app='excel.exe' title='*Budget*.xlsx*' />

<!-- Match with single character wildcard -->
<webctrl tag='INPUT' id='field_?' />
```

### Index-Based Selection
Use sparingly - index is position-dependent:
```xml
<!-- Third button on page -->
<webctrl tag='BUTTON' idx='3' />

<!-- Second row in table body -->
<webctrl tag='TBODY' />
<webctrl tag='TR' idx='2' />
```

### Anchor-Based Selection
Use a stable element as anchor to find dynamic elements:
```xml
<!-- Using anchor in UiPath -->
<ui:Click>
  <ui:Click.Target>
    <ui:Target Selector="<webctrl tag='A' innertext='Edit' />" />
  </ui:Click.Target>
  <ui:Click.Anchor>
    <ui:AnchorableTarget Selector="<webctrl tag='TD' innertext='{{customerName}}' />" />
  </ui:Click.Anchor>
</ui:Click>
```

---

## Best Practices

### Selector Reliability Rules

1. **Avoid Index (idx)** - Changes with UI modifications
2. **Prefer IDs** - Most stable attribute
3. **Use Partial Wildcards** - Handle dynamic content
4. **Minimize Hierarchy** - Fewer tags = more stable
5. **Test with UI Explorer** - Validate selectors before use

### Reliability Priority Order
1. `id` (most reliable)
2. `name`
3. `class` + `tag`
4. `innertext` with wildcard
5. `idx` (least reliable, use as last resort)

### Common Selector Issues

| Issue | Solution |
|-------|----------|
| Dynamic IDs | Use wildcards or partial match |
| Changing index | Use anchor or find stable parent |
| Multiple matches | Add more attributes to narrow |
| Timing issues | Use Element Exists + Wait |
| iFrame content | Include iframe in selector hierarchy |

### Selector Optimization Examples

**Bad - Uses index:**
```xml
<html app='chrome.exe' />
<webctrl tag='DIV' idx='5' />
<webctrl tag='BUTTON' idx='2' />
```

**Good - Uses stable attributes:**
```xml
<html app='chrome.exe' />
<webctrl tag='DIV' class='action-panel' />
<webctrl tag='BUTTON' innertext='Save' />
```

**Bad - Full title match:**
```xml
<wnd title='Invoice #12345 - Application v2.1.0' />
```

**Good - Partial match:**
```xml
<wnd title='Invoice*' />
```

### Fuzzy Selector (Modern Activities)
```xml
<ui:ClickX DisplayName="Click Save">
  <ui:ClickX.Target>
    <ui:Target>
      <ui:Target.FuzzySelector>
        <ui:FuzzySelector WindowSelector="<wnd app='myapp.exe' />" TargetText="Save" ScreenRegion="Right" />
      </ui:Target.FuzzySelector>
    </ui:Target>
  </ui:ClickX.Target>
</ui:ClickX>
```

### Strict vs Fuzzy vs Image

| Method | Use When |
|--------|----------|
| **Strict Selector** | Elements have stable attributes |
| **Fuzzy Selector** | Text-based identification needed |
| **Image Matching** | No usable selector, Citrix/VDI |

### Selector in XAML Pattern
Complete selector with all options:

```xml
<ui:Click DisplayName="Click Dynamic Element" Timeout="30000">
  <ui:Click.CursorPosition>
    <ui:CursorPosition Position="Center" />
  </ui:Click.CursorPosition>
  <ui:Click.Target>
    <ui:Target Timeout="5000" WaitForReady="COMPLETE">
      <ui:Target.Selector>
        <InArgument x:TypeArguments="x:String">["<html app='chrome.exe' title='*Dashboard*'/><webctrl tag='BUTTON' class='action-btn' innertext='" + strButtonText + "'/>"]</InArgument>
      </ui:Target.Selector>
    </ui:Target>
  </ui:Click.Target>
</ui:Click>
```
