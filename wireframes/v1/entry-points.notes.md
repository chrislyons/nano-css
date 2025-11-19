# Entry Points - Extended Documentation

## Overview

nano-css provides multiple entry points for different use cases: direct package imports, presets, individual addons, and various style definition APIs.

## Package Entry Points

### Main Entry (`index.js`)

The primary entry point exports the `create` factory:

```javascript
const {create} = require('nano-css');

const nano = create({
    pfx: '_',
    h: React.createElement,
    verbose: process.env.NODE_ENV !== 'production'
});
```

**Use when**: You want full control over addon selection

### Preset Entries (`preset/*.js`)

Pre-configured bundles for common use cases:

#### Basic Preset (`preset/sheet.js`)

```javascript
import {preset} from 'nano-css/preset/sheet';

const nano = preset();
```

**Includes**: stable, nesting, atoms, keyframes, rule, sheet, sourcemaps (dev)

**Use when**: Basic styling without React

#### React Preset (`preset/react.js`)

```javascript
import {preset} from 'nano-css/preset/react';

const nano = preset({h: React.createElement});
```

**Includes**: All of sheet + cache, snake, jsx, style, styled, decorator

**Use when**: Full-featured React application

#### VDOM Preset (`preset/vdom.js`)

```javascript
import {preset} from 'nano-css/preset/vdom';

const nano = preset({h: h}); // Any hyperscript
```

**Includes**: cache, stable, nesting, atoms, keyframes, rule, sheet, jsx

**Use when**: Non-React VDOM libraries

## Configuration Options

### Required Options

| Option | Type | Required For |
|--------|------|--------------|
| `h` | `Function` | jsx, style, styled addons |

### Optional Options

| Option | Default | Purpose |
|--------|---------|---------|
| `pfx` | `'_'` | Class name prefix |
| `sh` | auto | Custom style sheet element |
| `verbose` | `false` | Enable debug logging |
| `assign` | `Object.assign` | Custom object merger |
| `stringify` | `JSON.stringify` | Custom stringifier |

### Example Configuration

```javascript
const nano = create({
    pfx: 'app-',           // Class names: app-a3f2
    h: React.createElement,
    verbose: true,
    sh: document.getElementById('my-styles'),
    assign: require('object-assign')
});
```

## Addon Loading

### Individual Addon Import

```javascript
import {create} from 'nano-css';
import {addon as rule} from 'nano-css/addon/rule';
import {addon as sheet} from 'nano-css/addon/sheet';
import {addon as nesting} from 'nano-css/addon/nesting';
import {addon as atoms} from 'nano-css/addon/atoms';

const nano = create();

// Apply addons in order
rule(nano);
sheet(nano);
nesting(nano);
atoms(nano);
```

### Order Matters

Addons wrap existing methods, so order affects behavior:

```javascript
// This order:
atoms(nano);     // 1. Expand shorthand first
prefixer(nano);  // 2. Then add prefixes

// Different from:
prefixer(nano);  // 1. Prefixer sees shorthand names
atoms(nano);     // 2. Atoms expand but prefixer missed them
```

### Common Addon Combinations

**Minimal**:
```javascript
rule(nano);
```

**Basic Styling**:
```javascript
rule(nano);
sheet(nano);
nesting(nano);
atoms(nano);
unitless(nano);
```

**React Components**:
```javascript
rule(nano);
cache(nano);
jsx(nano);
style(nano);
styled(nano);
nesting(nano);
atoms(nano);
prefixer(nano);
```

## Style Definition APIs

### Object API

#### nano.rule()

Generate a single class name from CSS object:

```javascript
const className = nano.rule({
    color: 'red',
    fontSize: '14px'
});
// Returns: ' _a3f2'

<div className={className} />
// or
<div className={'btn' + className} />
```

#### nano.sheet()

Create a collection of lazy-evaluated styles:

```javascript
const styles = nano.sheet({
    button: {
        color: 'white',
        backgroundColor: 'blue'
    },
    link: {
        color: 'blue',
        textDecoration: 'underline'
    }
});

// CSS injected only when accessed
<button className={styles.button}>Click</button>
```

#### nano.cache()

Memoized version of rule():

```javascript
// Good for dynamic styles
<div className={nano.cache({
    color: props.color,
    size: props.size
})} />
```

### Component API

#### nano.jsx()

Create styled VDOM components:

```javascript
const Button = nano.jsx('button', {
    padding: '10px 20px',
    borderRadius: '4px',
    border: 'none'
}, 'Button');

// Use
<Button onClick={handleClick}>Click me</Button>

// With additional styles
<Button css={{color: 'red'}}>Red button</Button>

// Override element type
<Button as="a" href="#">Link button</Button>
```

#### nano.style()

Styled components with dynamic styles:

```javascript
const Button = nano.style('button')({
    padding: '10px 20px',
    color: props => props.primary ? 'white' : 'black',
    backgroundColor: props => props.primary ? 'blue' : 'gray'
});

<Button primary>Primary</Button>
<Button>Secondary</Button>
```

#### nano.styled()

Tagged template literal API:

```javascript
const Button = nano.styled('button')`
    padding: 10px 20px;
    color: ${props => props.primary ? 'white' : 'black'};
    background: ${props => props.primary ? 'blue' : 'gray'};

    &:hover {
        opacity: 0.8;
    }
`;

<Button primary>Click</Button>
```

### HOC/Decorator API

#### withStyles()

Higher-order component pattern:

```javascript
import {addon as withStyles} from 'nano-css/addon/withStyles';

const styles = {
    root: {padding: '20px'},
    button: {color: 'blue'}
};

const MyComponent = ({styles}) => (
    <div className={styles.root}>
        <button className={styles.button}>Click</button>
    </div>
);

export default withStyles(styles)(MyComponent);
```

