%% nano-css High-Level Architecture
%% 5th generation CSS-in-JS library with plugin system
%% Core: ~0.5KB, extensible through addons

graph TB
    subgraph UserCode["User Application"]
        APP["Application Code"]
        STYLES["Style Definitions<br/>Objects / Templates"]
    end

    subgraph Core["Core Renderer (~0.5KB)"]
        CREATE["create(config)<br/>Factory Function"]

        subgraph RendererMethods["Renderer Instance"]
            PUT["put(selector, css)<br/>Main CSS Injection"]
            PUTRAW["putRaw(css)<br/>Raw CSS Injection"]
            DECL["decl(prop, val)<br/>Declaration Builder"]
            KEBAB["kebab(prop)<br/>camelCase to kebab"]
            HASH["hash(obj)<br/>Unique Class Name"]
            SELECTOR["selector(parent, sel)<br/>Selector Builder"]
        end

        subgraph RendererState["Renderer State"]
            RAW["raw: string<br/>SSR CSS Accumulator"]
            PFX["pfx: string<br/>Class Prefix"]
            CLIENT["client: boolean<br/>Browser Detection"]
        end
    end

    subgraph AddonLayer["Addon Layer (Middleware)"]
        subgraph Generation["CSS Generation Addons"]
            RULE["rule()<br/>Object to Class"]
            SHEET["sheet()<br/>Lazy Collections"]
            CACHE["cache()<br/>Memoization"]
        end

        subgraph Preprocessing["CSS Preprocessing"]
            ATOMS["atoms<br/>Shorthand Props"]
            NESTING["nesting<br/>& Selectors"]
            PREFIXER["prefixer<br/>Vendor Prefixes"]
            UNITLESS["unitless<br/>Auto px"]
        end

        subgraph Components["Component APIs"]
            JSX["jsx()<br/>VDOM Components"]
            STYLED["styled<br/>Tagged Templates"]
            WITHSTYLES["withStyles()<br/>HOC Pattern"]
        end

        subgraph Advanced["Advanced Features"]
            KEYFRAMES["keyframes<br/>Animations"]
            VCSSOM["vcssom<br/>Virtual CSSOM"]
            HYDRATE["hydrate<br/>SSR Rehydration"]
        end
    end

    subgraph Output["CSS Output"]
        subgraph Browser["Browser Environment"]
            STYLEELEM["&lt;style&gt; Element"]
            INSERTRULE[".insertRule()<br/>Fast Injection"]
        end

        subgraph Server["Server Environment"]
            RAWSTRING["raw String<br/>CSS Accumulator"]
            SSR["SSR HTML Output"]
        end
    end

    subgraph External["External Dependencies"]
        CSSTYPE["csstype<br/>TS CSS Types"]
        STRINGIFY["fastest-stable-stringify<br/>Deterministic Hash"]
        INLINEPREFIX["inline-style-prefixer<br/>Vendor Prefixes"]
        RTLCSS["rtl-css-js<br/>RTL Transform"]
        STYLISLIB["stylis<br/>CSS Preprocessor"]
    end

    subgraph VDOMSupport["VDOM Libraries"]
        REACT["React"]
        PREACT["Preact"]
        VUE["Vue"]
        HYPERSCRIPT["Hyperscript"]
    end

    %% Flow connections
    APP --> STYLES
    STYLES --> Generation

    CREATE --> RendererMethods
    CREATE --> RendererState

    Generation --> Preprocessing
    Preprocessing --> PUT

    PUT --> Browser
    PUT --> Server

    INSERTRULE --> STYLEELEM
    RAWSTRING --> SSR

    Components --> Generation
    Advanced --> PUT

    External -.-> AddonLayer
    VDOMSupport -.-> Components

    %% Style definitions
    classDef core fill:#e1f5fe,stroke:#01579b
    classDef addon fill:#f3e5f5,stroke:#4a148c
    classDef output fill:#e8f5e9,stroke:#1b5e20
    classDef external fill:#fff3e0,stroke:#e65100

    class Core core
    class AddonLayer addon
    class Output output
    class External,VDOMSupport external
