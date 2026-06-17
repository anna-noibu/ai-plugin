---
name: tech-diagnosis
description: "Diagnose technical issues and Core Web Vital performance problems using Noibu data, and recommend a fix. Use when you want to know what's broken across the store, why an error is happening, what's causing errors, the root cause of poor page performance, how to fix a technical issue, why an error rate spiked, or where errors are concentrated."
---

# Tech Diagnosis

- **Proactive** — no specific issue in hand. Scans for highest-value errors and CWV problems, ranks them, surfaces a prioritized list.
- **Reactive** — a specific signal is already named. Produces a self-contained finding (Summary → Cause → Fix) the operator can act on, share, or escalate.

---

## Setup (always runs first)

1. **Noibu MCP** — confirm connected. If not, stop.
2. **Domain** — UUID provided → use it. Name only → resolve via lookup. Nothing → list domains and ask. Capture the **platform field** (Shopify, Magento, etc.) for fix tailoring.
3. **Mode + window** — detect mode (see below). Default window: 30 days. Override on request.
4. **TodoList** — create with mode-appropriate tasks.

## Mode detection

- **Proactive** — no specific issue given. A domain name alone is proactive — it provides context, not a signal. Examples: "run tech diagnosis", "what's broken", "find tech issues", "scan for issues", "/tech-diagnosis for store.com".
- **Share with vendor** — triggered when the message starts with "Share with vendor:". Skip setup, skip diagnosis. Go directly to Step P4's Share with vendor flow.
- **Reactive** — a specific issue is named alongside a domain: a named error, metric + page ("LCP on checkout"), Noibu issue ID, specific page URL, or structured handoff with domain + signal + context. Also reactive when the operator asks to update connectors ("update my setup", "change my connectors", "reconfigure") — the connector check at the start of reactive mode handles this.
- **Ambiguous** — ask: "Are you looking for a scan of what's broken across the store, or do you have a specific issue in mind?"

---

## Rules

- **No client-side data joins.** Each Noibu source on its own terms — no cross-table correlations.
- **Don't overstate impact.** "Present in X sessions", not "blocking X users". No revenue projections or revLost figures.
- **No console/replay links by default.** Inline the data; links only on explicit request.
- **Don't hardcode tool names.** Describe intent; the routing skill handles selection.
- **Suppress citation sections.**
- **Plain language.** Translate Noibu labels; keep identifiers in code style. Describe what to change, not how to navigate.
- **Proactive findings: call `show_widget` immediately.** Do not output findings as prose first. The tool call comes before any text output.
- **Closing offer after the fix: call `show_widget` for the next-steps buttons.** Never render the closing offer as plain text or generate a custom single button. Load `show_widget` via ToolSearch and use the button template from Step 6.
- **Live page inspection hierarchy.** Prefer Claude in Chrome. If Chrome isn't available, fall back to web_fetch using URLs constructed from the domain (already resolved in Setup) and affected paths from Noibu data — no need to ask the user. If neither is available, proceed with pattern-based recommendations only.
- **First visible output is the result, never narration.**
- **Never narrate scan criteria, filter logic, fallback decisions, or API internals.** This includes tool names, query parameters, API state values (PRIORITY, SPIKING, "new", "open"), error pool logic, or why one query returned nothing and another was tried. Fallbacks and retries happen silently.
- **Never surface config state.** Whether `$HOME/.tech-diagnosis-config.json` exists, is empty, or has specific values is never mentioned. Config reads and writes are silent. If nothing is saved, just ask for what's needed, save it on first use, and proceed — no admin questions about the config itself.
- **If any enrichment endpoint errors, skip it immediately and proceed** — do not retry or wait for a timeout. Erroring enrichment (Noibu AI diagnosis, GitHub, Shopify) is treated the same as "not available".
- **Live page inspection fallback is web_fetch, not WebSearch.** WebSearch is for looking up information, not fetching a specific store URL. Tool calls are silent. No preambles, filler, or "let me pull X". TodoList signals progress. After fetching, extract only `<script>` `src` and `crossorigin` attributes — discard the full page HTML immediately.
- **Pause only at designated points.** Reactive mode: pause once after the fix recommendation. Proactive mode: pause after surfacing the findings list. Open-ended questions only; never binary prompts or AskUserQuestion modals.
- **Propose a default and proceed.** Commit to the most likely cause/fix; mention alternatives in one sentence.
- **Page criticality is internal only.** Checkout > Cart > PDP > Collection > Home > Other. Prose only, never a labeled field.
- **Name third-party services specifically.** Stack trace names Klaviyo, GTM, Shop Pay → use that name.

