# MockKit Architecture

How the pieces fit together — from the developer typing a prompt, through the AgentSkills layer, the MCP server, MockKit core, and out to the artifacts that flow through the lifecycle.

## The full picture

```mermaid
---
config:
  layout: elk
---
flowchart TB
    Dev(["Developer<br/>(natural language prompt)"])

    subgraph Clients["AI Clients<br/>(30+ AgentSkills-compatible)"]
        CC["Claude Code"]
        CR["Cursor"]
        GH["GitHub Copilot"]
        GM["Gemini CLI"]
        OT["Codex · JetBrains · ..."]
    end

    subgraph Skills[".agents/skills/<br/>(AgentSkills — auto-discovered)"]
        Atomic["<b>8 Atomic Skills</b><br/>spec-mock · record-api<br/>setup-msw · setup-playwright<br/>record-flow-as-test · audit-tests<br/>chaos · circuit-breaker"]
        Workflow["<b>4 Workflow Skills</b><br/>build-component · ship-feature<br/>harden-tests · spec-to-prod"]
    end

    subgraph MCP["MCP Servers<br/>(configured in .mcp.json)"]
        MockMCP["<b>@up/mockkit-mcp</b><br/>12 tools · 13 prompts"]
        PlayMCP["<b>@playwright/mcp</b><br/>(used by record-flow-as-test only)"]
    end

    subgraph Core["MockKit Core<br/>(npm: @up/mockkit)"]
        CLI["CLI<br/>start · record · replay · curl"]
        Server["Hono Server<br/>mock · replay · chaos · breaker"]
        RecEng["Recording Engine<br/>capture · templatize · snapshot"]
    end

    subgraph Browser["Browser Layer<br/>(alt path — no proxy)"]
        Chrome["Chrome Extension<br/>(offline mode, capture)"]
        MSWH["MSW Handlers<br/>(in-app interception)"]
    end

    subgraph Artifacts["Artifacts<br/>(committed to git)"]
        Spec[("api.yaml<br/>spec")]
        Recordings[("recordings.json<br/><b>THE artifact</b>")]
        Tests[("tests/*.spec.ts<br/>generated")]
    end

    subgraph External["External Systems"]
        App["App Under Test<br/>(browser or backend)"]
        Upstream["Real Upstream API<br/>(record only)"]
    end

    Dev -->|prompt| Clients
    Clients -.auto-activate.-> Atomic
    Clients -.activate workflow.-> Workflow
    Workflow -.body-instructed composition.-> Atomic

    Atomic -->|tool / prompt call| MockMCP
    Atomic -.record-flow-as-test only.-> PlayMCP

    MockMCP -->|spawn / invoke| CLI
    MockMCP -->|HTTP| Server
    CLI --> Server
    CLI --> RecEng
    Server --> RecEng

    RecEng <-.reads/writes.-> Recordings
    CLI -.reads.-> Spec

    App <-->|"HTTP<br/>(or via Browser Layer)"| Server
    Server -->|"forward<br/>(record only)"| Upstream
    Upstream -.captured response.-> RecEng

    App -->|fetch/XHR| Chrome
    App -->|fetch/XHR| MSWH
    Chrome -.reads.-> Recordings
    MSWH -.reads.-> Recordings

    PlayMCP -->|drives browser| App
    Atomic -.writes.-> Tests

    classDef artifact fill:#1a1a1a,stroke:#00F5FF,stroke-width:2px,color:#fff
    classDef skill fill:#0f1929,stroke:#00F5FF,stroke-width:1px,color:#fff
    classDef mcp fill:#1a0f29,stroke:#A78BFA,stroke-width:1px,color:#fff
    classDef mcpAlt fill:#1a0f29,stroke:#A78BFA,stroke-width:1px,color:#fff,stroke-dasharray: 5 3
    classDef core fill:#1a1a1a,stroke:#fff,stroke-width:1px,color:#fff
    class Spec,Recordings,Tests artifact
    class Atomic,Workflow skill
    class MockMCP mcp
    class PlayMCP mcpAlt
    class CLI,Server,RecEng core
```

