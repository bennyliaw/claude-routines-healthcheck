# claude-routines-healthcheck

A daily Claude Routine that reports on the status of all Claude Routines (cloud-scheduled Claude Code agents) on this account — name, schedule, last run, next run, enabled state.

No external services, no tokens, no secrets. Delivery is entirely native to the Claude platform:

- **Data source:** `mcp__claude_code_remote__list_triggers` — a first-party MCP tool, automatically available to every Claude Routine via the built-in `Claude_Code_Remote` connector. Returns live data on every routine on the account.
- **Delivery:** `SendUserFile` — a first-party tool that delivers a file back to the account owner. In practice this lands in `~/Downloads/` on the machine where you're logged into claude.ai in a browser. **Known limitation:** this only surfaces when checking the browser client — it does not currently push to the native macOS/Android/iOS apps or trigger any notification. Confirmed by testing (see `docs/routine.md`).

## Why this exists

Built after an extended feasibility investigation (see `docs/routine.md` for the full trail) into whether a cloud Claude Routine can:
1. Query the status of other routines on the account — **yes**, via `mcp__claude_code_remote__list_triggers` (an MCP tool, not raw `curl`/Bash, which is blocked by the environment's network proxy).
2. Push a notification externally (Telegram, WhatsApp, Gmail) — **no**, for every channel tested:
   - Telegram: raw `curl` to `api.telegram.org` blocked by network proxy (`403`), and the routine's own model refuses "probe external status + report to third party" as unauthorized reconnaissance.
   - Gmail: connector exists but exposes no send-email tool, only draft creation — and even that tool wasn't available in-session (`enabledInChat: false` on the connector).
   - `SendUserFile` is the only delivery channel that actually worked end-to-end, with the caveat above.

## The routine

Live at `https://claude.ai/code/routines/trig_01UEV8NahFKtnyKCFsHMwhrT` — see `docs/routine.md` for the exact prompt and creation config, since the routine's logic lives on claude.ai, not as code in this repo.

Schedule: daily at 21:00 UTC (5am Asia/Singapore).

**Status: verified working.** First manual run (2026-07-03) produced a correct digest covering all 6 routines on the account, including correctly flagging a disabled routine (`Algora bounty watch`, `auto_disabled_init_failed`) as needing attention. First scheduled run: 2026-07-03 21:00 UTC.
