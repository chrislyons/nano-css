# Component Map - Extended Documentation

## Module Overview

nano-css consists of a core renderer and 40+ addon modules. Each addon has a single responsibility and clearly defined dependencies.

## Core Renderer (`index.js`)

### Public API

| Method | Signature | Description |
|--------|-----------|-------------|
| `put` | `(selector: string, css: object, atrule?: string) => void` | Main CSS injection |
| `putRaw` | `(rawCss: string) => void` | Inject raw CSS string |
| `decl` | `(prop: string, val: any) => string` | Build CSS declaration |
| `kebab` | `(prop: string) => string` | Convert camelCase to kebab-case |
| `hash` | `(obj: object) => string` | Generate unique class name |
| `selector` | `(parent: string, sel: string) => string` | Build nested selector |
| `putAt` | `(selector: string, css: object, atrule: string) => void` | Handle @-rules |

### State Properties

| Property | Type | Description |
|----------|------|-------------|
| `client` | `boolean` | Browser environment detection |
| `raw` | `string` | SSR CSS accumulator |
| `pfx` | `string` | Class name prefix (default: `_`) |
| `sh` | `CSSStyleSheet` | Style sheet element reference |

## CSS Generation Addons

### rule (`addon/rule.js`)

**Responsibility**: Convert CSS objects to class names

**API**:
```javascript
nano.rule(css: object, block?: string): string
// Returns: ' _a' (space-prefixed class name)
```

**Dependencies**: Core `put` method

**How it works**:
1. Hash the CSS object to generate unique class name
2. Call `put()` to inject the CSS
3. Return the class name (space-prefixed for easy concatenation)

### sheet (`addon/sheet.js`)

**Responsibility**: Lazy-evaluated style collections

**API**:
```javascript
nano.sheet(map: object, block?: string): object
// Returns: Object with getters for each style
```

**Dependencies**: `rule` addon

**How it works**:
1. Creates result object with property getters
2. On first access, calls `rule()` for that style
3. Caches result by redefining property as value

**Example**:
```javascript
const styles = nano.sheet({
    button: {color: 'red'},
    link: {color: 'blue'}
});
// CSS injected only when accessed:
<div className={styles.button} />
```

### cache (`addon/cache.js`)

**Responsibility**: Memoize rule generation

**API**:
```javascript
nano.cache(css: object): string
// Returns: Memoized class name
```

**Dependencies**: `rule` addon

**How it works**:
- Maintains internal cache keyed by CSS object
- Returns cached class name if already generated
- Useful for dynamic styles that may repeat

### stable (`addon/stable.js`)

**Responsibility**: Deterministic class name hashing

**API**: Replaces `renderer.hash()`

**Dependencies**: `fastest-stable-stringify` package

**Why needed**: Default `JSON.stringify` produces different output for same object with different key order. This addon ensures consistent hashing.

## Dynamic CSS Addons

### drule (`addon/drule.js`)

**Responsibility**: Dynamic rules with runtime overrides

**API**:
```javascript
const dynamicBtn = nano.drule({
    color: 'red',
    fontSize: 12
}, 'btn');

// Returns function that accepts overrides
dynamicBtn({color: 'blue'}) // Returns class name
```

**Dependencies**: `rule`, `cache` addons

**Use case**: Responsive styles, theme variations

### dsheet (`addon/dsheet.js`)

**Responsibility**: Dynamic sheets with runtime overrides

**API**:
```javascript
const sheet = nano.dsheet({
    button: {color: 'red'}
}, 'block');

sheet.button({color: 'blue'}) // Override at runtime
```

**Dependencies**: `sheet`, `cache` addons

## React/VDOM Component Addons

### jsx (`addon/jsx.js`)

**Responsibility**: Create styled VDOM components

**API**:
```javascript
const Button = nano.jsx('button', {
    color: 'red',
    padding: '10px'
}, 'Button');

// Use: <Button onClick={...}>Click</Button>
```

**Dependencies**: `rule`, `cache` addons, `h` config option

**Features**:
- Accepts `css` prop for additional styles
- Accepts `as` prop for element type override
- Merges class names properly

### style (`addon/style.js`)

**Responsibility**: Styled components with dynamic templates

**API**:
```javascript
const Button = nano.style('button')({
    color: props => props.primary ? 'blue' : 'gray'
});
```

**Dependencies**: `jsx` addon

**Features**:
- Supports function values for dynamic styles
- Props are passed to style functions

### styled (`addon/styled.js`)

**Responsibility**: Tagged template literal API

**API**:
```javascript
const Button = nano.styled('button')`
    color: ${props => props.primary ? 'blue' : 'gray'};
    padding: 10px;
`;
```

**Dependencies**: `style` addon

**Features**: Familiar API for styled-components users

### withStyles (`addon/withStyles.js`)

**Responsibility**: HOC to inject styles prop

**API**:
```javascript
const enhance = withStyles({
    button: {color: 'red'}
});

const MyComponent = enhance(({styles}) => (
    <button className={styles.button}>Click</button>
));
```

**Dependencies**: `sheet` addon

### useStyles (`addon/useStyles.js`)

**Responsibility**: Inject styles as second argument

**API**:
```javascript
const enhance = useStyles('block');

const MyComponent = enhance((props, styles) => (
    <button className={styles.button}>Click</button>
), {
    button: {color: 'red'}
});
```

**Dependencies**: `sheet` addon

