# Vastrix secure file transfer

This workspace contains the Vastrix React + Vite frontend and its companion Replit workspace packages.

## Current behavior

The secure-file-transfer app currently runs in **local preview mode**:

- Preview profile labels, transfer metadata, and selected file contents are saved in the current browser using IndexedDB.
- You can create a transfer, set an expiry and optional download cap, open its local share page, download the original files, search saved transfers, and remove them.
- An optional passphrase encrypts file contents in the browser with AES-GCM before they are saved. The passphrase is not stored; file names and transfer metadata are not encrypted.
- Preview files are limited to 64 MiB each and 250 MiB per transfer.
- Preview links work only in the browser and device where the files were saved. They are not remote share links.
- Passphrase protection and download caps are enforced only by this browser preview. They do not replace backend security or enforce limits on a remote service.
- Preview access does not authenticate a user. Email delivery, recipient verification, cross-device downloads, and server-side admin data are not connected.

The separate .NET API described in the supplied architecture is not included in this workspace. Connect it when its source and current request/response contract are available; the local preview does not upload or migrate its data automatically.

## Run on Windows or macOS

Install a current Node.js LTS release and enable Corepack. In PowerShell on Windows or Terminal on macOS, run these commands from the project root:

```sh
corepack enable
pnpm install
pnpm --filter @workspace/secure-file-transfer run dev
```

Open the local Vite URL printed in the terminal (normally `http://localhost:5173`). No API key or `.env` file is needed for the local preview.

## Verify / build

```sh
pnpm --filter @workspace/secure-file-transfer run typecheck
pnpm --filter @workspace/secure-file-transfer run build
```

The preview artifact is in `artifacts/secure-file-transfer`. Other artifacts and shared libraries remain part of this pnpm workspace.
