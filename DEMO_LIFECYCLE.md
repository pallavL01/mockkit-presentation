# Demo Script — The Full Lifecycle

**Goal:** A 5–7 minute live demo that proves the thesis of `pitch-deck-LIFECYCLE.html` — *one artifact, whole lifecycle*. The audience watches the same recording file travel from spec (Day 0) → working component → committable test (Pre-merge) → hardened suite (Forever), driven by natural-language prompts that auto-activate AgentSkills.

**Audience:** Engineering leaders, platform teams, AI-first devs already using Claude Code / Cursor / Copilot / Gemini CLI. They've seen "AI writes code" demos. The novelty here is **artifact continuity** — the same JSON file shows up in three lifecycle phases without a single human edit.

**Why this lands:** Most demos show one phase (record, or replay, or test gen). This one shows the *seam* between phases dissolved. The audience sees `recordings.json` referenced in Act 1 (mock data source), Act 2 (test fixture), and Act 3 (audit input) — and it's the same file, byte for byte. That's the wow moment.

**Position in the deck:** Run after **Slide 12 (Composition)**, before the CTA. The deck builds the case; the demo is the proof.

---

## Pre-demo checklist (do all of this 15 minutes before going live)

### Repos and processes

| Item | Verify |
|---|---|
| `mockkit/main` checked out at commit ≥ `78ade54` (skills + recording docs landed) | `git log -1 --oneline` |
| `mockkit-enterprise-demo` cloned, on a branch where the audit findings are still present | Pre-cut a `demo/lifecycle` branch where `tests/resilience.spec.ts` still has the vacuous assertions |
| `.agents/skills/` symlinked or copied into the demo repo's `.claude/skills/` (Claude Code path) | `ls .claude/skills/atomic/spec-mock/SKILL.md` |
| `@up/mockkit-mcp` and Playwright MCP both configured in `.mcp.json` | `cat .mcp.json` should show both servers |
| AI client restarted recently enough to discover the skills | Test cold: ask the AI *"what skills do you have for MockKit?"* — should list spec-mock, record-flow-as-test, audit-tests, etc. |
| `api.yaml` spec file present and valid | `mockkit validate -s api.yaml` |
| Component scaffold target known and committed-to (e.g., `IncidentList.tsx`) | Pick something with a single API call, ~30 lines |
| Recording file used in all three acts: pre-recorded, committed, named obviously | `fixtures/lifecycle-demo.json` — refer to this file by name on every act |

### Setup tactics

- **Pre-warm everything.** Dev server up, MockKit pre-installed, AI session open in the demo repo. Cold starts kill momentum.
- **Two screens or split.** Left: the prompt + AI reasoning. Right: the running app + the recording file in a code editor (so the audience can see the JSON when you reference it).
- **Big monospace font** (18–22pt) in the terminal. Dark IDE. Audience needs to read line 30 of a SKILL.md if you reveal it.
- **Have the recording file open in a tab the whole time.** When you say *"this same file"*, you tab to it and they see the bytes.
- **Pre-record a screen capture** of the exact same demo. If MCP flakes, you cut to the recording mid-sentence and keep going. Don't apologize. Don't restart.
- **Pin the file modification times.** `stat fixtures/lifecycle-demo.json` showing the same mtime across acts is a tiny but real proof.

### What NOT to demo

- **Skip Week 1 (recording capture)** — narrate it with one screenshot. Live network captures are slow and the audience already gets it.
- **Skip Day-to-Day (offline mode)** in the live demo — mention it as the bridge. Browser extension demos don't sit well in a deck flow.
- **Don't show the chaos endpoint** — it's the slide-10 callout, not part of the lifecycle arc. Keep the demo on the throughline.

---

## The 5–7 minute arc

### Beat 1 — The setup (0:00 – 0:30)

Don't open AI yet. Open `fixtures/lifecycle-demo.json` in a giant editor font. Scroll to one entry. Read it out loud:

```json
{
  "request": { "method": "GET", "path": "/api/incidents" },
  "response": { "status": 200, "body": { "incidents": [...] } }
}
```

Then say:

> *"Watch this file. We're going to use it three times. It will not change. Same bytes, three lifecycle phases."*

Now switch to the AI session.

---

### Beat 2 — Day 0 (0:30 – 2:00) — *spec → working component*

Type this single sentence into the AI:

```
Stand up a mock from ./api.yaml using ./fixtures/lifecycle-demo.json as the recording, and scaffold the IncidentList component.
```

What you should see in real time:

