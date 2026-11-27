# PonWrt AN7581 apk 软件仓库

> 当前分支：`repo`（主线固件用）。

本仓库是 PonWrt AN7581（Airoha）固件的 **apk 软件包 + 固件更新索引** 仓库。两条固件线各用一个分支，互不混装：

| 分支 | 服务哪条固件线 | 机型 | 源码分支 / label |
| --- | --- | --- | --- |
| `repo` | 主线固件（`fw-all.targets=all/mainline`） | 五台：HG5585F-CT、HG5585F-CT(USB-SFP)、HG5382A、XG-040G-MD、ZN504XG-D | `master` / `all` |
| `repo-wed` | WED+NPU 硬件卸载固件（`targets=wed`） | 两台：HG5585F-CT、HG5585F-CT(USB-SFP) | `feat/wed-npu-offload-6.18` / `wed` |

固件镜像里已按分支内置源地址（主线读 `repo`、WED 读 `repo-wed`），所以设备不需要手动加源，也不应该跨分支取包。

## 目录结构

```
aarch64_cortex-a53/ponwrt-k<内核版本>/
  common/  usb/  wifi/  bt/        # 按机型能力分组，各一份签名索引
    *.apk
    packages.adb                   # 本钥签名的 apk 索引（源地址指向它）
    SHA256SUMS
    README.txt                     # 本目录的包数量与完整源地址
meta/fw/<device>.json              # 固件更新索引（luci-app-ponwrt-update 读它）
public-key.pem                     # 签名公钥（全仓共用一把固定私钥）
.snapshot-id                       # 分支内容指纹（单提交快照）
```

- 目录按**内核版本**分（`ponwrt-k6.18.60` 这类），能力子目录再区分机型差异。
- kmod 也在这些目录里发布：与固件**同一次构建**产出，`kernel=<内核版本>~<config 指纹>` 与该分支的内核 config 绑定。换内核配置后，`pkg-publish` 会先清掉不匹配的旧 kmod 再重建索引，所以不会残留装不上的 kmod。
- **不要跨分支装 kmod**：`repo` 的 kmod 只配主线固件，`repo-wed` 的只配 WED 固件（内核 config 不同）。

## 固件 Release

固件镜像在本仓库发 Release，tag 形如 `fw-<device>-r<run>-<commit7>`，资产是 sysupgrade `.itb` + 同名 `.sha256` + `manifest-*.json`；`meta/fw/<device>.json` 指向它（含 sha256、run、commit、内核版本）。

归档用的整批产物在固件仓 `ufo1cn/ponwrt-ZN504-HG5585F`：

- `v<日期>-<时间>-all|wed-r<run>`：五台/两台的 itb + sha256 + manifest + config + build-info
- `kmods-all|wed-r<run>`：同一次构建的 kmod apk 归档

## 安装

设备侧（源已内置，直接装）：

```sh
apk update
apk add <包名>
```

离线安装：

```sh
cp public-key.pem /etc/apk/keys/
apk add --no-network /tmp/xxx.apk
```

公钥就位后不需要 `--allow-untrusted`。

## 口径与维护

- 签名：与固件、源码仓所有工作流共用同一把固定私钥；公钥 sha256 `d557dca1a8b8d30b48f7e4b910908342789d389a52db3d7225c6b72840eda4ed`，设备公钥在 `/etc/apk/keys/public-key.pem`。
- target：`airoha/an7581` → `aarch64_cortex-a53`（musl）。
- 写入方：`ufo1cn/ponwrt-ZN504-HG5585F` 的 `pkg-publish`（apk + 索引）、`fw-publish`（固件 Release + `meta/fw`）；本仓 `Reindex apk repository` 工作流每 2 小时兜底验签/补签并重建索引。手工改目录内容会被下一次重建覆盖。
- 分支内容是**单提交快照**（强推覆盖），没有可用的 git 历史；比对版本看 `.snapshot-id` 与各目录 `README.txt`。
- 清空前备份 tag：`backup-pre-wipe-20261126`（`repo`）、`backup-wed-pre-wipe-20261126`（`repo-wed`），需要回溯旧包时从 tag 取。
- 可选 Gitee 只读镜像：由 Gitee「仓库镜像 Pull 模式」定时拉取本仓分支，路径同名；本仓库不主动推送 Gitee。