#### useStyles()

Inject styles as second argument:

```javascript
import {addon as useStyles} from 'nano-css/addon/useStyles';

const styles = {
    root: {padding: '20px'}
};

const MyComponent = (props, styles) => (
    <div className={styles.root}>
        {props.children}
    </div>
);

export default useStyles('block')(MyComponent, styles);
```

#### @css Decorator

Class and method decorators:

```javascript
import {addon as decorator} from 'nano-css/addon/decorator';

@css({
    root: {padding: '20px'},
    title: {fontSize: '24px'}
})
class MyComponent extends React.Component {
    render() {
        return (
            <div className={this.css.root}>
                <h1 className={this.css.title}>Title</h1>
            </div>
        );
    }
}
```

### Dynamic API

#### nano.drule()

Rules with runtime overrides:

```javascript
const buttonStyle = nano.drule({
    padding: '10px',
    color: 'black'
}, 'button');

// Base style
<button className={buttonStyle()}>Default</button>

// With overrides
<button className={buttonStyle({color: 'red'})}>Red</button>
```

#### nano.dsheet()

Sheets with runtime overrides:

```javascript
const styles = nano.dsheet({
    button: {color: 'black'},
    link: {color: 'blue'}
}, 'block');

// With overrides
<button className={styles.button({color: 'red'})}>
    Red Button
</button>
```

## Advanced APIs

### nano.keyframes()

Define CSS animations:

```javascript
const fadeIn = nano.keyframes({
    from: {opacity: 0},
    to: {opacity: 1}
}, 'fadeIn');

const styles = nano.sheet({
    animated: {
        animation: `${fadeIn} 0.3s ease-in`
    }
});
```

### nano.global()

Inject global styles:

```javascript
nano.global({
    body: {
        margin: 0,
        fontFamily: 'sans-serif'
    },
    '*': {
        boxSizing: 'border-box'
    }
});
```

### nano.hydrate()

Re-hydrate SSR styles:

```javascript
// Client-side only
if (typeof window !== 'undefined') {
    nano.hydrate(document.getElementById('nano-css'));
}
```

### nano.createRule()

Virtual CSSOM for efficient updates:

```javascript
const dynamicStyle = nano.createRule({
    transform: 'scale(1)'
});

// Update efficiently
element.onmousemove = (e) => {
    dynamicStyle.update({
        transform: `translate(${e.clientX}px, ${e.clientY}px)`
    });
};

// Cleanup
element.onmouseleave = () => {
    dynamicStyle.remove();
};
```

## Development Entry Points

### Storybook

```bash
yarn storybook
# Opens http://localhost:6010
```

Visual testing for all addons. Stories located in `.storybook/`.

### Demo

```bash
yarn demo
# Opens webpack-dev-server
```

Example application in `demo/` directory.

### Tests

```bash
yarn test           # Run all tests
yarn test:coverage  # With coverage report
```

Jest tests in `addon/__tests__/`.

## Build Scripts

### Linting

```bash
yarn lint           # ESLint check
yarn prettier       # Format TypeScript files
```

### CI Pipeline

Defined in `.github/workflows/`:

- **checks.yml** - Runs on PRs, tests Node 20.x
- **gh-pages.yml** - Deploys documentation
- **release.yml** - Automated npm releases

## Environment Differences

### Browser Environment

```javascript
// Detection
renderer.client = typeof window !== 'undefined';

// Behavior
if (renderer.client) {
    // Use .insertRule() for fast injection
    sheet.insertRule(css, sheet.cssRules.length);
}
```

**Characteristics**:
- Direct CSSOM manipulation
- Fast style injection
- No string accumulation

### Server Environment (SSR)

```javascript
// Behavior
if (!renderer.client) {
    // Accumulate CSS string
    renderer.raw += css;
}
```

**Characteristics**:
- String accumulation in `raw`
- No DOM access
- Output for HTML embedding

### Test Environment

```javascript
// Setup in addon/__tests__/setup.js
process.env.NODE_ENV = 'production';

// Mock requestAnimationFrame
global.requestAnimationFrame = function (callback) {
    return setTimeout(callback, 17);
};
```

**Characteristics**:
- JSDOM for DOM simulation
- RAF mocked for animations
- Snapshot testing for CSS output

## TypeScript Entry Points

### Main Types (`index.d.ts`)

```typescript
import {create, NanoRenderer} from 'nano-css';

const nano: NanoRenderer = create();
```

### Addon Types (`addon/*.d.ts`)

Each addon has corresponding type definitions:

```typescript
import {addon as rule} from 'nano-css/addon/rule';
import {RuleAddon} from 'nano-css/addon/rule';

declare module 'nano-css' {
    interface NanoRenderer extends RuleAddon {}
}
```

### Core Interfaces (`types/nano.d.ts`)

```typescript
interface NanoRenderer {
    client: boolean;
    raw: string;
    pfx: string;
    // ...
}

interface NanoOptions {
    pfx?: string;
    h?: Function;
    // ...
}

type CreateNano = (options?: NanoOptions) => NanoRenderer;
```

## Common Integration Patterns

### Next.js Integration

```javascript
// pages/_document.js
import Document from 'next/document';

class MyDocument extends Document {
    static async getInitialProps(ctx) {
        const initialProps = await Document.getInitialProps(ctx);
        return {
            ...initialProps,
            styles: (
                <>
                    {initialProps.styles}
                    <style id="nano-css">{nano.raw}</style>
                </>
            )
        };
    }
}
```

### Create React App

```javascript
// src/nano.js
import {preset} from 'nano-css/preset/react';
import {createElement} from 'react';

export const nano = preset({h: createElement});
```

### Webpack Configuration

No special configuration needed - nano-css works with standard ES module bundling.
