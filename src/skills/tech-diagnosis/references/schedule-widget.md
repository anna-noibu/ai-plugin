# Scheduling widget

Called from Step P3.5. Render automatically — no conversational prompt. Load `show_widget` via ToolSearch and call it with `title: "schedule_tech_scan"` and loading messages `["Setting up schedule..."]`.

The widget renders **collapsed by default**. The operator clicks to expand when ready. This keeps it visible without competing with the findings cards above.

## Widget

```html
<style>
.chip{padding:6px 14px;font-size:13px;font-weight:500;border-radius:100px;cursor:pointer;background:var(--color-background-primary) !important;border:1px solid var(--color-border-tertiary) !important;color:var(--color-text-secondary) !important;}
.chip.on{background:#E6F1FB !important;border:1.5px solid #185FA5 !important;color:#0C447C !important;}
.fcard{padding:14px 10px;font-size:13px;font-weight:500;text-align:center;border-radius:var(--border-radius-md);cursor:pointer;display:flex;flex-direction:column;align-items:center;gap:6px;background:var(--color-background-primary) !important;border:1px solid var(--color-border-tertiary) !important;color:var(--color-text-secondary) !important;}
.fcard.on{background:#E6F1FB !important;border:1.5px solid #185FA5 !important;color:#0C447C !important;}
.slabel{font-size:11px;font-weight:500;color:var(--color-text-secondary);text-transform:uppercase;letter-spacing:0.07em;margin:0 0 10px;}
</style>

<div style="border:0.5px solid var(--color-border-tertiary);border-radius:var(--border-radius-lg);background:var(--color-background-primary);overflow:hidden;">
  <!-- Collapsed header — always visible, click to expand -->
  <div onclick="toggleSchedule()" style="display:flex;align-items:center;justify-content:space-between;padding:16px 20px;cursor:pointer;">
    <div>
      <p style="font-size:14px;font-weight:500;color:var(--color-text-primary);margin:0 0 2px;">Schedule recurring scan</p>
      <p style="font-size:12px;color:var(--color-text-tertiary);margin:0;">Get this report delivered weekly, bi-weekly, or monthly</p>
    </div>
    <span id="sched-chevron" style="font-size:18px;color:var(--color-text-tertiary);transition:transform 0.2s;">›</span>
  </div>

  <!-- Expandable body — hidden by default -->
  <div id="sched-body" style="display:none;padding:0 20px 20px;border-top:0.5px solid var(--color-border-tertiary);">
    <div style="height:16px;"></div>
    <p style="font-size:15px;font-weight:500;color:var(--color-text-primary);margin:0 0 24px;">Schedule tech scan</p>

  <p class="slabel">Frequency</p>
  <div style="display:grid;grid-template-columns:repeat(3,1fr);gap:8px;margin-bottom:24px;" id="freq-wrap">
    <button class="fcard on" onclick="selectFreq(this)" data-value="weekly"><i class="ti ti-calendar" aria-hidden="true"></i>Weekly</button>
    <button class="fcard" onclick="selectFreq(this)" data-value="biweekly"><i class="ti ti-calendar-stats" aria-hidden="true"></i>Bi-weekly</button>
    <button class="fcard" onclick="selectFreq(this)" data-value="monthly"><i class="ti ti-calendar-month" aria-hidden="true"></i>Monthly</button>
  </div>

  <div id="day-section" style="margin-bottom:24px;">
    <p class="slabel">Day</p>
    <div style="display:flex;gap:8px;flex-wrap:wrap;">
      <button class="chip on" onclick="selectDay(this)" data-value="Monday">Mon</button>
      <button class="chip" onclick="selectDay(this)" data-value="Tuesday">Tue</button>
      <button class="chip" onclick="selectDay(this)" data-value="Wednesday">Wed</button>
      <button class="chip" onclick="selectDay(this)" data-value="Thursday">Thu</button>
      <button class="chip" onclick="selectDay(this)" data-value="Friday">Fri</button>
    </div>
  </div>

  <div style="margin-bottom:24px;">
    <p class="slabel">Time</p>
    <div style="display:flex;gap:8px;flex-wrap:wrap;" id="time-section">
      <button class="chip" onclick="selectTime(this)" data-value="7:00 AM">7 am</button>
      <button class="chip on" onclick="selectTime(this)" data-value="9:00 AM">9 am</button>
      <button class="chip" onclick="selectTime(this)" data-value="12:00 PM">12 pm</button>
      <button class="chip" onclick="selectTime(this)" data-value="5:00 PM">5 pm</button>
    </div>
  </div>

  <p class="slabel">Send to</p>
  <p style="font-size:12px;color:var(--color-text-tertiary);margin:0 0 10px;">At least one required. Only Email, Slack, and Notion — do not add any other options.</p>
  <div style="display:flex;gap:8px;flex-wrap:wrap;margin-bottom:28px;">
    [If Email connected:]   <button class="chip" onclick="toggleChip(this)" data-value="email"><i class="ti ti-mail" style="font-size:14px;vertical-align:-1px;margin-right:4px;" aria-hidden="true"></i>Email</button>
    [If Email not connected:] <button onclick="sendPrompt('Connect email so I can schedule the tech scan digest to it')" style="padding:6px 14px;font-size:13px;font-weight:500;border-radius:100px;border:1px dashed var(--color-border-tertiary);background:var(--color-background-primary);color:var(--color-text-tertiary);cursor:pointer;"><i class="ti ti-mail" style="font-size:14px;vertical-align:-1px;margin-right:4px;" aria-hidden="true"></i>Email <span style="font-size:11px;">+ Connect</span></button>
    [If Slack connected:]   <button class="chip" onclick="toggleChip(this)" data-value="slack"><i class="ti ti-brand-slack" style="font-size:14px;vertical-align:-1px;margin-right:4px;" aria-hidden="true"></i>Slack</button>
    [If Slack not connected:] <button onclick="sendPrompt('Connect Slack so I can schedule the tech scan digest to it')" style="padding:6px 14px;font-size:13px;font-weight:500;border-radius:100px;border:1px dashed var(--color-border-tertiary);background:var(--color-background-primary);color:var(--color-text-tertiary);cursor:pointer;"><i class="ti ti-brand-slack" style="font-size:14px;vertical-align:-1px;margin-right:4px;" aria-hidden="true"></i>Slack <span style="font-size:11px;">+ Connect</span></button>
    [If Notion connected:]  <button class="chip" onclick="toggleChip(this)" data-value="notion"><i class="ti ti-brand-notion" style="font-size:14px;vertical-align:-1px;margin-right:4px;" aria-hidden="true"></i>Notion</button>
    [If Notion not connected:] <button onclick="sendPrompt('Connect Notion so I can schedule the tech scan digest to it')" style="padding:6px 14px;font-size:13px;font-weight:500;border-radius:100px;border:1px dashed var(--color-border-tertiary);background:var(--color-background-primary);color:var(--color-text-tertiary);cursor:pointer;"><i class="ti ti-brand-notion" style="font-size:14px;vertical-align:-1px;margin-right:4px;" aria-hidden="true"></i>Notion <span style="font-size:11px;">+ Connect</span></button>
  </div>

    <div style="display:flex;justify-content:flex-end;align-items:center;gap:12px;border-top:0.5px solid var(--color-border-tertiary);padding-top:16px;">
      <button onclick="sendPrompt('Skip scheduling for now')" style="padding:7px 16px;font-size:13px;font-weight:500;color:var(--color-text-secondary);background:transparent;border:none;cursor:pointer;">Skip</button>
      <button onclick="submitSchedule()" style="padding:7px 20px;font-size:13px;font-weight:500;color:var(--color-text-primary);background:var(--color-background-secondary);border:0.5px solid var(--color-border-secondary);border-radius:var(--border-radius-md);cursor:pointer;">Schedule scan</button>
    </div>
  </div>
</div>

<script>
function toggleSchedule(){
  const body=document.getElementById('sched-body');
  const chevron=document.getElementById('sched-chevron');
  const open=body.style.display==='none';
  body.style.display=open?'block':'none';
  chevron.style.transform=open?'rotate(90deg)':'';
}
function toggleChip(el){el.classList.toggle('on');}
function selectDay(el){document.querySelectorAll('#day-section .chip').forEach(b=>b.classList.remove('on'));el.classList.add('on');}
function selectTime(el){document.querySelectorAll('#time-section .chip').forEach(b=>b.classList.remove('on'));el.classList.add('on');}
function selectFreq(el){
  document.querySelectorAll('#freq-wrap .fcard').forEach(b=>b.classList.remove('on'));
  el.classList.add('on');
  document.getElementById('day-section').style.display=el.dataset.value==='monthly'?'none':'block';
}
function submitSchedule(){
  const freq=document.querySelector('#freq-wrap .fcard.on')?.dataset.value||'weekly';
  const day=document.querySelector('#day-section .chip.on')?.dataset.value||'Monday';
  const time=document.querySelector('#time-section .chip.on')?.dataset.value||'9:00 AM';
  const delivery=[...document.querySelectorAll('[data-value="email"],[data-value="slack"],[data-value="notion"]')].filter(b=>b.classList.contains('on')).map(b=>b.dataset.value);
  if(!delivery.length){alert('Select at least one delivery method.');return;}
  sendPrompt('Schedule tech scan: frequency='+freq+', day='+day+', time='+time+', delivery='+delivery.join(' and '));
}
</script>
```

