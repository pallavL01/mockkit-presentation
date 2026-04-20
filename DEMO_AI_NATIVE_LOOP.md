# Demo Script — The AI-Native Testing Loop

**Goal:** A live 4-minute demo that lands the wow moment for `pitch-deck-E.html` (the AI-native variant). Proves the closed-loop thesis — *AI generates disciplined tests forward, AI audits legacy tests backward, the loop closes on quality not just functionality* — by showing it run live, with verified prompts and verified outcomes.

**Audience:** Engineering leaders, AI-platform teams, anyone who's seen "AI writes tests" demos before and is rightfully skeptical.

**Why this lands:** Most "AI writes a test" demos show generation only. This one adds the part nobody's seen — *AI tells you which of your existing tests are lying*. The audit half is the dramatic core; generation is the supporting act.

---

## Pre-demo checklist (do all of this 10 minutes before going live)

### Repos and processes

| Item | Verify |
|---|---|
| `mockkit-enterprise-demo` cloned at the latest main (commit ≥ `34c340b`) | `git log -1` |
| `@up/mockkit-mcp` rebuilt with the latest prompts (record-flow-as-test + audit-test-quality + Pattern H/I + networkidle warning) | Commit ≥ `8e89d51` in mockkit |
| Dashboard running on `localhost:4000` | `pnpm dev` from enterprise-demo |
| MockKit running on `localhost:3000` with `--seed 42` | `pnpm exec mockkit start -s api-spec.yaml -p 3000 --seed 42` |
| AI client (Claude Code / Gemini CLI) restarted recently enough to spawn the freshest MCP server | Test with: "List the mockkit MCP prompts." Should return 8 including `audit-test-quality` and `record-flow-as-test`. |
| `tests/` directory contains 7 spec files in their *pre-fix* state | Cherry-pick a "demo" branch where the audit findings are still present. **Don't demo on already-fixed code.** |

### Setup tactics

- **Pre-load everything.** Dashboard, MockKit, AI session open. Every "loading…" second kills energy.
- **Big terminal font.** 18–22pt monospace, dark theme, no IDE chrome. Audience needs to read row 30.
- **Have a screen-recording backup.** MCP can flake live. If anything craps out, segue *"let me show you what just happened earlier"* and play the recording. Don't apologize, don't restart.
- **Pin the commit history.** Have `git log --oneline` ready in another tab so you can show recent commits as proof points.

---

## The 4-minute arc

### Beat 1 — The trap (0:00 – 0:30)

Open `tests/resilience.spec.ts` in a giant editor font. Read this line out loud, slowly:

```ts
const text = await userInfo.textContent();
expect(text).toBeTruthy();
```

