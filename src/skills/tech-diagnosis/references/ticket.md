# Ticket

Reached when the operator selects "Open a ticket" from the closing offer.

Full content always included (Summary + Technical details + Cause + Fix). No widget, no content selection.

## Apply the fix automatically

Only available when **both** conditions are met:
- GitHub is connected (MCP + app installed + repo in config)
- A specific file and line was identified from source enrichment

If either condition isn't met, do not offer or mention this option.

When both conditions are met:
1. **Confirm the fix** — briefly restate the specific change in plain language and ask the operator to confirm before touching their code.
2. **Create a branch** — name it descriptively (e.g. `fix/[plain-English-description]`).
3. **Apply the fix** — write the exact code change to the correct file in the repo.
4. **Open a review request** — title: plain-English summary of what was fixed. Body: Summary → Cause → Fix in markdown. Link to the Noibu finding by issue ID.
5. **Confirm it's ready** — surface the link and tell them someone needs to review and approve it before it goes live.

---

## Creating a ticket

1. **Show all ticket options as multi-select** — present Linear, Jira, GitHub Issues, and Notion as toggleable chips. The operator can select one or more. No "Connected" badge or status indicator. If destinations are already saved in `tickets` config, pre-select them. Show a confirm button to proceed.

   Use this chip pattern (load `show_widget` via ToolSearch):

   ```html
   <div style="border:0.5px solid var(--color-border-tertiary);border-radius:var(--border-radius-lg);background:var(--color-background-primary);padding:24px;">
     <p style="font-size:15px;font-weight:500;color:var(--color-text-primary);margin:0 0 16px;">Where should the ticket go?</p>
     <div style="display:flex;gap:8px;flex-wrap:wrap;margin-bottom:24px;">
       <button class="chip [on if saved]" onclick="toggleChip(this)" data-value="linear">Linear</button>
       <button class="chip [on if saved]" onclick="toggleChip(this)" data-value="jira">Jira</button>
       <button class="chip [on if saved]" onclick="toggleChip(this)" data-value="github-issues">GitHub Issues</button>
       <button class="chip [on if saved]" onclick="toggleChip(this)" data-value="notion">Notion</button>
     </div>
     <div style="display:flex;justify-content:flex-end;border-top:0.5px solid var(--color-border-tertiary);padding-top:16px;">
       <button onclick="submitTicket()" style="padding:7px 20px;font-size:13px;font-weight:500;color:var(--color-text-primary);background:var(--color-background-secondary);border:0.5px solid var(--color-border-secondary);border-radius:var(--border-radius-md);cursor:pointer;">Open ticket ↗</button>
     </div>
   </div>
   <style>.chip{padding:6px 14px;font-size:13px;font-weight:500;border-radius:100px;cursor:pointer;background:var(--color-background-primary) !important;border:1px solid var(--color-border-tertiary) !important;color:var(--color-text-secondary) !important;}.chip.on{background:#E6F1FB !important;border:1.5px solid #185FA5 !important;color:#0C447C !important;}</style>
   <script>
   function toggleChip(el){el.classList.toggle('on');}
   function submitTicket(){
     const sel=[...document.querySelectorAll('.chip.on')].map(b=>b.dataset.value);
     if(!sel.length){alert('Select at least one destination.');return;}
     sendPrompt('Open ticket in: '+sel.join(' and '));
   }
   </script>
   ```

   After the operator confirms: for each selected destination, check if it's connected. If connected, proceed directly. If not connected, **do not narrate that it's unavailable** — call `suggest_connectors` immediately to show the install UI. Load `suggest_connectors` via ToolSearch if needed. UUIDs:
   - Linear: search registry for "linear"
   - Jira: search registry for "jira"
   - GitHub Issues: `fe983ccb-92c7-4df1-85af-b1c3340b89bb`
   - Notion: `69f3a300-cc60-48c4-b237-dfac56530dbf`

   Wait for the operator to connect, then proceed with ticket creation for that destination.

2. **Check `tickets` in config** for saved destinations for each selected connector. If found, use silently. If not found, ask before creating and let the operator know it will be saved: e.g. "Which Linear team should this go to? I'll save your answer for next time." Collect all missing details before creating any tickets. **Write config via shell tool immediately after all details are confirmed** — before creating the tickets.

3. **Create tickets in parallel** — one per finding per selected destination:
   - Title: `[severity] [summary] (affecting [segment])`
   - Body in markdown: Summary → Technical details → Cause → Fix
   - Priority mapping (where supported): critical → high; standard → medium; third-party-origin → default/low
   - Labels (only if pre-existing): `tech-diagnosis`, `error` or `performance`, severity, `noibu`
   - Assignee: unset unless operator specifies

4. **Surface ticket URLs** after creation. Report partial failures honestly.