> **Renderer note:** the `layout: elk` directive needs Mermaid v11+. GitHub renders Mermaid 10.x as of late 2025 and silently falls back to dagre — still readable, just less cleanly routed. VS Code, Obsidian (with the Mermaid v11 plugin), and most modern doc viewers will render with ELK.

### Render-anywhere fallback (SVG)

If your viewer blocks Mermaid or you want pixel-perfect output:

![MockKit Architecture](./architecture.svg)

The thick cyan border + glow on `recordings.json` is intentional — it's the focal point. Every layer either reads from it or writes to it.

---

## Complete system map (reference)

The lifecycle view above is narrative — focused on the AI-driven dev loop. This map is exhaustive: every published package, every MCP server, every input/output, every consumer. Use it to answer "what connects to what."

```mermaid
---
config:
  layout: elk
---
flowchart TB
    %% ─── INPUTS ───
    subgraph Inputs["📥 Inputs"]
        OAS[("OpenAPI<br/>api.yaml")]
        SS[("Simple Schema<br/>schema.yaml")]
        HAR[("HAR import<br/>browser export")]
    end

    %% ─── AI CONSUMER LAYER ───
    subgraph AI["🤖 AI Layer"]
        AIClient["AI Client<br/>(Claude Code · Cursor · Copilot ·<br/>Gemini · Codex · 30+)"]
        SkillsRepo["AgentSkills<br/>.agents/skills/<br/>(8 atomic + 4 workflow)"]
    end

    %% ─── MCP SERVERS (configured in .mcp.json) ───
    subgraph MCPLayer["🔌 MCP Servers"]
        MockMCP["@up/mockkit-mcp<br/>12 tools · 13 prompts"]
        PlayMCP["@playwright/mcp<br/>browser primitives<br/>(snapshot · click · navigate)"]
        DevToolsMCP["chrome-devtools-mcp<br/>Lighthouse · Performance<br/>(used by Scenario 18)"]
    end

    %% ─── MOCKKIT PACKAGES (the monorepo) ───
    subgraph MK["📦 MockKit Monorepo (4 packages)"]
        direction TB

        subgraph CorePkg["@up/mockkit · core"]
            CLI["CLI<br/>start · record · replay · curl<br/>validate · routes · init · examples<br/>recordings · import · snapshots<br/>chaos · cache · browser"]
            Hono["Hono Server<br/>3 modes: mock · record · replay<br/>+ /__mock__/* admin endpoints"]
            RecEng["Recording Engine<br/>capture · templatize · snapshot<br/>HAR export · diff · merge"]
            ChaosE["Chaos Engine<br/>error rate · latency · timeout<br/>circuit breaker (per-endpoint)"]
            PWFix["Playwright Fixture<br/>@up/mockkit-playwright<br/>(re-exported from core)"]
            BrowserBundle["Browser Bundle<br/>generated MSW handlers<br/>from spec"]
        end

        MCPPkg["@up/mockkit-mcp<br/>thin wrapper over CLI + HTTP"]

        subgraph CXT["chrome-extension"]
            DevPanel["DevTools Panel<br/>(React UI)"]
            BG["Background SW<br/>state · proxy · injection"]
            CSIso["offline-content.ts<br/>(ISOLATED world bridge)"]
            CSMain["offline-inject.ts<br/>(MAIN world fetch override)"]
        end

        Preview["@up/mockkit-preview<br/>Vite app · visualize<br/>MSW handlers from spec"]
    end

    %% ─── BROWSER LAYER (in the app under test) ───
    subgraph BrowserLayer["🌐 Browser Layer (in App Under Test)"]
        MSWBrowser["MSW Service Worker<br/>(in app's public/)"]
        ExtRuntime["Chrome Extension Runtime<br/>(injected per tab)"]
    end

    %% ─── TEST LAYER ───
    subgraph TestLayer["🧪 Test Layer"]
        PWRunner["Playwright Test Runner<br/>npx playwright test"]
        CIRunner["CI Runner<br/>GitHub Actions · GitLab · etc."]
    end

    %% ─── OUTPUTS / ARTIFACTS ───
    subgraph Outputs["📤 Outputs (committed to git)"]
        Recs[("recordings.json<br/><b>THE artifact</b>")]
        Snaps[("snapshots/<br/>named scenarios")]
        Tests[("tests/*.spec.ts<br/>Playwright specs")]
        HAREx[("HAR exports<br/>/__mock__/recordings/export")]
        Lighthouse[("Lighthouse reports<br/>JSON · perf metrics")]
        MSWFiles[("src/mocks/handlers.ts<br/>generated MSW handlers")]
    end

    %% ─── CONSUMERS ───
    subgraph Apps["💻 App Under Test"]
        FE["Frontend App<br/>(browser)"]
        BE["Backend Service<br/>(HTTP client)"]
    end

    %% ─── EXTERNAL ───
    subgraph Ext["🌍 External"]
        Upstream["Real Upstream API<br/>(only during record)"]
    end

    %% ═══ EDGES ═══

    %% Inputs → Core
    OAS -->|spec| CLI
    SS -->|spec| CLI
    HAR -.import.-> CLI

    %% AI → Skills → MCP
    AIClient -.auto-discover.-> SkillsRepo
    SkillsRepo -->|tool / prompt call| MockMCP
    SkillsRepo -.record-flow-as-test only.-> PlayMCP
    SkillsRepo -.scenario 18 only.-> DevToolsMCP

    %% MCP → Core
    MockMCP -->|spawn / HTTP| CLI
    MockMCP -.HTTP /__mock__/* .-> Hono

    %% Core internal
    CLI --> Hono
    CLI --> RecEng
    CLI --> ChaosE
    Hono --> RecEng
    Hono --> ChaosE
    CLI -.generates.-> BrowserBundle
    CLI -.generates.-> MSWFiles
    BrowserBundle --> PWFix

    %% Browser injection paths
    DevPanel <-->|chrome.runtime| BG
    BG -->|chrome.scripting| CSIso
    BG -->|chrome.scripting + MAIN world| CSMain
    CSIso <-.postMessage.-> CSMain
    BG -.imports.-> ExtRuntime
    CSMain -.intercepts fetch.-> ExtRuntime

    %% Preview
    Preview -.reads.-> OAS
    Preview -.renders.-> MSWFiles

    %% App ↔ MockKit (multiple paths)
    FE <-->|HTTP fetch/XHR| Hono
    BE <-->|HTTP| Hono
    FE -->|fetch/XHR| MSWBrowser
    FE -->|fetch/XHR via injected| ExtRuntime
    MSWBrowser -.uses.-> MSWFiles

    %% Recording read paths
    RecEng <-.reads/writes.-> Recs
    RecEng <-.reads/writes.-> Snaps
    RecEng -.exports.-> HAREx
    ExtRuntime -.reads.-> Recs
    MSWBrowser -.reads.-> Recs

    %% Real upstream
    Hono -->|"forward<br/>(record only)"| Upstream
    Upstream -.captured response.-> RecEng

    %% Playwright orchestration
    PWMCP_drive["drives browser"]
    PlayMCP -.drives.-> FE
    PWFix -.starts mock for tests.-> Hono
    PWRunner -->|uses fixture| PWFix
    PWRunner -.executes.-> Tests
    SkillsRepo -.writes via record-flow-as-test.-> Tests

    %% chrome-devtools-mcp
    DevToolsMCP -.measures.-> FE
    DevToolsMCP -.outputs.-> Lighthouse

    %% CI
    CIRunner -->|invokes| PWRunner
    CIRunner -.starts via fixture.-> Hono
    CIRunner -.reads.-> Recs

    %% ─── Styling ───
    classDef artifact fill:#1a1a1a,stroke:#00F5FF,stroke-width:2px,color:#fff
    classDef recording fill:#062a30,stroke:#00F5FF,stroke-width:3px,color:#fff
    classDef skill fill:#0f1929,stroke:#00F5FF,stroke-width:1px,color:#fff
    classDef mcp fill:#1a0f29,stroke:#A78BFA,stroke-width:1px,color:#fff
    classDef mcpAlt fill:#1a0f29,stroke:#A78BFA,stroke-width:1px,color:#fff,stroke-dasharray: 5 3
    classDef pkg fill:#161616,stroke:#9ca3af,stroke-width:1px,color:#fff
    classDef ext fill:#1a1a1a,stroke:#6b7280,stroke-width:1px,color:#9ca3af,stroke-dasharray: 4 4
    class Recs recording
    class Snaps,Tests,HAREx,Lighthouse,MSWFiles,OAS,SS,HAR artifact
    class SkillsRepo skill
    class MockMCP mcp
    class PlayMCP,DevToolsMCP mcpAlt
    class CLI,Hono,RecEng,ChaosE,PWFix,BrowserBundle,MCPPkg,DevPanel,BG,CSIso,CSMain,Preview pkg
    class Upstream ext
```

