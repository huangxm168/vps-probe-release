# VPS Probe Releases

Signed release binaries of **vps-probe**, the Linux (amd64 / arm64) agent of a self-hosted VPS monitoring system. Source code is private; this repository only hosts releases. Releases are immutable: once published, assets cannot be changed or replaced.

vps-probe 的发布仓库：只放签过名的二进制与安装脚本，源码是私有的。本仓库开启了 GitHub 不可变发布，版本一经发布，文件不能再修改或替换。

## Assets / 每个版本的文件

| File | Description |
| --- | --- |
| `vps-probe_linux_amd64`, `vps-probe_linux_arm64` | Static binaries / 静态二进制 |
| `install.sh` | Installer, pinned to the two binaries' SHA-256 / 安装脚本，内含两个二进制的 SHA-256 |
| `manifest.json` | Version, minimum allowed version, source commit, and SHA-256 and size of every file above / 版本号、签名下限、源码提交与上面每个文件的 SHA-256 和大小 |
| `manifest.json.minisig` | [minisign](https://jedisct1.github.io/minisign/) signature of `manifest.json`, trusted comment `vps-probe manifest <version>` / 清单的 minisign 签名 |

Third-party license notices are embedded in the binary: run `vps-probe licenses`. / 第三方许可声明编在二进制里，`vps-probe licenses` 打印。

Binaries are built twice from the same commit, by CI and locally, and signed only when both builds are byte-identical. The signing keys never leave the maintainer's machines.

每个版本由 CI 与维护者本地各编译一次，两边逐字节一致才签名；签名私钥不在 GitHub 上。

## Public keys / 公钥

Primary / 主钥：

```
RWQjl7fcVYCYoa1hmaZkwDHP/tQOF2Jr/pW/4gKy1Ywj/LLx/FkPqLXd
```

Backup (used only for key rotation) / 备用钥（只在换钥时使用）：

```
RWQmx/p6AyHcEgCgzBMCx4QhtskK6mg4DjD+jK2pxaUupgsfsGG3zo7x
```

## Verify / 手动验签

```bash
VERSION=v0.1.0
BASE=https://github.com/huangxm168/vps-probe-release/releases/download/$VERSION
curl -fsSLO "$BASE/manifest.json" -fsSLO "$BASE/manifest.json.minisig" -fsSLO "$BASE/vps-probe_linux_amd64"

# 1. The signature must verify, and the printed trusted comment must be exactly "vps-probe manifest $VERSION"
#    签名验证通过，且输出的 Trusted comment 必须正好是「vps-probe manifest $VERSION」
minisign -V -H -P '<primary public key above>' -m manifest.json

# 2. Every downloaded file must match the manifest / 下载的每个文件都要与清单一致
jq -r '.files | to_entries[] | "\(.value.sha256)  \(.key)"' manifest.json | sha256sum -c --ignore-missing
```

A release that replaces the primary key is signed with the backup key; verify it with the backup public key instead.

更换主钥的那个版本由备用钥签名，验签时改用上面的备用公钥。

GitHub's release attestation can be checked too: `gh release verify $VERSION -R huangxm168/vps-probe-release`.
