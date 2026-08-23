# MoxFile

[English](./README.md)

MoxFile 是 MoxChat 的一次性文件中继。它负责创建上传会话、接收加密文件流或分块、生成面向接收者的下载地址、记录下载回执，并在文件不再需要后删除远端数据。

## 发布文件

从发布页下载与你的部署目标匹配的文件：

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

1. 下载 `moxfile.lpk`。
2. 在懒猫应用界面安装，或使用 CLI：

```sh
lzc-cli app install moxfile.lpk
```

3. 打开分配到的 `moxfile` 子域名。
4. 检查 `https://<moxfile-host>/healthz`。

LPK 内包含 MoxFile 进程、PostgreSQL 服务，并把文件持久化目录配置到 `/lzcapp/var/storage`。

## Linux 部署

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
