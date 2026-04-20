# Demo Script — Spec to Ship in 6 Minutes

**Goal:** A live 6-minute demo proving that an OpenAPI spec is the *only* artifact a human writes — AI ships the working frontend, the deterministic mock backend, and the regression test from that single source of truth.

**Audience:** CTOs, VPs of Engineering, product leadership, anyone evaluating velocity gains from AI-native development. Less effective for staff engineers — for that crowd, use [`DEMO_AI_NATIVE_LOOP.md`](./DEMO_AI_NATIVE_LOOP.md) (the audit demo) instead.

**Why this lands:** Most "AI codes" demos show generation in isolation — a function, a component, a test. This demo shows the *convergence*: spec → mock → UI → test, all driven by AI from one source of truth, with the chain visibly closing on screen. No other tool composes like this; demonstrating MockKit's full thesis end-to-end.

> ## ⚠️ Verification status
>
> Unlike `DEMO_AI_NATIVE_LOOP.md` (which was verified end-to-end during its authoring session), this demo's full chain has **not** been run end-to-end yet. Component verifications:
>
> | Component | Status |
> |---|---|
> | `mockkit_browser_generate` MCP tool | ✅ Verified — exists, works, generates MSW handlers from spec |
> | `record-flow-as-test` prompt | ✅ Verified — Scenario 19 in `mockkit_demo` repo, ships passing self-cleaning tests |
> | `setup-playwright` prompt | ✅ Verified — used in `setup-playwright` integration in enterprise-demo |
> | AI scaffolding Vite + React + TS app from a single prompt | ⚠️ Unverified — depends on the AI client's general code-gen + filesystem ability, not a MockKit-specific tool |
> | AI writing a React component that consumes the API and renders correctly | ⚠️ Unverified — same dependency on general code-gen |
> | The full chain executing cleanly in 6 minutes without manual intervention | ⚠️ Unverified — proposed, not battle-tested |
>
> **Run this end-to-end at least once before going live.** Update this doc with verified outcomes (and mark this section as ✅ verified) after a successful dry run. The tactics below are the recommended structure; treat the timings and exact AI behavior as estimates until proven.

---

## Pre-demo checklist

### Repos and processes

| Item | Verify |
|---|---|
| `@up/mockkit-mcp` rebuilt with the latest prompts (record-flow-as-test, setup-playwright, browser_generate, etc.) | Commit ≥ `8e89d51` in mockkit |
| Playwright MCP configured in `.mcp.json` (`@playwright/mcp@latest`) | `cat .mcp.json` |
| AI client restarted recently enough to spawn the freshest MCP servers | Test: "List all mockkit MCP prompts." Should return 8 including `setup-playwright`, `record-flow-as-test`. |
| OpenAPI spec ready and **simple** (one resource, GET + POST is enough) | A 30-line `api.yaml` is better than a 300-line one for demo purposes |
| **Decision:** which scaffolding tier? (see below) | Pick before going live, not during |

### Scaffolding tier — pick one before the demo

The riskiest step is "AI scaffolds a Vite + React + TS app from prompts." Three tiers from most ambitious to safest. Pick based on your appetite for live failure.

| Tier | Pre-baked | AI does live | Risk | When to use |
|---|---|---|---|---|
| **Tier 1 — full scratch** | Empty directory only | Scaffold + handlers + feature + test | High (multiple compile errors possible) | Conference keynote where the audience cares about authenticity over polish |
| **Tier 2 — pre-baked scaffold** | Vite + React + TS shell already running, MockKit MCP wired up | MSW handlers + feature component + test | Medium | Investor pitch, exec demo — most common choice |
| **Tier 3 — pre-baked scaffold + handlers** | Tier 2 + MSW handlers already generated from a previous run | Feature component + test only | Low | Fallback if Tier 2 has been flaky in dry runs |

**Recommendation:** Tier 2 unless you've dry-run Tier 1 successfully ≥3 times. Tier 2 keeps the dramatic "AI generates handlers from spec" beat (the visually impressive one) while removing the most fragile part (initial scaffolding).

The arc below assumes **Tier 2**. If you pick Tier 1, add 60–90 seconds for scaffolding; if Tier 3, skip Beat 2.

### Setup tactics

- **Pre-load everything.** Vite dev server running, AI session open, spec file open in a side panel. Every "loading…" second kills energy.
- **Big editor + terminal font.** 18–22pt monospace. Audience reads row 30 from the back of the room.
- **Have a screen-recording backup.** This demo has more failure surface than the audit demo. If anything craps out, segue to recording smoothly.
- **Pre-test in the exact environment.** Run the full arc once, in the same terminal, same AI session, same screen size you'll use live. Don't trust "it worked yesterday on my laptop."

---

## The 6-minute arc (Tier 2)

### Beat 1 — The constraint (0:00 – 1:00)

Open the spec file in a side panel. Read the relevant snippet out loud:

