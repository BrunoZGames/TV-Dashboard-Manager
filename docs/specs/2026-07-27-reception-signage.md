# Reception TV Signage Implementation Plan

> This is an implementation guide. Validate security, dependency versions, and Windows deployment behavior before production use.

**Goal:** Build a single Electron app that loops reception videos, displays scheduled image/PDF covers uploaded through a LAN web page, and returns automatically to the video loop.

**Tech stack:** Electron, Express, multer, PDF-to-PNG conversion, Vitest, Supertest, vanilla HTML/CSS/JS, and PowerShell deployment scripts.

## File structure

```text
reception-signage/
  package.json
  .gitignore
  electron/
    main.js
    paths.js
    store.js
    scheduler.js
    playlist.js
    logger.js
    server.js
    convert.js
    preload-display.js
  display/
    index.html
    display.js
    display.css
  admin/
    index.html
    admin.css
    admin.js
    i18n.js
  scripts/
    launch.cmd
    watchdog.ps1
    install-tasks.ps1
  tests/
  docs/
```

## Implementation tasks

### 1. Scaffold

- Create `package.json` with Electron start and Vitest test scripts.
- Add Express, multer, PDF conversion, Electron, Vitest, and Supertest dependencies.
- Ignore `node_modules`, logs, runtime data, and local configuration.

### 2. Data paths and store

Implement testable storage helpers that:

- Resolve `SIGNAGE_DATA_DIR` for development and tests.
- Create videos, covers, expired, and logs directories.
- Create configuration with a generated session secret.
- Hash a password only during explicit setup or password change.
- Load corrupt or missing schedules safely.
- Save schedules atomically through a temporary file and rename.
- Archive and restore covers safely.

### 3. Scheduler

Implement pure functions:

```js
function selectActive(covers, now) {
  const active = covers.filter(c => c.start <= now && now < c.end);
  if (active.length === 0) return null;
  active.sort((a, b) =>
    (a.end - a.start) - (b.end - b.start) ||
    b.uploadedAt - a.uploadedAt
  );
  return active[0];
}

function findExpired(covers, now) {
  return covers.filter(c => c.end <= now);
}
```

Test empty schedules, overlap priority, equal durations, and boundary times.

### 4. Playlist and logging

- Rescan the video directory between playback items.
- Filter configured extensions and sort case-insensitively.
- Cycle deterministically and handle an empty directory.
- Write daily logs without throwing from the logger.
- Remove logs older than the configured retention period.

### 5. Upload and API

- Accept only configured image and PDF formats.
- Convert the first PDF page to PNG.
- Validate required fields, date ranges, and file sizes.
- Protect API routes with secure, signed, HTTP-only cookies.
- Add login throttling and clear error responses.
- Implement schedule CRUD, status, expired-cover restore, and permanent deletion.
- Ensure temporary uploads are removed on success and failure.

### 6. Display renderer

- Render video with sound and automatic sequencing.
- Switch to covers with a short fade.
- Resume with the next video after a cover ends.
- Show a useful standby card when no videos exist.
- Retry empty playlists periodically.
- Hide the cursor and keep the display fit-to-screen.

### 7. Admin UI

Build a responsive, modern, framework-free page with:

- Login card.
- Current status banner.
- Upload drop zone and preview.
- Presets and custom schedule fields.
- Scheduled/expired cover lists.
- Edit, delete, reschedule, and overlap indicators.
- ES/EN translations stored in a dedicated dictionary module.

### 8. Electron main process

- Ensure directories and load configuration.
- Start the server on the configured port and trusted interface.
- Create the kiosk window with a preload bridge.
- Run the five-second scheduler tick.
- Archive expired covers and send display updates only when content changes.
- Recreate the window after renderer failures.
- Maintain a heartbeat and perform a controlled nightly restart if required.
- Register a documented maintenance shortcut.
- Log uncaught errors and handle them safely.

### 9. Windows scripts

- Add a launch command that starts the app from its installed directory.
- Add a watchdog that checks the heartbeat and only terminates the intended app process.
- Add an installation script for scheduled tasks, power settings, and a narrowly scoped firewall rule.
- Print reminders for manual Windows configuration rather than silently weakening security.

### 10. Tests and documentation

- Unit-test scheduler, store, playlist, upload validation, and authentication.
- Add API tests with temporary data directories.
- Run the complete test suite before deployment.
- Perform manual reboot, crash-recovery, remote-upload, and nested-cover acceptance tests.
- Keep deployment documentation free of passwords, tokens, private addresses, and user-specific secrets.

## Data contract

A schedule entry should contain an opaque ID, stored filename, original filename, start/end timestamps, optional note, and upload timestamp. Configuration should contain the port, password hash, generated session secret, and allowed video extensions. Runtime secrets and content belong outside Git.

## Acceptance checklist

- Videos loop in alphabetical order.
- Uploading a valid image displays it during its scheduled window.
- PDFs display their first page.
- Nested covers follow the most-specific-wins rule.
- Expired covers can be restored or deleted.
- The service recovers from renderer crashes and process restarts.
- The admin page is unavailable from untrusted networks.
- No credentials or runtime data are present in the repository.
