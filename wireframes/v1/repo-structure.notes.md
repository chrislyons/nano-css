# Repository Structure - Extended Documentation

## Overview

nano-css follows a plugin-based architecture where the core is minimal (~0.5KB) and all features are provided through addons. This structure enables extreme customization and tree-shaking.

## Key Directories

### Root Level Files

| File | Purpose |
|------|---------|
| `index.js` | Main entry point - exports the `create()` function that instantiates a renderer |
| `index.d.ts` | TypeScript type definitions for the core module |
| `package.json` | Project configuration, dependencies, and npm scripts |
| `README.md` | Project overview and quick start guide |

### `/addon` - The Plugin System

This is the heart of nano-css. Contains 40+ addon modules that extend the base renderer.

#### Organization Pattern

Each addon typically has:
- `{name}.js` - The addon implementation
- `{name}.d.ts` - TypeScript definitions
- Related tests in `__tests__/{name}.test.js`
- Documentation in `docs/{name}.md`

#### Addon Categories

**Core CSS Generation** (`rule.js`, `sheet.js`, `cache.js`, `stable.js`)
- Foundation for generating CSS class names
- Start here when understanding the codebase

**Dynamic CSS** (`drule.js`, `dsheet.js`)
- Runtime style overrides
- Used for responsive/interactive styles

**React/VDOM Integration** (`jsx.js`, `style.js`, `styled.js`, etc.)
- Component-based styling APIs
- Familiar patterns for React developers

**CSS Enhancement** (`atoms.js`, `emmet.js`, `nesting.js`, etc.)
- Syntactic sugar and preprocessing
- Improves developer experience

**Advanced Rendering** (`keyframes.js`, `cssom.js`, `vcssom.js`, etc.)
- Performance optimizations
- Animation support
- Virtual CSSOM for diffing

**Utilities** (`hydrate.js`, `extract.js`, `validate.js`, etc.)
- SSR support
- Development tools
- Production optimizations

#### Sub-directories

| Directory | Contents |
|-----------|----------|
| `__dev__/` | Development-only utilities like dependency warnings |
| `__tests__/` | Jest test suites with client/server/dev variants |
| `animate/` | Pre-built animation keyframes (fadeIn, fadeOut, etc.) |
| `reset/` | 13 different CSS reset stylesheets |
| `stylis/` | Stylis preprocessor plugins |
| `util/` | Shared utility functions |
| `vcssom/` | Virtual CSSOM utilities |

### `/preset` - Pre-configured Bundles

Presets combine commonly-used addons for specific use cases:

| Preset | Use Case | Addons Included |
|--------|----------|-----------------|
| `sheet.js` | Basic styling | stable, nesting, atoms, keyframes, rule, sheet |
| `react.js` | React apps | All of sheet + cache, jsx, style, styled, decorator |
| `vdom.js` | Virtual DOM | cache, stable, nesting, atoms, keyframes, rule, sheet, jsx |

**When to use**: Start with a preset for common use cases, then add individual addons as needed.

### `/types` - TypeScript Support

| File | Contents |
|------|----------|
| `nano.d.ts` | Core interfaces (NanoRenderer, NanoOptions, CreateNano) |
| `common.d.ts` | CSS property types (leverages `csstype` package) |
| `demo.ts` | Example usage with types |

### `/docs` - Documentation

Contains 46+ markdown files documenting:
- Installation and setup
- Each addon's API and usage
- Presets and configuration
- Server-side rendering guides

**Note**: Documentation is deployed to GitHub Pages via CI/CD.

### `/demo` - Example Applications

Simple demo setup showing nano-css in action:
- `demo1.html` - HTML entry point
- `demo1.tsx` - React component demo

Run with: `yarn demo`

### `/.storybook` - Visual Testing

Comprehensive visual test suite with 40+ stories:
- Stories for each major addon
- Sub-directories for complex features (`styled/`, `vcssom/`)
- Custom webpack configuration for JSX support

**Key for**: Understanding how addons work visually, regression testing

Run with: `yarn storybook`

### `/.github` - CI/CD Workflows

| Workflow | Purpose |
|----------|---------|
| `checks.yml` | Runs tests on Node 20.x for PRs |
| `gh-pages.yml` | Deploys documentation |
| `release.yml` | Automated npm releases |

## Code Organization Patterns

### Finding Code by Feature

| Want to... | Look in... |
|------------|------------|
| Understand core rendering | `index.js` |
| Add a new CSS feature | `addon/` directory |
| Create React components | `addon/jsx.js`, `addon/styled.js` |
| Add animations | `addon/keyframes.js`, `addon/animate/` |
| Support SSR | `addon/hydrate.js` |
| Add vendor prefixes | `addon/prefixer.js` |
| Use shorthand props | `addon/atoms.js` |

### Test Files Convention

- `*.test.js` - Standard tests (browser environment)
- `*.server.test.js` - Server-side rendering tests
- `*.dev.test.js` - Development mode tests

### Type Definition Convention

Every addon that exports functionality has a corresponding `.d.ts` file in the same directory.

## Technical Debt and Complexity

### Areas of Complexity

1. **Addon interdependencies** - Many addons depend on others (e.g., `styled` → `style` → `jsx` → `rule`)
2. **Multiple test environments** - Client/server/dev modes require separate test setups
3. **Type definitions maintenance** - TypeScript types must be manually kept in sync

### Improvement Opportunities

1. **Convert to TypeScript** - Currently JS with separate type definitions
2. **Modernize build system** - Currently no build step for source files
3. **Consolidate presets** - Could auto-generate from addon metadata

## Common Workflows

### Adding a New Addon

1. Create `addon/{name}.js` following the addon pattern
2. Add `addon/{name}.d.ts` for TypeScript support
3. Write tests in `addon/__tests__/{name}.test.js`
4. Document in `docs/{name}.md`
5. Add a Storybook story for visual testing

### Modifying Core Behavior

1. Start with `index.js` to understand the renderer
2. Check which addons wrap the method you need to modify
3. Consider backwards compatibility with existing addons

### Debugging Style Generation

1. Enable `verbose: true` in nano options
2. Use `sourcemaps` addon in development
3. Check `validate` addon for CSS syntax errors