### Reading the system map

Six logical groupings (matched to the icons in the subgraph titles):

| Group | What it contains | Lives where |
|---|---|---|
| 📥 **Inputs** | OpenAPI / Simple Schema specs, HAR imports | User-provided, in repo |
| 🤖 **AI Layer** | AI client + AgentSkills | Client config + `.agents/skills/` |
| 🔌 **MCP Servers** | mockkit-mcp + playwright-mcp + chrome-devtools-mcp | `.mcp.json` |
| 📦 **MockKit Monorepo** | Core, MCP wrapper, Chrome extension, Preview | `packages/` in mockkit repo |
| 🌐 **Browser Layer** | MSW worker, Chrome extension runtime | Injected into the App Under Test |
| 🧪 **Test Layer** | Playwright runner, CI runner | Wherever you run tests |
| 📤 **Outputs** | recordings, snapshots, tests, HAR exports, Lighthouse reports, MSW handlers | All committed to git |
| 💻 **Apps** | Frontend (browser), Backend (HTTP client) | The thing being tested |
| 🌍 **External** | Real upstream API (only during record) | Third party |

### Lookup: "What connects to what?"

Common questions and where to find the answer in the map:

| Question | Path through the map |
|---|---|
| How does `mockkit start -s api.yaml` work? | `OAS → CLI → Hono`, then App fetches `Hono` over HTTP |
| How does the AI generate a Playwright test? | `AIClient → SkillsRepo (record-flow-as-test) → PlayMCP → drives FE`, simultaneously `→ MockMCP → Hono` for deterministic backend, then writes to `Tests` |
| How does Lighthouse audit fit in? | `AIClient → SkillsRepo → DevToolsMCP → measures FE → outputs Lighthouse reports` (used in Scenario 18) |
| How does the Chrome extension intercept fetch? | `BG → chrome.scripting → CSMain (MAIN world) → intercepts fetch in ExtRuntime → reads Recs` |
| How does MSW serve the recording? | `MSWBrowser (running in app's service worker) → reads MSWFiles → falls back to Recs for capture-mode replays` |
| How does CI run a Playwright suite without a backend? | `CIRunner → PWRunner → PWFix (starts Hono in mock/replay mode) → Tests run against Hono → reads Recs` |
| What's the difference between `mockkit-mcp` and `mockkit`? | `mockkit-mcp` is a thin wrapper that spawns `mockkit` CLI / talks HTTP to its `Hono` server. The MCP server doesn't replicate Core capabilities — it exposes them. |
| Is the Preview app required? | No. `Preview` reads `OAS` and renders `MSWFiles` for visualization only. Orthogonal to the runtime path. |
| When does the proxy hit upstream? | `Hono → forward → Upstream` only fires during `mockkit record`. In `mock` and `replay` modes, that arrow is dormant. |

