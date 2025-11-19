%% nano-css Data Flow
%% How CSS moves through the system from definition to DOM
%% Shows processing pipeline and state management

flowchart TB
    subgraph Input["Style Input"]
        OBJ["CSS Object<br/>{color: 'red', fz: 12}"]
        TEMPLATE["Tagged Template<br/>styled('div')`color: red`"]
        DYNAMIC["Dynamic Props<br/>(props) => ({color: props.c})"]
    end

    subgraph Preprocessing["Preprocessing Pipeline"]
        direction TB
        ATOMS_P["atoms addon<br/>fz → fontSize"]
        EMMET_P["emmet addon<br/>fz12 → fontSize: 12px"]
        NESTING_P["nesting addon<br/>& → parent selector"]
        UNITLESS_P["unitless addon<br/>100 → 100px"]
        PREFIXER_P["prefixer addon<br/>flex → -webkit-flex, flex"]
    end

    subgraph Generation["Class Generation"]
        direction TB
        HASH["hash(css)<br/>Generate unique ID"]
        CLASSNAME["Build class name<br/>pfx + hash = '_a3f2'"]
        CACHE_CHECK{"Cached?"}
        RETURN_CACHED["Return cached<br/>class name"]
    end

    subgraph CorePut["Core put() Method"]
        direction TB
        BUILD_SEL["Build selector<br/>'.\_a3f2'"]
        BUILD_DECL["Build declarations<br/>color: red;"]
        BUILD_RULE["Build CSS rule<br/>.\_a3f2 { color: red; }"]
    end

    subgraph BrowserPath["Browser Environment"]
        direction TB
        GET_SHEET["Get style sheet<br/>document.head"]
        INSERT_RULE[".insertRule()<br/>Fast CSSOM injection"]
        STYLE_ELEM["&lt;style&gt; element<br/>in DOM"]
        CSSOM["Browser CSSOM<br/>Parsed styles"]
    end

    subgraph ServerPath["Server Environment"]
        direction TB
        RAW_ACCUM["Accumulate in<br/>renderer.raw"]
        SSR_OUTPUT["SSR Output<br/>&lt;style&gt;...&lt;/style&gt;"]
        HTML_STRING["HTML String<br/>sent to client"]
    end

    subgraph Hydration["Client Hydration"]
        direction TB
        PARSE_EXISTING["Parse existing<br/>selectors"]
        REGISTER_CACHE["Register in cache<br/>skip duplicates"]
        CONTINUE["Continue with<br/>new styles"]
    end

    subgraph VCSSOMFlow["Virtual CSSOM Flow"]
        direction TB
        CREATE_VDOM["createRule()<br/>Virtual representation"]
        DIFF["Diff changes<br/>old vs new"]
        UPDATE["Update only<br/>changed rules"]
        REMOVE["Remove unused<br/>rules"]
    end

    %% Main flow connections
    OBJ --> ATOMS_P
    TEMPLATE --> ATOMS_P
    DYNAMIC --> ATOMS_P

    ATOMS_P --> EMMET_P
    EMMET_P --> NESTING_P
    NESTING_P --> UNITLESS_P
    UNITLESS_P --> PREFIXER_P

    PREFIXER_P --> HASH
    HASH --> CACHE_CHECK

    CACHE_CHECK -->|Yes| RETURN_CACHED
    CACHE_CHECK -->|No| CLASSNAME

    CLASSNAME --> BUILD_SEL
    BUILD_SEL --> BUILD_DECL
    BUILD_DECL --> BUILD_RULE

    BUILD_RULE --> GET_SHEET
    BUILD_RULE --> RAW_ACCUM

    GET_SHEET --> INSERT_RULE
    INSERT_RULE --> STYLE_ELEM
    STYLE_ELEM --> CSSOM

    RAW_ACCUM --> SSR_OUTPUT
    SSR_OUTPUT --> HTML_STRING

    HTML_STRING --> PARSE_EXISTING
    PARSE_EXISTING --> REGISTER_CACHE
    REGISTER_CACHE --> CONTINUE

    BUILD_RULE --> CREATE_VDOM
    CREATE_VDOM --> DIFF
    DIFF --> UPDATE
    DIFF --> REMOVE

    %% Styling
    classDef input fill:#e3f2fd,stroke:#1565c0
    classDef preprocess fill:#f3e5f5,stroke:#7b1fa2
    classDef generate fill:#e8f5e9,stroke:#2e7d32
    classDef core fill:#fff3e0,stroke:#ef6c00
    classDef browser fill:#e1f5fe,stroke:#0277bd
    classDef server fill:#fce4ec,stroke:#c2185b
    classDef hydrate fill:#f1f8e9,stroke:#558b2f

    class Input input
    class Preprocessing preprocess
    class Generation generate
    class CorePut core
    class BrowserPath browser
    class ServerPath server
    class Hydration hydrate