---

# Proactive mode

**The very first action in proactive mode — before the TodoList, before any queries — is to send this message:** "Scanning [domain] for the top technical errors and performance issues — this may take a moment."

## Tasks
1. Scan errors
2. Scan performance
3. Rank and surface findings
4. Deep-dive (added when operator picks a finding)

Mark `in_progress` on start, `completed` when done.

## Step P1: Scan for errors

Fetch a broad candidate pool, then classify post-fetch using the platform reference file.

**Fetch candidates:** Use the priority errors tool with issue types PRIORITY, SPIKING, TOP_ISSUES in sequence. If fewer than 5 candidates total, supplement with the advanced error search tool using the same importance filter pattern (the Noibu routing skill has the correct field values). Limit: 20 from the fallback.

**Apply 48-hour recency filter post-fetch** — regardless of which tool returned them, drop any candidate last seen more than 48 hours ago. This applies to priority tool results too. What remains is the active candidate pool.

**Fetch detail:** For the top 10 remaining candidates by occurrence count, fetch stack trace and error type detail.

**Classify using platform path patterns** — check `platform_overrides` in `$HOME/.tech-diagnosis-config.json` first; if set, use those `merchant_paths` and `vendor_paths`. If not set, read `references/platforms/[platform].md` (use the platform field from Setup; fall back to `generic.md` for unknown platforms). Classify each candidate as:
- **Fixable** — frames in merchant-owned code paths
- **Platform** — frames only in platform infrastructure, or HTTP error types (403/429) with no JS stack trace
- **Vendor** — frames in known third-party scripts

