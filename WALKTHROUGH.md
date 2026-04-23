# MockKit — Complete Walkthrough

> A single self-contained document covering every capability, package, integration, and architectural decision behind MockKit. Designed to be ingested whole by Google NotebookLM (or any RAG system) for end-to-end Q&A on the system.

---

## Part 1 — What MockKit Is

MockKit is an **AI-native API mocking platform** that replaces fragmented developer tooling — separate spec viewers, mock servers, fixtures, test stubs, chaos tools, performance auditors — with a single tool whose central artifact (a JSON recording file) flows through every phase of the development lifecycle.

It ships as:

1. **A CLI** (`@up/mockkit`) — `mockkit start`, `record`, `replay`, `curl`, `chaos`, `snapshots`, etc.
2. **An MCP server** (`@up/mockkit-mcp`) — exposes 12 tools and 13 prompts to any AI client (Claude Code, Cursor, Copilot, Gemini CLI, Codex, JetBrains, 30+ clients).
3. **A Chrome extension** (`mockkit-extension`) — DevTools panel for recording, mocking, and offline mode in the browser.
4. **A preview app** (`@up/mockkit-preview`) — Vite app that visualizes MSW handlers generated from a spec.
5. **A Playwright fixture** (`@up/mockkit-playwright`, exported from core) — turn-key E2E mocking for tests.
6. **AgentSkills** (`.agents/skills/`) — 8 atomic + 4 workflow skills following the open agentskills.io standard, auto-discovered by AI clients so developers describe intent in natural language and the model orchestrates the toolchain.

The core thesis: **one artifact, whole lifecycle**. The same `recordings.json` file is the mock data source on Day 0, the offline cache day-to-day, the test fixture pre-merge, and the contract that audits and chaos run against forever.

---

## Part 2 — The Lifecycle Story

MockKit organizes around five phases of the development lifecycle. Each phase produces the input the next phase needs — no rebuilds, no fixture rot, no contract drift between layers.

### Phase 1 — Day 0 (Spec → Mock)