```yaml
paths:
  /api/projects:
    get:
      responses:
        '200':
          content:
            application/json:
              schema: { type: array, items: { $ref: '#/components/schemas/Project' } }
    post:
      requestBody: ...
      responses:
        '201': ...
components:
  schemas:
    Project:
      type: object
      required: [id, title]
      properties:
        id: { type: string, format: uuid }
        title: { type: string }
        description: { type: string }
        createdAt: { type: string, format: date-time }
```

Then deliver the framing line:

> *"This is the only thing a human is going to write today. Watch the rest happen."*

Show the audience the empty state — Vite running with a placeholder page on `localhost:5173`. **No `ProjectsPage` exists yet.** No mock handlers. No tests. *"None of this exists. We have a spec. Nothing else."*

---

### Beat 2 — AI generates the mock backend (1:00 – 2:00)

Switch to the AI assistant. Type one prompt:

```
Generate MSW handlers from api.yaml using mockkit_browser_generate.
Write the output to src/mocks/handlers.ts and wire up the worker
in src/main.tsx so it starts before React renders.
```

Audience watches `src/mocks/handlers.ts` materialize. The AI also edits `src/main.tsx` to start MSW.

Open `src/mocks/handlers.ts` briefly so audience sees real handlers — `http.get('/api/projects', ...)` returning typed faker-generated data.

Reload the browser. The Vite dev page now shows a network tab full of `[MSW] GET /api/projects → 200` log lines.

> *"That's the spec, in code, as a working mock. 60 seconds. No backend running anywhere on this laptop."*

---

### Beat 3 — AI builds the feature (2:00 – 3:30)

Type the next prompt:

```
Add a ProjectsPage component at src/pages/ProjectsPage.tsx that:
- fetches GET /api/projects on mount
- renders the list with title and createdAt
- has a CreateProjectForm with title and description fields
- POSTs to /api/projects on submit, then refetches the list

Use TanStack Query for the data layer if you can; useState if not.
Mount the component at the root of App.tsx.
```

Watch files appear:
- `src/pages/ProjectsPage.tsx`
- `src/components/CreateProjectForm.tsx` (if AI splits it out)
- `src/App.tsx` updated to mount the page

Reload the browser. **The list renders with real-looking data.** Type into the form, click Create. **A new row appears.**

Show the network tab — real `POST /api/projects` followed by `GET /api/projects`. *"That's a working CRUD feature. Form to API to UI refresh. From the spec, in 90 seconds."*

---

### Beat 4 — AI ships the regression test (3:30 – 5:00)

Type the next prompt:

```
Use the mockkit record-flow-as-test prompt with:
  spec:       api.yaml
  appUrl:     http://localhost:5173
  outputPath: tests/projects-create.spec.ts
  seed:       42
  flow: |
    1. Open the projects page
    2. Click the button that creates a new project
    3. Fill the title field with "Demo project"
    4. Fill the description field with "Created by autonomous test"
    5. Submit the form
    6. Verify the new project appears in the list with the correct title
```

Watch the browser fly around — Playwright MCP visibly drives the page. Form fills, submit clicks, new row appears.

Then the AI emits `tests/projects-create.spec.ts`. Open it briefly. Audience sees:
- Semantic locators (`getByRole`, `getByLabel`)
- Unique title with `Date.now()`
- `test.afterEach` cleanup that survives mid-test failure

Run it:

```bash
npx playwright test tests/projects-create.spec.ts --reporter=list
```

Passes in ~1 second.

> *"And the regression test that proves it works — generated from the same spec, deterministic forever, runs in CI without a backend."*

---

### Beat 5 — Synthesis (5:00 – 6:00)

Walk the audience back through what just happened:

> *"Six minutes ago we had a YAML file. Now we have:*
>
> *— A working React frontend that lists and creates projects.*
> *— A deterministic mock backend serving the same spec.*
> *— A Playwright regression test that runs in CI without any backend, deterministic forever.*
>
> *All generated from one source of truth. All driven by AI. Three prompts."*

Long pause.

> *"That's not 'AI assists with code.' That's 'spec is the only artifact a human writes. AI ships the rest.'"*

End slide:

> # Six minutes. Three prompts. One spec.
>
> Mock backend • Frontend feature • Regression test
> Generated, not assisted

---

## Demo tactics that make or break it

| Tactic | Why |
|---|---|
| **Live, not recorded** | Same as audit demo — recording reads as marketing. Must be live. |
| **Silence while AI thinks** | Don't narrate during the AI's work. The output speaks. |
| **One spec snippet on screen at all times** | Audience needs the visual anchor — *this* is what everything traces back to. Pin it in a side panel. |
| **Show the network tab in Beat 3** | The audience needs to see real HTTP calls happening to trust the mock is real. |
| **Big editor font, dark theme** | 18–22pt monospace. Read row 30 from row 1. |
| **The "three prompts" line** | The most quotable line in the whole arc. Write it on the closing slide. |
| **Don't add styling/polish during the demo** | The AI may emit ugly default UI. Resist the urge to ask it to "make it pretty" — that wastes 60 seconds and adds nothing to the thesis. |
| **Recording backup** | More failure surface than audit demo. Have one ready. |

