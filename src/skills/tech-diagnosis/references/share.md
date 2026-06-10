# Share findings

Reached when the operator selects "Share findings" from the closing offer.

Load `show_widget` via ToolSearch, then call it with `title: "share_findings"` and loading messages `["Setting up sharing options..."]`.

Before rendering, read the config for previously saved share preferences (`share.destinations`, `share.include`). Use as defaults — pre-select saved chips. If no saved preferences, default to PDF selected, Cause and Fix selected.

After submit, save selections back to config under `share`.

---

## Widget

Show all destinations as equal multi-select chips — no connected/unconnected distinction in the UI. Pre-select any previously saved destinations from config. Connection is handled after the operator submits, per destination.

```html
<style>
.chip{padding:6px 14px;font-size:13px;font-weight:500;border-radius:100px;cursor:pointer;background:var(--color-background-primary) !important;border:1px solid var(--color-border-tertiary) !important;color:var(--color-text-secondary) !important;}
.chip.on{background:#E6F1FB !important;border:1.5px solid #185FA5 !important;color:#0C447C !important;}
.slabel{font-size:11px;font-weight:500;color:var(--color-text-secondary);text-transform:uppercase;letter-spacing:0.07em;margin:0 0 10px;}
</style>

<div style="border:0.5px solid var(--color-border-tertiary);border-radius:var(--border-radius-lg);background:var(--color-background-primary);padding:24px;">
  <p style="font-size:15px;font-weight:500;color:var(--color-text-primary);margin:0 0 4px;">Share findings for [domain]</p>
  <p style="font-size:13px;color:var(--color-text-secondary);margin:0 0 24px;">Summary is always included. Pick a destination and any extras.</p>

  <p class="slabel">Destination</p>
  <div style="display:flex;gap:8px;flex-wrap:wrap;margin-bottom:24px;">
    <button class="chip on" onclick="toggleChip(this)" data-value="pdf"><i class="ti ti-file" style="font-size:14px;vertical-align:-1px;margin-right:4px;" aria-hidden="true"></i>PDF</button>
    <button class="chip [on if previously selected]" onclick="toggleChip(this)" data-value="email"><i class="ti ti-mail" style="font-size:14px;vertical-align:-1px;margin-right:4px;" aria-hidden="true"></i>Email</button>
    <button class="chip [on if previously selected]" onclick="toggleChip(this)" data-value="slack"><i class="ti ti-brand-slack" style="font-size:14px;vertical-align:-1px;margin-right:4px;" aria-hidden="true"></i>Slack</button>
    <button class="chip [on if previously selected]" onclick="toggleChip(this)" data-value="notion"><i class="ti ti-brand-notion" style="font-size:14px;vertical-align:-1px;margin-right:4px;" aria-hidden="true"></i>Notion</button>
  </div>

  <p class="slabel">Include</p>
  <div style="display:flex;gap:8px;flex-wrap:wrap;margin-bottom:28px;">
    <button class="chip on" onclick="toggleChip(this)" data-value="cause">Cause</button>
    <button class="chip on" onclick="toggleChip(this)" data-value="fix">Fix</button>
    <button class="chip [on if previously selected]" onclick="toggleChip(this)" data-value="technical">Technical details</button>
  </div>

  <div style="display:flex;justify-content:flex-end;align-items:center;gap:12px;border-top:0.5px solid var(--color-border-tertiary);padding-top:16px;">
    <button onclick="submitShare()" style="padding:7px 20px;font-size:13px;font-weight:500;color:var(--color-text-primary);background:var(--color-background-secondary);border:0.5px solid var(--color-border-secondary);border-radius:var(--border-radius-md);cursor:pointer;">Share ↗</button>
  </div>
</div>

<script>
function toggleChip(el){el.classList.toggle('on');}
function submitShare(){
  const dest=[...document.querySelectorAll('[data-value="pdf"],[data-value="email"],[data-value="slack"],[data-value="notion"]')].filter(b=>b.classList.contains('on')).map(b=>b.dataset.value);
  const inc=[...document.querySelectorAll('[data-value="cause"],[data-value="fix"],[data-value="technical"]')].filter(b=>b.classList.contains('on')).map(b=>b.dataset.value);
  if(!dest.length){alert('Select at least one destination.');return;}
  sendPrompt('Share: destinations='+dest.join(',')+' include='+inc.join(','));
}
</script>
```

**Per-destination follow-up** (check config for saved values first; collect all missing details before sending anything):
- PDF → no follow-up; save and confirm path
- Email → if not connected, trigger `suggest_connectors` (search registry for "gmail"); then "What email address?" (skip if saved)
- Slack → if not connected, trigger `suggest_connectors` UUID `597f662f-36de-437e-836e-5a81013cbfbe`; then "Which channel?" (skip if saved)
- Notion → if not connected, trigger `suggest_connectors` UUID `69f3a300-cc60-48c4-b237-dfac56530dbf`; then "Which page or database?" (skip if saved)

Once all details are confirmed, **write config via shell tool immediately** (merge into `share` key), then process all selected destinations in parallel.

---

## Generating the artifact

Content order: Summary → Technical details (if selected) → Cause (if selected) → Fix (if selected).

**Static-document rewrite required:**
- No "let me know if" / "want me to walk through" hooks
- No pause-for-direction text
- Alternative-fix mentions become static: "If the primary fix doesn't resolve, the alternative is [X]"
- Reasoning disclaimer becomes a single-line caveat at the top

**Per-channel:**
- **PDF** — `tech-diagnosis-[domain]-[date].pdf`. Confirm path after saving.
- **Email** — subject: `Tech findings for [domain] — [N] items to investigate`. HTML body, scannable.
- **Slack** — lead-in: `*Tech findings for [domain]* — [N] items, last [window].` One mrkdwn section per finding.
- **Notion** — title: `Tech findings for [domain] — [date]`. Sectioned page.

No console/replay URLs in any shared content. Issue IDs as plain identifiers are fine. Links only on explicit request.
