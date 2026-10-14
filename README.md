# Reception TV Signage

Digital signage for a reception TV running Windows 11. The app loops videos from `C:\Signage\videos` and displays scheduled image or PDF covers uploaded by staff through a LAN web page at `http://<PC-NAME>:8080`.

- **Install on the TV PC:** [docs/INSTALL.md](docs/INSTALL.md)
- **Design spec:** [docs/specs/2026-07-27-reception-signage-design.md](docs/specs/2026-07-27-reception-signage-design.md)
- **Implementation plan:** [docs/specs/2026-07-27-reception-signage.md](docs/specs/2026-07-27-reception-signage.md)

## Development

```bash
npm ci
npm test
set SIGNAGE_DATA_DIR=C:\tmp\signage
set SIGNAGE_WINDOWED=1
npm start
```

Set the administrator password during first-run setup. Do not commit passwords, session secrets, uploaded media, logs, or local configuration files.

Maintenance exit on the TV: `Ctrl+Alt+Q`.
