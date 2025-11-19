# Architecture Overview - Extended Documentation

## Design Philosophy

nano-css is a 5th generation CSS-in-JS library built around these core principles:

1. **Minimal Core** - Base renderer is only ~0.5KB
2. **Plugin Architecture** - All features via composable addons
3. **Isomorphic** - Works identically on server and browser
4. **Library Agnostic** - Supports React, Preact, Vue, or standalone
5. **Performance First** - Uses `.insertRule()` for fast CSS injection

## Core Renderer

### The `create()` Factory

```javascript
const {create} = require('nano-css');
const nano = create({
    pfx: '_',           // Class name prefix
    h: React.createElement,  // Hyperscript function
    verbose: true       // Development logging
});
```

### Renderer Methods

| Method | Purpose | Used By |
|--------|---------|---------|
| `put(selector, css, atrule)` | Main CSS injection point | All addons |
| `putRaw(rawCss)` | Inject raw CSS string | keyframes, global styles |
| `decl(prop, val)` | Build CSS declaration | prefixer, atoms |
| `kebab(prop)` | Convert camelCase → kebab-case | decl |
| `hash(obj)` | Generate unique class name | rule, cache |
| `selector(parent, sel)` | Build nested selectors | nesting |

### Renderer State

| Property | Type | Purpose |
|----------|------|---------|
| `raw` | string | Accumulates CSS for SSR |
| `pfx` | string | Prefix for generated class names |
| `client` | boolean | Detects browser environment |
| `sh` | CSSStyleSheet | Reference to style element |

## Addon Layer Architecture

### Middleware Pattern

Addons extend the renderer using a middleware pattern:

```javascript
exports.addon = function (renderer) {
    var originalPut = renderer.put;

    renderer.put = function (selector, css, atrule) {
        // Pre-process CSS
        css = transform(css);

        // Call original
        return originalPut(selector, css, atrule);
    };
};
```

### Addon Categories

#### 1. CSS Generation Addons

**Purpose**: Convert style objects to CSS class names

| Addon | API | Returns |
|-------|-----|---------|
| `rule` | `nano.rule({color: 'red'})` | `' _a'` (class name) |
| `sheet` | `nano.sheet({btn: {...}}).btn` | Lazy-evaluated class |
| `cache` | `nano.cache({...})` | Memoized class name |

**Key Pattern**: These form the foundation other addons build upon.

#### 2. CSS Preprocessing Addons

**Purpose**: Transform CSS before injection

| Addon | Transforms |
|-------|------------|
| `atoms` | `{fz: 12}` → `{fontSize: 12}` |
| `nesting` | `{'&:hover': {...}}` → resolved selector |
| `prefixer` | `{display: 'flex'}` → with vendor prefixes |
| `unitless` | `{width: 100}` → `{width: '100px'}` |

**Key Pattern**: Each wraps `put` or `decl` to intercept and transform CSS.

#### 3. Component API Addons

**Purpose**: Create styled components for virtual DOM

| Addon | API Style | Example |
|-------|-----------|---------|
| `jsx` | Factory | `nano.jsx('div', {color: 'red'})` |
| `styled` | Tagged template | `nano.styled('div')\`color: red\`` |
| `withStyles` | HOC | `withStyles(styles)(Component)` |
| `decorator` | Decorator | `@css({color: 'red'})` |

**Key Pattern**: All ultimately use `rule` or `cache` to generate class names.

#### 4. Advanced Feature Addons

**Purpose**: Specialized capabilities

| Addon | Capability |
|-------|------------|
| `keyframes` | `@keyframes` animation support |
| `vcssom` | Virtual CSSOM with diffing |
| `hydrate` | Re-hydrate server-rendered CSS |
| `stylis` | Full CSS preprocessing |

## Data Flow Architecture

### Style Definition → CSS Output

```
Style Object
    ↓
[Preprocessing Addons]
atoms → nesting → prefixer → unitless
    ↓
[Generation Addons]
rule → hash → put
    ↓
[Core Renderer]
put → decl → kebab
    ↓
[Output]
Browser: .insertRule()
Server: raw += css
```

### Server-Side Rendering Flow

```
1. Server renders components
2. nano.put() calls accumulate in nano.raw
3. Output: <style>${nano.raw}</style>
4. Client receives pre-rendered HTML
5. nano.hydrate() prevents duplicate styles
6. Client continues with new styles
```

## External Dependencies

### Runtime Dependencies

| Package | Purpose | Size Impact |
|---------|---------|-------------|
| `csstype` | TypeScript CSS types | Types only |
| `fastest-stable-stringify` | Deterministic JSON | ~1KB |
| `inline-style-prefixer` | Vendor prefixes | ~8KB |
| `rtl-css-js` | RTL transformation | ~3KB |
| `stylis` | CSS preprocessing | ~3KB |

### Peer Dependencies

- `react` / `react-dom` - For React integration addons

## Key Architectural Decisions

### Why Plugin Architecture?

1. **Tree-shaking** - Only include what you use
2. **Customization** - Mix and match features
3. **Bundle size** - Keep base small
4. **Separation of concerns** - Each addon does one thing

### Why `.insertRule()`?

Performance comparison:
- `innerHTML` - Slower, triggers reparse
- `.insertRule()` - Fast, direct CSSOM manipulation

### Why Multiple Component APIs?

Different mental models for different developers:
- `jsx()` - Familiar to Hyperscript users
- `styled()` - Familiar to styled-components users
- `withStyles()` - Familiar to Material-UI users
- `decorator()` - Familiar to Angular/MobX users

## Technology Stack

### Languages
- JavaScript (ES5/ES6) - Source code
- TypeScript - Type definitions only

### Build & Test
- Jest - Unit testing
- Storybook - Visual testing
- Webpack - Demo/Storybook bundling
- ESLint + Prettier - Code quality

### CI/CD
- GitHub Actions
- Automated testing
- Automated releases
- Documentation deployment

## Patterns to Understand

### 1. The Addon Pattern

Every addon follows this structure:

```javascript
exports.addon = function (renderer, config) {
    // Check dependencies (dev only)
    if (process.env.NODE_ENV !== 'production') {
        require('./__dev__/warnOnMissingDependencies')('name', renderer, ['dep']);
    }

    // Extend renderer
    renderer.newMethod = function () { ... };
};
```

### 2. The Lazy Evaluation Pattern

`sheet()` uses property getters:

```javascript
Object.defineProperty(result, key, {
    get: function () {
        var cn = renderer.rule(styles[key]);
        Object.defineProperty(result, key, {value: cn});
        return cn;
    }
});
```

### 3. The Wrapping Pattern

Addons wrap existing methods:

```javascript
var original = renderer.put;
renderer.put = function (sel, css, atrule) {
    // Transform
    return original(sel, transformed, atrule);
};
```

## Common Architecture Questions

### Q: Where do I add a new CSS transformation?

Create an addon that wraps `put` or `decl`:
- Wrap `decl` for property-level transforms
- Wrap `put` for selector/rule-level transforms

### Q: How do I support a new VDOM library?

Pass the hyperscript function in config:
```javascript
create({h: VueCreateElement})
```

### Q: How do I optimize for production?

1. Use presets or cherry-pick addons
2. Enable `stable` addon for deterministic class names
3. Use `extract` addon for external stylesheets
4. Remove `sourcemaps` addon in production

### Q: How do I debug style issues?

1. Enable `verbose: true` in config
2. Add `sourcemaps` addon
3. Add `validate` addon for CSS syntax checking
4. Check browser DevTools for generated styles
