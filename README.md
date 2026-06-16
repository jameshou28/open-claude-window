# Open Claude Window

A tiny GitHub Actions workflow that opens your Claude **usage window** at a fixed time each day by sending one throwaway prompt to Claude Code.

## Why this exists

Claude Pro and Max usage limits reset on a rolling **5-hour window** that starts the moment you send your first message in a session — not on a fixed wall-clock schedule. If you want that window to be open and ready at a predictable time (say, the start of your workday), you need *something* to send a prompt at that time.

This workflow does exactly that: every morning it sends a single `"hi"` to Claude, which starts the clock so your 5-hour window aligns with when you actually want to work.

## How it works

The workflow lives at [`.github/workflows/start-claude.yml`](.github/workflows/start-claude.yml).

1. **Scheduled trigger** — a cron entry fires the job once a day.
2. **Fresh runner** — GitHub spins up a clean `ubuntu-latest` machine.
3. **Install Claude Code** — `npm install -g @anthropic-ai/claude-code`. The runner is ephemeral, so this runs on every invocation (~15–30s).
4. **Send the prompt** — `claude -p "hi"` runs Claude Code in headless mode. That one prompt starts your 5-hour window.

You can also trigger it manually from the **Actions** tab via `workflow_dispatch`.

## Schedule

```yaml
- cron: "0 10 * * *"   # 10:00 UTC = 6:00 AM EDT (UTC-4)
```

Cron always runs in **UTC** — it has no timezone awareness. The current setting targets **6:00 AM EDT**.

To change the time, convert your desired local time to UTC and edit the cron.

## Setup

This requires a **Claude Pro or Max subscription** — authentication is via a subscription OAuth token, not an API key. (An API key would bill per token and would *not* trigger your subscription window.)

1. **Generate a token** on your own machine:
   ```bash
   claude setup-token
   ```
   It prints a token like `sk-ant-oat01-…`, valid for **one year**. It's shown only once — save it.

2. **Add it as a repo secret:** GitHub repo → **Settings → Secrets and variables → Actions → New repository secret**. Name it `CLAUDE_CODE_OAUTH_TOKEN` and paste the token.

3. **Adjust the cron** to your preferred time (in UTC) and commit.

## Things to watch

- **Cron is UTC** and doesn't follow daylight saving — see the note above.
- **GitHub doesn't run scheduled jobs exactly on time.** They can lag several minutes, sometimes more under load. Your window opens *roughly*, not precisely, at the set time.
- **GitHub disables scheduled workflows after 60 days of repo inactivity.** Push a commit occasionally, or re-enable it from the Actions tab.
- **Regenerate the token once a year** when it expires.
- **The `"hi"` prompt uses a small slice of your quota** — that's the cost of opening the window when you want.