You have a spec (OpenAPI YAML, OpenAPI JSON, or MockKit's Simple Schema). No backend yet.

```bash
mockkit start -s api.yaml
```

A live HTTP server stands up on `localhost:9876` (default port for `start`, configurable). It generates deterministic responses from the spec — same seed produces same data, every time. Frontend developers can build against a real-feeling API in seconds.

The same outcome via AI:

> "Stand up a mock from `./api.yaml`."

The AI auto-activates the `spec-mock` skill, validates the spec, starts the server, confirms health, and reports the URL.

### Phase 2 — Week 1 (Capture)

Backend ships their first endpoints. Instead of throwing away the spec-driven mock, MockKit captures the real responses next to it via the recording proxy:

```bash
mockkit record -t https://api.example.com -o ./fixtures/api.json --templatize
```

Your app points at `http://localhost:3000` (the proxy). The proxy forwards every request upstream, returns the real response unchanged, and writes the request/response pair to `./fixtures/api.json`. With `--templatize`, dynamic values (UUIDs, JWTs, timestamps) become placeholders so the recording isn't pinned in time.

Spec drift between the documented API and the real API is caught at capture time — not in QA weeks later.

### Phase 3 — Day-to-Day (Offline Dev)

The recording becomes the offline cache. Three paths to consume it:

- **`mockkit replay -r ./fixtures/api.json`** — fastest, runs an HTTP server from the file.
- **MSW handlers** — generated from the spec, served in-browser by Mock Service Worker; integrated into React/Next.js apps.
- **Chrome extension** — the extension's "Offline Mode" intercepts `fetch`/XHR in any page and serves recordings directly. No proxy, no VPN, plane mode works.

### Phase 4 — Pre-Merge (Tests Write Themselves)

The killer feature. Describe a UI flow in plain language:

> "Record a Playwright test for: load the dashboard, click the first incident, fill the title with X, submit, verify it appears in the list."

The AI auto-activates `record-flow-as-test`. It composes two MCP servers — MockKit (for the deterministic backend) + Playwright MCP (for the browser) — drives the app, captures aria snapshots after each step, synthesizes a `.spec.ts` file with `afterEach` cleanup baked in, and runs it once to confirm it passes. Self-cleaning, no suite pollution.

### Phase 5 — Forever (Quality Stays)

Six months later your test suite has 200 specs and three different selector styles. Test rot is real. The AI auto-activates `audit-tests`:

```
applyFixes: true
```

The skill walks 9 fragility patterns — brittle selectors, missing cleanup, hardcoded counts, vacuous assertions, `networkidle` on SSE pages, etc. — reports findings with concrete fixes, optionally auto-applies HIGH and MEDIUM severity fixes. Real result from the enterprise demo: suite went from **13s flaky to 1.9s green**, all without deleting a single test (just removing test theater).

Chaos and circuit breaker stress error paths against the same recordings. Resilience verified.

---

## Part 3 — Architecture

### 3.1 Lifecycle View

The narrative architecture: how a developer's prompt becomes a working artifact.

```mermaid
---
config:
  layout: elk
---
flowchart TB
    Dev(["Developer<br/>(natural language prompt)"])

    subgraph Clients["AI Clients (30+ AgentSkills-compatible)"]
        CC["Claude Code"]
        CR["Cursor"]
        GH["GitHub Copilot"]
        GM["Gemini CLI"]
        OT["Codex · JetBrains · ..."]
    end

    subgraph Skills[".agents/skills/  (AgentSkills — auto-discovered)"]
        Atomic["8 Atomic Skills<br/>spec-mock · record-api<br/>setup-msw · setup-playwright<br/>record-flow-as-test · audit-tests<br/>chaos · circuit-breaker"]
        Workflow["4 Workflow Skills<br/>build-component · ship-feature<br/>harden-tests · spec-to-prod"]
    end

    subgraph MCP["MCP Servers (configured in .mcp.json)"]
        MockMCP["@up/mockkit-mcp<br/>12 tools · 13 prompts"]
        PlayMCP["@playwright/mcp<br/>(used by record-flow-as-test only)"]
    end

    subgraph Core["MockKit Core (npm: @up/mockkit)"]
        CLI["CLI<br/>start · record · replay · curl"]
        Server["Hono Server<br/>mock · replay · chaos · breaker"]
        RecEng["Recording Engine<br/>capture · templatize · snapshot"]
    end

    subgraph Browser["Browser Layer (alt path — no proxy)"]
        Chrome["Chrome Extension<br/>(offline mode, capture)"]
        MSWH["MSW Handlers<br/>(in-app interception)"]
    end

    subgraph Artifacts["Artifacts (committed to git)"]
        Spec[("api.yaml<br/>spec")]
        Recordings[("recordings.json<br/>THE artifact")]
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

    App <-->|HTTP or via Browser Layer| Server
    Server -->|forward (record only)| Upstream
    Upstream -.captured response.-> RecEng

    App -->|fetch/XHR| Chrome
    App -->|fetch/XHR| MSWH
    Chrome -.reads.-> Recordings
    MSWH -.reads.-> Recordings

    PlayMCP -->|drives browser| App
    Atomic -.writes.-> Tests
```

### 3.2 Three-Layer Stack

The vertical decomposition:

| Layer | Purpose | Lives in |
|---|---|---|
| **AgentSkills** | Natural-language entry point; spec-compliant skill files auto-discovered by AI clients | `.agents/skills/` — 8 atomic + 4 workflow |
| **MCP Server** | Tool/prompt API for AI; thin wrapper over Core | `@up/mockkit-mcp` — 12 tools, 13 prompts |
| **MockKit Core** | The actual mocking implementation | `@up/mockkit` — CLI, Hono server, recording engine, chaos, circuit breaker |

The AI never talks to Core directly. It talks to the MCP server, which talks to the CLI (for spawning) and HTTP (for runtime control of `/__mock__/*` admin endpoints). Skills are the prompt-engineering layer that makes intent map cleanly to tools.

### 3.3 The Browser Layer (Alt Path)

For browser-side mocking, two options that **bypass the HTTP server entirely**:

- **Chrome extension** — injects a MAIN-world content script (`offline-inject.ts`) that overrides `window.fetch` at `document_start`, intercepts every request, and serves matching responses from `recordings.json` (read via the extension's background service worker). Communication between page-context inject and isolated-world content script (`offline-content.ts`) happens via `postMessage`.
- **MSW handlers** — generated from the spec, run inside a service worker installed by the app. Best for unit/component tests and Storybook.

Both read the same `recordings.json` the HTTP server reads. The artifact is shared.

### 3.4 The Central Artifact

Every component in the architecture either reads from or writes to `recordings.json`:

- **Recording Engine** — reads/writes (capture mode) or reads (replay mode).
- **Hono Server** — reads (replay mode) when serving cached responses.
- **Chrome extension** — reads (offline mode).
- **MSW handlers** — read (when generated from a recording, not just from spec).
- **Playwright fixture** — reads (test fixture in CI).
- **Audit prompt** — reads tests that consume it.
- **AI skills** — read for context, write via record-flow-as-test (which extends it).

The lifecycle thesis collapses to: **the file is the contract, every layer is plumbing**.

### 3.5 Complete System Map

The exhaustive reference — every package, every MCP server, every input/output. Use this for "what connects to what" lookups.

```mermaid
---
config:
  layout: elk
---
flowchart TB
    subgraph Inputs["Inputs"]
        OAS[("OpenAPI · api.yaml")]
        SS[("Simple Schema · schema.yaml")]
        HAR[("HAR import · browser export")]
    end

    subgraph AI["AI Layer"]
        AIClient["AI Client (Claude Code, Cursor, Copilot, Gemini, Codex, 30+)"]
        SkillsRepo["AgentSkills · .agents/skills/ · 8 atomic + 4 workflow"]
    end

    subgraph MCPLayer["MCP Servers"]
        MockMCP["@up/mockkit-mcp · 12 tools · 13 prompts"]
        PlayMCP["@playwright/mcp · browser primitives"]
        DevToolsMCP["chrome-devtools-mcp · Lighthouse · Performance"]
    end

    subgraph MK["MockKit Monorepo"]
        subgraph CorePkg["@up/mockkit · core"]
            CLI["CLI · start, record, replay, curl, validate, routes, init, examples, recordings, import, snapshots, chaos, cache, browser"]
            Hono["Hono Server · 3 modes: mock, record, replay · /__mock__/* admin endpoints"]
            RecEng["Recording Engine · capture, templatize, snapshot, HAR export, diff, merge"]
            ChaosE["Chaos Engine · error rate, latency, timeout, circuit breaker"]
            PWFix["Playwright Fixture · @up/mockkit-playwright"]
            BrowserBundle["Browser Bundle · generated MSW handlers from spec"]
        end
        MCPPkg["@up/mockkit-mcp · thin wrapper over CLI + HTTP"]
        subgraph CXT["chrome-extension"]
            DevPanel["DevTools Panel (React UI)"]
            BG["Background SW · state, proxy, injection"]
            CSIso["offline-content.ts (ISOLATED world bridge)"]
            CSMain["offline-inject.ts (MAIN world fetch override)"]
        end
        Preview["@up/mockkit-preview · Vite app · visualize MSW handlers"]
    end

    subgraph BrowserLayer["Browser Layer (in App Under Test)"]
        MSWBrowser["MSW Service Worker"]
        ExtRuntime["Chrome Extension Runtime"]
    end

    subgraph TestLayer["Test Layer"]
        PWRunner["Playwright Test Runner"]
        CIRunner["CI Runner (GitHub Actions, GitLab, etc.)"]
    end

    subgraph Outputs["Outputs (committed to git)"]
        Recs[("recordings.json · THE artifact")]
        Snaps[("snapshots/ · named scenarios")]
        Tests[("tests/*.spec.ts · Playwright specs")]
        HAREx[("HAR exports")]
        Lighthouse[("Lighthouse reports")]
        MSWFiles[("src/mocks/handlers.ts")]
    end

    subgraph Apps["App Under Test"]
        FE["Frontend App (browser)"]
        BE["Backend Service (HTTP client)"]
    end

    Upstream["Real Upstream API (only during record)"]

    OAS -->|spec| CLI
    SS -->|spec| CLI
    HAR -.import.-> CLI

    AIClient -.auto-discover.-> SkillsRepo
    SkillsRepo -->|tool / prompt call| MockMCP
    SkillsRepo -.record-flow-as-test only.-> PlayMCP
    SkillsRepo -.scenario 18 only.-> DevToolsMCP

    MockMCP -->|spawn / HTTP| CLI
    MockMCP -.HTTP /__mock__/*.-> Hono

    CLI --> Hono
    CLI --> RecEng
    CLI --> ChaosE
    Hono --> RecEng
    Hono --> ChaosE
    CLI -.generates.-> BrowserBundle
    CLI -.generates.-> MSWFiles
    BrowserBundle --> PWFix

    DevPanel <-->|chrome.runtime| BG
    BG -->|chrome.scripting| CSIso
    BG -->|chrome.scripting + MAIN world| CSMain
    CSIso <-.postMessage.-> CSMain
    BG -.imports.-> ExtRuntime
    CSMain -.intercepts fetch.-> ExtRuntime

    Preview -.reads.-> OAS
    Preview -.renders.-> MSWFiles

    FE <-->|HTTP fetch/XHR| Hono
    BE <-->|HTTP| Hono
    FE -->|fetch/XHR| MSWBrowser
    FE -->|fetch/XHR via injected| ExtRuntime
    MSWBrowser -.uses.-> MSWFiles

    RecEng <-.reads/writes.-> Recs
    RecEng <-.reads/writes.-> Snaps
    RecEng -.exports.-> HAREx
    ExtRuntime -.reads.-> Recs
    MSWBrowser -.reads.-> Recs

    Hono -->|forward (record only)| Upstream
    Upstream -.captured response.-> RecEng

    PlayMCP -.drives.-> FE
    PWFix -.starts mock for tests.-> Hono
    PWRunner -->|uses fixture| PWFix
    PWRunner -.executes.-> Tests
    SkillsRepo -.writes via record-flow-as-test.-> Tests

    DevToolsMCP -.measures.-> FE
    DevToolsMCP -.outputs.-> Lighthouse

    CIRunner -->|invokes| PWRunner
    CIRunner -.starts via fixture.-> Hono
    CIRunner -.reads.-> Recs
```

---

## Part 4 — Capabilities (What MockKit Does)

### 4.1 Server Modes

The Hono server runs in three modes, all from the same binary:

| Mode | Command | Behavior |
|---|---|---|
| **mock** | `mockkit start -s api.yaml` | Generates responses from spec. Deterministic with seed. No upstream. |
| **record** | `mockkit record -t https://api.example.com` | Forwards to upstream, returns the real response unchanged, writes the request/response pair to disk. |
| **replay** | `mockkit replay -r recordings.json` | Serves saved responses from a recording file. No upstream. |

Modes can be combined — a `replay` server can have `chaos` enabled for error injection on top of replays, or a `mock` server can fall back to recordings for routes the spec doesn't cover.

### 4.2 Capturing API Traffic — Two Paths

**Path A — `mockkit curl`** (no proxy, one-shot per endpoint):

```bash
mockkit curl https://api.example.com/users -o ./fixtures/api.json
mockkit curl https://api.example.com/users/1 -o ./fixtures/api.json
mockkit curl https://api.example.com/posts -o ./fixtures/api.json --templatize
```

Each call makes a direct `fetch()` to the URL and appends to the recording file. Best for: building fixtures from a known list of endpoints, CI seed data, scripting against documented APIs, capturing third-party APIs with known cURL examples.

**Path B — `mockkit record`** (proxy, captures real app traffic):

```bash
mockkit record -t https://api.example.com -o ./fixtures/api.json --templatize
```

App points at `http://localhost:3000` (NOT `https://` — the proxy is plain HTTP). Every request is forwarded upstream, response unchanged, request/response pair saved. Best for: capturing realistic multi-request flows, auth handshakes (login → token → use), streaming responses (SSE, NDJSON), discovering "what does my app actually fire."

**Common gotcha**: don't use curl's `--proxy` flag against the recording proxy. MockKit isn't a CONNECT/MITM proxy — it's a forwarder. Swap the URL host directly: `https://api.example.com/...` → `http://localhost:3000/...`.

### 4.3 Templatization

Add `--templatize` to `record` or `curl` to auto-detect dynamic values and replace them with template placeholders. Patterns from `packages/mockkit/src/recording/templatize.ts`:

| Pattern | Detected | Replaced with |
|---|---|---|
| UUID v4 | `550e8400-e29b-41d4-a716-446655440000` | `{{uuid}}` |
| MongoDB ObjectId | `507f1f77bcf86cd799439011` | `{{mongoId}}` |
| ISO 8601 date | `2026-04-21T10:00:00Z` | `{{isoDate}}` |
| Unix timestamp (seconds) | `1745234567` | `{{timestamp}}` |
| Unix timestamp (ms) | `1745234567890` | `{{timestampMs}}` |
| JWT token | `eyJhbGc...` | `{{jwt}}` |
| Email | `user@example.com` | `{{email}}` |
| IPv4 | `192.0.2.1` | `{{ipv4}}` |
| URL | `https://example.com/foo` | `{{url}}` |

Templates resolve to fresh values per-request during replay. To templatize an existing recording (without re-capture), POST to the running server: `curl -X POST http://localhost:3000/__mock__/recordings/templatize`.

For arbitrary fields not covered by patterns, edit the JSON file and use `{{your-placeholder}}` directly in `response.body`.

### 4.4 Replay & Matching Strategies

```bash
mockkit replay -r recordings.json --match path-query --latency
```

Four matching strategies (configurable via `--match`):

| Strategy | Matches on | Use when |
|---|---|---|
| `exact` | method + path + query + body | Strictest — body-sensitive POSTs |
| `path-query` (default) | method + path + query string | Most common — REST APIs with query params |
| `path-only` | method + path | Loose — when query strings are noisy |
| `fuzzy` | method + path + query subset | Most permissive — tests pass extra params |

`--latency` flag replays the recorded `responseTime` for each entry. Without it, replays return instantly.

### 4.5 Snapshots

Named variants of recordings, switchable at runtime without restarting the server.

```bash
mockkit start -s api.yaml
mockkit snapshots save -n happy-path
mockkit snapshots save -n edge-cases
mockkit snapshots save -n error-states

mockkit snapshots activate -n error-states
mockkit snapshots activate -n happy-path
```

Live management via runtime endpoints:

```bash
GET  /__mock__/snapshots                    # list all
POST /__mock__/snapshots/<name>/activate    # switch
POST /__mock__/snapshots/deactivate         # back to live
```

Snapshots live in `.mockkit/snapshots/` by default (configurable with `--data-dir`).

### 4.6 Chaos & Circuit Breaker

**Chaos** injects random failures: error responses (5xx), latency, timeouts. Enabled via CLI flags on `mockkit start` (NOT on `mockkit replay`):

```bash
mockkit start -s api.yaml --chaos --chaos-error-rate 0.3 --chaos-latency 200-800
```

For `replay` servers, use the runtime endpoint:

```bash
curl -X PUT http://localhost:3000/__mock__/chaos \
  -H 'content-type: application/json' \
  -d '{"enabled": true, "errorRate": 0.3, "latencyMin": 200, "latencyMax": 800}'
```

Disable: `{"enabled": false}`. Inspect: `GET /__mock__/chaos`.

**Circuit Breaker** trips per-endpoint after N failures, stays tripped for a cooldown:

```bash
curl -X POST http://localhost:3000/__mock__/circuit-breaker/enable \
  -H 'content-type: application/json' \
  -d '{"failureThreshold": 5, "cooldownMs": 30000}'

# Manually trip an endpoint for testing
curl -X POST http://localhost:3000/__mock__/circuit-breaker/trip \
  -d '{"path": "/api/incidents"}'

# Inspect open/half-open/closed circuits
curl http://localhost:3000/__mock__/circuit-breaker
```

Pairs with chaos: chaos creates failures, circuit breaker trips when threshold hits, app exercises fallback paths.

### 4.7 Cache

Response caching with TTL and max-entry limits.

```bash
mockkit start -s api.yaml --cache --cache-ttl 300 --cache-max 1000
```

Inspect/invalidate at runtime:

```bash
GET  /__mock__/cache                  # stats
POST /__mock__/cache/invalidate       # by pattern
```

Useful for simulating expensive backends or testing client-side cache-busting logic.

### 4.8 Built-in Admin Endpoints (`/__mock__/*`)

Every running server exposes:

| Endpoint | Purpose |
|---|---|
| `GET /__mock__/info` | Server info, routes, config |
| `GET /__mock__/health` | Health check |
| `GET /__mock__/chaos` | Chaos config and stats |
| `PUT /__mock__/chaos` | Update chaos config |
| `GET /__mock__/cache` | Cache config and stats |
| `POST /__mock__/cache/invalidate` | Invalidate cache by pattern |
| `GET /__mock__/circuit-breaker` | Circuit state and stats |
| `POST /__mock__/circuit-breaker/enable` | Enable circuit breaker |
| `POST /__mock__/circuit-breaker/trip` | Manually trip a circuit |
| `POST /__mock__/circuit-breaker/reset` | Reset a circuit |
| `GET /__mock__/recordings` | Stats on current recordings |
| `GET /__mock__/recordings/export` | HAR export |
| `POST /__mock__/recordings` | Push new recordings (replay mode) |
| `POST /__mock__/recordings/save` | Persist in-memory store to disk |
| `POST /__mock__/recordings/templatize` | Bulk templatize all recordings |
| `DELETE /__mock__/recordings` | Clear store |
| `GET /__mock__/snapshots` | List snapshots |
| `POST /__mock__/snapshots` | Create a snapshot |
| `POST /__mock__/snapshots/<name>/activate` | Activate a snapshot |
| `POST /__mock__/snapshots/deactivate` | Deactivate snapshot, return to live |

There is no per-entry PATCH/DELETE on recordings. To edit a single entry, three options: edit the JSON file directly + restart, use the Chrome extension's row-level Edit/Delete buttons, or use snapshots to keep multiple variants.

### 4.9 Other CLI Subcommands

| Command | Purpose |
|---|---|
| `mockkit init` | Interactively create a new Simple Schema spec |
| `mockkit examples` | Browse/run built-in example specs |
| `mockkit validate -s <spec>` | Validate an OpenAPI/Simple Schema spec |
| `mockkit routes -s <spec>` | List all routes from a spec |
| `mockkit recordings` | Manage recordings on a running server |
| `mockkit import --har <file>` | Import HAR file as recordings |
| `mockkit chaos` | Manage chaos config from CLI |
| `mockkit cache` | Manage cache from CLI |
| `mockkit browser` | Browser-mode utilities |

---

## Part 5 — Packages

### 5.1 `@up/mockkit` (Core)

The primary CLI and library. Hono-based HTTP server, recording engine, chaos engine, snapshot manager, OpenAPI parser, Simple Schema parser, templatizer.

**Install**: `npm install -g @up/mockkit`
**Source**: `packages/mockkit/`
**Subdirs**: `cli/`, `core/`, `recording/`, `openapi/`, `playwright/`, `streaming/`, `browser/`, `types/`, `utils/`

Standalone — works without any AI integration. The MCP server and Chrome extension are thin layers on top.

### 5.2 `@up/mockkit-mcp` (MCP Server)

Exposes MockKit's capabilities as MCP tools and prompts. Configured in `.mcp.json`:

```json
{
  "mcpServers": {
    "mockkit": {
      "command": "npx",
      "args": ["-y", "@up/mockkit-mcp"]
    }
  }
}
```

**12 tools** — server lifecycle (start, stop, status), recordings (list, push, save, clear, debug), snapshots (list, activate, save), chaos (config, get-state).

**13 prompts** (callable as `/mcp__mockkit__<name>` in Claude Code, or auto-activated by AgentSkills):

| Prompt name | Purpose |
|---|---|
| `record-and-replay` | Guide through recording API traffic from a live server and replaying it |
| `troubleshoot-replay` | Debug why a replay is not matching incoming requests |
| `chaos-testing-setup` | Configure chaos engineering for resilience testing |
| `setup-playwright` | Full Playwright integration setup for browser-based API mocking |
| `generate-tests` | Generate test cases from recorded MockKit sessions |
| `api-mock-quickstart` | Fastest path from zero to a running mock server |
| `snapshot-workflow` | Manage named snapshots for different test scenarios |
| `compare-api-versions` | Compare two API versions using recordings to find breaking changes |
| `msw-integration` | Set up Mock Service Worker in a React/Next.js app |
| `e2e-test-setup` | Complete end-to-end workflow from running app to automated Playwright tests |
| `circuit-breaker-setup` | Configure circuit breaker for resilience testing |
| `record-flow-as-test` | Record a UI flow against MockKit and emit a committable Playwright .spec.ts |
| `audit-test-quality` | Audit existing Playwright tests for fragility patterns; report + auto-fix |

### 5.3 Chrome Extension (`mockkit-extension`)

A Manifest V3 extension that adds a "MockKit DevTools" panel to Chrome DevTools. Features:

- **Record tab** — capture browser fetch/XHR traffic, edit body, push recordings to a running MockKit server or save offline.
- **Mock tab** — three sub-views:
  - **Live** — point the browser at a running MockKit server.
  - **Offline** — serve recordings directly from the browser, no server needed. Uses MAIN-world content script injection to override `window.fetch` at `document_start`.
  - **Hosts** — manage which hosts get intercepted.
- **Explore tab** — browse recorded sessions, diff, replay individual requests.
- **Chaos panel** — error injection sliders (offline mode only; for server-side chaos use the runtime endpoint).
- **Failover monitoring** — watches the configured MockKit server and surfaces health changes.

**Distribution**: built as a self-contained zip (`mockkit-extension.zip`), installable via `chrome://extensions` "Load unpacked" or a future Chrome Web Store listing.

**Browser-side architecture**: two content scripts.

- `offline-content.ts` (ISOLATED world) — bridge to background SW via `chrome.runtime`.
- `offline-inject.ts` (MAIN world) — registered as a persistent content script at `document_start` with `world: "MAIN"`, so it overrides `window.fetch` BEFORE any inline page script fires its first request. Communicates with the bridge via `postMessage`.

### 5.4 `@up/mockkit-preview`

A small Vite app for visualizing MSW handlers generated from a spec. Renders the spec's routes, lets you tweak responses, and produces handler files committable to your app's `src/mocks/`. Orthogonal to the runtime server path — purely a generation/visualization tool.

**Install**: `npm install --save-dev @up/mockkit-preview`
**Use**: `npx mockkit-preview` (opens local dev server with the visualizer).

### 5.5 `@up/mockkit-playwright` (Fixture)

A Playwright test fixture that handles MockKit lifecycle for E2E tests. Re-exported from `@up/mockkit` core (so `import { test } from '@up/mockkit-playwright'` works).

**What it does**:

- Starts a MockKit server per worker (configurable port, spec, recording file).
- Tears down on test completion.
- Optionally enables chaos / templatization per test via fixture options.
- Provides `mockkit` fixture object with helpers for runtime control (`mockkit.startSnapshot('error-states')`, `mockkit.resetRecordings()`, etc.).
- Adds `mockkit.url` to the Playwright `page` so tests can `page.goto(mockkit.url + '/...')`.

**Setup**:

```ts
// playwright.config.ts
import { defineConfig } from '@up/mockkit-playwright';

export default defineConfig({
  mockkit: {
    spec: './api.yaml',
    seed: 42,
    hosts: ['api.example.com', 'cdn.example.com'],
  },
  // ... rest of Playwright config
});
```

```ts
// tests/example.spec.ts
import { test, expect } from '@up/mockkit-playwright';

test('happy path', async ({ page, mockkit }) => {
  await page.goto(`${mockkit.appUrl}/dashboard`);
  await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible();
});
```

---

## Part 6 — AI Integration

### 6.1 The MCP Server Layer

MockKit's MCP server (`@up/mockkit-mcp`) is a thin Node.js process that:

1. Listens on stdio for MCP requests from the AI client.
2. Exposes 12 tools (callable functions with typed Zod schemas).
3. Exposes 13 prompts (parameterized prompt templates with arg schemas).
4. Spawns the MockKit CLI as needed (`mockkit start ...`) and tracks its lifecycle.
5. Talks HTTP to running MockKit servers for runtime control (`PUT /__mock__/chaos`, etc.).

Adding it to an AI client is one config block:

```json
{
  "mcpServers": {
    "mockkit": {
      "command": "npx",
      "args": ["-y", "@up/mockkit-mcp"]
    }
  }
}
```

Once configured, the AI can call any tool or prompt by name. For Claude Code specifically, prompts surface as `/mcp__mockkit__<prompt-name>` slash commands.

### 6.2 The AgentSkills Layer

AgentSkills is an open standard (https://agentskills.io) for AI client-discovered, model-activated skill files. Each skill is a `SKILL.md` file with YAML frontmatter:

```markdown
---
name: spec-mock
description: Stand up a MockKit server from an OpenAPI/Simple Schema spec. Use when the user has a spec but no backend yet, says "stand up a mock", or is starting frontend work before the backend exists.
license: MIT
metadata:
  category: atomic
  mcp-server: mockkit
  mcp-prompt: api-mock-quickstart
---

# spec-mock

[Markdown body — instructions the AI follows when this skill activates]
```

The model **auto-activates** a skill when the user's prompt matches the description. No slash command typing — the user describes intent in natural language, the model picks the right skill.

**Discovery paths** (vary by client):
- Claude Code: `.claude/skills/` and `~/.claude/skills/`
- VS Code Copilot, Cursor, etc.: `.agents/skills/` (the spec quickstart default)
- Always supported: directory scan for `SKILL.md` files

MockKit ships skills at `mockkit/main/.agents/skills/`, organized by kind:

```
.agents/skills/
├── README.md
├── atomic/         # 8 1:1 wrappers around individual MCP prompts
└── workflows/      # 4 multi-step compositions
```

### 6.3 The 8 Atomic Skills

Each atomic skill wraps one MCP prompt with a clear `When to activate` section so the model can match user intent.

| Skill | Wraps | Activate when... |
|---|---|---|
| **spec-mock** | `api-mock-quickstart` | User has a spec, wants a mock server |
| **record-api** | `record-and-replay` | User wants to capture real API traffic |
| **setup-msw** | `msw-integration` | User wants browser-side mocking in React/Next.js |
| **setup-playwright** | `setup-playwright` | User wants E2E tests with shared MockKit fixture |
| **record-flow-as-test** | `record-flow-as-test` | User describes a UI flow and wants a Playwright spec |
| **audit-tests** | `audit-test-quality` | User wants to find/fix flaky tests |
| **chaos** | `chaos-testing-setup` | User wants to inject errors for resilience testing |
| **circuit-breaker** | `circuit-breaker-setup` | User wants per-endpoint failure thresholds |

### 6.4 The 4 Workflow Skills

Workflows compose atomic skills via body instructions (the AgentSkills spec deliberately omits a composition primitive — the model follows the markdown).

| Workflow | Composes | Purpose |
|---|---|---|
| **build-component** | spec-mock + setup-msw | Stand up a mock and scaffold a frontend component against it |
| **ship-feature** | build-component + record-flow-as-test + audit-tests | End-to-end feature delivery from spec to merge-ready PR |
| **harden-tests** | chaos + circuit-breaker + audit-tests | Stress error paths, find untested code, clean fragility |
| **spec-to-prod** | All of the above | Full lifecycle — spec to PR-ready in ~10 minutes |

A workflow's body tells the model the order. Example excerpt from `ship-feature`:

```
1. Activate the build-component skill with the given spec.
2. Confirm the component works in the browser.
3. Activate record-flow-as-test with the user's flow description.
4. Run the generated test once to confirm it passes.
5. Activate audit-tests on the new file with applyFixes: true.
6. Re-run the suite, summarize for the user.
```

### 6.5 How Activation Actually Works

Three-stage progressive disclosure (per the AgentSkills spec):

1. **Startup** — AI client scans skill directories, loads only `name` + `description` of each skill (~100 tokens each). Just enough to know the skill exists.
2. **Match** — When the user prompts, the model compares the prompt against every loaded skill description. It auto-activates the best match (or no match if nothing fits).
3. **Execute** — Only when activated does the full SKILL.md body load. The model follows the body as a runbook, calling MCP tools/prompts as needed.

Cost: ~100 tokens per skill per session for discovery, only loading the full body of skills that fire. Cheap enough to ship dozens of skills without context bloat.

---

## Part 7 — Browser Integration

### 7.1 The Chrome Extension (Detail)

**Manifest V3**, single bundled extension installable from a zip. Three core pieces:

**1. Background service worker** (`background.ts`)
- Holds extension state (active hosts, recordings, offline config, failover state).
- Mediates between DevTools panel UI and content scripts.
- Persists state to `chrome.storage.local`.
- Registers content scripts via `chrome.scripting.registerContentScripts`.
- Handles `injectOfflinePageScript` requests by seeding `window.__OFFLINE_CONFIG__` in MAIN world before injecting `offline-inject.js`.

**2. ISOLATED-world content script** (`offline-content.ts`)
- Runs in the content-script world, has access to `chrome.runtime`.
- Bridge between the page-context inject script and the background SW.
- Translates `postMessage` events from the page to `chrome.runtime.sendMessage` and back.

**3. MAIN-world inject script** (`offline-inject.ts`)
- Runs in the page's actual JavaScript context, has access to `window.fetch`.
- Registered as a content script with `world: "MAIN"` and `runAt: "document_start"` — so it activates BEFORE any inline page script fires its first request.
- Overrides `window.fetch` with `interceptedFetch`. For each request, decides via `shouldIntercept(url, method)` whether to handle it locally (offline cache) or pass to original fetch.
- Handles streaming responses (SSE replay with `entryToSSEResponse`).
- 5-second timeout falls back to original fetch if background is unresponsive.

**Why MAIN-world matters**: ISOLATED-world content scripts can't override `window.fetch` for the page. The page would still see the native fetch. MAIN world gives the inject script the same JS context as the page itself, so the override is real.

**Why `document_start`**: inline `<script>` tags at the top of the HTML can fire fetches before any later script runs. By registering as a persistent content script at `document_start`, the inject script wins the race.

### 7.2 MSW Integration

[Mock Service Worker](https://mswjs.io) is a popular browser-side API mocking library that uses a service worker. MockKit's `setup-msw` skill (and the `msw-integration` MCP prompt) generates handlers from your spec.

```bash
# Manual flow (without the skill):
npx msw init public/                        # install service worker file
npx mockkit browser --generate-msw \
  -s api.yaml -o src/mocks/handlers.ts      # generate handlers
```

Then in your app entry point (e.g., `src/main.tsx`):

```ts
if (process.env.NODE_ENV === 'development') {
  const { worker } = await import('./mocks/browser');
  await worker.start();
}
```

MSW handlers can additionally consume `recordings.json` to serve recorded responses (not just spec-generated ones). Useful for Storybook scenarios and component tests.

### 7.3 The Preview UI

`@up/mockkit-preview` is a separate Vite app for visualizing what MSW would serve. Run it pointed at your spec:

```bash
npx mockkit-preview -s ./api.yaml
```

Renders every route, lets you trigger requests against generated handlers, and exports the handler files. Useful for design review with non-engineers ("does this response look right?") before committing fixtures.

---

## Part 8 — Workflows (End-to-End Recipes)

### 8.1 Day 0 — Spec to Working Mock

```bash
# Validate the spec
mockkit validate -s api.yaml

# Start the server
mockkit start -s api.yaml -p 9876

# Confirm it's up
curl http://localhost:9876/__mock__/health

# Inspect routes
mockkit routes -s api.yaml
```

Or via AI: *"Stand up a mock from `./api.yaml`."*

### 8.2 Capture Real API Traffic

```bash
# Start the recording proxy targeting real API
mockkit record -t https://api.example.com -o ./fixtures/api.json --templatize

# (Point your app at http://localhost:3000 and use it normally)

# Ctrl+C to stop and auto-save
```

For a single endpoint with a known curl, skip the proxy:

```bash
mockkit curl https://api.example.com/users \
  -H 'Authorization: Bearer <token>' \
  -o ./fixtures/users.json --templatize
```

### 8.3 Replay for Local Dev / CI

```bash
# Basic replay
mockkit replay -r ./fixtures/api.json

# With original timings (slower but realistic)
mockkit replay -r ./fixtures/api.json --latency

# Looser matching for noisy query strings
mockkit replay -r ./fixtures/api.json --match path-only
```

### 8.4 Auto-Generate a Playwright Test

Via AI:

> "Record a Playwright test for: load the dashboard, click the first incident, fill the title with 'DB outage', set severity to high, submit, verify it appears in the list. Output to `tests/create-incident.spec.ts`."

The AI activates `record-flow-as-test`, composes MockKit MCP + Playwright MCP, drives the app, captures aria snapshots, writes the spec file with `afterEach` cleanup, runs it once to confirm.

### 8.5 Audit Existing Tests

Via AI:

> "Audit `tests/` with `applyFixes: true` and report what changed."

The AI activates `audit-tests`, walks 9 fragility patterns, applies HIGH/MEDIUM fixes automatically, re-runs the suite, reports before/after numbers.

### 8.6 Resilience Testing

Server already running in another terminal. Inject moderate chaos:

```bash
curl -X PUT http://localhost:3000/__mock__/chaos \
  -H 'content-type: application/json' \
  -d '{"enabled": true, "errorRate": 0.2, "latencyMin": 200, "latencyMax": 800}'
```

Enable circuit breaker:

```bash
curl -X POST http://localhost:3000/__mock__/circuit-breaker/enable \
  -d '{"failureThreshold": 5, "cooldownMs": 30000}'
```

Run your test suite. Tests that fail under chaos fall into three buckets: real bugs (app doesn't handle errors), fragile tests (assumed happy path), or flaky timing. The `harden-tests` workflow does this end-to-end and pairs it with the audit pass to clean up the fragility category automatically.

### 8.7 Full Lifecycle in One Prompt

> "Implement the IncidentList component end-to-end from `./api.yaml`."

The AI activates `ship-feature` (or `spec-to-prod` for the longer arc with chaos+audit). It composes:

1. `spec-mock` — start MockKit on the spec.
2. `setup-msw` — wire MSW handlers if not already.
3. *(scaffold the component)*
4. `record-flow-as-test` — generate a Playwright spec.
5. `audit-tests` — clean up the new file.
6. *(report PR-ready summary)*

Total time: ~10 minutes for a small component, with most time spent on the user describing the flow in step 4.

---

## Part 9 — Distribution

MockKit ships as four artifacts, all built into `~/projects/mockkit-builds/` from the monorepo root:

| Artifact | Build command | Install |
|---|---|---|
| `up-mockkit-1.0.0.tgz` | `pnpm pack:mockkit` | `npm install -g <tgz>` |
| `up-mockkit-mcp-1.0.0.tgz` | `pnpm pack:mcp` | `npm install -g <tgz>` |
| `up-mockkit-preview-1.0.0.tgz` | `pnpm pack:preview` | `npm install -g <tgz>` |
| `mockkit-extension.zip` | `pnpm zip:extension` | Load unpacked at `chrome://extensions` |

Standard distribution will be via the npm registry once published. The Chrome extension is not yet on the Chrome Web Store; ships via direct zip for now.

---

## Part 10 — Documentation Index

The `mockkit/main/docs/` folder contains:

| Doc | Covers |
|---|---|
| `CLI_GUIDE.md` | Every CLI subcommand, every flag, with examples (565 lines) |
| `RECORDING_WITH_CURL.md` | One-shot capture via `mockkit curl`, no proxy |
| `RECORDING_WITH_PROXY.md` | Proxy-based capture via `mockkit record`, with worked example and gotchas |
| `BROWSER_MODE.md` | Browser-side mocking via MSW and the Chrome extension |
| `chrome-extension-guide.md` | Chrome extension features in depth |
| `playwright-integration.md` | Playwright fixture setup and use |
| `gitlab-ci-integration.md` | CI integration recipes |
| `MOCKKIT_VS_WIREMOCK.md` | Comparison with WireMock for migration audiences |
| `FAQ.md` | Common questions |
| `FEATURES.md` | Feature inventory |
| `USER_GUIDE.md` | High-level onboarding |
| `WORKFLOW_DIAGRAMS.md` | Workflow visuals |
| `DISTRIBUTING.md` | How to build/ship the four artifacts |

In `mockkit-presentation/` (this repo):

| Doc | Covers |
|---|---|
| `ARCHITECTURE.md` | Lifecycle view + complete system map (mermaid + SVG) |
| `architecture.svg` | Hand-crafted vector architecture diagram |
| `WALKTHROUGH.md` | This document |
| `pitch-deck-LIFECYCLE.html` | 14-slide deck telling the lifecycle story |
| `pitch-deck-A.html` through `pitch-deck-E.html` | Earlier deck variants (different angles) |
| `DEMO_LIFECYCLE.md` | 5–7 minute demo recipe for the lifecycle deck |
| `DEMO_AI_NATIVE_LOOP.md` | 4-minute demo recipe for deck E |
| `DEMO_SPEC_TO_SHIP.md` | 6-minute spec-to-ship demo recipe |

In `mockkit-enterprise-demo/`:

| Doc | Covers |
|---|---|
| `scenarios/mcp-prompts.md` | 20+ MCP scenarios covering common workflows |
| `scenarios/scenario-*.sh` | Shell scripts for individual scenarios |

---

## Part 11 — Glossary

| Term | Meaning |
|---|---|
| **AgentSkills** | Open standard (agentskills.io) for AI-discovered, model-activated skill files. SKILL.md per folder. |
| **Atomic skill** | A skill that wraps one MCP prompt with intent-matching description. |
| **Workflow skill** | A skill that composes atomic skills via body instructions (no spec primitive). |
| **MCP** | Model Context Protocol — standard for AI clients to talk to tool/prompt servers. |
| **MCP server** | A process that exposes tools and prompts to MCP-compatible AI clients. |
| **MCP tool** | A typed callable function exposed by an MCP server. |
| **MCP prompt** | A parameterized prompt template exposed by an MCP server. |
| **Recording** | A captured request/response pair, stored in `recordings.json`. |
| **Snapshot** | A named variant of recordings, switchable at runtime. |
| **Templatization** | Auto-replacing dynamic values (UUIDs, JWTs) with placeholders. |
| **Chaos** | Random failure injection — error responses, latency, timeouts. |
| **Circuit breaker** | Per-endpoint failure threshold pattern; trip after N failures, cool down before retry. |
| **MSW** | Mock Service Worker — browser-side API mocking via service worker. |
| **MAIN world** | Chrome extension content script context with same JS as the page; can override `window.fetch`. |
| **ISOLATED world** | Default content script context; isolated from page JS but has `chrome.runtime`. |
| **Hono** | The web framework MockKit's server is built on. |
| **OpenAPI** | API specification standard (a.k.a. Swagger). |
| **Simple Schema** | MockKit's lightweight YAML spec format, simpler than OpenAPI. |
| **HAR** | HTTP Archive — browser's network log export format. Importable via `mockkit import`. |
| **Replay matching strategies** | `exact`, `path-query` (default), `path-only`, `fuzzy` — how MockKit decides which recording matches an incoming request. |
| **`/__mock__/*`** | Built-in admin endpoints on every MockKit server for runtime control. |

---

## Appendix A — Why MockKit Exists

The verification gap in AI-augmented development:

- AI generates code in seconds.
- Verification of that code (does it actually work against the real API?) takes hours.
- Most AI assistants have no API runtime — they read prose docs, infer types, and guess.
- Tests AI writes mock the function it just wrote, proving nothing.

MockKit closes this gap by giving the AI an API runtime: a tool layer (MCP) it can call, deterministic responses (from spec or recordings), and a feedback loop (recording-flow-as-test). The AI generates code, hits a real mock, sees a real response, fixes a real bug — all in one prompt.

The lifecycle thesis ("one artifact, whole lifecycle") is the unifier. Most tools cover one phase. MockKit covers all five because the recording artifact has the same shape across phases — it's just consumed differently each time.

## Appendix B — The Three Most Common Misconceptions

1. **"It's just a wrapper around `mockkit` CLI."** — The MCP server is, but AgentSkills + the lifecycle integration are the actual product. The CLI is the implementation detail.
2. **"You need to use the AI features to use MockKit."** — No. The CLI is fully standalone. The AI layer is additive.
3. **"It's another mock server like WireMock or json-server."** — Those are passive mock servers. MockKit is opinionated about the lifecycle (record→replay→test→audit), ships an AI tool layer, and provides a Chrome extension for browser-side capture/replay. It overlaps in "serve fake responses" but the value is in the surrounding workflow.

## Appendix C — Quick Reference — Skills to MCP Prompts

For LLM/RAG retrieval, here's the explicit mapping:

| Skill name | MCP prompt name | Required args |
|---|---|---|
| spec-mock | `api-mock-quickstart` | `spec`, optional `port` |
| record-api | `record-and-replay` | `spec`, `targetUrl` |
| setup-msw | `msw-integration` | `spec` |
| setup-playwright | `setup-playwright` | `spec`, `hosts` |
| record-flow-as-test | `record-flow-as-test` | `spec`, `appUrl`, `flow`, optional `outputPath`, `seed` |
| audit-tests | `audit-test-quality` | optional `testDir`, `testFiles`, `applyFixes` |
| chaos | `chaos-testing-setup` | optional `errorRate` |
| circuit-breaker | `circuit-breaker-setup` | optional `failureThreshold`, `cooldownMs` |

Workflow skills don't have direct MCP-prompt mappings — they orchestrate atomic skills via their bodies.

---

End of walkthrough.