## How to read it

The diagram has **six horizontal layers** plus two side groupings.

1. **Developer** — types a natural-language prompt. Doesn't know or care which skill runs.
2. **AI Clients** — Claude Code, Cursor, Copilot, Gemini CLI, etc. Auto-discover the skills in `.agents/skills/` at startup.
3. **AgentSkills** — model auto-activates the matching atomic skill based on the prompt. Workflow skills compose atomics via body instructions (no spec primitive).
4. **MCP Servers** — atomic skills call into MockKit MCP for tools/prompts; the `record-flow-as-test` skill additionally orchestrates Playwright MCP for browser control.
5. **MockKit Core** — the actual mocking implementation: CLI, Hono server, recording engine. The MCP server is a thin wrapper that calls into here.
6. **Browser Layer** (alt path) — for browser-side mocking, the Chrome extension and MSW handlers serve recordings directly without going through the core HTTP server.

**Side groupings:**
- **Artifacts** — files committed to git. `recordings.json` is the central one; spec is input, tests are output.
- **External** — the app under test and (during recording) the real upstream API.

## The throughline

The whole point of the diagram: **`recordings.json` is the only thing that flows through every lifecycle phase**.

- **Day 0**: Recording is empty. Mock generates from spec.
- **Week 1**: Recording captures real upstream responses.
- **Day-to-day**: Recording is the offline cache (extension, MSW, replay all consume it).
- **Pre-merge**: Recording is the test fixture.
- **Forever**: Recording stays in git. Audit tests against it; chaos perturbs the responses on top of it.

