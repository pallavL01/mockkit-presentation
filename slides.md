---
theme: mockkit
title: 'MockKit: Spec-Driven API Mocking Infrastructure'
info: |
  Replacing fragile static JSON fixtures with programmable, realistic,
  and highly resilient mock infrastructure.
author: Pallav Laskar
transition: slide-left
drawings:
  enabled: false
colorSchema: light
hideInToc: true
---

# _MockKit:_ The Complete Spec-Driven API Mocking Infrastructure

<p class="subtitle">
Replacing fragile static JSON fixtures with programmable, realistic,
and highly resilient mock infrastructure.
</p>

<p style="margin-top: 16px; font-size: 14px; color: var(--slate-500);">
by <strong style="color: var(--slate-700);">Pallav Laskar</strong>
</p>

<div class="card" style="margin-top: 10px; display: flex; align-items: center; justify-content: center; padding: 20px;">
  <div style="display: flex; align-items: center; gap: 20px;">
    <div style="text-align: center;">
      <div style="font-size: 36px; margin-bottom: 6px;">📄</div>
      <div style="font-size: 16px; font-weight: 600;">Spec</div>
      <div style="font-size: 11px; color: var(--slate-400);">(YAML/JSON)</div>
    </div>
    <div style="display: flex; flex-direction: column; gap: 28px; margin: 0 20px;">
      <div style="font-size: 28px; color: var(--slate-400);">➜</div>
      <div style="font-size: 28px; color: var(--slate-400);">➜</div>
      <div style="font-size: 28px; color: var(--slate-400);">➜</div>
    </div>
    <div style="display: flex; flex-direction: column; gap: 16px;">
      <div style="display: flex; align-items: center; gap: 12px;">
        <div style="font-size: 28px;">🗄</div>
        <div style="font-weight: 700; font-size: 18px;">Running Server</div>
      </div>
      <div style="display: flex; align-items: center; gap: 12px;">
        <div style="font-size: 28px;">🖥</div>
        <div style="font-weight: 700; font-size: 18px;">Browser Service Worker</div>
      </div>
      <div style="display: flex; align-items: center; gap: 12px;">
        <div style="font-size: 28px;">✅</div>
        <div style="font-weight: 700; font-size: 18px;">Test Suite</div>
      </div>
    </div>
  </div>
</div>

---
layout: default
routeAlias: toc
hideInToc: true
---

# What We'll Cover

<p class="subtitle" style="text-align: center;">A complete tour of MockKit's capabilities</p>

<TocGrid />

---
layout: default
---

# The Gridlock of Modern API Development

<div class="g2" style="margin-top: 10px;">
  <div class="pain-card">
    <h3>Slow Frontend Iteration</h3>
    <p>Hand-crafting static JSON fixtures for every endpoint is tedious, error-prone, and falls out of sync the moment the API contract changes.</p>
  </div>
  <div class="pain-card">
    <h3>Fragile CI/CD Pipelines</h3>
    <p>Tests that depend on live staging environments break unpredictably due to downtime, rate limits, or data drift between runs.</p>
  </div>
  <div class="pain-card">
    <h3>Untestable Real-Time Features</h3>
    <p>SSE, NDJSON, and chunked streaming endpoints are nearly impossible to test locally without complex infrastructure.</p>
  </div>
  <div class="pain-card">
    <h3>Fixture Maintenance Hell</h3>
    <p>A single API contract change requires manually updating dozens of stale JSON files.</p>
  </div>
</div>

---
layout: two-cols
---

# One Spec, Unlimited Mocks

<div class="badge">Prototyping</div>

<div class="card" style="margin-top: 16px;">
  <h3>OpenAPI 3.x</h3>
  <p style="margin-bottom: 12px; font-size: 0.85em;">Full schema support. Re-use existing industry-standard contracts.</p>

```yaml
openapi: 3.0.0
info:
  title: Sample API
  version: 1.0.0
paths:
  /users:
    get:
      responses:
        '200':
          description: OK
```

</div>

::right::

<div style="margin-top: 36px;"></div>

<div class="card">
  <h3>Simple Schema</h3>
  <p style="margin-bottom: 6px; font-size: 0.75em;">A lightweight shorthand for rapid prototyping.</p>

```json
{
  "$schema": "mockkit",
  "name": "My API",
  "basePath": "/api",
  "routes": [{
    "path": "/users",
    "method": "GET",
    "response": {
      "type": "array",
      "items": { "id": "uuid" }
    }
  }]
}
```

</div>