---

## Optional flourishes

### Add chaos at the end (extends to 7 minutes)

After Beat 4, before synthesis:

```
Use mockkit_chaos_configure to enable 50% errors on POST /api/projects.
Then re-run the test from Beat 4 and report what happens.
```

The test fails — POST returns 500 half the time. *"Now I'll prove the test catches a real regression."* Disable chaos, re-run, passes.

This adds the "AI verifies its own work catches regressions" beat — but extends the demo by 60+ seconds and adds another failure surface.

### The "flip the spec" reveal (riskier)

After Beat 5, edit the spec live to add a new field (e.g. `priority: { type: string, enum: [low, medium, high] }`). Then ask AI to update the handlers, the form, and the test. Audience watches all three update from the spec change.

This is the **strongest** version of the thesis ("spec is the contract; everything follows") — but it's an entirely separate run that adds 2–3 minutes and introduces compile errors as failure modes. Only attempt with a Tier 3 fallback ready.

---

## Fallback plan if something breaks live

| Failure | Recovery |
|---|---|
| AI returns wrong/empty response in Beat 2 | Fall back to Tier 3 — show the pre-baked handlers as if AI just generated them. *"Here's what it produces normally."* |
| Generated React component doesn't compile | Skip running it; just show the file structure. *"In the normal flow this renders. The interesting bit is that it's traced from the spec — let me show you the spec-to-code lineage."* |
| Browser doesn't open in Beat 4 | Skip the visual flair; show the emitted .spec.ts as the artifact. *"The test was generated — here it is. In a normal run you'd see the browser drive the flow."* |
| Test fails in Beat 4 | This is fatal. Either: (a) restart from a known-good branch, or (b) acknowledge and pivot to the audit demo. Don't try to debug live. |
| MCP server returns errors | Switch to screen recording. Don't restart MCP live — takes too long. |
| Vite server crashes | Restart it (5–10 seconds). Apologize once, pick up from where you were. |

---

## Pairing this demo with deck E

Slide 6 (Autonomous Loops) is the backdrop. The actual deck slide that maps to this demo is **slide 5 (The Solution)** — *"One spec. One AI. Zero guessing."* That's literally what you're demonstrating.

After Beat 5, flip to slide 10 (proof bullets) and let it sit. The audience just watched every claim on that slide happen live.

---

## Demo selection guide

| Audience | Use |
|---|---|
| Engineering team / staff engineers | `DEMO_AI_NATIVE_LOOP.md` (audit) — they recognize their own pain |
| Tech leadership (CTO/VP) / product | `DEMO_SPEC_TO_SHIP.md` (this) — velocity story |
| Mixed audience, ~10 minutes total | This demo for the first 6 min, then 4 min of `DEMO_AI_NATIVE_LOOP.md` Beat 2 only (the audit reveal) |
| AI-platform / DevTools focused | This demo, plus the "flip the spec" flourish |
| Conference keynote (high stakes, must work) | `DEMO_AI_NATIVE_LOOP.md` — verified, lower failure surface |

---

## Verified facts you can cite

- **`mockkit_browser_generate`** — generates MSW handlers from OpenAPI/Simple Schema specs; verified working in `mockkit_demo` Scenario 10
- **`record-flow-as-test`** — verified end-to-end in `mockkit_demo` Scenario 19; produces self-cleaning tests
- **`audit-test-quality`** — verified end-to-end in `mockkit_demo` Scenario 20; surfaced 27 real findings including a HIGH-severity bug humans missed
- **8 built-in MCP prompts** in mockkit-mcp
- **20 enterprise scenarios** in `mockkit_demo/scenarios/mcp-prompts.md`

**What you cannot yet cite as verified for this specific demo:**
- The exact 6-minute timing (proposed; needs dry-run)
- The exact AI scaffolding behavior (depends on AI client and current state)
- That the chain runs cleanly without manual intervention (proposed; needs dry-run)

If asked *"has anyone actually run this end-to-end?"*, answer honestly: *"The components are all proven; the full chain is what we're showing live now. If anything breaks, it's because the demo is genuinely live, not staged."* That framing turns potential failure into authenticity.

---

## After the first successful live run

Update the **Verification status** section at the top of this doc:
1. Mark "the full chain executing cleanly in 6 minutes" as ✅ Verified
2. Add a date and the actual elapsed time
3. Add any caveats or fixes the live run surfaced
4. Update the "Verified facts" section to include the live-run outcome

Same pattern that produced `DEMO_AI_NATIVE_LOOP.md`'s honest verification trail. The doc gets more trustworthy each time it's executed.