Every other component is plumbing. The recording is the contract.

## Data flow examples

### Example A — Day 0 (spec → mock → component)

```
Dev → "Stand up a mock from api.yaml and scaffold IncidentList"
   ↓
AI Client auto-activates spec-mock + setup-msw + (component scaffold)
   ↓
spec-mock calls @up/mockkit-mcp  →  CLI: mockkit start -s api.yaml
   ↓
Hono Server starts on :9876, generates responses from api.yaml
   ↓
App fetches → Server responds (no recording yet, generated from spec)
```

### Example B — Pre-merge (flow → test)

```
Dev → "Record this flow as a Playwright test"
   ↓
AI Client auto-activates record-flow-as-test
   ↓
Skill calls @up/mockkit-mcp     →  Server up with seed=42 (deterministic)
Skill calls @playwright/mcp     →  Drives the browser
   ↓
Browser fetches → Server responds from spec (or recordings.json)
   ↓
Skill captures aria snapshots → synthesizes tests/<flow>.spec.ts
   ↓
Test runs against same Server → green
```

### Example C — Forever (audit + chaos)

```
Dev → "Audit tests for fragility, apply HIGH/MEDIUM fixes"
   ↓
AI Client auto-activates audit-tests
   ↓
Skill calls @up/mockkit-mcp → audit-test-quality prompt
   ↓
Reads tests/*.spec.ts, walks 9 patterns, applies fixes
   ↓
Re-runs suite against same Server → green, faster
```

## Key design choices

| Choice | Why |
|---|---|
| Skills wrap MCP prompts (don't reinvent) | Single source of truth for prompt content; spec-compliant skills with proprietary `metadata` fields for the wiring |
| MCP server is thin (delegates to core) | Core works standalone via CLI; MCP is just a calling convention for AI |
| Recording is JSON, not a binary | Diff-able in git, hand-editable, language-agnostic |
| Chrome extension reads recordings directly | No proxy required for offline dev; faster than message round-trip |
| Workflow skills compose via body instructions | Spec deliberately omits composition primitive — model chains atomics by following the markdown body |

## What's NOT in the diagram (intentionally)

- **CI runners** — they just invoke `mockkit replay` like any other process.
- **The preview package** — it's a separate Next.js app for visualizing recordings; orthogonal to the runtime path.
- **GitHub / Linear / Slack MCP servers** — the AI client may have many MCPs; only MockKit + Playwright are part of MockKit's story.
- **Telemetry / observability** — none today; would be a future layer above Core.