<div class="takeaway" style="margin-top: 16px;"><strong>Takeaway:</strong>&ensp;One spec file replaces hundreds of hand-written JSON fixtures.</div>

---
layout: default
---

# Stateful Persistence Without a Database

<div class="badge">Under the Hood</div>

<p class="subtitle">Every mock instance includes an automatic CRUD store. State persists across requests within the same session.</p>

<div class="pipeline" style="margin-top: 10px;">
  <div class="pipe-node">Request</div>
  <div class="pipe-arrow">➜</div>
  <div class="pipe-node">CORS</div>
  <div class="pipe-arrow">➜</div>
  <div class="pipe-node">Logger</div>
  <div class="pipe-arrow">➜</div>
  <div class="pipe-node">Delay</div>
  <div class="pipe-arrow">➜</div>
  <div class="pipe-node">Chaos</div>
  <div class="pipe-arrow">➜</div>
  <div class="pipe-node">Cache</div>
  <div class="pipe-arrow">➜</div>
  <div class="pipe-node">Route Handler</div>
  <div class="pipe-arrow">➜</div>
  <div class="pipe-node">Response</div>
</div>

<div class="card-teal" style="margin-top: 8px;">
  <div style="display: flex; align-items: center; gap: 8px; margin-bottom: 6px;">
    <span style="font-size: 16px;">🗄</span>
    <span class="sol" style="font-size: 13px;">In-Memory Store</span>
  </div>
  <div style="font-size: 0.7em; color: var(--slate-600); display: grid; grid-template-columns: 1fr 1fr; gap: 3px 16px;">
    <div><strong>POST /users</strong>: Stores with auto-generated ID</div>
    <div><strong>GET /users/:id</strong>: Returns specific item</div>
    <div><strong>PUT / PATCH</strong>: Replaces or merges updates</div>
    <div><strong>DELETE</strong>: Removes item, returns 204</div>
  </div>
</div>

<div class="takeaway"><strong>Takeaway:</strong>&ensp;Full CRUD lifecycle with zero database setup — state persists across requests.</div>

---
layout: two-cols
---

# Intelligent Data Generation

<div class="badge">Local Dev</div>

<p class="subtitle" style="font-size: 0.95em;">MockKit analyzes property names and generates contextually appropriate data.</p>

<div style="margin-top: 10px;">
  <div style="font-weight: 700; margin-bottom: 14px;">Generation Priority</div>
  <div style="display: flex; flex-direction: column; gap: 3px; align-items: center;">
    <div class="funnel-row" style="width: 100%; background: #6b7a96;">Example Value</div>
    <div class="funnel-row" style="width: 88%; background: #7585a0;">Default Value</div>
    <div class="funnel-row" style="width: 76%; background: #8190aa;">Enum Pick</div>
    <div class="funnel-row" style="width: 64%; background: #8d9bb4;">Format-based</div>
    <div class="funnel-row" style="width: 52%; background: #99a6be;">Smart Field Matching</div>
    <div class="funnel-row" style="width: 40%; background: #a5b1c8;">Type Fallback</div>
  </div>
</div>

::right::

<div style="margin-top: 36px;"></div>

<div style="font-weight: 700; margin-bottom: 14px;">Smart Field Detection</div>

| Field Name | Generated Data |
|---|---|
| **userEmail** | john.doe@example.com |
| **avatar** | https://example.com/image.jpg |
| **createdAt** | 2024-01-15T10:30:00.000Z |
| **phone** | +1-555-123-4567 |
| **firstName** | John |

<div class="card-light" style="text-align: center; margin-top: 16px; padding: 10px 16px; font-size: 0.8em;">
  Reproducible Data: Use <strong>--seed 12345</strong> for deterministic responses.
</div>

<div class="takeaway"><strong>Takeaway:</strong>&ensp;Smart defaults mean zero config for realistic data — add a seed for determinism.</div>

---
layout: default
---

# Mocking Real-Time Endpoints Out of the Box

<div class="badge">Local Dev</div>

<p class="subtitle">If your OpenAPI spec declares a streaming content type, MockKit automatically streams. No configuration required.</p>

<div class="g3" style="margin-top: 10px;">
  <div class="box" style="padding: 14px 20px;">SSE (text/event-stream)</div>
  <div class="box" style="padding: 14px 20px;">NDJSON</div>
  <div class="box" style="padding: 14px 20px;">Chunked Transfer</div>
</div>

<div style="margin-top: 10px;">
  <p style="margin-bottom: 12px;">Tune stream timing on the fly without changing the spec:</p>

