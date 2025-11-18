%% nano-css Entry Points
%% All ways to interact with the codebase
%% Application initialization, APIs, CLI, and environments

flowchart TB
    subgraph PackageEntries["Package Entry Points"]
        MAIN["index.js<br/>Main Entry<br/>exports.create()"]
        PRESET_SHEET["preset/sheet.js<br/>Basic Preset"]
        PRESET_REACT["preset/react.js<br/>React Preset"]
        PRESET_VDOM["preset/vdom.js<br/>VDOM Preset"]
    end

    subgraph Initialization["Initialization Patterns"]
        CREATE["create(config)<br/>Manual Setup"]
        PRESET_INIT["preset({h})<br/>Pre-configured"]

        subgraph ConfigOptions["Configuration Options"]
            PFX_OPT["pfx: string<br/>Class prefix"]
            H_OPT["h: Function<br/>Hyperscript"]
            SH_OPT["sh: Element<br/>Style sheet"]
            VERBOSE_OPT["verbose: boolean<br/>Debug mode"]
            ASSIGN_OPT["assign: Function<br/>Object.assign"]
        end
    end

    subgraph AddonLoading["Addon Loading"]
        INDIVIDUAL["Individual Addons<br/>import {addon}"]
        REQUIRE_ADDON["require('nano-css/addon/x')"]

        subgraph CommonAddons["Commonly Loaded"]
            RULE_A["addon/rule"]
            SHEET_A["addon/sheet"]
            JSX_A["addon/jsx"]
            STYLED_A["addon/styled"]
            NESTING_A["addon/nesting"]
            ATOMS_A["addon/atoms"]
        end
    end

    subgraph StyleAPIs["Style Definition APIs"]
        subgraph ObjectAPI["Object API"]
            NANO_RULE["nano.rule(css)<br/>Single rule"]
            NANO_SHEET["nano.sheet(map)<br/>Multiple rules"]
            NANO_CACHE["nano.cache(css)<br/>Memoized"]
        end

        subgraph ComponentAPI["Component API"]
            NANO_JSX["nano.jsx(tag, css)<br/>VDOM component"]
            NANO_STYLE["nano.style(tag)(css)<br/>Dynamic styles"]
            NANO_STYLED["nano.styled(tag)`css`<br/>Tagged template"]
        end

        subgraph HOCAPI["HOC/Decorator API"]
            NANO_WITH["withStyles(styles)<br/>HOC wrapper"]
            NANO_USE["useStyles(block)<br/>Inject styles"]
            NANO_DEC["@css(styles)<br/>Decorator"]
        end

        subgraph DynamicAPI["Dynamic API"]
            NANO_DRULE["nano.drule(css)<br/>Dynamic rule"]
            NANO_DSHEET["nano.dsheet(map)<br/>Dynamic sheet"]
        end
    end

    subgraph AdvancedAPIs["Advanced APIs"]
        NANO_KF["nano.keyframes(timeline)<br/>Animations"]
        NANO_GLOBAL["nano.global(css)<br/>Global styles"]
        NANO_HYDRATE["nano.hydrate(sh)<br/>SSR hydration"]
        NANO_VCSSOM["nano.createRule(css)<br/>Virtual CSSOM"]
    end

    subgraph DevelopmentTools["Development Entry Points"]
        STORYBOOK["yarn storybook<br/>Port 6010"]
        DEMO["yarn demo<br/>Webpack Dev"]
        TEST["yarn test<br/>Jest Suite"]
    end

    subgraph BuildScripts["Build & CI Scripts"]
        LINT["yarn lint<br/>ESLint"]
        PRETTIER["yarn prettier<br/>Format code"]
        COVERAGE["yarn test:coverage<br/>Coverage report"]
    end

    subgraph Environments["Environment Differences"]
        subgraph BrowserEnv["Browser"]
            B_CLIENT["client: true"]
            B_INSERT[".insertRule()"]
            B_SHEET["Live style sheet"]
        end

        subgraph ServerEnv["Server (SSR)"]
            S_CLIENT["client: false"]
            S_RAW["raw accumulator"]
            S_STRING["CSS string output"]
        end

        subgraph TestEnv["Test (Jest)"]
            T_MOCK["Mock RAF"]
            T_JSDOM["JSDOM environment"]
            T_SNAP["Snapshot testing"]
        end
    end

    subgraph TypeScriptSupport["TypeScript Entry"]
        TS_MAIN["index.d.ts<br/>Main types"]
        TS_ADDON["addon/*.d.ts<br/>Addon types"]
        TS_TYPES["types/nano.d.ts<br/>Core interfaces"]
    end

    %% Connections
    MAIN --> CREATE
    PRESET_SHEET --> PRESET_INIT
    PRESET_REACT --> PRESET_INIT
    PRESET_VDOM --> PRESET_INIT

    CREATE --> ConfigOptions
    PRESET_INIT --> ConfigOptions

    INDIVIDUAL --> CommonAddons
    REQUIRE_ADDON --> CommonAddons

    CREATE --> StyleAPIs
    CREATE --> AdvancedAPIs

    STORYBOOK --> StyleAPIs
    DEMO --> StyleAPIs
    TEST --> StyleAPIs

    StyleAPIs --> Environments

    TS_MAIN --> CREATE
    TS_ADDON --> INDIVIDUAL

    %% Styling
    classDef entry fill:#e8f5e9,stroke:#2e7d32
    classDef init fill:#e3f2fd,stroke:#1565c0
    classDef api fill:#f3e5f5,stroke:#7b1fa2
    classDef dev fill:#fff3e0,stroke:#ef6c00
    classDef env fill:#fce4ec,stroke:#c2185b

    class PackageEntries entry
    class Initialization init
    class StyleAPIs,AdvancedAPIs api
    class DevelopmentTools,BuildScripts dev
    class Environments env
