# MoxFile 构建信息

| 项目 | 内容 |
| --- | --- |
| 包版本 | `1.3.1` |
| 服务源码修订 | [f01e745b](https://github.com/ponzS/MoxFile/commit/f01e745b118d2ef28b9e02e1b34c0fc99532e164) |
| 构建开始时间（UTC） | `2026-11-22T10:56:17Z` |
| 工作区构建入口 | [scripts/github-build.sh](https://github.com/ponzS/MoxWorkSpace/blob/1c04a18c10549910f3d7711038cca9ce04f75c83/scripts/github-build.sh) |
| 工具链 | Go `1.26.5`、LazyCat CLI `2.0.9` |
| 二进制构建参数 | `CGO_ENABLED=0`、`-trimpath`、`-ldflags='-s -w'` |
| 目标平台 | Linux、macOS、Windows，均提供 amd64 和 arm64 |
| 懒猫 LPK v2 | `moxfile.lpk`，内含同批次的 Linux amd64 和 arm64 二进制 |

完善上传大小限制和头像替换的并发处理，并支持 Windows 部署的磁盘配额检查。

为固定下载版本，请使用本发布仓库的提交 SHA。服务源码修订用于追踪构建来源。校验摘要见 [SHA256SUMS](./SHA256SUMS)。
