# Data Flow - Extended Documentation

## Overview

nano-css processes CSS through a pipeline of transformations, from user-defined style objects to injected CSS in the browser or accumulated strings for SSR.

## Input Types

### 1. CSS Objects

The most common input format:

```javascript
const styles = {
    color: 'red',
    fontSize: 12,
    '&:hover': {
        color: 'blue'
    }
};

const className = nano.rule(styles);
```

### 2. Tagged Template Literals

For developers familiar with styled-components:

```javascript
const Button = nano.styled('button')`
    color: red;
    padding: 10px;

    &:hover {
        color: blue;
    }
`;
```

### 3. Dynamic Functions

For runtime-computed styles:

```javascript
const Button = nano.style('button')({
    color: props => props.primary ? 'blue' : 'gray',
    fontSize: props => props.size || 14
});
```

## Preprocessing Pipeline

Styles pass through addon middleware in the order they're applied. Each addon transforms the CSS before passing it on.

### Typical Pipeline Order

1. **atoms** - Expand shorthand properties
2. **emmet** - Expand Emmet abbreviations
3. **nesting** - Resolve `&` selectors
4. **unitless** - Add `px` units
5. **prefixer** - Add vendor prefixes

### Example Transformation

```javascript
// Input
{fz: 12, d: 'flex', '&:hover': {c: 'blue'}}

// After atoms
{fontSize: 12, display: 'flex', '&:hover': {color: 'blue'}}

// After unitless
{fontSize: '12px', display: 'flex', '&:hover': {color: 'blue'}}

// After prefixer
{fontSize: '12px', display: ['-webkit-flex', 'flex'], '&:hover': {color: 'blue'}}

// After nesting (with parent .btn)
// Main: {fontSize: '12px', display: ['-webkit-flex', 'flex']}
// Nested: .btn:hover {color: blue}
```

## Class Generation

### Hashing Process

```javascript
renderer.hash = function (obj) {
    // 1. Stringify the object
    var str = JSON.stringify(obj);
    // (or fastest-stable-stringify with 'stable' addon)

    // 2. Generate hash
    var hash = 0;
    for (var i = 0; i < str.length; i++) {
        hash = ((hash << 5) - hash) + str.charCodeAt(i);
        hash |= 0;
    }

    // 3. Return base36 string
    return (hash >>> 0).toString(36);
};
```

### Class Name Format

```
[prefix][hash]
   _     a3f2

Default prefix: '_'
Result: '_a3f2'
```

### Cache Lookup

The `cache` addon prevents regenerating styles:

```javascript
var cache = {};

renderer.cache = function (css) {
    var key = renderer.hash(css);

    if (!cache[key]) {
        cache[key] = renderer.rule(css);
    }

    return cache[key];
};
```

## Core put() Method

### Building Declarations

```javascript
renderer.decl = function (prop, val) {
    // Convert camelCase to kebab-case
    var kebabProp = renderer.kebab(prop);

    // Build declaration
    return kebabProp + ':' + val + ';';
};

// Example:
// decl('fontSize', '12px') → 'font-size:12px;'
```

### Building Rules

```javascript
renderer.put = function (selector, css, atrule) {
    var str = '';

    // Build all declarations
    for (var prop in css) {
        if (typeof css[prop] === 'object') {
            // Handle nested selectors
            renderer.put(
                renderer.selector(selector, prop),
                css[prop]
            );
        } else {
            str += renderer.decl(prop, css[prop]);
        }
    }

    // Output the rule
    if (str) {
        renderer.putRaw(selector + '{' + str + '}');
    }
};
```

## Browser Environment Flow

### Style Sheet Injection

```javascript
// Get or create style element
if (!renderer.sh) {
    renderer.sh = document.createElement('style');
    document.head.appendChild(renderer.sh);
}

// Get the CSSStyleSheet
var sheet = renderer.sh.sheet;
```

### Using .insertRule()

```javascript
renderer.putRaw = function (rawCss) {
    if (renderer.client) {
        // Browser: use fast CSSOM API
        sheet.insertRule(rawCss, sheet.cssRules.length);
    } else {
        // Server: accumulate string
        renderer.raw += rawCss;
    }
};
```

### Why .insertRule()?

Performance comparison:

| Method | Speed | Repaint |
|--------|-------|---------|
| `innerHTML +=` | Slow | Full repaint |
| `textContent +=` | Slow | Full repaint |
| `.insertRule()` | Fast | Minimal |

`.insertRule()` directly manipulates CSSOM without re-parsing.

## Server Environment Flow

