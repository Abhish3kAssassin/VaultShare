# Vault Share

### Local-first file sharing with receiver approval

Vault Share is a LAN file-sharing application built with **React and Node.js**. Users can create accounts, see other active users, and send files or folders. The receiver must accept each request before downloading.

One computer runs the host. Everyone else connects through a browser using that computer's LAN address. The host serves the React interface and API together over **HTTP on port 8080**. No database or cloud storage service is required.

![Vault Share dashboard](previews/VaultShare-React-dashboard.png)

## Features

- **Registration and login:** Create an account with a username, display name, and password. Passwords are hashed with scrypt.
- **Online presence:** See other active users connected to the same Vault Share host. The dashboard refreshes every five seconds.
- **File and folder sharing:** Upload individual files, multiple files, or a folder. Folders and multiple files download as ZIP archives with relative paths preserved.
- **Receiver approval:** A request starts as pending. Only its receiver can accept or decline it; downloading is blocked until acceptance.
- **Encrypted storage:** File payloads are encrypted on the host using AES-256-GCM.
- **Integrity verification:** The host authenticates encrypted payloads and verifies SHA-256 before serving downloads.
- **Private access:** Only the selected receiver can download. Transfer records and logs are visible to the participants.
- **Expiry windows:** Choose one hour, six hours, 24 hours, three days, or seven days.
- **Revocation:** The sender can revoke pending or accepted transfers to block future downloads.
- **Transfer logs:** Track requests, approvals, declines, revocations, expiry, and verified downloads served.
- **Responsive interface:** Use the dashboard on desktop and mobile browsers.
- **No database:** Accounts, transfer metadata, and logs use authenticated JSON files. Browser sessions and presence are held in memory.

> **HTTP security:** File contents are encrypted in storage, but passwords and files are not encrypted during network transmission. The host holds the decryption key; this is not end-to-end encryption. Use a trusted host and LAN. See [SECURITY.md](SECURITY.md).

## Technology stack

| Component | Technology |
| --- | --- |
| Interface | React, CSS, and inline SVG icons |
| Frontend build and development | Vite |
| Host and API | Node.js and Express |
| Upload handling | Multer with aggregate input limits |
| Folder archives | JSZip |
| Encryption and hashing | Node.js crypto: AES-256-GCM, SHA-256, scrypt, and HMAC-SHA-256 |
| Persistent storage | Local signed JSON metadata and encrypted file payloads |
| Tests | Node.js test runner, Supertest, and Playwright |

## Requirements

**On the host computer:**

- Node.js **22.12 or newer**, with npm installed.
- Internet access for the initial dependency installation.
- Enough disk space and RAM for transfers, compression, encryption, and downloads.
- A LAN connection that allows other devices to reach the host.

**On other devices:** A browser and access to the same LAN. They do not need Node.js, Python, or a separate app installation.

