# MoxFile

[中文文档](./README.zh-CN.md)

MoxFile is the ephemeral file relay for MoxChat. It creates upload sessions, accepts encrypted file streams or chunks, provides receiver-specific download URLs, records download receipts, and deletes remote data after it is no longer needed.

## Release Files

Download the files from the release page that match your target platform:

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

1. Download `moxfile.lpk`.
2. Install it from the Lazycat app UI, or with the CLI:

```sh
lzc-cli app install moxfile.lpk
```

3. Open the app at the assigned `moxfile` subdomain.
4. Check `https://<moxfile-host>/healthz`.

The LPK includes the MoxFile app process, a PostgreSQL service, and persistent storage under `/lzcapp/var/storage`.

## Linux Deployment

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
