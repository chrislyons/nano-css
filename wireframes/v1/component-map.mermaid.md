%% nano-css Component Map
%% Detailed breakdown of all modules and their relationships
%% Shows dependencies, responsibilities, and APIs

classDiagram
    direction TB

    class CoreRenderer {
        +client: boolean
        +raw: string
        +pfx: string
        +sh: CSSStyleSheet
        +put(selector, css, atrule)
        +putRaw(rawCss)
        +decl(prop, val)
        +kebab(prop)
        +hash(obj)
        +selector(parent, sel)
        +putAt(selector, css, atrule)
    }

    class RuleAddon {
        +rule(css, block): string
        --
        Generates class names from
        CSS objects
    }

    class SheetAddon {
        +sheet(map, block): object
        --
        Lazy-evaluated style
        collections with getters
    }

    class CacheAddon {
        +cache(css): string
        --
        Memoizes rule() calls
        for performance
    }

    class StableAddon {
        +hash(obj): string
        --
        Deterministic class names
        using fastest-stable-stringify
    }

    class DruleAddon {
        +drule(css, block): function
        --
        Dynamic rules with
        runtime overrides
    }

    class DsheetAddon {
        +dsheet(map, block): object
        --
        Dynamic sheets with
        runtime overrides
    }

    class JsxAddon {
        +jsx(tag, styles, block): Component
        --
        Creates styled VDOM
        components
    }

    class StyleAddon {
        +style(tag)(css): Component
        --
        Styled components with
        dynamic template support
    }

    class StyledAddon {
        +styled(tag)`css`: Component
        --
        Tagged template literal
        API like styled-components
    }

    class ComponentAddon {
        +Component: class
        --
        React Component base
        with this.css()
    }

    class DecoratorAddon {
        +css(styles): decorator
        --
        @css class/method
        decorators
    }

    class UseStylesAddon {
        +useStyles(block)(Component): Component
        --
        Injects styles as
        second argument
    }

    class WithStylesAddon {
        +withStyles(styles)(Component): Component
        --
        HOC that injects
        styles prop
    }

    class AtomsAddon {
        +decl(prop, val): declaration
        --
        Short property names
        fz -> fontSize
    }

    class EmmetAddon {
        +decl(prop, val): declaration
        --
        Emmet-style
        abbreviations
    }

    class NestingAddon {
        +selector(parent, sel): string
        --
        Advanced & selector
        nesting support
    }

    class PrefixerAddon {
        +decl(prop, val): declaration
        --
        Auto vendor prefix
        addition
    }

    class UnitlessAddon {
        +decl(prop, val): declaration
        --
        Auto px unit for
        numeric values
    }

    class GlobalAddon {
        +putRaw(css): void
        --
        :global pseudo-selector
        support
    }

    class KeyframesAddon {
        +keyframes(timeline, block): string
        --
        @keyframes animation
        definitions
    }

    class HydrateAddon {
        +hydrate(sh): void
        --
        Re-hydrate SSR CSS
        prevent duplicates
    }

    class VcssomAddon {
        +createRule(styles): object
        +cssToTree(css): tree
        +removeRule(tree): void
        --
        Virtual CSSOM with
        diffing support
    }

    class CssomAddon {
        +createRule(css): object
        --
        CSSOM API for
        rule manipulation
    }

    class ValidateAddon {
        +putRaw(css): void
        --
        CSS syntax
        validation
    }

    class SourcemapsAddon {
        +putRaw(css): void
        --
        Generate CSS
        sourcemaps (dev)
    }

    %% Core Dependencies
    CoreRenderer <|-- RuleAddon : extends
    RuleAddon <|-- SheetAddon : depends on
    RuleAddon <|-- CacheAddon : depends on

    %% Dynamic addons
    RuleAddon <|-- DruleAddon : depends on
    CacheAddon <|-- DruleAddon : depends on
    SheetAddon <|-- DsheetAddon : depends on
    CacheAddon <|-- DsheetAddon : depends on

    %% React/VDOM chain
    RuleAddon <|-- JsxAddon : depends on
    CacheAddon <|-- JsxAddon : depends on
    JsxAddon <|-- StyleAddon : depends on
    StyleAddon <|-- StyledAddon : depends on

    %% Other React patterns
    RuleAddon <|-- ComponentAddon : depends on
    CacheAddon <|-- ComponentAddon : depends on
    RuleAddon <|-- DecoratorAddon : depends on
    CacheAddon <|-- DecoratorAddon : depends on
    SheetAddon <|-- UseStylesAddon : depends on
    SheetAddon <|-- WithStylesAddon : depends on

    %% Preprocessing (wrap decl)
    CoreRenderer <|-- AtomsAddon : wraps decl
    CoreRenderer <|-- EmmetAddon : wraps decl
    CoreRenderer <|-- PrefixerAddon : wraps decl
    CoreRenderer <|-- UnitlessAddon : wraps decl

    %% Selector processing
    CoreRenderer <|-- NestingAddon : wraps selector
    CoreRenderer <|-- GlobalAddon : wraps putRaw

    %% Advanced features
    CoreRenderer <|-- KeyframesAddon : uses putRaw
    CoreRenderer <|-- HydrateAddon : uses put
    CoreRenderer <|-- CssomAddon : uses sh
    CssomAddon <|-- VcssomAddon : depends on
    CoreRenderer <|-- ValidateAddon : wraps putRaw
    CoreRenderer <|-- SourcemapsAddon : wraps putRaw
