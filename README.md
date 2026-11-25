# PonWrt AN7581 apk 软件仓库

FiberHome AN7581 系列（HG5585F-ct / HG5382A）固件用的 apk 包仓库。
所有 apk 都在 `ufo1cn/ponwrt-ZN504-HG5585F` 同一内核源码树上编译，
并用**同一把固定私钥**签名（与固件、与两个源码仓库的所有工作流一致）。

- 签名公钥：`public-key.pem`
- 公钥 sha256：`d557dca1a8b8d30b48f7e4b910908342789d389a52db3d7225c6b72840eda4ed`
- target：`airoha/an7581` → `aarch64_cortex-a53` (musl)

## 安装

设备上只需一次：

```sh
cp public-key.pem /etc/apk/keys/
apk add /tmp/xxx.apk
```

公钥已就位后**不需要** `--allow-untrusted`；离线安装可加 `--no-network`。

## Release 命名

| tag | 内容 |
| --- | --- |
| `legacy-<device>-r<run>` | 旧仓有、新仓无的用户态软件（SMB/NFS/工具/组网等） |
| `pkgs-incremental-<device>-r<run>` | 自有用户态包（ser2net/go2rtc/camlive/bluez 等） |
| `pkgs-extra-<device>-r<run>` | 外挂 feed 包（iptv/nikki/passwall2/homeproxy） |

每个 Release 都带 `public-key.pem`、`public-key.pem.sha256`、`abi-info.txt`，
以及每个 apk 的 `.sha256`。

## 注意

内核模块（`kmod-*`）不在本仓库重复发布：它们与固件的内核版本及 vermagic 强绑定，
跨版本装不上，由固件镜像直接内置（USB/无线/音视频驱动在 USB 版固件里全部 `=y`）。
