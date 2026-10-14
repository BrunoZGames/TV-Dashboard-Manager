# Installation Guide — Reception TV Signage

How to install the signage app on the Windows 11 PC connected to the reception TV. You need an administrator account on that machine.

## What you end up with

- The TV loops videos stored in `C:\Signage\videos`.
- Staff on the office network open `http://<PC-NAME>:8080` to schedule image or PDF covers.
- Covers switch in and out automatically; when covers overlap, the most specific one wins.
- The app can recover from crashes, closed windows, reboots, and Windows Updates.

## 1. Install Node.js

1. Download the current LTS installer from <https://nodejs.org>.
2. Run it with the default options.
3. Verify the installation in a new terminal:

```bash
node --version
```

## 2. Copy the project

Copy the project folder to the TV PC, for example:

```text
C:\Apps\reception-signage
```

Avoid moving the folder after installing the scheduled tasks. From an administrator terminal:

```bash
cd C:\Apps\reception-signage
npm ci
```

## 3. First run and password setup

Start the app once by hand:

```bash
npm start
```

From another PC, open `http://<PC-NAME>:8080`. Complete the first-run password setup immediately. Use a strong password and share it only with the staff who manage signage. Never store the password in source control or paste it into support tickets.

If you need to reset the password, stop the app and update the local configuration file documented by the application. Restart the app and verify that the old password no longer works.

## 4. Add videos

Copy the looping videos into:

```text
C:\Signage\videos\
```

MP4 with H.264 is the safest format. WebM, MKV, MOV, and AVI may also be accepted depending on the installed media support. Files play in alphabetical order; prefix names with `01-`, `02-`, and so on when order matters.

Files can be added or removed without restarting the app.

## 5. Install the 24/7 services

From an administrator PowerShell in the project folder:

```powershell
powershell -ExecutionPolicy Bypass -File scripts\install-tasks.ps1
```

This installs the startup and watchdog tasks, disables unwanted sleep/display power timeouts, and adds a firewall rule for TCP port 8080 on trusted office networks.

| Item | Purpose |
|---|---|
| `ReceptionSignage` | Starts the app at logon |
| `ReceptionSignageWatchdog` | Restarts the app when its heartbeat becomes stale |
| Power settings | Prevents the signage display from sleeping |
| Firewall rule | Allows the admin page on the configured trusted networks |

## 6. Windows settings

1. Configure Windows auto-logon only if permitted by your organization's security policy.
2. Set Windows Update active hours so planned restarts occur outside reception hours.
3. Disable notifications and enable Do Not Disturb for the signage account.
4. Optionally rename the PC to a friendly name such as `tv-recepcion`.
5. Restrict the Windows account and network access to what is required for signage operation.

## 7. Verify

1. **Video loop:** reboot the PC and confirm the TV resumes playback without manual action.
2. **Crash recovery:** exit with `Ctrl+Alt+Q` or stop the process and confirm the watchdog restarts it.
3. **Remote upload:** upload a test image from another PC and confirm it appears within the scheduler interval.
4. **Nested covers:** schedule a long cover and a shorter cover inside its window; confirm the shorter cover takes priority and the long cover returns afterward.

## Daily use

1. Open the admin page and sign in.
2. Upload an image or PDF. For PDFs, only the first page is displayed.
3. Choose a preset or enter an exact start and end time.
4. Add an optional note and schedule the cover.
5. Reschedule or delete expired covers when needed.
6. Use the ES/EN toggle to change the interface language.

## Troubleshooting

| Symptom | Check |
|---|---|
| Black screen | Wait for the watchdog interval, then inspect the application and watchdog logs. |
| No videos | Confirm `C:\Signage\videos` contains supported files. |
| Admin page unreachable | Check the PC name, firewall rule, configured port, and whether the app is running. |
| Video skipped | Re-encode the file to MP4/H.264 and inspect the logs. |
| Cover not shown | Check its start/end window, the PC clock, file format, and overlap priority. |
| Forgot the password | Follow the local password-reset procedure, then restart the app. |
| Need maintenance access | Disable both scheduled tasks temporarily before performing extended maintenance. |

## Updating the app

```bash
cd C:\Apps\reception-signage
git pull
npm ci
```

Restart the app after updating. Content and settings stored under `C:\Signage\` should be backed up separately and must not be committed to the repository.
