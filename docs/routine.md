# Routine config: routines-daily-digest

This is the source of truth for the live routine at `https://claude.ai/code/routines/trig_01UEV8NahFKtnyKCFsHMwhrT`. The routine's actual logic lives on claude.ai, not as code in this repo — this file exists so it's reproducible and versioned.

## Schedule

`0 21 * * *` UTC = daily 5am Asia/Singapore.

## Model

`claude-haiku-4-5-20251001` — cheap, matches the tier of the other lightweight routines on this account (`ok-ping`, `diffusiongemma-availability-check`).

## Repo attached

`https://github.com/bennyliaw/claude-routines-healthcheck` (this repo).

## Allowed tools

`preset:default`, `SendUserFile`

(`preset:default` is required for the `mcp__claude_code_remote__*` MCP tools to be available — confirmed via probing that these are NOT available under a narrow `allowed_tools: ["Bash"]` config alone; they need the default preset.)

## Exact prompt

```
Call mcp__claude_code_remote__list_triggers to get the full list of Claude Routines on this account.

For each routine, extract: name, cron_expression (converted to a human-readable schedule), enabled state, last_fired_at, next_run_at.

Write a plain-text digest to a file called routines_digest.txt with one block per routine, most-recently-fired first. Include a one-line summary at the top: total routine count, how many are enabled, and whether any show a next_run_at more than 24 hours in the past relative to their cron schedule (a sign they may be stuck/failing).

Send the file to the user via SendUserFile.

Do not modify, create, update, or delete any triggers. This is a read-only reporting task.
```

## Full create request body

```json
{
  "name": "routines-daily-digest",
  "cron_expression": "0 21 * * *",
  "enabled": true,
  "job_config": {
    "ccr": {
      "environment_id": "env_01W9j1MUXSDqJcTZDBWVt4Lt",
      "session_context": {
        "model": "claude-haiku-4-5-20251001",
        "sources": [
          {"git_repository": {"url": "https://github.com/bennyliaw/claude-routines-healthcheck"}}
        ],
        "allowed_tools": ["preset:default", "SendUserFile"]
      },
      "events": [
        {"data": {
          "uuid": "<generated fresh per creation>",
          "session_id": "",
          "type": "user",
          "parent_tool_use_id": null,
          "message": {
            "role": "user",
            "content": "<see Exact prompt above>"
          }
        }}
      ]
    }
  }
}
```

## Known limitation

`SendUserFile` delivers to `~/Downloads/` but **only surfaces when accessing claude.ai via a browser** — confirmed through direct testing. It does not push to the native macOS, Android, or iOS Claude apps, and does not trigger any OS-level notification. This was the best available option after testing:

- **Telegram** — blocked at the network-proxy level for raw `curl` (`403`), and the routine's own model refused the "probe status + report to external channel" pattern as unauthorized reconnaissance, independent of the network block.
- **Gmail** — connector exists (`installedServerId: a5dee38f-72d1-4c6d-b71a-64bc927cf9bf`) but its tool surface only supports draft creation, not sending. Even the draft tool wasn't available in-session at time of testing (`enabledInChat: false` on the connector as reported by `ListConnectors`).
- **WhatsApp** — assumed similar limitation to Telegram (network egress), not separately tested.

If claude.ai adds native-app or push delivery for `SendUserFile`, or exposes a working send-capable connector, this routine should be updated to use it — check back periodically.

## Cleanup note

Two throwaway probe routines were created during this investigation and left **disabled** (not deleted — the `delete_trigger` MCP tool call did not take effect when tested):
- `PROBE-delete-me-routines-api-test` (`trig_0114rArfn8KCtDmptMsAQ5Js`)
- `PROBE2-delete-me-routines-api-test` (`trig_01KFfAQbwLGTdcaAdfCftPPP`)

Both are scheduled for Jan 2027 and disabled, so they're inert. To be deleted manually at `claude.ai/code/routines`.

## Verification (2026-07-03)

Manual run via `RemoteTrigger action:"run"` on `trig_01UEV8NahFKtnyKCFsHMwhrT` produced a correct `routines_digest.txt`, delivered via `SendUserFile` to `~/Downloads/`. Output covered all 6 routines then on the account, correctly computed human-readable schedules, and correctly flagged `Algora bounty watch — AB target orgs` as disabled (`auto_disabled_init_failed`) needing review. Confirms the full pipeline (`list_triggers` → format → `SendUserFile`) works end-to-end.
