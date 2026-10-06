# MoxFile

[中文文档](./README.zh-CN.md)

MoxFile is the ephemeral file relay for MoxChat. It creates upload sessions, accepts encrypted file streams or chunks, provides receiver-specific download URLs, records download receipts, and deletes remote data after it is no longer needed.

Service source code and target specifications are maintained in the main Mox source repository under `spec/files/moxfile/`. This repository contains deployment documentation and distributable artifacts only.

## MoxChat Clients

- iOS: [Download on the App Store](https://apps.apple.com/us/app/moxchat/id6775016915)
- Web: [Open MoxChat](https://app.ponzs.com)

## Get the Files from GitHub

This repository provides deployment documentation, prebuilt binaries, and Lazycat LPK packages. You do not need Go or a local build. The current package version is `1.3.1`; see [build information](./BUILD-INFO.md) for the source revision and build timestamp, and [SHA256SUMS](./SHA256SUMS) for file digests.

### Clone the Release Repository

With Git installed, run these commands on Linux, macOS, or Windows PowerShell:

```sh
git clone --depth 1 https://github.com/MoxChat/MoxFile.git
cd MoxFile
```

Verify the files after cloning and before editing configuration. On Linux:

```sh
sha256sum --check SHA256SUMS
```

On macOS:

```sh
shasum -a 256 --check SHA256SUMS
```

### Download Only the Files for Your Platform

Linux x64 example, using a new deployment directory:

```sh
mkdir moxfile-release
cd moxfile-release
curl -fL --retry 3 -o moxfile-linux-amd64 https://raw.githubusercontent.com/MoxChat/MoxFile/main/moxfile-linux-amd64
curl -fL --retry 3 -o SHA256SUMS https://raw.githubusercontent.com/MoxChat/MoxFile/main/SHA256SUMS
awk '$2 == "moxfile-linux-amd64" {print}' SHA256SUMS | sha256sum --check
chmod +x ./moxfile-linux-amd64
```

For Linux arm64, replace `linux-amd64` with `linux-arm64`. On macOS, use the matching `darwin-*` filename from the table below and replace the checksum command with `shasum -a 256 --check`.

For Windows x64, download and verify the files in a new directory using PowerShell:

```powershell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/MoxChat/MoxFile/main/moxfile-windows-amd64.exe" -OutFile ".\moxfile-windows-amd64.exe"
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/MoxChat/MoxFile/main/SHA256SUMS" -OutFile ".\SHA256SUMS"
$expected = ((Get-Content .\SHA256SUMS | Select-String '  moxfile-windows-amd64\.exe$').Line -split '\s+')[0]
if ((Get-FileHash .\moxfile-windows-amd64.exe -Algorithm SHA256).Hash -ne $expected) { throw "SHA-256 verification failed" }
```

For Windows arm64, replace `windows-amd64` with `windows-arm64`. Individual binary downloads do not include `.env`; set environment variables as shown in the deployment examples below. Start or install the service only after verification succeeds.

These direct URLs read the `main` branch. To pin a version, replace `main` in every download URL with the same release repository commit SHA. The source revision in the build information belongs to the main source repository and cannot be used in these URLs. If the branch changes during download and verification fails, pin one commit and download the files again.

## Release Files

Use the GitHub commands above to obtain the files that match your target platform:

| Target | File |
| --- | --- |
| Lazycat MicroServer | `moxfile.lpk` |
| Linux x64 | `moxfile-linux-amd64` |
| Linux arm64 | `moxfile-linux-arm64` |
| macOS Intel | `moxfile-darwin-amd64` |
| macOS Apple Silicon | `moxfile-darwin-arm64` |
| Windows x64 | `moxfile-windows-amd64.exe` |
| Windows arm64 | `moxfile-windows-arm64.exe` |

## Lazycat MicroServer Deployment

1. Use `moxfile.lpk` from your clone, or download and verify it in a new directory (Linux example; on macOS, replace `sha256sum` with `shasum -a 256`):

```sh
curl -fL --retry 3 -o moxfile.lpk https://raw.githubusercontent.com/MoxChat/MoxFile/main/moxfile.lpk
curl -fL --retry 3 -o SHA256SUMS https://raw.githubusercontent.com/MoxChat/MoxFile/main/SHA256SUMS
awk '$2 == "moxfile.lpk" {print}' SHA256SUMS | sha256sum --check
```

2. Install it from the Lazycat app UI, or with the CLI:

```sh
lzc-cli app install moxfile.lpk
```

3. Open the app at the assigned `moxfile` subdomain.
4. Check `https://<moxfile-host>/healthz`.

The LPK includes the MoxFile app process, a PostgreSQL service, and persistent storage under `/lzcapp/var/storage`.

## Linux Deployment

These commands assume you are in the binary directory and PostgreSQL is installed and running. Run the SQL in a PostgreSQL session with permission to create users and databases. Replace `change-me` in both the SQL and the connection string with the same database password.

Create a PostgreSQL database:

```sql
CREATE USER moxfile WITH PASSWORD 'change-me';
CREATE DATABASE moxfile OWNER moxfile;
```

Create a storage directory and run the binary:

```sh
mkdir -p ./moxfile-storage
chmod +x ./moxfile-linux-amd64
export MOXFILE_ADDR=:8981
export MOXFILE_DB_DSN='postgres://moxfile:change-me@127.0.0.1:5432/moxfile?sslmode=disable'
export MOXFILE_STORAGE_DIR="$PWD/moxfile-storage"
export MOXFILE_MAX_FILE_SIZE_BYTES=107374182400
./moxfile-linux-amd64
```

Use `moxfile-linux-arm64` on arm64 hosts.

## macOS Deployment

Use the matching macOS binary:

```sh
mkdir -p ./moxfile-storage
chmod +x ./moxfile-darwin-arm64
export MOXFILE_ADDR=:8981
export MOXFILE_DB_DSN='postgres://moxfile:change-me@127.0.0.1:5432/moxfile?sslmode=disable'
export MOXFILE_STORAGE_DIR="$PWD/moxfile-storage"
./moxfile-darwin-arm64
```

If macOS blocks a downloaded binary, remove the quarantine attribute:

```sh
xattr -d com.apple.quarantine ./moxfile-darwin-arm64
```

## Windows Deployment

Create the PostgreSQL database first, then start MoxFile from PowerShell:

```powershell
New-Item -ItemType Directory -Force .\moxfile-storage
$env:MOXFILE_ADDR = ":8981"
$env:MOXFILE_DB_DSN = "postgres://moxfile:change-me@127.0.0.1:5432/moxfile?sslmode=disable"
$env:MOXFILE_STORAGE_DIR = "$PWD\moxfile-storage"
$env:MOXFILE_MAX_FILE_SIZE_BYTES = "107374182400"
.\moxfile-windows-amd64.exe
```

Use `moxfile-windows-arm64.exe` on Windows arm64 hosts.

## Environment Variables

This release directory includes a default `.env` file. When the binary is started from this directory, MoxFile loads `.env` automatically; set `MOXFILE_ENV_FILE` to point at a different file. The default file is a deployment template, so replace `change-me` database credentials and choose a persistent `MOXFILE_STORAGE_DIR` before production use.

| Variable | Required | Description |
| --- | --- | --- |
| `MOXFILE_ADDR` | No | Network address for the HTTP upload, download, health, and operations endpoints. Default: `:8981`. |
| `MOXFILE_DB_DSN` | Yes | PostgreSQL connection string used for file metadata, upload sessions, file tokens, receipts, and schema state. |
| `MOXFILE_STORAGE_DIR` | Recommended | Local directory for encrypted file bytes. Use persistent storage in production or uploaded files may be lost. |
| `MOXFILE_ENV_FILE` | No | Alternate env file path. If omitted, the service searches for `.env` in the current directory and parent directories. |
| `MOXFILE_FILE_TTL_SECONDS` | No | Default lifetime for file objects before they expire and become cleanup candidates. Default: `86400`. |
| `MOXFILE_MAX_FILE_SIZE_BYTES` | No | Maximum accepted file size in decimal bytes. Default: `107374182400`. |
| `MOXFILE_DATA_RETENTION_DAYS` | No | Retention window for expired database records and on-disk file cleanup. Default: `30`. |
| `MOXFILE_CHALLENGE_TTL_SECONDS` | No | Lifetime of challenge-response authentication codes. Default: `60`. |
| `MOXFILE_AUTH_FAIL_WINDOW_MINUTES` | No | Time window used to count failed authentication attempts per source IP. Default: `30`. |
| `MOXFILE_AUTH_FAIL_BAN_MINUTES` | No | Temporary ban duration after a source IP exceeds the failure threshold. Default: `30`. |
| `MOXFILE_AUTH_FAIL_BAN_THRESHOLD` | No | Number of failed authentication attempts allowed in the window before banning the source IP. Default: `10`. |

## Health and Operations

- `GET /healthz` verifies that the HTTP process is reachable.
- `POST /api/secure/health` verifies authenticated relay functionality, database access, and storage writability.
- `GET /` opens the operations/status page.
- `GET /status.json` returns a status snapshot.
- `GET /status/stream` streams live status updates.

Expose the service through HTTPS in production. Upload, manifest, and download URLs must be reachable by the sender and all receivers. MoxChat clients should use the externally reachable file relay URL, for example `https://moxfile.example.com`.

## Update an Existing Deployment

Back up the database, persistent data, and runtime configuration, then stop the old process. Keep real configuration and data outside the release repository; the included `.env` is a template, so updates should not overwrite your local configuration. For a Git clone with a clean working tree:

```sh
git pull --ff-only
```

Verify `SHA256SUMS` again (Linux: `sha256sum --check SHA256SUMS`; macOS: `shasum -a 256 --check SHA256SUMS`), then start the new binary with your existing runtime configuration. For individual downloads, download the binary and checksum file from the same commit again. For LPK deployments, run the installation command again. Do not start the service if file verification fails.

After startup, check the actual listening port from another terminal (default: `8981`):

```sh
curl -f http://127.0.0.1:8981/healthz
```

Expect HTTP 200. Then check `/healthz` through the public service URL and verify the corresponding feature in MoxChat. `/healthz` only confirms that the HTTP process is reachable; it does not replace database, mesh, file transfer, SFU media, or APNs delivery validation.
