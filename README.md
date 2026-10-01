# Vault Share

Vault Share is a local-first file-sharing application built with **React and Node.js**. Users on the same LAN can create accounts, see other active users, and share files or folders. Every transfer requires the receiver's approval before downloading.

The application uses encrypted file storage and authenticated JSON metadata, with **no database**.

![Vault Share dashboard](previews/VaultShare-React-dashboard.png)

## Features

- User registration and login with scrypt password hashing.
- Online user list that refreshes every five seconds.
- File and folder sharing; folders and multiple files download as ZIP archives.
- Receiver approval or rejection for each transfer request.
- AES-256-GCM encryption for stored file payloads.
- SHA-256 integrity verification before downloads are served.
- Receiver-only downloads and participant-only transfer records and logs.
- Configurable expiry windows and sender-controlled revocation.
- Responsive interface for desktop and mobile browsers.
- Local file storage without a database or cloud storage service.

## Tech Stack

| Layer | Technology |
| --- | --- |
| Frontend | React, Vite, CSS |
| Backend | Node.js, Express |
| Uploads and archives | Multer, JSZip |
| Security | Node.js crypto, AES-256-GCM, SHA-256, scrypt, HMAC-SHA-256 |
| Storage | Authenticated JSON metadata and encrypted payload files |
| Testing | Node.js test runner, Supertest, Playwright |

## Installation

The host requires **Node.js 22.12 or newer** and npm. Other devices only need a browser and access to the same LAN.

Extract the project, open a terminal in the `VaultShare-React` folder, and run:

```bash
npm ci
npm start
```

`npm start` builds the React interface and serves the interface and API together over **HTTP on port 8080**. Keep the terminal open while sharing. Stop the host with **Ctrl+C**.

## LAN Access

On the host computer, open:

```text
http://127.0.0.1:8080
```

On other devices, open the host's LAN URL printed in the terminal. For example:

```text
http://192.168.1.42:8080
```

Replace the example IP with the host's actual address. All users must connect to the same host. The host's firewall and network must allow incoming LAN connections.

## Usage

1. Create an account and sign in.
2. Choose an online receiver under **People nearby** or select **New transfer**.
3. Select files or a folder, choose an expiry window, and send the request.
4. The receiver reviews the request and selects **Accept** or **Decline**.
5. After acceptance, the receiver can download the verified payload.
6. Review events in **Activity log** or revoke access from **Sent files**.

Expiry starts when a request is sent. Expired, declined, revoked, and corrupt payloads are removed from active storage. Revocation and expiry block future downloads; they cannot remove copies already downloaded.

## Commands

| Command | Purpose |
| --- | --- |
| `npm start` | Build and run the complete application |
| `npm run dev` | Run the development interface on port 3001 and API on port 8080 |
| `npm run build` | Build the React interface |
| `npm test` | Run API and launcher tests |
| `npm run test:browser` | Run browser workflow tests |

To change the host port:

```bash
npm start -- --port 8081
```

Use the new port in every device's URL. For browser tests, build the interface and install Chromium first:

```bash
npm run build
npx playwright install chromium
npm run test:browser
```

## Project Structure

| Directory | Purpose |
| --- | --- |
| `src/` | React components, API client, and styles |
| `server/` | Authentication, transfers, encryption, access control, and HTTP hosting |
| `scripts/` | Application launch and development utilities |
| `tests/` | API, launcher, and browser tests |
| `public/` | Static assets |
| `dist/` | Built React interface |
| `data/` | Private runtime data created on first launch |

## Storage and Security

The `data` folder contains signed account and transfer metadata, encryption keys, and encrypted payloads. Stop the host before backing up the entire folder. Keep this folder private and exclude it from version control.

**HTTP traffic is not encrypted.** AES-256-GCM protects stored file contents; the host holds the decryption key. File names, notes, and logs are authenticated but not encrypted. Use a trusted host and LAN. See [SECURITY.md](SECURITY.md) for the security model.