### component (`addon/component.js`)

**Responsibility**: React Component base class

**API**:
```javascript
class MyComponent extends nano.Component {
    render() {
        return <div className={this.css({color: 'red'})} />;
    }
}
```

**Dependencies**: `rule`, `cache` addons

### decorator (`addon/decorator.js`)

**Responsibility**: @css class/method decorators

**API**:
```javascript
@css({color: 'red'})
class MyComponent extends React.Component {
    render() {
        return <div className={this.css.root} />;
    }
}
```

**Dependencies**: `rule`, `cache` addons

## CSS Enhancement Addons

### atoms (`addon/atoms.js`)

**Responsibility**: Shorthand property names

**Wraps**: `decl` method

**Transformations**:
```javascript
{fz: 12}        → {fontSize: 12}
{m: 10}         → {margin: 10}
{p: '10px 20px'} → {padding: '10px 20px'}
{d: 'flex'}     → {display: 'flex'}
{ai: 'center'}  → {alignItems: 'center'}
// ... many more
```

### emmet (`addon/emmet.js`)

**Responsibility**: Emmet-style abbreviations

**Wraps**: `decl` method

**Transformations**:
```javascript
{fz12: 1}       → {fontSize: '12px'}
{m10-20: 1}     → {margin: '10px 20px'}
{c_red: 1}      → {color: 'red'}
```

### nesting (`addon/nesting.js`)

**Responsibility**: Advanced selector nesting with `&`

**Wraps**: `selector` method

**Examples**:
```javascript
{
    color: 'red',
    '&:hover': {color: 'blue'},
    '& .child': {color: 'green'},
    '.parent &': {color: 'purple'}
}
```

### prefixer (`addon/prefixer.js`)

**Responsibility**: Auto vendor prefix addition

**Wraps**: `decl` method

**Dependencies**: `inline-style-prefixer` package

**Transforms**:
```javascript
{display: 'flex'}
→ {display: ['-webkit-flex', 'flex']}
```

### unitless (`addon/unitless.js`)

**Responsibility**: Auto-add 'px' to numeric values

**Wraps**: `decl` method

**Transforms**:
```javascript
{width: 100}    → {width: '100px'}
{opacity: 0.5}  → {opacity: 0.5}  // No change (unitless prop)
{lineHeight: 1.5} → {lineHeight: 1.5}  // No change
```

### global (`addon/global.js`)

**Responsibility**: `:global` pseudo-selector support

**Wraps**: `putRaw` method

**Example**:
```javascript
nano.put(':global', {
    body: {margin: 0}
});
```

### array (`addon/array.js`)

**Responsibility**: Array values for multiple declarations

**Example**:
```javascript
{
    display: ['-webkit-flex', 'flex']
}
// Outputs both declarations
```

## Advanced Feature Addons

### keyframes (`addon/keyframes.js`)

**Responsibility**: @keyframes animation support

**API**:
```javascript
const fadeIn = nano.keyframes({
    from: {opacity: 0},
    to: {opacity: 1}
}, 'fadeIn');

// Use: {animation: `${fadeIn} 0.3s`}
```

**Uses**: `putRaw` method

### hydrate (`addon/hydrate.js`)

**Responsibility**: Re-hydrate server-rendered CSS

**API**:
```javascript
nano.hydrate(document.getElementById('nano-css'));
```

**How it works**:
1. Reads existing CSS selectors from style element
2. Registers them as already generated
3. Prevents duplicate CSS injection on client

### vcssom (`addon/vcssom.js`)

**Responsibility**: Virtual CSSOM with diffing

**API**:
```javascript
const style = nano.createRule({color: 'red'});
// Later:
style.update({color: 'blue'}); // Only diffs changed
style.remove();
```

**Dependencies**: `cssom` addon

**Use case**: Frequently updating styles, avoiding style bloat

### cssom (`addon/cssom.js`)

**Responsibility**: CSSOM API for rule manipulation

**API**:
```javascript
const style = nano.createRule({color: 'red'});
```

**Use case**: Foundation for vcssom

### sourcemaps (`addon/sourcemaps.js`)

**Responsibility**: Generate CSS sourcemaps in development

**Dependencies**: `stacktrace-js` package

**Note**: Should only be used in development

### validate (`addon/validate.js`)

**Responsibility**: Validate CSS syntax

**Dependencies**: `css-tree` package

**Note**: Useful for catching errors during development

## Dependency Graph Summary

```
Core: put, putRaw, decl, kebab, hash, selector
                    ↓
Generation: rule → sheet, cache → drule, dsheet
                    ↓
Components: jsx → style → styled
           rule/cache → component, decorator
           sheet → useStyles, withStyles
                    ↓
Preprocessing: atoms, nesting, prefixer, unitless (wrap decl/selector)
                    ↓
Advanced: keyframes, cssom → vcssom, hydrate
```

## Adding New Components

To add a new component addon:

1. Identify dependencies (usually `rule` or `cache`)
2. Follow the addon pattern
3. Add TypeScript definitions
4. Write tests
5. Add Storybook story
6. Document in `/docs`

## Common Extension Points

| Want to... | Wrap/Extend |
|------------|-------------|
| Transform CSS properties | `decl` method |
| Transform selectors | `selector` method |
| Transform entire rules | `put` method |
| Add new generation method | Add to `renderer` |
| Create component pattern | Build on `jsx` or `rule` |
