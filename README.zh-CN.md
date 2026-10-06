# MoxFile

[English](./README.md)

MoxFile 是 MoxChat 的一次性文件中继。它负责创建上传会话、接收加密文件流或分块、生成面向接收者的下载地址、记录下载回执，并在文件不再需要后删除远端数据。

服务源码与目标规格由主 Mox 源码仓库维护，相关规格位于 `spec/files/moxfile/`。本仓库只提供部署说明与可分发产物。

## MoxChat 客户端

- iOS：[在 App Store 下载](https://apps.apple.com/us/app/moxchat/id6775016915)
- Web：[打开 MoxChat 网页版](https://app.ponzs.com)

## 从 GitHub 获取

本仓库提供部署文档、预编译二进制和懒猫 LPK，无需安装 Go 或自行编译。当前包版本为 `1.3.1`；源码修订、构建时间见 [构建信息](./BUILD-INFO.md)，文件摘要见 [SHA256SUMS](./SHA256SUMS)。

### 克隆整个发布仓库

安装 Git 后执行，Linux、macOS 和 Windows PowerShell 均适用：

```sh
git clone --depth 1 https://github.com/MoxChat/MoxFile.git
cd MoxFile
```

克隆后，在修改配置之前校验文件。Linux 使用：

```sh
sha256sum --check SHA256SUMS
```

macOS 使用：

```sh
shasum -a 256 --check SHA256SUMS
```

### 只下载当前平台的文件

Linux x64 示例（在新的部署目录中执行）：

```sh
mkdir moxfile-release
cd moxfile-release
curl -fL --retry 3 -o moxfile-linux-amd64 https://raw.githubusercontent.com/MoxChat/MoxFile/main/moxfile-linux-amd64
curl -fL --retry 3 -o SHA256SUMS https://raw.githubusercontent.com/MoxChat/MoxFile/main/SHA256SUMS
awk '$2 == "moxfile-linux-amd64" {print}' SHA256SUMS | sha256sum --check
chmod +x ./moxfile-linux-amd64
```

Linux arm64 将命令中的 `linux-amd64` 替换为 `linux-arm64`。macOS 将文件名替换为下表对应的 `darwin-*`，校验命令改用 `shasum -a 256 --check`。

Windows x64 可在新的目录中通过 PowerShell 下载并校验：

```powershell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/MoxChat/MoxFile/main/moxfile-windows-amd64.exe" -OutFile ".\moxfile-windows-amd64.exe"
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/MoxChat/MoxFile/main/SHA256SUMS" -OutFile ".\SHA256SUMS"
$expected = ((Get-Content .\SHA256SUMS | Select-String '  moxfile-windows-amd64\.exe$').Line -split '\s+')[0]
if ((Get-FileHash .\moxfile-windows-amd64.exe -Algorithm SHA256).Hash -ne $expected) { throw "SHA-256 校验失败" }
```

Windows arm64 将 `windows-amd64` 替换为 `windows-arm64`。单文件下载不会包含 `.env`，可按下方部署示例设置环境变量。只有校验通过后才启动或安装。

以上直链读取 `main` 分支。需要固定版本时，把所有下载地址中的 `main` 替换为同一个发布仓库提交 SHA；构建信息中的源码修订属于主源码仓库，不能用于这些下载地址。若下载期间分支更新导致校验失败，请固定同一提交后重新下载。

## 发布文件

通过上面的 GitHub 命令获取与你的部署目标匹配的文件：

| 目标 | 文件 |
| --- | --- |
| 懒猫微服 | `moxfile.lpk` |
| Linux x64 | `moxfile-linux-amd64` |
| Linux arm64 | `moxfile-linux-arm64` |
| macOS Intel | `moxfile-darwin-amd64` |
| macOS Apple Silicon | `moxfile-darwin-arm64` |
| Windows x64 | `moxfile-windows-amd64.exe` |
| Windows arm64 | `moxfile-windows-arm64.exe` |

## 懒猫微服部署

1. 使用已克隆的 `moxfile.lpk`，或在新的目录中直接下载并校验（Linux 示例；macOS 把 `sha256sum` 换成 `shasum -a 256`）：

```sh
curl -fL --retry 3 -o moxfile.lpk https://raw.githubusercontent.com/MoxChat/MoxFile/main/moxfile.lpk
curl -fL --retry 3 -o SHA256SUMS https://raw.githubusercontent.com/MoxChat/MoxFile/main/SHA256SUMS
awk '$2 == "moxfile.lpk" {print}' SHA256SUMS | sha256sum --check
```

2. 在懒猫应用界面安装，或使用 CLI：

```sh
lzc-cli app install moxfile.lpk
```

3. 打开分配到的 `moxfile` 子域名。
4. 检查 `https://<moxfile-host>/healthz`。

LPK 内包含 MoxFile 进程、PostgreSQL 服务，并把文件持久化目录配置到 `/lzcapp/var/storage`。

## Linux 部署

以下命令假定已进入二进制所在目录，且已安装并启动 PostgreSQL；SQL 在具备创建用户和数据库权限的 PostgreSQL 会话中执行。数据库密码 `change-me` 必须在 SQL 和连接串中同时替换。

先创建 PostgreSQL 数据库：

```sql
CREATE USER moxfile WITH PASSWORD 'change-me';
CREATE DATABASE moxfile OWNER moxfile;
```

创建存储目录并启动二进制：

```sh
mkdir -p ./moxfile-storage
chmod +x ./moxfile-linux-amd64
export MOXFILE_ADDR=:8981
export MOXFILE_DB_DSN='postgres://moxfile:change-me@127.0.0.1:5432/moxfile?sslmode=disable'
export MOXFILE_STORAGE_DIR="$PWD/moxfile-storage"
export MOXFILE_MAX_FILE_SIZE_BYTES=107374182400
./moxfile-linux-amd64
```

arm64 主机使用 `moxfile-linux-arm64`。

## macOS 部署

使用匹配的 macOS 二进制：

```sh
mkdir -p ./moxfile-storage
chmod +x ./moxfile-darwin-arm64
export MOXFILE_ADDR=:8981
export MOXFILE_DB_DSN='postgres://moxfile:change-me@127.0.0.1:5432/moxfile?sslmode=disable'
export MOXFILE_STORAGE_DIR="$PWD/moxfile-storage"
./moxfile-darwin-arm64
```

如果 macOS 拦截下载的二进制，移除 quarantine 属性：

```sh
xattr -d com.apple.quarantine ./moxfile-darwin-arm64
```

## Windows 部署

先创建 PostgreSQL 数据库，然后在 PowerShell 中启动：

```powershell
New-Item -ItemType Directory -Force .\moxfile-storage
$env:MOXFILE_ADDR = ":8981"
$env:MOXFILE_DB_DSN = "postgres://moxfile:change-me@127.0.0.1:5432/moxfile?sslmode=disable"
$env:MOXFILE_STORAGE_DIR = "$PWD\moxfile-storage"
$env:MOXFILE_MAX_FILE_SIZE_BYTES = "107374182400"
.\moxfile-windows-amd64.exe
```

Windows arm64 主机使用 `moxfile-windows-arm64.exe`。

## 环境变量

发布目录中已经包含默认 `.env` 文件。从该目录启动二进制时，MoxFile 会自动加载 `.env`；如需使用其他文件，可设置 `MOXFILE_ENV_FILE`。默认 `.env` 是部署模板，生产环境必须先替换 `change-me` 数据库密码，并把 `MOXFILE_STORAGE_DIR` 指向持久化目录。

| 变量 | 必填 | 说明 |
| --- | --- | --- |
| `MOXFILE_ADDR` | 否 | HTTP 上传、下载、健康检查和运维端点监听地址，默认 `:8981`。 |
| `MOXFILE_DB_DSN` | 是 | PostgreSQL 连接串，用于存储文件元数据、上传会话、文件 token、回执和表结构状态。 |
| `MOXFILE_STORAGE_DIR` | 建议填写 | 加密文件字节的本地存储目录；生产环境必须使用持久化目录，否则上传文件可能丢失。 |
| `MOXFILE_ENV_FILE` | 否 | 指定其他 env 文件路径；不填时服务会在当前目录或上级目录查找 `.env`。 |
| `MOXFILE_FILE_TTL_SECONDS` | 否 | 文件对象默认有效期，过期后会成为后台清理候选，默认 `86400` 秒。 |
| `MOXFILE_MAX_FILE_SIZE_BYTES` | 否 | 接受的单文件最大字节数，必须是十进制整数，默认 `107374182400`。 |
| `MOXFILE_DATA_RETENTION_DAYS` | 否 | 过期数据库记录和磁盘文件的清理保留窗口，默认 `30` 天。 |
| `MOXFILE_CHALLENGE_TTL_SECONDS` | 否 | challenge-response 认证验证码有效期，默认 `60` 秒。 |
| `MOXFILE_AUTH_FAIL_WINDOW_MINUTES` | 否 | 按源 IP 统计认证失败次数的时间窗口，默认 `30` 分钟。 |
| `MOXFILE_AUTH_FAIL_BAN_MINUTES` | 否 | 源 IP 超过失败阈值后的临时封禁时长，默认 `30` 分钟。 |
| `MOXFILE_AUTH_FAIL_BAN_THRESHOLD` | 否 | 统计窗口内允许的认证失败次数，超过后封禁该源 IP，默认 `10`。 |

## 健康检查和运维

- `GET /healthz` 检查 HTTP 进程是否可达。
- `POST /api/secure/health` 检查已认证的文件中继功能、数据库访问和存储目录可写性。
- `GET /` 打开运维状态页。
- `GET /status.json` 返回状态快照。
- `GET /status/stream` 返回实时状态流。

生产环境建议通过 HTTPS 暴露服务。上传、manifest 和下载地址必须能被发送方和所有接收方访问。MoxChat 客户端应填写外部可访问的文件中继地址，例如 `https://moxfile.example.com`。

## 更新已有部署

先备份数据库、持久化数据和运行配置，停止旧进程。真实配置和数据应放在发布仓库之外；仓库内 `.env` 仅为模板，避免更新时覆盖本地配置。对于通过 Git 克隆且工作区干净的部署：

```sh
git pull --ff-only
```

更新后重新校验 `SHA256SUMS`（Linux：`sha256sum --check SHA256SUMS`；macOS：`shasum -a 256 --check SHA256SUMS`），再使用原有运行配置启动新二进制。单文件部署重新下载同一提交下的二进制与校验文件，LPK 部署重新执行安装命令。文件校验失败时不要继续启动。

启动后在另一终端检查实际监听端口（默认 `8981`）：

```sh
curl -f http://127.0.0.1:8981/healthz
```

预期返回 HTTP 200。再通过公网服务地址检查 `/healthz`，并在 MoxChat 中验证对应功能。`/healthz` 只表明 HTTP 进程可达，不能替代数据库、Mesh、文件传输、SFU 媒体或 APNs 投递验证。