```bash
curl "api/events?_mockCount=5"          # Fixed count
curl "api/events?_mockInfinite=true"    # Stream forever
curl "api/events?_mockInterval=1000"    # Slow down to 1 msg/sec
```

</div>

<div class="takeaway"><strong>Takeaway:</strong>&ensp;If your spec declares streaming, MockKit streams automatically — no config required.</div>

---
layout: default
---

# Capture Reality and Replay Deterministically

<div class="badge">E2E Testing / QA</div>

<p class="subtitle">Proxy requests to a real API and capture every response for later replay.</p>

<RecordReplayDiagram />

<div class="g2" style="margin-top: 10px;">
  <div class="card-light">
    <h3 style="font-size: 1em;">Matching Strategies</h3>
    <p style="font-size: 0.8em;">Control strictness with <strong>path-query</strong>, <strong>path-only</strong>, <strong>exact</strong>, or <strong>fuzzy</strong> (ignores dynamic POST bodies).</p>
  </div>
  <div class="card-light">
    <h3 style="font-size: 1em;">HAR Import</h3>
    <p style="font-size: 0.8em; margin-bottom: 8px;">Instantly import recordings from Chrome DevTools or Playwright.</p>
    <code style="font-size: 0.75em; background: var(--card); padding: 6px 12px; border-radius: 6px;">mockkit import --har recording.har</code>
  </div>
</div>

<div class="takeaway"><strong>Takeaway:</strong>&ensp;Record once against the real API, replay forever — complete staging independence.</div>

---
layout: default
---

# Version Control for Your Mock Data

<div class="badge">Debugging</div>

<p class="subtitle">Snapshots are immutable, named sets of recordings. Reproduce bugs by hot-swapping to the exact API state when the error occurred.</p>

<SnapshotTimeline />

<div class="takeaway"><strong>Takeaway:</strong>&ensp;Hot-swap API state instantly — reproduce any bug with a single CLI command.</div>

---
layout: default
---

# Testing the Edges of Resilience and Performance

<div class="badge">Resilience</div>

<div class="g2" style="margin-top: 10px;">
  <div class="card" style="text-align: center;">
    <div style="font-size: 48px; margin-bottom: 12px;">⚠️</div>
    <h3 style="font-size: 1.2em;">Chaos Engineering</h3>
    <p style="margin: 12px 0 18px; font-size: 0.85em;">Inject 400/500 errors, add 200-2000ms latencies, or simulate timeouts. Target specific routes with path-based rules.</p>

```bash
mockkit start --chaos \
  --chaos-error-rate 0.2 \
  --chaos-latency 500
```

  </div>
  <div class="card" style="text-align: center;">
    <div style="font-size: 48px; margin-bottom: 12px;">⚡</div>
    <h3 style="font-size: 1.2em;">Response Caching</h3>
    <p style="margin: 12px 0 18px; font-size: 0.85em;">Built-in LRU cache, MD5-based ETags, and 304 Not Modified support. Test caching behavior locally.</p>

```bash
mockkit start --cache \
  --cache-ttl 60 \
  --cache-max 500
```

  </div>
</div>

<div class="takeaway"><strong>Takeaway:</strong>&ensp;Test failure modes before they hit production — chaos and caching with zero infrastructure.</div>

---
layout: default
---

# Serverless Mocking for Frontend Development

<div class="badge">Frontend Dev</div>

<p class="subtitle">Generate MSW handlers directly from your API spec. No server, no ports — work completely offline.</p>

<div class="card" style="margin-top: 8px; padding: 14px;">
  <div style="border: 2px dashed var(--slate-300); border-radius: 8px; padding: 12px; position: relative;">
    <div style="position: absolute; top: -10px; left: 14px; background: white; padding: 0 6px; font-size: 10px; color: var(--slate-500);">Browser Environment</div>
    <div style="display: flex; align-items: center; gap: 14px; justify-content: center;">
      <div class="card-light" style="padding: 8px 16px; text-align: center; font-weight: 700; font-size: 12px;">App</div>
      <div style="text-align: center;">
        <div style="font-size: 10px; color: var(--slate-500);">fetch('/api/users')</div>
        <div style="font-size: 18px; color: var(--slate-400);">➜</div>
      </div>
      <div class="card-light" style="padding: 8px 16px; text-align: center; font-weight: 700; font-size: 12px; border: 2px solid var(--slate-400);">MSW Worker</div>
      <div style="display: flex; align-items: center; gap: 4px;">
        <div style="font-size: 18px; color: var(--red);">✗</div>
        <div style="font-size: 9px; color: var(--red);">No network<br>request</div>
      </div>
    </div>
    <div style="text-align: center; margin-top: 6px; font-size: 10px; color: var(--slate-500);">⬅ Mock Response returned to App</div>
  </div>