1. AI activates `/spec-mock` (validates the spec, starts MockKit).
2. AI activates `/setup-msw` (or detects it's already wired).
3. AI scaffolds `IncidentList.tsx` — types from the spec, fetch from the running mock.
4. Browser shows the component rendering real data **from the recording you just opened**.

Tab to the recording file. Tab to the browser. They are the same data.

> *"That's Day 0. No backend. The component is rendering data from this exact JSON file."*

**If MCP flakes here**: cut to the screen recording. Same prompt, same outcome. Keep narrating.

---

### Beat 3 — Pre-merge (2:00 – 4:00) — *flow → committable test*

Without resetting anything, type:

```
Record a Playwright test for: load the dashboard, click the first incident, verify the title shows. Output to tests/incident-detail.spec.ts.
```

What you should see:

1. AI activates `/record-flow-as-test`.
2. It composes two MCP servers — MockKit for the deterministic backend, Playwright for the browser.
3. It drives the app, captures aria snapshots after each step.
4. It writes the spec file (~18 lines), self-cleaning, no test pollution.
5. It runs the test once: green in ~1.5s.

Open the generated spec file. Show the `afterEach` cleanup. Show the `getByRole` selectors. Then —

Tab back to `fixtures/lifecycle-demo.json`. **The test runs against the same file.** Say:

> *"Same JSON. Different lens. Day 0 it was a mock data source. Pre-merge it's a test fixture. I never touched the file."*

This is the wow moment. Land it. Pause. Move on.

**If the AI describes the wrong flow**: it's because Playwright MCP missed an aria element. Re-prompt with one more sentence of clarification. Do NOT debug live.

---

### Beat 4 — Forever (4:00 – 5:30) — *quality stays*

Type:

```
Audit tests/ with applyFixes: true, then run the suite.
```

What you should see:

1. AI activates `/audit-tests`.
2. Walks 9 fragility patterns.
3. Reports findings (HIGH/MEDIUM/LOW) — show the report on screen.
4. Auto-applies HIGH+MEDIUM fixes.
5. Re-runs the suite. Green. Faster than before.

The killer line:

> *"Suite went from 13 seconds flaky to 1.9 seconds green. We didn't delete tests — we removed test theater."*

(Use real numbers from your repo. Don't fudge.)

Tab to `fixtures/lifecycle-demo.json` one more time:

> *"Same JSON. Audited the tests that consume it. The artifact carried the contract through three phases."*

---

### Beat 5 — The close (5:30 – 6:30)

Drop back to the deck. Slide 13 (CTA).

Three commands. One prompt. Done.

If you have time for one more sentence:

> *"Most tools cover one phase. The win isn't any single skill — it's that the same file flows through all five. That's what 'one artifact, whole lifecycle' actually means."*

Then take questions.

---

## Failure modes and recoveries

| What goes wrong | Recovery |
|---|---|
| AI doesn't auto-activate the skill (description didn't match) | Retype with more keywords from the skill's description. If still nothing, prefix: *"Use the spec-mock skill to..."* |
| MockKit MCP not discovered | `cat .mcp.json` in front of the audience, restart the AI client. Costs 30s. |
| Playwright MCP fails mid-recording | Cut to the screen-recording fallback. Keep narrating. |
| Generated test fails on first run | Don't debug. Say *"the prompt has a re-run loop built in"* and re-prompt with the same flow. |
| Audit finds zero issues | You demoed against a clean branch by mistake. Switch to `demo/lifecycle` branch. (Why we cut a dirty branch in the checklist.) |
| Suite is slow (>5s) on the audit re-run | Don't apologize. Say *"this is real numbers on a real repo, your mileage may vary"* and move on. |

---

## Optional extensions (if the audience is hot)

These are bonus rounds — only if Q&A is going well and you have 2+ extra minutes.

| Extension | Time | Worth showing if... |
|---|---|---|
| Capture phase live (`mockkit record` proxy) | +90s | Audience asks "how do you make the recording?" |
| Offline mode in Chrome extension | +60s | Audience asks "what about plane mode / no VPN?" |
| Chaos injection on the running mock | +60s | Audience asks "how do you test error paths?" |
| `/spec-to-prod` workflow (the whole arc in one prompt) | +5min | Audience pushes back on "this only works in pieces" |

The `/spec-to-prod` extension is the *real* mic-drop, but it's a 5-minute commit. Save it for an executive review or longer slot.

---

## What this demo proves (post-mortem talking points)

If anyone asks *"why is this different from \<other tool\>?"*, the answer is:

1. **Artifact continuity** — same JSON across three phases. No translation, no rebuild, no drift.
2. **Open standard skill format** — the same SKILL.md files work in Claude Code, Cursor, Copilot, Gemini CLI, 30+ clients. No vendor lock.
3. **Composition** — workflow skills chain atomic skills. Devs describe intent; the model picks the chain.
4. **Auditability** — the recording file is in git. Every phase reads it. Every phase's output is a normal file (component, test, report) that you can review.

If they want to go deeper, hand them `RECORDING_WITH_CURL.md` and `RECORDING_WITH_PROXY.md` from the `mockkit/main/docs/` folder.