## On submit

Ask for any missing delivery details (email address, Slack channel, Notion page) if not already in config. Then create the scheduled task with `create_scheduled_task`. Task name: `tech-scan-[domain]`.

If a "+ Connect" button is clicked: call `suggest_connectors` with the appropriate UUID (Slack: `597f662f-36de-437e-836e-5a81013cbfbe`; Notion: `69f3a300-cc60-48c4-b237-dfac56530dbf`; Email/Gmail: search registry for "gmail"). After connecting, re-render the widget with that connector unlocked.

## Scheduled task prompt

> Run /tech-diagnosis for [domain]. When the proactive scan completes, do not show the findings widget — instead format the results as a [email/Slack/Notion] digest and send to [destination]. Use short Cowork prompts for each finding — errors use their Noibu issue ID, performance findings use metric + page URL.

**Cowork prompts per finding type:**
- Fixable error: `/tech-diagnosis fix #[issue-id] on [domain]`
- Vendor error: `/tech-diagnosis share with vendor #[issue-id] on [domain]`
- Performance: `/tech-diagnosis fix [LCP|INP|CLS] on [page-url] for [domain]`

**Email:** Subject `Tech scan — [domain] — [date]`. Each finding: title, 1-sentence description, metric/count. Show the Cowork command wrapped in an HTML `<code>` tag to prevent Gmail from auto-linking the domain — e.g. `Open Cowork and type: <code>/tech-diagnosis fix #61 on louloulollipop.com</code>`

**Slack:** Lead `*Tech scan — [domain]* — [date]`. Each finding with title, one-liner, and "Open Cowork and type:" followed by the command in a code block (backtick-wrapped so it's copyable): `/tech-diagnosis fix #[id] on [domain]`

**Notion:** Page titled `Tech scan — [domain] — [date]` with sections (Issues you can fix / Vendor issues / Performance). Each finding as a callout block with title and description, followed by a text line "Open Cowork and type:" and then the prompt as a code block. E.g.:
> Open Cowork and type:
> `/tech-diagnosis fix #61 on louloulollipop.com`