Download Node.js from [nodejs.org](https://nodejs.org/). Check your installed version with:

```bash
node --version
npm --version
```

Fresh installations do not require Python. Python 3 is needed only for the optional migration from the previous edition.

## Installation and startup

1. Extract `VaultShare-React.zip` and keep the complete project folder together.
2. Stop any previous Python or Node.js Vault Share host with **Ctrl+C**, especially if it uses port 8080.
3. Open a terminal in the extracted `VaultShare-React` folder.
4. Install dependencies and start the host:

```bash
npm ci
npm start
```

For example, on macOS:

```bash
cd ~/Downloads/VaultShare-React
npm ci
npm start
```

Adjust the folder path if you extracted the project elsewhere. On Windows, open Command Prompt or PowerShell in the extracted folder and use the same npm commands.

`npm start` builds the React interface, starts the HTTP server, checks that it responds, and opens the host's browser. It binds to `0.0.0.0:8080` so other LAN devices can connect. Subsequent starts need only `npm start`; after dependencies are installed, the app can run without internet.

Keep the terminal open and the host awake while sharing. Stop the host with **Ctrl+C**.

### Optional launchers

- **macOS:** Run `bash Start-Mac.command` from the project folder, or double-click the launcher if permitted by macOS.
- **Windows:** Double-click `Start-Windows.bat`.

These launchers install dependencies if `node_modules` is missing, then start the application. Node.js must already be installed.

## Connect from other devices

The host terminal prints addresses similar to:

```text
On this computer: http://127.0.0.1:8080
On Windows / other LAN devices: http://192.168.1.42:8080 (en0)
```

| Device | Address to open |
| --- | --- |
| Computer running the host | `http://127.0.0.1:8080` |
| Windows computer, phone, or another LAN device | The host's printed LAN URL, such as `http://192.168.1.42:8080` |

**Replace the example IP with the host's actual address.** Everyone must connect to the same host to share accounts, presence, and transfers.

`localhost` and `127.0.0.1` refer to the device using them. Opening `localhost` on Windows will not connect to a host running on your Mac. `0.0.0.0` is the server's bind setting, not a browser address. Use **HTTP**, the printed LAN IP, and the correct port.

## Sharing workflow

1. Open the host URL, create an account, and sign in. Registration requires a password of at least 12 characters.
2. Ask the receiver to open the same host URL and sign in. They appear under **People nearby**.
3. Select **New transfer**, choose the receiver, and select files or a folder.
4. Choose an expiry window, optionally add a message, and select **Send request**.
5. The receiver reviews the incoming request and chooses **Accept** or **Decline**.
6. After acceptance, the receiver selects **Download**. The host verifies integrity before sending the payload.
7. Use **Sent files** to monitor transfers or revoke access, and **Activity log** to review events.

Expiry starts when the request is sent, including time spent awaiting approval. Closed or disconnected browsers disappear from online presence after about 35 seconds. Declined, revoked, expired, and corrupt payloads are removed from active storage.

Revocation and expiry cannot remove copies already downloaded. Empty files are supported; empty directories are not preserved. Logs record verified downloads served, not proof that the receiver saved a file.

## Configuration

Pass host options after `--`:

```bash
# Use a different port
npm start -- --port 8081

# Use a different data folder
npm start -- --data-dir "/absolute/path/to/vault-data"

# Start without opening a browser
npm start -- --no-browser

# Set a 64 MiB input limit and 1 GiB storage quota
npm start -- --max-upload-mb 64 --quota-mb 1024

# Explicitly bind to all IPv4 interfaces
npm start -- --host 0.0.0.0
```

Use a path appropriate to your host's operating system. Changing the port requires updating the URL on every device. A custom data folder starts a separate vault unless it already contains compatible data.

| Setting | Default |
| --- | --- |
| HTTP port | 8080 |
| Bind address | `0.0.0.0` |
| Data folder | `data` inside the project |
| Total file input per transfer | 128 MiB |
| Files per transfer | 2,000 |
| Concurrent uploads | 2 |
| Active payload storage | 2 GiB |
| Active payload storage per sender | Up to 512 MiB, within the host quota |
| Active transfers per sender | 100 |
| Active transfers per host | 500 |

The configurable input limit is 1–256 MiB. The storage quota must cover at least one transfer at the configured input limit. Uploads and downloads use memory buffers, so this edition is intended for bounded transfers rather than very large files.

## Troubleshooting LAN access

### Another device says “refused to connect”

1. Confirm the host terminal shows **VAULT SHARE REACT IS READY**. Follow any startup error instead of assuming the server is running.
2. Use the host's current Wi-Fi/Ethernet IPv4 address and printed port. Prefer the physical network address over a VPN or virtual interface.
3. Keep the host terminal open and the computer awake.
4. Connect both devices to a LAN that permits device-to-device traffic. Guest Wi-Fi and client isolation can block access.
5. Allow incoming connections for the Node.js host through the host computer's firewall. On a Mac, check System Settings → Network → Firewall → Options. Permissions previously granted to Python do not apply to Node.js.
6. Check whether a VPN is routing local traffic away from the LAN.

From Windows PowerShell, replace the example IP and test:

```powershell
Test-NetConnection 192.168.1.42 -Port 8080
```

If `TcpTestSucceeded` is `False`, Windows cannot establish a TCP connection to that host and port. Check the address, host process, firewall, and network isolation.

From a computer with Node.js and this project installed, test the app's HTTP endpoint:

```bash
npm run check -- http://192.168.1.42:8080
```

A successful check confirms that the address serves Vault Share React. The endpoint is also available at `/healthz`.

### Other startup problems

| Message or symptom | Action |
| --- | --- |
| `node` or `npm` is not found | Install Node.js and reopen the terminal. |
| Unsupported Node.js version | Update to Node.js 22.12 or newer. |
| Port is occupied | Stop the previous host or use another port. |
| Another host is using the data folder | Stop that process; use one host per data folder. |
| React UI is not built | Start with `npm start`, which builds and serves the UI. |
| Session verification failed | Reload the page and sign in again. |
| Missing keys or invalid metadata signature | Restore a consistent backup; do not delete or regenerate the original key file. |

## Migrate from the Python edition

Migration is optional. New users can start with an empty vault.

To retain existing accounts, compatible password hashes, logs, transfer records, and active encrypted files:

1. Stop the Python host.
2. Back up its complete `data` folder.
3. Before the first React launch, run this command from the new project folder:

```bash
python3 scripts/migrate-python.py --source "/path/to/old/VaultShare/data"
```

Then install dependencies and start normally:

```bash
npm ci
npm start
```

The migration verifies the old metadata signature, copies the keys and active ciphertext, and creates a compatible data folder. The source remains unchanged. Accepted transfers stay accepted, pending transfers still require approval, and expiry remains enforced. Browser sessions are not migrated; users sign in with their existing credentials.

**The destination must not already exist.** If React has already created its default `data` folder, import into a new destination:

```bash
python3 scripts/migrate-python.py --source "/path/to/old/data" --target "/path/to/new/imported-data"
npm start -- --data-dir "/path/to/new/imported-data"
```

Do not directly copy a Python data folder into React. Do not delete `keys.bin` to resolve a migration or startup error. The migration command uses Python 3's standard library; the running React edition does not require Python.

## Project structure

| Path | Purpose |
| --- | --- |
| `src/App.jsx` | React state, authentication, polling, and transfer actions |
| `src/api.js` | API requests, uploads, and downloads |
| `src/components/` | Login, dashboard, transfer cards, icons, and dialogs |
| `src/style.css` | Responsive interface styles |
| `server/app.mjs` | Accounts, presence, uploads, approval, access control, and downloads |
| `server/store.mjs` | Authenticated JSON storage, encryption, integrity, and expiry |
| `server/security.mjs` | Password hashing and upload-path validation |
| `server/index.mjs` | HTTP startup, configuration, and host locking |
| `server/network.mjs` | LAN address discovery and HTTP health checks |
| `scripts/` | Launch, development, diagnostics, and migration utilities |
| `tests/` | API, launcher, and browser tests |
| `public/` | Static public assets |
| `dist/` | Built React interface |
| `previews/` | Screenshots of the application |
| `data/` | Private runtime data, created on first launch |
| `SECURITY.md` | Security model and limitations |

## Development and testing

Start the development environment:

```bash
npm run dev
```

Vite serves the interface on **HTTP port 3001** with hot reload and proxies API requests to the Node.js host on **port 8080**. For normal LAN sharing, use `npm start` so the interface and API share one port.

| Command | Purpose |
| --- | --- |
| `npm start` | Build React and start the complete host |
| `npm run build` | Build the interface into `dist` |
| `npm run server` | Start the backend using the existing build |
| `npm run dev` | Start the development interface and API |
| `npm test` | Run API and launcher tests |
| `npm run test:browser` | Run the browser workflow checks |
| `npm run check -- http://HOST:PORT` | Check a running host |
| `npm run format` | Format the supported project files |

Run the checks with:

```bash
npm run build
npm test
npx playwright install chromium
npm run test:browser
```

API and launcher tests cover authorization, approvals, integrity failures, expiry, persistence, participant privacy, upload limits, migration, LAN binding, and startup failures. Browser tests use two isolated sessions to check presence, folder sharing, approval, verified ZIP downloads, revocation, logs, logout, session recovery, and desktop/mobile layouts.

Tests use temporary accounts and data. The migration test is skipped if Python 3 is unavailable. Set `VS_BROWSER_PATH` to an installed Chromium executable if necessary.

## Storage and backups

The default `data` folder contains:

| Item | Contents |
| --- | --- |
| `state.json` | Accounts, transfer metadata, and logs, authenticated with HMAC-SHA-256 |
| `keys.bin` | Encryption and metadata-signing keys |
| `blobs/` | AES-256-GCM encrypted file payloads |

Stop the host before backing up the **entire data folder**. Restore the metadata, keys, and payloads together. Losing `keys.bin` makes encrypted payloads unrecoverable. Do not commit `data` or distribute it with the application.

Sessions are stored in memory, so a host restart requires users to sign in again. File names, notes, account records, and logs are authenticated but not encrypted. The server verifies every download; the browser additionally verifies SHA-256 when Web Crypto is available on its origin. Some HTTP LAN origins disable that browser capability.

The current edition has no password recovery or administrator interface. For implementation details and security boundaries, read [SECURITY.md](SECURITY.md).
