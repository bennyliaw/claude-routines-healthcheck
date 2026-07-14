# Session recap — Slack integration for routines-daily-digest (2026-07-14)

## Context

`routines-daily-digest` (live at `https://claude.ai/code/routines/trig_01UEV8NahFKtnyKCFsHMwhrT`) currently delivers via `SendUserFile`, which only surfaces in a browser (see `docs/routine.md` for the full history — Telegram and Gmail were both dead ends). This session's goal: replace/supplement that with Slack, which looks like it actually supports sending, unlike Gmail.

## What was done this session

1. **Cleaned up Slack workspace confusion.** Ben's Slack login was tied to an old defunct company workspace ("DirectHome") via Google SSO. Left that workspace (did not delete it — other members are still in it). Created a brand-new personal workspace **"Ben Personal"** (`benpersonalnetwork.slack.com`) with a channel `#claude-routine-healthcheck`.
2. **Connected Slack in claude.ai Connectors.** Authorized the "Claude" Slack app against the **Ben Personal** workspace (not DirectHome).
3. **Fixed tool permission gate.** On the Connectors detail page for Slack, the **"Send message"** tool (under Write/delete tools) defaulted to "Needs approval" — which would stall an unattended routine forever since there's no human to approve. Changed it to **Always allow**. (Left other write tools — reactions, canvas, schedule message — on their defaults.)
4. **Probed tool availability via a reused throwaway routine** (`trig_0114rArfn8KCtDmptMsAQ5Js`, currently named `PROBE-slack-tool-check-delete-me` — this is one of the two leftover probes from the original investigation, repurposed rather than creating a third). Confirmed via `ListConnectors` + `ToolSearch` run inside a routine session:
   - Slack connector: **connected, authenticated, org-level**. `directoryUuid: 597f662f-36de-437e-836e-5a81013cbfbe`, `installedServerId: 6e588c20-18a6-41b2-ba93-a51d1f0237bf`.
   - **11 Slack tools available**, including `slack_send_message` — this is the key difference from Gmail, which had no send tool at all.
   - Caveat surfaced by the probe: `"Enabled in Current Chat: false"` — the connector is authenticated but not toggled on for a given session by default. This likely means the real routine will need an explicit `mcp_connections` entry (like `ok-ping`/Gmail probes needed) rather than relying on auto-attach.

## What's still pending / blocking

**We do not yet have the Slack connector's `url` field**, which is required for the `mcp_connections` array when updating the real routine (`connector_uuid` + `name` + `url`). The first probe reported `directoryUuid` and `installedServerId` but not `url`. A second probe was fired (session `cse_01LzGme3GDwWQ42FNC8uvecs`, updated trigger `trig_0114rArfn8KCtDmptMsAQ5Js` with prompt asking for the full raw `ListConnectors` JSON) requesting the complete JSON output to a file called `connectors_raw.txt`, sent via `SendUserFile`. **This had not landed in `~/Downloads/connectors_raw.txt` as of session end** — either still running, or check the run status directly at `https://claude.ai/code/routines/trig_0114rArfn8KCtDmptMsAQ5Js`.

## Natural next step

1. Check `~/Downloads/connectors_raw.txt` (or re-fire the probe if it never landed) to get the Slack connector's `url`.
2. Update the real routine (`trig_01UEV8NahFKtnyKCFsHMwhrT`, `routines-daily-digest`) via `RemoteTrigger action:"update"`:
   - Add `mcp_connections: [{"connector_uuid": "597f662f-36de-437e-836e-5a81013cbfbe", "name": "slack", "url": "<from connectors_raw.txt>"}]`
   - Update the prompt to call `slack_send_message` (post the digest to `#claude-routine-healthcheck` in the Ben Personal workspace) instead of/alongside `SendUserFile`
   - Double check whether `allowed_tools` needs the Slack tool names explicitly listed, or whether `preset:default` + the `mcp_connections` entry is sufficient (uncertain — verify empirically, same as the Gmail investigation pattern)
3. Manually run the updated routine (`RemoteTrigger action:"run"`) and confirm a message actually lands in the `#claude-routine-healthcheck` Slack channel.
4. Once verified working, update `docs/routine.md` and `README.md` in this repo to reflect the new Slack delivery (mirroring how the SendUserFile section is currently documented), and re-disable/rename `trig_0114rArfn8KCtDmptMsAQ5Js` back to a "probe, delete me" state (or note that it's earmarked for cleanup along with its sibling `trig_01KFfAQbwLGTdcaAdfCftPPP` — both still can't be deleted via API or web GUI, see `docs/routine.md`'s Cleanup note).
5. Commit and push these repo doc changes.

## Continuation prompt (paste after reboot)

```
Continue the Slack integration for routines-daily-digest. Read
~/dev/claude-routines-healthcheck/docs/session-recap-2026-07-14.md for full context.
Short version: Slack is connected (Ben Personal workspace, #claude-routine-healthcheck
channel), Send message permission is set to Always allow, and we confirmed
slack_send_message exists via a probe routine (trig_0114rArfn8KCtDmptMsAQ5Js).
We were waiting on a second probe run to get the connector's `url` field
(needed for mcp_connections) — check ~/Downloads/connectors_raw.txt first,
re-fire the probe if it's not there. Then update the real routine
(trig_01UEV8NahFKtnyKCFsHMwhrT, routines-daily-digest) to post the digest to
Slack via slack_send_message instead of/alongside SendUserFile, verify it
actually lands in the channel, then update docs/routine.md and README.md in
this repo and commit/push.
```
