# Tech diagnosis setup

Runs on first use (no config found) or when the operator asks to update their setup. Ask one connector at a time — send a message, wait for the reply, then move to the next. Never present multiple connectors at once.

---

## Intro message

Send this first, then stop and wait:

**First run:**
> "Before we dive in, I have a couple of quick optional connections that make the diagnosis more precise. Each one is optional and only takes a moment — let's go through them one at a time."

**Update:**
> "Sure — let's update your setup. I'll go through each one and you can change any of your previous choices."

---

## Step 1 — Platform connector (Shopify only)

**Only relevant if the domain's platform is Shopify.** Skip entirely for other platforms.

Check if the Shopify MCP tools are available in the current session (they appear as Shopify tools in the tool list). If they are available, save `status: connected` to config and skip this step — don't ask. If they are not available, send this message and **wait for reply before continuing:**

> "**Shopify** — connecting your store lets me check your actual settings when diagnosing an issue. For example, if a product variant looks like the cause, I can confirm whether it's really out of stock rather than just guessing. Connect, skip for now, or never ask?"

**Outcomes:**
- **Connect** → call `suggest_connectors` with UUID `80917cb7-3071-4fca-b053-a4262d356c60` and keyword `store`. Ask: "Once connected, let me know — or say 'skip for now' or 'never ask'." Wait for reply. Save `status: connected` on success.
- **Skip for now** → save nothing. Will ask again next session.
- **Never ask** → save `status: skipped`. Won't ask again.

---

## Step 2 — Code repository

Send this message and **wait for reply before continuing:**

> "**Code repository** — if your store's theme or code lives in a Git repo (GitHub, GitLab, Bitbucket), connecting it lets me point to the specific file and line that needs changing instead of giving general advice. Connect, skip for now, or never ask?"

**Outcomes:**
- **Connect** → two steps are required:
  1. Check if GitHub Integration tools are available. If they are not, setup the GitHub Integration tools.
  2. Then tell them: "You also need to install the GitHub app to grant access to your repo — visit https://github.com/apps/claude-github-mcp-connector and install it on the account or organisation that owns your store's repo."
  3. Once they confirm the app is installed, ask: "What's the repo? (e.g. `mycompany/my-store`)" — **this is required, not optional**. Wait for reply. Save `status: connected` and `repo` to config.
- **Skip for now** → save nothing. Will ask again next session.
- **Never ask** → save `status: skipped`. Won't ask again.

---

## Step 3 — Claude in Chrome

First, silently check if Claude in Chrome tools are available. If they are, skip this step entirely — Chrome is already active and will be used automatically. If not, send this message and **wait for reply before continuing:**

> "**Claude in Chrome** — enables me to open your live pages in a real browser and run performance tests directly. This gives much more accurate speed diagnosis, especially on mobile, compared to working from data alone. You can turn it on in your Claude settings under Customize. Enable it and let me know, or say 'skip for now' or 'never ask'."

**Outcomes:**
- **Enabled / let me know** → verify Chrome tools are now available. If yes, confirm. If still not active, let them know to restart and try again. Save nothing (detected at runtime).
- **Skip for now** → save nothing. Will ask again next session.
- **Never ask / security reason** → save `chrome_inspection: skipped`. Won't surface this again.

---

## Confirm and proceed

Once all steps are answered, **always write `$HOME/.tech-diagnosis-config.json` using the shell tool — even if every answer was "skip for now".** The file must exist after setup so future sessions can read it. Use the Python merge pattern from SKILL.md → Configuration. Confirm in one sentence:

> "All set — [one-line summary, e.g. 'Shopify and Chrome are connected, code repo skipped for now.']. Let's get into the diagnosis."

Then proceed into the skill — if a domain was already provided, continue with Setup from where it left off.
