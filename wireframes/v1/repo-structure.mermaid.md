%% nano-css Repository Structure
%% A visual map of the complete directory organization
%% Last updated: 2025

graph TB
    subgraph Root["Root Directory"]
        INDEX["index.js<br/>Main Entry Point"]
        INDEXD["index.d.ts<br/>TypeScript Types"]
        PKG["package.json<br/>Project Config"]
        README["README.md<br/>Documentation"]
    end

    subgraph Addon["addon/ - Plugin System"]
        subgraph CoreAddons["Core CSS Generation"]
            RULE["rule.js<br/>Class Name Generator"]
            SHEET["sheet.js<br/>Lazy Style Collections"]
            CACHE["cache.js<br/>Memoization"]
            STABLE["stable.js<br/>Deterministic Names"]
        end

        subgraph DynamicAddons["Dynamic CSS"]
            DRULE["drule.js<br/>Dynamic Rules"]
            DSHEET["dsheet.js<br/>Dynamic Sheets"]
        end

        subgraph ReactAddons["React/VDOM Integration"]
            JSX["jsx.js<br/>Styled Components"]
            STYLE["style.js<br/>Dynamic Templates"]
            STYLED["styled.js<br/>Tagged Templates"]
            COMPONENT["component.js<br/>React Base Class"]
            DECORATOR["decorator.js<br/>@css Decorators"]
            USESTYLES["useStyles.js<br/>Style Injection"]
            WITHSTYLES["withStyles.js<br/>HOC Wrapper"]
        end

        subgraph EnhancementAddons["CSS Enhancement"]
            ATOMS["atoms.js<br/>Shorthand Props"]
            EMMET["emmet.js<br/>Emmet Syntax"]
            NESTING["nesting.js<br/>Selector Nesting"]
            PREFIXER["prefixer.js<br/>Vendor Prefixes"]
            UNITLESS["unitless.js<br/>Auto px Units"]
            GLOBAL["global.js<br/>:global Support"]
            RTL["rtl.js<br/>RTL Transform"]
        end

        subgraph RenderingAddons["Advanced Rendering"]
            KEYFRAMES["keyframes.js<br/>Animations"]
            STYLIS["stylis.js<br/>CSS Preprocessor"]
            VIRTUAL["virtual.js<br/>Atomic CSS"]
            CSSOM["cssom.js<br/>CSSOM API"]
            VCSSOM["vcssom.js<br/>Virtual CSSOM"]
        end

        subgraph UtilityAddons["Utilities"]
            HYDRATE["hydrate.js<br/>SSR Rehydration"]
            EXTRACT["extract.js<br/>CSS Extraction"]
            VALIDATE["validate.js<br/>CSS Validation"]
            GOOGLEFONT["googleFont.js<br/>Font Loading"]
        end

        subgraph Presets["Pre-built Styles"]
            RESET["reset/<br/>13 CSS Resets"]
            ANIMATE["animate/<br/>5 Animation Presets"]
        end

        subgraph AddonUtils["Utilities & Tests"]
            DEV["__dev__/<br/>Dev Utilities"]
            TESTS["__tests__/<br/>Test Suites"]
            UTIL["util/<br/>Helper Functions"]
        end
    end

    subgraph Preset["preset/ - Addon Bundles"]
        PSHEET["sheet.js<br/>Basic Preset"]
        PREACT["react.js<br/>React Preset"]
        PVDOM["vdom.js<br/>VDOM Preset"]
    end

    subgraph Types["types/ - TypeScript"]
        NANOD["nano.d.ts<br/>Core Types"]
        COMMOND["common.d.ts<br/>CSS Types"]
    end

    subgraph Docs["docs/ - Documentation"]
        INSTALL["Installation.md"]
        ADDONDOC["Addons.md"]
        PRESETDOC["Presets.md"]
        SSRDOC["SSR.md"]
        ADDONDOCS["46 Addon Docs"]
    end

    subgraph Demo["demo/ - Examples"]
        DEMO1["demo1.html"]
        DEMO1TSX["demo1.tsx"]
    end

    subgraph Storybook[".storybook/ - Visual Tests"]
        CONFIG["config.js"]
        WEBPACK["webpack.config.js"]
        STORIES["40+ Stories"]
    end

    subgraph GitHub[".github/ - CI/CD"]
        CHECKS["checks.yml<br/>Test Workflow"]
        GHPAGES["gh-pages.yml<br/>Docs Deploy"]
        RELEASE["release.yml<br/>Release Auto"]
    end

    INDEX --> Addon
    INDEX --> Preset
    INDEX --> Types
    Addon --> Docs
    Addon --> Storybook