### CSS Accumulation

```javascript
renderer.putRaw = function (rawCss) {
    renderer.raw += rawCss;
};
```

### SSR Output

```javascript
// After rendering components
const html = `
<!DOCTYPE html>
<html>
<head>
    <style id="nano-css">${nano.raw}</style>
</head>
<body>
    ${renderedContent}
</body>
</html>
`;
```

## Client Hydration Flow

When the client receives SSR HTML, it needs to avoid re-injecting existing styles.

### Hydration Process

```javascript
nano.hydrate = function (sh) {
    // 1. Get existing style element
    var sheet = sh.sheet;

    // 2. Parse all existing selectors
    var rules = sheet.cssRules;
    for (var i = 0; i < rules.length; i++) {
        var selector = rules[i].selectorText;
        // 3. Register in cache
        hydrated[selector] = true;
    }
};

// Modified put checks hydration
renderer.put = function (selector, css) {
    if (hydrated[selector]) {
        return; // Skip - already exists
    }
    // Continue with normal injection
};
```

### Usage

```javascript
// Client-side initialization
import {create} from 'nano-css';
import {addon as hydrate} from 'nano-css/addon/hydrate';

const nano = create();
hydrate(nano);

// Hydrate existing styles
nano.hydrate(document.getElementById('nano-css'));
```

## Virtual CSSOM Flow

For frequently updating styles, the `vcssom` addon provides efficient diffing.

### Creating Virtual Rules

```javascript
const style = nano.createRule({
    color: 'red',
    fontSize: '14px'
});

// Returns handle with methods
```

### Updating Styles

```javascript
// Only changed properties are updated
style.update({
    color: 'blue',      // Changed - update
    fontSize: '14px'    // Same - skip
});
```

### Removing Styles

```javascript
style.remove(); // Removes from CSSOM
```

### Internal Diffing

```javascript
function diff(oldStyles, newStyles) {
    var changes = [];

    for (var prop in newStyles) {
        if (oldStyles[prop] !== newStyles[prop]) {
            changes.push({prop, value: newStyles[prop]});
        }
    }

    for (var prop in oldStyles) {
        if (!(prop in newStyles)) {
            changes.push({prop, value: null}); // Remove
        }
    }

    return changes;
}
```

## State Management

### Renderer State

nano-css doesn't use external state management. State is contained in the renderer:

```javascript
{
    raw: '',        // Accumulated CSS (SSR)
    client: true,   // Environment flag
    pfx: '_',       // Class prefix
    sh: null        // Style sheet reference
}
```

### Caching State

The `cache` addon maintains internal state:

```javascript
var cache = {};  // {hash: className}
```

### Hydration State

The `hydrate` addon tracks existing selectors:

```javascript
var hydrated = {};  // {selector: true}
```

## Event Flows

### Component Render Flow

```
Component Render
    ↓
Call nano.rule/cache/jsx
    ↓
Check cache
    ↓ (miss)
Process through addons
    ↓
Generate class name
    ↓
Inject CSS
    ↓
Return class name
    ↓
Apply to element
```

### Style Update Flow (vcssom)

```
Props Change
    ↓
Call style.update(newStyles)
    ↓
Diff old vs new
    ↓
Calculate changes
    ↓
Apply only changes to CSSOM
```

## Performance Considerations

### Minimize Unique Styles

Each unique style object creates a new CSS rule:

```javascript
// Bad: Creates new rule each render
<div className={nano.rule({color: props.color})} />

// Good: Cache dynamic styles
<div className={nano.cache({color: props.color})} />
```

### Use sheet() for Static Styles

```javascript
// Creates rules lazily, caches automatically
const styles = nano.sheet({
    button: {color: 'red'},
    link: {color: 'blue'}
});
```

### Use vcssom for Frequent Updates

```javascript
// For animations or real-time updates
const style = nano.createRule({transform: 'scale(1)'});

// Efficient updates
onMouseMove = (e) => {
    style.update({transform: `translate(${e.x}px, ${e.y}px)`});
};
```

## Debugging Data Flow

### Enable Verbose Mode

```javascript
const nano = create({verbose: true});
```

### Use Sourcemaps

```javascript
import {addon as sourcemaps} from 'nano-css/addon/sourcemaps';
sourcemaps(nano);
```

### Validate CSS

```javascript
import {addon as validate} from 'nano-css/addon/validate';
validate(nano);
// Throws on invalid CSS syntax
```

### Inspect Generated CSS

```javascript
// Browser: Check <style> element in DevTools
// Server: console.log(nano.raw)
```