Ask the audience two questions in sequence (don't reveal answers yet):

1. *"How many of you have tests like this in your codebase?"* (Hands go up.)
2. *"What is this test actually testing?"*

The audience will guess things like *"that user info loaded"* — wrong. It asserts `textContent` is non-null, which is true even if the page rendered nothing useful. The test passes whether the feature works or not.

**Don't reveal yet.** Move directly to Beat 2.

---

### Beat 2 — The audit reveal (0:30 – 1:30)

Switch to your AI assistant. Type this single sentence:

```
Use the mockkit audit-test-quality prompt on tests/, applyFixes: false
```

Let the AI work in **silence**. Do not narrate. Audience watches the structured report appear: 27 findings across 7 files, 9 patterns, severity ranks.

When output finishes, scroll to the **Pattern I (vacuous assertion)** finding for `resilience.spec.ts`. Say one line:

> *"The AI just told us that test passes whether the feature works or not."*

Then scroll to the **HIGH-severity Pattern H** finding for `dashboard-unit.test.ts:57` (invalid `import` inside a function body). Read the AI's note out loud:

> *"Code that was silently not running."*

**This is your detonation moment.** Stop talking for 3 full seconds. Let it sit.

Then deliver the punchline:

> *"This file claims 3 tests. It actually runs zero. It's been passing CI for months because failing-to-parse looks the same as no-tests-to-run."*

---

### Beat 3 — The fix (1:30 – 2:30)

```
Re-run the audit with applyFixes: true
```

Watch the diffs appear in the editor in real-time. Audience sees:
- `expect(text).toBeTruthy()` → `await expect(page.locator("#user-info")).toContainText(/Unauthorized/i)`
- The broken `import` moves from line 57 to the top of the file
- Hardcoded `toHaveCount(3)` → `expect(await rows.count()).toBeGreaterThanOrEqual(3)`

Run the suite live:

```bash
npx playwright test --reporter=list
```

`15 passed (1.9s)`. Then deliver the keeper line:

> *"Notice — runtime dropped from 13 seconds to 1.9 seconds. The fake tests were the slow ones. Real tests are faster than test theater."*

This is the most quotable single line in the demo. Write it on a slide if nothing else.

---

### Beat 4 — The forward loop (2:30 – 3:30)

> *"That was the audit. Here's the generator."*

```
Use the mockkit record-flow-as-test prompt:
  spec:    api-spec.yaml
  appUrl:  http://localhost:4000
  flow:    click create incident, fill title with "Demo incident",
           set severity to high, submit, verify it appears
```

The browser flies around on screen — Playwright MCP visibly drives the dashboard. Form fills. Submit clicks. The new incident appears in the list.

The AI emits the `.spec.ts` to `tests/demo.spec.ts`. Open the file. Audience sees:
- Semantic locators (`getByRole`, `getByLabel`)
- Unique title: `${TITLE_PREFIX}${Date.now()}`
- `test.afterEach` cleanup that survives mid-test failure

Run it:

```bash
npx playwright test tests/demo.spec.ts
```

Passes in ~800ms. Closing line for this beat:

> *"That test will run forever. In CI, against a deterministic mock. Self-cleaning. Permanent regression coverage. From one prompt."*

---

### Beat 5 — The synthesis (3:30 – 4:00)

Close with the closed-loop frame:

> *"Forward — AI ships disciplined tests. Backward — AI audits the ones you already have. Neither half is new on its own. The composition is."*
>
> *"Your test suite gets better every time you run this. Quality compounds. You stop being the test-quality enforcement layer."*

Long pause.

> *"That's the loop closing on quality, not just functionality."*

End slide:

> # One prompt. 15/15 tests. 1.9 seconds. Forever.
>
> Forward generation • Retroactive audit • Loop closes on quality

---

## Demo tactics that make or break it

| Tactic | Why |
|---|---|
| **Live, not recorded** | The audience needs to feel the AI is actually working. A pre-recorded video reads as marketing. |
| **Silence while AI thinks** | Don't narrate during the AI's work. Silence makes the output hit harder. You're not selling it — the AI is showing it. |
| **Audience prediction in Beat 1** | Their wrong guess makes the AI's catch land 3× harder. Engaged > passive. |
| **Pre-load everything** | Dashboard running, AI session open, file paths ready, terminal positioned. Every second of "loading…" kills energy. |
| **Big terminal font** | 18–22pt monospace, dark theme, no IDE chrome. Audience reads row 30 from the back of the room. |
| **Screen-recording backup** | MCP can flake live. If your session craps out, segue smoothly to a recording. Don't apologize, don't restart. |
| **The "13s → 1.9s" line** | Most quotable line in the whole arc. Write it on the closing slide. |
| **Don't fix everything** | Skip LOW-severity findings during the demo. The point is impact, not completeness. |

---

## Optional flourish — the audience callout

In Beat 2, after the audit completes, screenshot the report and drop it in the audience's Slack/Zoom chat. Then say:

> *"Run this prompt against your own codebase tonight. Email me what it finds. I'll bet 80% of you have a Pattern H or Pattern I you didn't know about."*

This is a commitment device that turns the demo into action. Days later they're showing colleagues *"look what MockKit found in our tests"* — that's how the loop closes outside your room.

---

## Fallback plan if something goes wrong live

| Failure | Recovery |
|---|---|
| AI returns a wrong/empty response | *"Let me show you what this normally returns"* — switch to screen recording, narrate over it. Don't restart the AI. |
| Browser doesn't open in Beat 4 | Skip the visual flair, just show the emitted .spec.ts as-if. *"In a normal run you'd see the browser fly around — here's the artifact it produces."* |
| Test suite fails on Beat 3 (post-apply) | Pre-emptively cherry-pick a known-good apply-fixes commit; run from that branch. If the live apply breaks, switch to the cherry-pick. |
| MockKit not running | The autonomous test fixture spawns its own — run anyway. Mention in passing: *"Notice the test is starting MockKit on its own — that's the fixture."* |
| Dashboard not running | This is fatal. Restart it (10 seconds), apologize once, pick up from where you were. Don't start over. |

---

## Pairing this demo with deck E

Slide 6 (Autonomous Loops) is your **backdrop** during the demo, not the focus. Have it open in another window so when you're between beats, audience can read the supporting copy.

The actual demo is the wow moment; the deck is the validation that this is a real product, not a magic trick. After Beat 5, flip to the closing slide of deck E and let it sit.

---

## Verified facts you can cite

These all come from real verification runs in `mockkit-enterprise-demo` during the session that produced this demo:

- **27 audit findings** across 7 hand-written test files (8 HIGH, 11 MEDIUM, 8 LOW)
- **AI-generated `autonomous-create-incident.spec.ts` had zero findings** — the only file in the suite that passed the audit by construction
- **`dashboard-unit.test.ts:57`** had an invalid `import` inside a function body — a real HIGH-severity bug humans missed even though the file appeared to "pass CI" (it parsed-to-skip rather than parse-and-run)
- **Suite runtime: 13s → 1.9s** after audit fixes, because the new resilience tests short-circuit on intercepted error routes instead of waiting for full SSE-laden page loads
- **8 built-in MCP prompts** in mockkit-mcp, including the two stars of this demo
- **20 enterprise scenarios** in `mockkit-enterprise-demo/scenarios/mcp-prompts.md`

If anyone asks for receipts, point them at:
- Repo: `github.com/pallavL01/mockkit_demo` — Scenarios 19 and 20 in `scenarios/mcp-prompts.md`
- Repo: `github.com/pallavL01/mockkit` — `packages/mcp-server/src/prompts/index.ts`