</div>

<div style="display: flex; align-items: center; gap: 12px; margin-top: 8px; flex-wrap: wrap;">
  <code style="background: var(--card); padding: 4px 10px; border-radius: 6px; font-size: 0.7em;">mockkit browser generate -s ./api.yaml</code>
  <span style="font-size: 0.7em; color: var(--slate-600);"><strong>Storage:</strong> memory, sessionStorage, localStorage</span>
</div>

<div class="takeaway"><strong>Takeaway:</strong>&ensp;Full offline development — no server, no ports, just your browser and a spec.</div>

---
layout: two-cols
---

# In-Browser Power Tools

<div class="badge">QA / Debugging</div>

<p class="subtitle" style="font-size: 0.9em;">A bidirectional bridge between the browser and MockKit.</p>

<div style="margin-top: 16px; display: flex; flex-direction: column; gap: 16px;">
  <div>
    <div style="display: flex; align-items: center; gap: 8px; margin-bottom: 4px;">
      <div style="width: 10px; height: 10px; border-radius: 50%; background: var(--teal);"></div>
      <strong>Smart Response Diffing</strong>
    </div>
    <p style="font-size: 0.8em;">Deep JSON comparison to catch API regressions instantly.</p>
  </div>
  <div>
    <div style="display: flex; align-items: center; gap: 8px; margin-bottom: 4px;">
      <div style="width: 10px; height: 10px; border-radius: 50%; background: #6366f1;"></div>
      <strong>GraphQL Intelligence</strong>
    </div>
    <p style="font-size: 0.8em;">Auto-detects, groups, and tracks variables for queries and mutations.</p>
  </div>
  <div>
    <div style="display: flex; align-items: center; gap: 8px; margin-bottom: 4px;">
      <div style="width: 10px; height: 10px; border-radius: 50%; background: #d97706;"></div>
      <strong>Automatic Failover</strong>
    </div>
    <p style="font-size: 0.8em;">Automatically switches to mocks when staging APIs go down.</p>
  </div>
</div>

::right::

<div style="margin-top: 36px;"></div>

<DevToolsMockup />

<div class="takeaway"><strong>Takeaway:</strong>&ensp;Debug API issues live in the browser — smart diffing catches regressions instantly.</div>

---
layout: default
tocColor: '#d97706'
---

# From Live Traffic to Mocked Responses in 4 Steps

<div class="badge">QA / Debugging</div>

<p class="subtitle">Browse any website, record its API calls, and replay them locally — no backend required.</p>

<InterceptionDiagram />

<div style="display: flex; gap: 8px; margin-top: 6px;">
  <div class="card-light" style="text-align: center; padding: 6px 10px; flex: 1;">
    <div style="font-weight: 700; font-size: 0.7em;">Failover Mode</div>
    <p style="font-size: 0.6em;">Auto-switches when API is down</p>
  </div>
  <div class="card-light" style="text-align: center; padding: 6px 10px; flex: 1;">
    <div style="font-weight: 700; font-size: 0.7em;">Offline Support</div>
    <p style="font-size: 0.6em;">Fetch override, zero network</p>
  </div>
  <div class="card-light" style="text-align: center; padding: 6px 10px; flex: 1;">
    <div style="font-weight: 700; font-size: 0.7em;">Host-Level Control</div>
    <p style="font-size: 0.6em;">Intercept specific hosts only</p>
  </div>
</div>

<div class="takeaway"><strong>Takeaway:</strong>&ensp;Record real traffic from any website, replay locally — complete backend independence.</div>

---
layout: default
---

# Spec-Driven Fixtures for Flake-Free E2E Testing

<div class="badge">CI/CD</div>

<p class="subtitle">Eliminate manual page.route() boilerplate. Record once, replay in CI forever.</p>

<div style="margin-top: 8px; background: white; border: 2px solid var(--teal); border-radius: 10px; overflow: hidden; position: relative;">
  <div style="display: flex; align-items: center; gap: 8px; padding: 8px 14px; border-bottom: 1px solid var(--card-border); font-size: 0.75em; color: var(--slate-600);">
    playwright.config.ts
  </div>
  <div style="padding: 10px 16px;">

```typescript
// Intercept API calls and redirect to mock server
import { setupMockServer } from '@up/mockkit/playwright';

await setupMockServer(page, {
  interceptHosts: ['api.example.com'],
  mode: 'replay',
});
```

  </div>
  <div style="position: absolute; top: 6px; right: 12px;">
    <div style="background: var(--green); color: white; padding: 4px 12px; border-radius: 12px; font-size: 0.7em; font-weight: 700;">✔ Pass</div>
  </div>