Take top 2 fixable and top 2 vendor/platform candidates. Discard pure platform errors (no Fix or Share with vendor button needed for infrastructure issues the merchant can't touch).

**Score:** `occurrences_last_7_days × criticality_weight`
Weights (internal): Checkout=5, Cart=4, PDP=3, Collection=2, Home=1, Other=0.5

The 48-hour recency filter ensures only currently active errors surface. Scoring on last 7 days (rather than 30-day total) further deprioritizes errors that spiked and then resolved.

Mark "Scan errors" `completed`, "Scan performance" `in_progress`.

## Step P2: Scan for performance

Fetch page visit data with CWV (LCP, INP, CLS) across all page groups. Filter to pages where at least one metric is outside the Good band — see `references/performance.md` for thresholds.

**Score:** `traffic_weight × severity_score`
- traffic_weight — relative session volume, normalized (highest = 1.0)
- severity_score — Poor=3pts, Needs Improvement=1pt per metric. Max 9pts.

Take top 2. Capture: page/group, worst metric + p75 value, band, traffic volume.

Mark "Scan performance" `completed`, "Rank and surface findings" `in_progress`.

## Step P3: Rank and surface findings

Rank findings (score × criticality weight, ties by page criticality).

**Load `show_widget` via ToolSearch (`select:mcp__visualize__show_widget`) before calling it.** Then call `show_widget` with `title: "tech_findings"` and loading messages `["Scanning for errors...", "Checking performance...", "Ranking findings..."]`. Fill in the template below with real data. Always render both sections even if empty. Do not output findings as prose.

Badge inline styles — pick one per card:
- Active: `background:#FCEBEB;color:#A32D2D`
- Re-emerging (was quiet, now ticking up): `background:#FAEEDA;color:#854F0B`
- Improving (present but trending down): `background:#EAF3DE;color:#3B6D11`

sendPrompt formats — use these exactly, no verbose context:
- Fixable error Get fix button: `/tech-diagnosis fix #[issue-id] on [domain]`
- Vendor error Share with vendor button: `/tech-diagnosis share with vendor #[issue-id] on [domain]`
- Performance Fix button: `/tech-diagnosis fix [LCP|INP|CLS] on [page-url] for [domain]`

```html
<h2 style="position:absolute;width:1px;height:1px;overflow:hidden;">Tech findings for [domain] — last 30 days</h2>

<div style="display:flex;flex-direction:column;gap:24px;padding:4px;">

  <div>
    <p style="font-size:11px;font-weight:500;color:var(--color-text-secondary);text-transform:uppercase;letter-spacing:0.06em;margin:0 0 10px;">Issues you can fix</p>

    [Repeat for each fixable (first-party) error, up to 2:]
    <div style="border:0.5px solid var(--color-border-tertiary);border-radius:var(--border-radius-lg);overflow:hidden;margin-bottom:10px;">
      <div style="background:var(--color-background-secondary);padding:8px 20px;border-bottom:0.5px solid var(--color-border-tertiary);display:flex;align-items:center;justify-content:space-between;">
        <span style="font-size:12px;font-weight:500;color:var(--color-text-secondary);text-transform:uppercase;letter-spacing:0.06em;">[Error type — e.g. 403 Forbidden, JavaScript TypeError]</span>
        <span style="font-size:11px;font-weight:500;padding:2px 8px;border-radius:4px;[badge inline style]">[Active|Re-emerging|Improving]</span>
      </div>
      <div style="background:var(--color-background-primary);padding:20px;">
        <p style="font-size:15px;font-weight:500;margin:0 0 8px;color:var(--color-text-primary);">[Plain-English title]</p>
        <p style="font-size:14px;color:var(--color-text-secondary);line-height:1.6;margin:0 0 12px;">[1–2 sentences: what's wrong, where, who's affected]</p>
        <p style="font-size:13px;color:var(--color-text-secondary);margin:0 0 16px;">[occurrence count — must be a number e.g. "412 occurrences · last seen Jun 4 · product pages · Chrome/macOS" — always include the count, never substitute status words like "Active" for it]</p>
        <button onclick="sendPrompt('/tech-diagnosis fix #[issue-id] on [domain]')" style="padding:6px 14px;font-size:13px;font-weight:500;color:var(--color-text-primary);background:var(--color-background-secondary);border:0.5px solid var(--color-border-tertiary);border-radius:6px;cursor:pointer;">Get fix ↗</button>
      </div>
    </div>

    [If no fixable errors:]
    <p style="font-size:14px;color:var(--color-text-secondary);padding:16px 0;">No active errors found that you can fix directly.</p>

  </div>

  [If there are vendor errors, render a separate section:]
  <div>
    <p style="font-size:11px;font-weight:500;color:var(--color-text-secondary);text-transform:uppercase;letter-spacing:0.06em;margin:0 0 10px;">Vendor issues</p>

    [Repeat for each vendor (third-party) error, up to 2:]
    <div style="border:0.5px solid var(--color-border-tertiary);border-radius:var(--border-radius-lg);overflow:hidden;margin-bottom:10px;">
      <div style="background:var(--color-background-secondary);padding:8px 20px;border-bottom:0.5px solid var(--color-border-tertiary);display:flex;align-items:center;justify-content:space-between;">
        <span style="font-size:12px;font-weight:500;color:var(--color-text-secondary);text-transform:uppercase;letter-spacing:0.06em;">[Vendor name] — [Error type]</span>
        <span style="font-size:11px;font-weight:500;padding:2px 8px;border-radius:4px;background:var(--color-background-tertiary);color:var(--color-text-tertiary);">Vendor</span>
      </div>
      <div style="background:var(--color-background-primary);padding:20px;">
        <p style="font-size:15px;font-weight:500;margin:0 0 8px;color:var(--color-text-primary);">[Plain-English title]</p>
        <p style="font-size:14px;color:var(--color-text-secondary);line-height:1.6;margin:0 0 12px;">[1–2 sentences: what's happening, which vendor, impact]</p>
        <p style="font-size:13px;color:var(--color-text-secondary);margin:0 0 16px;">[occurrence count · vendor name · pages affected]</p>
        <button onclick="sendPrompt('/tech-diagnosis share with vendor #[issue-id] on [domain]')" style="padding:6px 14px;font-size:13px;font-weight:500;color:var(--color-text-secondary);background:var(--color-background-secondary);border:0.5px solid var(--color-border-tertiary);border-radius:6px;cursor:pointer;">Share with vendor ↗</button>
      </div>
    </div>

    [If no vendor errors:]
    [Omit vendor section entirely — don't show an empty vendor section]
  </div>

  <div>
    <p style="font-size:11px;font-weight:500;color:var(--color-text-secondary);text-transform:uppercase;letter-spacing:0.06em;margin:0 0 10px;">Performance</p>

    [Repeat for each performance finding, up to 2:]
    <div style="border:0.5px solid var(--color-border-tertiary);border-radius:var(--border-radius-lg);overflow:hidden;margin-bottom:10px;">
      <div style="background:var(--color-background-secondary);padding:8px 20px;border-bottom:0.5px solid var(--color-border-tertiary);display:flex;align-items:center;justify-content:space-between;">
        <span style="font-size:12px;font-weight:500;color:var(--color-text-secondary);text-transform:uppercase;letter-spacing:0.06em;">[LCP|INP|CLS]</span>
        <span style="font-size:11px;font-weight:500;padding:2px 8px;border-radius:4px;[badge inline style]">[Active|Re-emerging|Improving]</span>
      </div>
      <div style="background:var(--color-background-primary);padding:20px;">
        <p style="font-size:15px;font-weight:500;margin:0 0 8px;color:var(--color-text-primary);">[Plain-English title]</p>
        <p style="font-size:14px;color:var(--color-text-secondary);line-height:1.6;margin:0 0 12px;">[1–2 sentences: what's slow, where, how bad]</p>

        [Benchmark bar — set flex values and marker position per metric:
          LCP: green=25/yellow=15/red=60, scale 0–10s, labels "2.5s" "4.0s", marker=value/10*100%
          INP: green=20/yellow=30/red=50, scale 0–1000ms, labels "200ms" "500ms", marker=value/1000*100%
          CLS: green=10/yellow=15/red=25, scale 0–0.5, labels "0.1" "0.25", marker=value/0.5*100%
          Cap marker at 95% to keep it visible]
        <div style="margin-bottom:16px;">
          <div style="display:flex;align-items:center;gap:12px;margin-bottom:4px;">
            <span style="font-size:20px;font-weight:500;color:var(--color-text-primary);min-width:48px;">[p75 value]</span>
            <div style="flex:1;position:relative;height:24px;">
              <div style="display:flex;height:100%;border-radius:5px;overflow:hidden;">
                <div style="flex:[green-flex];background:#D1FAE5;display:flex;align-items:center;justify-content:center;"><span style="font-size:10px;color:#065F46;">✓</span></div>
                <div style="width:1px;background:white;"></div>
                <div style="flex:[yellow-flex];background:#FEF3C7;display:flex;align-items:center;justify-content:center;"><span style="font-size:10px;color:#92400E;">!</span></div>
                <div style="width:1px;background:white;"></div>
                <div style="flex:[red-flex];background:#FEE2E2;display:flex;align-items:center;justify-content:center;"><span style="font-size:10px;color:#991B1B;">✕</span></div>
              </div>
              <div style="position:absolute;top:-3px;left:[marker-position];transform:translateX(-50%);width:2.5px;height:30px;background:var(--color-text-primary);border-radius:2px;"></div>
            </div>
          </div>
          <div style="display:flex;padding-left:60px;">
            <div style="flex:[green-flex];"></div>
            <span style="font-size:10px;color:var(--color-text-tertiary);transform:translateX(-50%);">[threshold-1]</span>
            <div style="flex:[yellow-flex];"></div>
            <span style="font-size:10px;color:var(--color-text-tertiary);transform:translateX(-50%);">[threshold-2]</span>
            <div style="flex:[red-flex];"></div>
          </div>
        </div>

        <p style="font-size:13px;color:var(--color-text-secondary);margin:0 0 16px;">[traffic volume · page group · device]</p>
        <button onclick="sendPrompt('/tech-diagnosis fix [LCP|INP|CLS] on [page-url] for [domain]')" style="padding:6px 14px;font-size:13px;font-weight:500;color:var(--color-text-primary);background:var(--color-background-secondary);border:0.5px solid var(--color-border-tertiary);border-radius:6px;cursor:pointer;">Get fix ↗</button>
      </div>
    </div>

    [If no performance issues:]
    <p style="font-size:14px;color:var(--color-text-secondary);padding:16px 0;">No significant performance issues found.</p>
  </div>

</div>
```

Mark "Rank and surface findings" `completed`. The Fix button handles routing to reactive mode — do not auto-advance.

**Immediately after rendering the widget** (do not wait for a Fix button click), proceed to the scheduling offer below.

## Step P3.5: Scheduling offer

Call `list_scheduled_tasks` — if a scheduled tech scan already exists for this domain, skip this step entirely.

Otherwise, load `show_widget` via ToolSearch and render the collapsed schedule widget (see `references/schedule-widget.md`). It appears automatically in a collapsed state — no conversational prompt needed. The operator expands it when they're ready.

If yes, load `show_widget` via ToolSearch and call it with `title: "schedule_tech_scan"` and loading messages `["Setting up schedule..."]`. Pre-select weekly and Monday 9am as defaults.

See `references/schedule-widget.md` for the full widget template, submit handling, and scheduled task prompt format.

## Step P4: Hand off to reactive mode

**Fix button** → add "Deep-dive" task, mark `in_progress`. Enter reactive flow — domain, platform, and window already resolved. Signal is a pass-through; follow the normal reactive flow from Step 2 onwards.

**Share with vendor button** → skip the full diagnostic flow. Fetch the error detail, then generate a clear vendor-facing summary: what the error is, which pages it affects, occurrence count, browser/OS breakdown, and what the operator would like the vendor to investigate. Deliver it as a copyable block the operator can paste into a support ticket or email. Offer to send it via email or Slack if those connectors are available.

---

# Reactive mode

## Connector check (runs once before diagnosis)

Read `$HOME/.tech-diagnosis-config.json` silently. For each applicable connector (Shopify if on Shopify, GitHub, Chrome), check if it has a saved status (`connected`, `skipped`, or `pending_connection`). If all applicable connectors have a saved status, proceed directly — no setup, no mention of connectors.

If any connector has no saved status, or if mode is Setup (operator asked to reconfigure): read `references/first-run-setup.md` silently, then **stop and ask only about unset connectors before running any queries**. Do not call `suggest_connectors` for a connector that is already available in the current session. The first visible output must be the intro message. Do not fetch any data until setup is complete and config is written.

## Tasks
- Understanding the issue
- Measure the impact
- Trace the cause
- Recommend a fix
- Optional handoff *(conditional — if operator wants to share or ticket)*

Mark `in_progress` on start, `completed` when done.

## Step 1: Identify the signal

Translate the prompt into symptom, scope, and context (for handoffs). If the signal is clear, go straight to fetching — don't narrate it back. Examples:

| Prompt | Signal |
|---|---|
| "Why is LCP poor on checkout?" | Performance — LCP — checkout |
| "What's causing errors on Safari mobile?" | Errors — Safari mobile |

If ambiguous, ask one clarifying question. Mark "Understanding the issue" `in_progress`.

## Step 2: Fetch focused data

Targeted queries only — no broad surveys.
- **Errors** — priority issues matching scope; full detail for top 1–2.
- **Performance** — CWV at p75 for affected URLs/page group, segmented by device and browser.
- **Combined** — both, scoped to the relevant page/segment.

Run in parallel where possible. Mark "Understanding the issue" `completed`, "Measure the impact" `in_progress`.

## Step 3: Cross-reference, filter, render Summary

**Post-processing:**
- Errors: flag third-party origin, compute page criticality, read funnel stage. Apply cookie-consent observability check. See `references/errors.md`.
- Performance: map p75 to band, identify worst segment. See `references/performance.md`.
- Ranking: severity → page criticality → impacted sessions or distance from threshold. Never by revenue. Cap at 3–5.

**Render Summary** for each finding — in this order:
1. Finding title as a markdown `##` heading
2. Narrative paragraph (2–3 sentences): impact only — who's affected, how many, what they experience, trend. No technical identifiers, error codes, or endpoint names — those belong in Cause or Fix.
3. Impacted-sessions widget: two metric cards only (session count + % of total traffic) + line/area chart. No trend card — the chart shows the trend.
4. Top pages affected: 3–5 row table (URL + impacted sessions)

Mark "Measure the impact" `completed`. Continue directly to cause tracing.

## Step 4: Trace the cause

- **Noibu AI diagnosis** — use the Why/Impact content as the basis for the cause narrative.
- **Live page inspection** — use Chrome if available (navigate to affected pages, apply device emulation matching where the issue is worst — see `references/performance.md`). If Chrome isn't available, fall back to web_fetch using `https://{domain}{path}` constructed from the resolved domain and affected URLs from Noibu data. After fetching, extract only `<script>` `src` and `crossorigin` attributes — do not retain the full page HTML in context. For INP without Chrome, fetch `CLICKED_SELECTORS_WITH_COUNTS` for the affected page from Noibu to identify likely slow interaction targets. If neither is available, proceed with pattern-based recommendations.
- **GitHub enrichment** — search for the affected element, fetch ~20 lines context. SPA caveat: hashed class names may produce multiple candidate files — surface them.
- **Platform enrichment** — query store data to confirm/refute hypotheses. With confirmation, state cause confidently; without, stay in working-hypothesis framing. Note: Shopify MCP doesn't expose theme files — use GitHub.

See `references/errors.md` for error-specific guidance. See `references/performance.md` for performance-specific guidance.

**Render cause:** open with "From here on, I'm reasoning over the data rather than reporting it…" If multiple causes, rank by likelihood and commit to the primary.

Mark "Trace the cause" `completed`. Continue directly to fix recommendation.

## Step 5: Closing offer

Determine the fix internally (file, change, risk level, alternative) but **do not render it**. The fix is shown only when the operator clicks "Show me the fix".

Load `show_widget` via ToolSearch and call it with `title: "next_steps"` and loading messages `["What's next..."]`. Nothing renders after the widget — no closing summary, no prose. Never mention connectors, grep, WAF, CDN, or enrichment steps.

**Visibility rules:**
- **"Apply the fix automatically"** — hide only if GitHub config status is `skipped`. Otherwise:
  - **Disabled** (grey, no click) — only when GitHub IS connected but the fix isn't a code change in a specific file (e.g. vendor escalation, platform setting, app config) — there's no file to edit as a PR.
  - **Active** — all other cases: GitHub not connected, no repo saved, or fix is a code change in a known file. Clicking handles whatever is missing — prompts to connect GitHub, asks for the repo, or creates the PR.
- **"Show me the fix"** — always shown.
- **"Open a ticket"** — hide only if all ticket connectors have been explicitly declined. Show if any connector is connected or unset (clicking triggers `suggest_connectors` if none are connected).
- **"Share findings"** — always.
- **"Done for now"** — always.

```html
<div style="border:0.5px solid var(--color-border-tertiary);border-radius:var(--border-radius-lg);background:var(--color-background-primary);padding:20px;">
  <p style="font-size:13px;font-weight:500;color:var(--color-text-primary);margin:0 0 16px;">What would you like to do next?</p>
  [If GitHub skipped → omit entirely]
  [If GitHub IS connected AND fix is not a code change → show disabled container:]
  <div style="display:block;width:100%;padding:12px 16px;background:var(--color-background-secondary);border:0.5px solid var(--color-border-tertiary);border-radius:var(--border-radius-md);margin-bottom:8px;opacity:0.5;cursor:not-allowed;">
    <p style="font-size:14px;font-weight:500;color:var(--color-text-tertiary);margin:0 0 4px;">Apply the fix automatically</p>
    <p style="font-size:12px;color:var(--color-text-tertiary);margin:0;">The fix for this issue isn't a code change — there's no file to edit automatically.</p>
  </div>
  [All other cases → show active, with risk level as subtitle:]
  [Label: "Apply the fix automatically" + if repo saved append " — [org/repo]"]
  <div onclick="sendPrompt('Apply the fix automatically')" style="display:block;width:100%;padding:12px 16px;background:var(--color-background-secondary);border:0.5px solid var(--color-border-tertiary);border-radius:var(--border-radius-md);cursor:pointer;margin-bottom:8px;">
    <p style="font-size:14px;font-weight:500;color:var(--color-text-primary);margin:0 0 3px;">[Label] ↗</p>
    <p style="font-size:12px;color:var(--color-text-tertiary);margin:0;">[low|medium|high] risk · test on preview before publishing</p>
  </div>
  <button onclick="sendPrompt('Show me the fix')" style="display:block;width:100%;text-align:left;padding:12px 16px;font-size:14px;font-weight:500;color:var(--color-text-primary);background:var(--color-background-secondary);border:0.5px solid var(--color-border-tertiary);border-radius:var(--border-radius-md);cursor:pointer;margin-bottom:8px;">Show me the fix ↗</button>
  [If ticket connectors not all skipped:]
  [Label rules: 0 saved → "Open a ticket"; 1 saved → "Open a ticket in [connector] — [team]"; 2+ saved → "Open a ticket in [first connector] & [n] more"]
  <button onclick="sendPrompt('Open a ticket')" style="display:block;width:100%;text-align:left;padding:12px 16px;font-size:14px;font-weight:500;color:var(--color-text-primary);background:var(--color-background-secondary);border:0.5px solid var(--color-border-tertiary);border-radius:var(--border-radius-md);cursor:pointer;margin-bottom:8px;">[Label] ↗</button>
  [Label rules: 0 saved → "Share findings"; 1 saved → "Share findings to [destination]"; 2+ saved → "Share findings to [first] & [n] more"]
  <button onclick="sendPrompt('Share findings')" style="display:block;width:100%;text-align:left;padding:12px 16px;font-size:14px;font-weight:500;color:var(--color-text-primary);background:var(--color-background-secondary);border:0.5px solid var(--color-border-tertiary);border-radius:var(--border-radius-md);cursor:pointer;margin-bottom:8px;">[Label] ↗</button>
  <button onclick="sendPrompt('Done for now')" style="display:block;width:100%;text-align:left;padding:12px 16px;font-size:14px;color:var(--color-text-secondary);background:var(--color-background-primary);border:0.5px solid var(--color-border-tertiary);border-radius:var(--border-radius-md);cursor:pointer;">Done for now</button>
</div>
```

**When "Show me the fix" is clicked** — render the fix section only (no repeating Summary or Cause). Then show the closing offer again so the operator can still act on it.

Mark "Closing offer" `completed`.

### Output structure

Each section has one job — no repeating content from a prior section.

```
## [Finding title — plain English]

### Summary
[2–3 sentences — impact only: who's affected, how many, what they experience, trend.
No technical identifiers, error codes, or endpoint names — those belong in Cause or Fix.]
[Impacted-sessions widget: two metric cards only (session count + % of total) + line/area chart. No trend card.]
**Top pages affected:** 3–5 row table

### What's likely causing it
*From here on, I'm reasoning over the data rather than reporting it...*
[1–2 sentences — the hypothesis only. Don't restate what broke. Start from why.]

[Closing offer widget — rendered here. Fix is NOT shown by default.]

--- shown only after "Show me the fix" is clicked ---

### How to fix it
[The specific change only. No restating the cause. Code or config, testing note, alternative pointer.]

### Technical details  [on request or via share widget]
See references/errors.md or references/performance.md for field list.
```

## Step 6: Handoff *(conditional)*

**Ask:** "Share as tickets or as a report to send someone?"

See `references/ticket.md` for the ticket flow. See `references/share.md` for the share widget and output formats.

Mark "Optional handoff" `completed`.

---

## Configuration

`$HOME/.tech-diagnosis-config.json` — `$HOME` is a standard environment variable that works on Mac, Linux, and Windows. Accessible from any Cowork session regardless of which folder is mounted or how the session was opened (deep links, scheduled tasks, direct invocation).

**Always use the shell tool to read and write this file.** Never use the file read/write tools — those are workspace-relative and will fail in sessions without a mounted folder.

- Read: `cat "$HOME/.tech-diagnosis-config.json" 2>/dev/null || echo '{}'`
- Write (merge, never clobber existing keys):
  ```
  python3 -c "
  import json, os
  path = os.path.join(os.environ['HOME'], '.tech-diagnosis-config.json')
  cfg = json.load(open(path)) if os.path.exists(path) else {}
  cfg.update({...})  # merge new values
  json.dump(cfg, open(path,'w'), indent=2)
  "
  ```

**Write config immediately whenever any preference is confirmed** — do not defer to end of session. Triggers:
- Connector setup complete (first-run-setup.md)
- Ticket destination confirmed (first use or change)
- Share destination or include selection confirmed (first use or change)
- Schedule created (save scheduled task details)
- Any explicit "update my setup" / "change my connectors" flow

See `references/first-run-setup.md` for the guided setup flow.

```json
{
  "github": { "status": "connected|skipped|pending_connection", "repo": "org/repo" },
  "shopify": { "status": "connected|skipped|pending_connection" },
  "chrome_inspection": "skipped",
  "platform_overrides": {
    "merchant_paths": ["cdn.shopify.com/s/files/", "assets.mystore.com/"],
    "vendor_paths": ["static.klaviyo.com/", "heyethos.com/"]
  },
  "tickets": {
    "linear": { "team": "Issues Team", "project": "Tech Issues" },
    "notion": { "destination": "Bug Tracker", "url": "https://notion.so/..." },
    "jira": { "project": "TECH" },
    "github_issues": { "repo": "org/repo" }
  },
  "share": {
    "destinations": ["pdf", "slack"],
    "include": ["cause", "fix"],
    "slack_channel": "#tech-issues",
    "email": "team@example.com",
    "notion_page": "Tech findings"
  }
}
```

`platform_overrides` and `tickets` are optional. Operators can ask Claude to update any field in plain language — Claude will edit the file directly.
