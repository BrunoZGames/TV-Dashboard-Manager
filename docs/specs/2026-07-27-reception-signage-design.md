# Reception TV Signage — Design

**Date:** 2026-07-27  
**Status:** Approved

## Purpose

A Windows 11 PC drives a reception TV. By default it loops a folder of videos. Staff can upload an image or PDF cover and schedule it for a time window. Covers may nest: a short cover takes priority during its window, then the longer cover returns, and finally the display returns to the video loop.

## Decisions

| Topic | Decision |
|---|---|
| Upload access | Web page served over the trusted office LAN |
| Video source | Local folder of video files with sound |
| Overlap rule | Shortest total duration wins; ties use the newest upload |
| Access control | One administrator password with a browser session cookie |
| PDF handling | First page converted to PNG during upload |
| Expired covers | Archived separately and available for rescheduling or deletion |
| Admin UI | Responsive, bilingual ES/EN interface |
| Architecture | Single Electron app |

## Architecture

1. **Display window** — frameless, always-on-top fullscreen kiosk window. Renders either the video loop or the active cover.
2. **Embedded web server** — Express server bound to the configured LAN port. Serves the admin page, uploads, and schedule API.
3. **Scheduler** — evaluates the active cover every five seconds and updates the display.

## Data layout

```text
C:\Signage\
  videos\
  covers\
  covers\expired\
  schedule.json
  config.json
  logs\
```

Passwords, session secrets, uploaded media, and logs are local runtime data and must never be committed.

## Scheduling behavior

- Active windows use `[start, end)` semantics.
- If no cover is active, videos play in alphabetical order.
- If multiple covers are active, the shortest duration wins; newest upload breaks ties.
- Expired covers are removed from the active schedule and archived.
- Video folders are rescanned between videos so content changes do not require a restart.
- Covers fit within the screen with black bars and are never stretched or cropped.

## Reliability

- Windows scheduled task starts the app at logon.
- A separate watchdog checks a heartbeat and relaunches stale processes.
- Renderer failures recreate the display window.
- Unplayable videos are skipped and logged.
- A corrupted schedule falls back to video-only mode.
- Logs are rotated and old logs are removed.

## Admin page

The single-page admin UI provides:

- Login and session management.
- Current display status.
- Drag-and-drop image/PDF upload.
- Presets for common time ranges and custom date-time selection.
- Scheduled and expired cover lists.
- Edit, reschedule, and delete actions.
- ES/EN language toggle persisted per browser.

## Security and deployment notes

- Bind the admin service only to the office network as appropriate for the deployment.
- Use a strong administrator password and never commit it.
- Keep the TV PC patched and use a restricted Windows account where practical.
- Limit the firewall rule to trusted Domain/Private networks.
- Do not expose the admin page directly to the public internet.

## Out of scope

Multiple screens, video covers, per-user accounts, recurring schedules, remote internet access, and analytics are intentionally excluded from the initial version.