</div>

<div class="takeaway"><strong>Takeaway:</strong>&ensp;Spec-driven fixtures eliminate test flakiness — record in dev, replay in CI forever.</div>

---
layout: two-cols
tocGroup: extra
---

# Visual API Exploration

<div class="badge">Collaboration</div>

<p class="subtitle" style="font-size: 0.9em;">An interactive browser interface for testing generated MSW handlers.</p>

<div style="margin-top: 16px; display: flex; flex-direction: column; gap: 20px;">
  <div>
    <div style="display: flex; align-items: center; gap: 8px; margin-bottom: 4px;">
      <div style="width: 10px; height: 10px; border-radius: 50%; background: var(--slate-500);"></div>
      <strong>Browse all endpoints</strong>
    </div>
    <p style="font-size: 0.8em;">with method badges and streaming indicators.</p>
  </div>
  <div>
    <div style="display: flex; align-items: center; gap: 8px; margin-bottom: 4px;">
      <div style="width: 10px; height: 10px; border-radius: 50%; background: var(--slate-500);"></div>
      <strong>Watch SSE/NDJSON</strong>
    </div>
    <p style="font-size: 0.8em;">events arrive in real-time.</p>
  </div>
  <div>
    <div style="display: flex; align-items: center; gap: 8px; margin-bottom: 4px;">
      <div style="width: 10px; height: 10px; border-radius: 50%; background: var(--slate-500);"></div>
      <strong>Set Faker.js seeds</strong>
    </div>
    <p style="font-size: 0.8em;">for reproducible responses on the fly.</p>
  </div>
</div>

::right::

<div style="margin-top: 36px;"></div>

<PreviewMockup />

---
layout: default
tocGroup: extra
tocColor: 'var(--teal)'
---

# The 10x Workflow Transformation

<div class="compare-table" style="margin-top: 10px;">

| The Old Way | The MockKit Way |
|---|---|
| Cross-Team Blocking | <span class="sol">Parallel Development</span> — Frontend and Backend work simultaneously |
| Flaky Staging Environments | <span class="sol">Deterministic CI Replay</span> — Tests run in milliseconds |
| Hard-Coded Prototypes | <span class="sol">5-Minute Simple Schemas</span> — Fully functional APIs with CRUD |
| Silent API Breakages | <span class="sol">DevTools Smart Diffing</span> — Catch regressions immediately |
| No Streaming Mocks | <span class="sol">3 Protocols Out of the Box</span> — SSE, NDJSON, Chunked |
| Manual Fixture Updates | <span class="sol">Spec-Driven Generation</span> — One spec, unlimited mocks |

</div>

---
layout: default
tocGroup: extra
tocColor: 'var(--teal)'
---

# One Infrastructure for the Entire Lifecycle

<p class="subtitle">From 30-line prototypes to strict OpenAPI contracts, MockKit replaces fragmented tooling with a unified mocking engine.</p>

<div class="g4" style="margin-top: 10px;">
  <div class="env-card" style="padding: 12px;">
    <div style="font-weight: 700; font-size: 0.9em;">Local Dev</div>
    <div class="env-sub">MSW/CLI</div>
  </div>
  <div class="env-card" style="padding: 12px;">
    <div style="font-weight: 700; font-size: 0.9em;">E2E Tests</div>
    <div class="env-sub">Playwright</div>
  </div>
  <div class="env-card" style="padding: 12px;">
    <div style="font-weight: 700; font-size: 0.9em;">Debugging</div>
    <div class="env-sub">Snapshots</div>
  </div>
  <div class="env-card" style="padding: 12px;">
    <div style="font-weight: 700; font-size: 0.9em;">QA</div>
    <div class="env-sub">Chrome Extension</div>
  </div>
</div>

<div style="text-align: center; margin-top: 12px;">
  <p style="font-weight: 700; font-size: 1em; margin-bottom: 10px;">Start building instantly:</p>
  <div class="terminal">
    <div class="terminal-bar">
      <div class="terminal-dot" style="background: #ff5f57;"></div>
      <div class="terminal-dot" style="background: #ffbd2e;"></div>
      <div class="terminal-dot" style="background: #28c840;"></div>
    </div>
    <div class="terminal-body">
      <span style="color: #94a3b8;">$</span> mockkit start -s api.yaml<span class="cursor">&nbsp;</span>
    </div>
  </div>
</div>
