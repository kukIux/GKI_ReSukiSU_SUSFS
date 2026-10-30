<div align="center">

# GKI BakaSU SUSFS

用 GitHub Actions 在云端编译 Android GKI 内核，默认集成 **BakaSU**（root）和 **SUSFS**（隐藏 root 痕迹）。

简体中文 / [English](README_EN.md)

[![Release](https://img.shields.io/github/v/release/coolzyd9107/GKI_BakaSU_SUSFS?label=Release&style=flat-square&logo=github&logoColor=white&color=2ea44f)](https://github.com/zhuzhuzihan/GKI_BakaSU_SUSFS/releases)
[![构建 GKI 内核](https://github.com/zhuzhuzihan/GKI_BakaSU_SUSFS/actions/workflows/main.yml/badge.svg)](https://github.com/zhuzhuzihan/GKI_BakaSU_SUSFS/actions/workflows/main.yml)
[![Telegram](https://img.shields.io/static/v1?label=Telegram&message=Channel&color=0088cc)](https://t.me/BakaSUKernelBuilds)
[![BakaSU](https://img.shields.io/badge/KernelSU-BakaSU-5AA300?style=flat-square)](https://github.com/Baka-SU/BakaSU)
[![SUSFS](https://img.shields.io/badge/Filesystem-SUSFS-E67E22?style=flat-square)](https://gitlab.com/simonpunk/susfs4ksu)

</div>

> **先说明：** 这里产出的是可刷入的 **AnyKernel3 内核包**，不是 ROM，也不是整机固件。刷错内核可能开不了机；刷之前一定备份当前 boot，并确认自己知道怎么恢复。
>
> **赶时间？** Fork → Actions → Build GKI Kernels → 勾 `build_kernel_version`，填完整内核版本（如 `6.6.92`；5.10 / 5.15 还要选 `kernel_android_version`），`build_time` 清空 → Run。跑完在 Artifacts 下载。第一次用还是建议先看下面的 KMI 和 5 个坑。

## 先搞懂 6 个词

| 词 | 用大白话说 |
|---|---|
| **GKI** | 谷歌的统一内核（Generic Kernel Image）。这个项目编译的就是它。 |
| **KMI** | 内核和手机厂商模块之间的“接口版本”。KMI 不对，轻则相机、Wi-Fi 不正常，重则开不了机。 |
| **BakaSU** | KernelSU 的一个分支，用来提供 root。 |
| **SUSFS** | 一套内核补丁，用来隐藏 root 和文件系统痕迹。不保证绕过所有检测。 |
| **AnyKernel3** | 一种可以直接刷入的内核 ZIP。产物就是这个，不是 `boot.img`。 |
| **Artifacts** | Actions 跑完后生成的文件。GitHub 会在外面再套一层 ZIP，需要先解压外层。 |

## 开始前：确认手机的 KMI（最重要）

KMI 可以理解成 **Android 版本 + 内核系列**，例如 `android15-6.6`。构建目标必须和设备实际使用的 KMI 一致。

**不要只看 Android 版本。** 同样是 Android 14，内核可能是 5.15，也可能是 6.1；同样是 5.15，Android 13 和 Android 14 的内核版本号还可能长得一样。判断依据是内核版本字符串里的 `androidXX` 和前面的 `5.xx` / `6.xx`。

怎么查：

1. 手机上打开 **设置 → 关于手机 → 内核版本**；
2. 或者用终端 / adb 执行 `uname -r`；
3. 或者直接问 ROM / 设备维护者。

| Android 版本 | 内核系列 | 内核版本里常见的标记 | 主工作流的 `build_target` |
|---|---|---|---|
| Android 12 | 5.10 | `android12` | `android12-5.10` |
| Android 13 | 5.10 | `android13` | `android13-5.10` |
| Android 13 | 5.15 | `android13` | `android13-5.15` |
| Android 14 | 5.15 | `android14` | `android14-5.15` |
| Android 14 | 6.1 | `android14` | `android14-6.1` |
| Android 15 | 6.6 | `android15` | `android15-6.6` |
| Android 16 | 6.12 | `android16` | `android16-6.12` |
| Android 17 | 6.18 | `android17` | `android17-6.18` |

几个例子：

- `5.15.94-android13-8-g...` → 内核系列 5.15，Android 13，选 `android13-5.15`
- `5.15.110-android14-11-...` → 内核系列 5.15，Android 14，选 `android14-5.15`
- `6.6.92-android15-8-...` → 内核系列 6.6，Android 15，选 `android15-6.6`

> **注意：** 5.10 和 5.15 各自都对应两个 Android KMI。只看到 `5.15.xxx` 不能判断是 Android 13 还是 14，必须看 `androidXX` 这一段。如果内核版本被第三方改过、看不到 `androidXX`，直接问 ROM / 设备维护者，不要猜。

## 开始前必看：5 个常见坑

| 坑 | 会发生什么 | 正确做法 |
|---|---|---|
| 选了 `build_target: all` | 会把所有 KMI 都展开，当前是一百多个 job，跑很久 | 只为一台设备构建时，选一个 KMI，或用下面的“版本筛选” |
| 没改 `build_time` | 表单预填了 `Sun Dec 01 08:10:00 UTC 2024`，不改就会把内核构建时间写死成这个值 | 清空输入框，或填 `N`，使用运行时当前 UTC 时间 |
| 自定义构建没改 `revision` | 表单默认是 `r11`；如果 JSON 条目没有 revision，会去找不存在的上游 tag，通常直接失败 | JSON 里没有 `revision` 就清空；有就原样填 |
| 勾了 `clean_build` | 构建会跳过 BakaSU / SUSFS，产物没有 root；这不是“更干净、更稳定” | 想要 root 就别勾 |
| 同一分支重复点 `Run workflow` | `Build GKI Kernels` 的新运行会取消正在跑的上一次 | 等上一次结束再开；或者换一个分支跑 |

## 三种构建方式：先选一个

| 我想要 | 用哪个入口 | 会跑几个 job |
|---|---|---|
| 只编当前设备的**一个**内核版本（推荐） | **Build GKI Kernels** + 版本筛选（填完整版本） | 1 个 |
| 把一个 KMI 的**所有补丁版本**都编出来 | **Build GKI Kernels** + `build_target` | 很多个 |
| 指定**补丁月份 / revision** | **Build Custom Android Kernel** | 1 个 |

下面分别说明这三种。

## 方式 A：只编一个内核版本（推荐）

适合：已经知道设备完整内核版本的人，例如 `uname -r` 显示 `6.6.92`，只想编这一个。

1. **Fork 本仓库。** 点右上角 **Fork**，在你自己的 fork 里操作，避免占用上游的构建队列。
2. **打开工作流。** 进入 fork 的 **Actions**；如果 GitHub 提示需要启用 workflows，先点启用。左侧选 **Build GKI Kernels**，再点 **Run workflow**。
3. **按下面填表。**

| 表单字段 | 填什么 | 例子（设备内核 6.6.92） |
|---|---|---|
| `build_kernel_version` | 勾选（true），表示按版本筛选 | 勾选 |
| `kernel_version_filter` | 完整内核版本 | `6.6.92` |
| `kernel_android_version` | 6.x 选 `auto`；5.10 选 `android12` / `android13`；5.15 选 `android13` / `android14` | `auto` |
| `build_target` | 会被忽略；保持默认 `all` 也不会全量构建 | `all` |
| `build_time` | 清空，或填 `N` | 留空 |
| `release_type` | fork 里保持 `Actions` | `Actions` |
| 其他功能开关 | 首次构建保持默认 | 默认 |

4. **点 Run workflow。** 这个模式只会跑 **1 个 job**，名字类似 `6.6.92-android15-2025-07`。
5. **下载产物。** 跑完后在运行页底部 **Artifacts** 下载 `BakaSU_kernel-...`，解压外层，刷里面的 `...-AnyKernel3.zip`。具体见[下载与刷入](#跑完之后下载解压刷入)。

> **为什么 `build_target` 可以不管？** 勾选 `build_kernel_version` 后，主工作流走版本筛选分支，`build_target` 不参与选择。保持默认 `all` 不会触发全量构建，可以放心。

如果提示 `kernel version not found in JSON`，说明 `data/` 里还没有这个子版本。可以改用 `6.6.X` 编整个系列，或等每周的数据同步；不要随便猜一个不存在的版本。

如果同一个子版本在 JSON 里出现多次（5.10 / 5.15 很常见），版本筛选取**第一条**记录。想指定后面的补丁月份或 revision，请用方式 C。

## 方式 B：把一个 KMI 的所有补丁版本都编出来

适合：不确定要哪个子版本，或者想一次拿到某个 KMI 的多份内核。

1. Fork 本仓库，进入 **Actions → Build GKI Kernels → Run workflow**。
2. `build_target` 选你的 KMI（见上面的表）；`build_time` 清空或填 `N`；其他保持默认。
3. 点 **Run workflow**。
4. 跑完后，在 **Artifacts** 里下载你需要的包。

> **不要选 `all`。** `all` 会把所有 KMI 都展开，当前是一百多个 job；只为一台手机编，选一个 KMI 就够了。

每个补丁版本是一个独立的 job，互不影响；某个 job 失败，其他 job 仍会继续。5.10 的矩阵会保留所有月度记录，任务更多；5.15 和 6.x 对重复的子版本只保留数据里的第一条记录。

如果只想编其中一个子版本，别用这个方式，回到方式 A 或方式 C。

## 方式 C：指定补丁月份或 revision

适合这些情况：

- 5.10 / 5.15 的同一个子版本有多个 revision，你要指定其中一个；
- 你知道自己需要某个特定的 `date`（补丁月份）；
- 你想让 `sub_level` 填 `X`，跟随所选分支的当前源码。

这个方式用 **Build Custom Android Kernel**，每次只编一个组合，不会创建 Release。

### 先查 data JSON

打开对应文件，例如 [data/android15/6.6.json](data/android15/6.6.json)。`entries` 里每一项长这样：

```json
{ "date": "2025-07", "kernel": "6.6.92" }
```

5.10 / 5.15 的条目可能还带 `revision`：

```json
{ "date": "2023-06", "kernel": "5.15.110", "revision": "r11" }
```

文件顶部的 `lts` 是单独的 LTS 目标。`date` 是谷歌 GKI 分支的发布月份，不一定等于手机设置里显示的完整安全补丁日期。

### 然后按表填

| 想构建的版本 | `android_version` | `kernel_version` | `sub_level` | `os_patch_level` | `revision` |
|---|---|---|---|---|---|
| Android 15 / 6.6.92，2025-07 | `android15` | `6.6` | `92` | `2025-07` | 清空 |
| Android 14 / 5.15.110，2023-06 | `android14` | `5.15` | `110` | `2023-06` | `r11` |

操作步骤：

1. 打开 **Actions → Build Custom Android Kernel → Run workflow**。
2. `android_version` 和 `kernel_version` 必须是支持表里的配对，例如 `android15` + `6.6`。
3. `sub_level` 只填最后一段：`6.6.92` 填 `92`，`5.15.110` 填 `110`。
4. `os_patch_level` 填 JSON 里的 `date`，格式是 `YYYY-MM`；LTS 目标填 `lts`。
5. `revision` 必须和 JSON 完全一致。JSON 里没有 `revision` 就清空，**特别是要清掉表单默认的 `r11`**。
6. 需要一加 8E 就勾 `supp_op`；其他功能按需开。
7. 点 **Run workflow**，跑完后从这次运行的 **Artifacts** 下载。这个工作流不创建 Release。

> **最容易错的是 `revision`。** 表单默认填 `r11` 只是示例；当前 6.1 / 6.6 / 6.12 / 6.18 的数据都没有 revision，必须清空。填错会去找不存在的上游 tag，通常直接失败。

LTS 目标：`os_patch_level` 填 `lts`，`revision` 一般清空（看 JSON 的 `lts_revision`，没有就留空）。

`sub_level` 也可以填 `X`，表示跟随所选分支当前源码；产物名字会带 `X`，不如数字版本明确，能用数字就用数字。

## 跑完之后：下载、解压、刷入

### 找到正确的文件

Actions 的 Artifact 名称通常长这样：

| 情况 | Artifact 名称 | 解压外层后，里面是 |
|---|---|---|
| 普通构建 | `BakaSU_kernel-android15-6.6.92-2025-07` | `android15-6.6.92-2025-07-BakaSU-AnyKernel3.zip` |
| 带 revision | `BakaSU_kernel-android13-5.10.198-2023-12-r7` | `android13-5.10.198-2023-12-r7-BakaSU-AnyKernel3.zip` |
| Clean build | `Clean_kernel-android15-6.6.92-2025-07` | `android15-6.6.92-2025-07-Clean-AnyKernel3.zip` |

GitHub 的 **Artifacts 下载下来是一个 ZIP**；只解压这一层，里面那个 `...-AnyKernel3.zip` 才是刷机包。**AK3 ZIP 本身不要再解压。** Artifacts 可能会过期，建议跑完尽快下载。

`AIO-REJ` 是所有 `.rej` 补丁冲突文件的汇总，给维护者排查补丁用，不是内核包；`...-Rejects` 同理，也可以忽略。

如果开了 `build_bypass`，AK3 包里会同时有普通 `Image` 和 `Bypass-Image`。刷入时会让你选：**音量- 普通模式**，**音量+ Bypass 模式**。Bypass 只是放宽内核模块版本校验，**不能隐藏 root，也不是绕过检测**；普通用户不用开。

### 刷入前

1. 确认设备的 KMI 和内核系列都和产物匹配，最好用和设备相同的子版本；不要只看 `6.6.xxx` 这种版本号。
2. 备份当前 boot 镜像，并确认自己能用 fastboot 或 Recovery 刷回去。
3. 用 Recovery，或支持 AnyKernel3 的刷写工具，刷入里面那个 `...-AnyKernel3.zip`。
4. 重启。第一次开机可能比平时久一点。
5. 开机后安装 [BakaSU 管理器](https://github.com/Baka-SU/BakaSU)，确认 root 授权正常。需要 SUSFS 模块时，可以安装发布说明推荐的 [ksu_module_susfs](https://github.com/sidex15/ksu_module_susfs)；需要 NoMount 时安装 [NoMount 模块](https://github.com/maxsteeel/nomount)。

如果开不了机：恢复之前备份的 boot，不要反复刷。把失败的 job 日志、设备型号、内核版本一起发到 [Issues](../../issues)。

> **不要**把产物当 ROM 或 `boot.img` 刷。它只替换内核；用户数据一般不动，但 boot 出问题就会开不了机。

## 参数说明

### 构建范围与发布（仅 Build GKI Kernels）

| 参数 | 默认值 | 怎么填 |
|---|---|---|
| `build_target` | `all` | 选设备对应的一个 KMI；一次只能选一个，多个 KMI 要分开跑。`all` 不要用于一台设备；版本筛选模式下这个参数会被忽略。 |
| `build_kernel_version` | `false` | 勾选后按完整内核版本筛选，只编一个版本；此时 `build_target` 失效。 |
| `kernel_version_filter` | 空 | 填完整版本，如 `6.6.92`；或 `6.6.X` 编该系列（`X` 大小写都行）。只填纯版本号，不要带 `v`、`-android15` 或后缀。不能留空。 |
| `kernel_android_version` | `auto` | 6.x 用 `auto`；5.10 必须选 `android12` / `android13`；5.15 必须选 `android13` / `android14`。 |
| `release_type` | `Actions` | fork 里保持 `Actions`。`Pre-Release` / `Release` 只在发布仓库且未启用版本筛选时创建，fork 不会创建 Release。 |
| `change_log` | 空 | 只在创建发布时用于通知文本，不影响构建。 |

### 通用参数（各工作流都有）

| 参数 | 默认值 | 怎么填 |
|---|---|---|
| `kernelsu_branch` | `main`（留空也按 `main`） | 一般留空或填 `main`。高级用法可以填 BakaSU 分支名或完整 40 位 commit SHA。 |
| `version` | 空 | 可选的内核本地版本后缀。它不是内核版本；填了会改 `uname -r` 后缀，一般留空。 |
| `build_time` | `Sun Dec 01 08:10:00 UTC 2024` | 清空或填 `N` 用当前 UTC 时间；也可以填 `Sun Dec 01 08:10:00 UTC 2024` 这种时间字符串。 |
| `clean_build` | `false` | 勾选后不集成 BakaSU、SUSFS 和可选功能补丁，产物是 `Clean_kernel-...`。要 root 就别勾；`build_bypass` 仍会额外编译一次。 |

### 功能开关

| 参数 | 默认 | 一句话说明 | 什么时候开 / 注意 |
|---|---|---|---|
| `cancel_susfs` | `false` | 勾选表示**关闭** SUSFS | 不勾才有 SUSFS；clean build 和 6.18 会跳过 |
| `use_zram` | `false` | 增强 ZRAM / LZ4KD 算法 | 想要 ZRAM 增强才开；6.12 通过 `build_target` 构建时会强制关闭；6.18 自动跳过 |
| `use_bbg` | `false` | BBG 防格机补丁 | 按需开；不是所有设备都需要 |
| `use_rekernel` | `false` | 实验性 Re-Kernel 驱动 | 一般不用；6.18 自动跳过 |
| `cve_2026_43499_patch` | `true` | 应用 CVE-2026-43499 修复链 | 建议保持开启；6.18 自动跳过 |
| `droidspaces` | `off` | Droidspaces 容器支持 | 不玩容器保持 `off`。5.10–6.6 选 `678` / `123` / `345`，不同设备可能要换槽位试；6.12 / 6.18 任一非 `off` 值都表示开启 |
| `droidspaces_ntsync` | `false` | 给 Droidspaces 加 NTSync | 必须先开 Droidspaces。支持 Android 12/5.10、13/5.15、14/6.1、15/6.6、16/6.12；Android 14/5.15 会失败；Android 13/5.10 和 17/6.18 会跳过 |
| `build_bypass` | `false` | 额外编一个放宽内核模块版本校验的 Image | 会明显增加构建时间；用于诊断模块兼容性，**不是隐藏 root** |
| `use_nomount` | `false` | NoMount 挂载支持 | 还需要单独安装配套的 [NoMount 模块](https://github.com/maxsteeel/nomount)；clean build 跳过 |
| `supp_op` | `false` | 一加 8E（OnePlus 8E）支持 | 主工作流和自定义工作流都有这个开关；6.18 自动跳过 |

> **Android 17 / 6.18 会自动跳过：** SUSFS、ZRAM/LZ4KD、NTSync、Re-Kernel、一加 8E 和 CVE-2026-43499 修复链。BBG 和 Droidspaces 仍可用。构建日志开头的“构建信息摘要”会列出实际启用的功能。

> **Android 16 / 6.12 的 ZRAM：** 通过 `build_target` 构建 6.12 时，ZRAM 会被强制关闭；6.18 自动跳过。6.12 建议保持关闭。

## 工作流清单

| 工作流 | 手动可运行？ | 做什么 |
|---|---|---|
| **Build GKI Kernels** | 可以 | 推荐入口。用版本筛选只编一个版本，或用 `build_target` 选一个 KMI 编整个矩阵。 |
| **Build Custom Android Kernel** | 可以 | 按表单构建一个精确组合（补丁月份 / revision）。 |
| **Update GKI Version Data** | 可以（也会每周自动运行） | 同步 `data/` 下的 GKI 版本数据，不构建内核。 |

补充说明：

- 整个 KMI 的矩阵统一用 **Build GKI Kernels** 的 `build_target`，没有单独的固定 KMI 入口了；`build_target` 一次只能选一个 KMI。
- **Prepare GKI Build Matrix**、**Build KMI Matrix** 和 **Kernel Build Progress** 是被主工作流调用的内部流程，不需要直接运行，也不会出现在可手动运行的工作流列表里。
- 只有 **Build GKI Kernels** 在发布仓库且未启用版本筛选时可能创建 Release；自定义工作流只产出 Artifacts。

## 常见问题

**Actions 里看不到工作流，或没有 Run workflow 按钮？**  
确认 fork 已经启用 Actions，并且默认分支里有 `.github/workflows/` 下的文件。只有 **Build GKI Kernels** 和 **Build Custom Android Kernel** 是手动构建入口，其余是内部流程或数据同步。

**我只想编一个版本，为什么跑了一堆？**  
你可能用了 `build_target` 模式，它会编该 KMI 的所有补丁版本。改用方式 A 的版本筛选。

**版本筛选提示找不到版本？**  
`data/` 里还没有这个子版本。改用 `6.6.X` 编整个系列，或等每周的数据同步。

**构建失败怎么办？**  
打开失败的 job，展开红色步骤看日志。自定义构建先检查 `revision` 是否和 JSON 一致；开了 NTSync 的话，确认没有选 Android 14 / 5.15。矩阵里的其他 job 仍会继续跑。

**主工作流显示失败，但有些版本成功了？**  
矩阵 job 是独立的。回到运行摘要页，下载成功 job 的 Artifacts；失败 job 的日志用来定位具体版本。

**下载的 ZIP 刷不了？**  
先解压 GitHub 下载的外层 ZIP；里面那个 `...-AnyKernel3.zip` 才是刷机包。AK3 ZIP 不要再解压。

**5.15 到底选 `android13-5.15` 还是 `android14-5.15`？**  
看 `uname -r` 里的 `androidXX`，不要看 Android 版本号。Android 14 的手机也可能用 6.1 内核。

**为什么没有 Release？**  
fork、版本筛选模式和自定义工作流都不会创建 Release；从本次运行的 **Artifacts** 下载。只有发布仓库在非筛选模式下才可能创建 Release。

**能同时跑多个构建吗？**  
`Build GKI Kernels` 同一分支的新运行会取消正在跑的上一次。等它结束，或换一个分支跑。

**支持 SukiSU 吗？**  
本仓库固定使用 BakaSU，不提供 SukiSU 选项。

## 可选：伪装 `/proc/config.gz`（Stock Config）

如果仓库里存在 `config/stock_defconfig`，构建会自动用它伪装 `/proc/config.gz`；文件不存在就跳过这一步。clean build 不会应用。

想用的话：从设备当前的官方内核取出 `/proc/config.gz`，解压后放到 `config/`，命名为 `stock_defconfig`，再重新构建。构建日志会显示是否应用成功。

## 维护者：更新 GKI 版本数据

[Update GKI Version Data](.github/workflows/update-gki-data.yml) 每周一 UTC 08:00 自动运行，也可以手动触发。它会运行同步测试、更新 `data/android*/` 下的 JSON、验证所有构建矩阵，并提交数据变更。

普通构建不需要运行它。如果版本筛选一直提示找不到版本，可以先看数据是否已经同步。

## 致谢

- [zzh20188](https://github.com/zzh20188)：曾经的上游 GKI 构建仓库作者；目前此仓库已脱离分支网络，`zzh20188/GKI_KernelSU_SUSFS` 不再是本仓库的上游仓库。
- [coolzyd9107](https://github.com/coolzyd9107)：本仓库维护者。
- [zhuzhuzihan](https://github.com/zhuzhuzihan)：工作流修复及 Telegram Bot 开发与维护。
- [TanakaLun](https://github.com/TanakaLun)：工作流修复与功能改进。
- [YC酱luyancib](https://github.com/luyanci)：Telegram Bot 与构建流程建议。
- [AlexLiuDev233](https://github.com/AlexLiuDev233)：工作流问题修复。
- [cctv18](https://github.com/cctv18)：工作流、6.12 支持及 SUSFS 问题修复建议。

也感谢 BakaSU、SUSFS4KSU、AnyKernel3 等上游项目。

新构建和重要变更通知见 [Telegram 频道](https://t.me/BakaSUKernelBuilds)；BakaSU 官方频道见 [BakaSU_Grp](https://t.me/BakaSU_Grp)。

## License

本项目以 GPL-2.0 发布，详见 [LICENSE](LICENSE)。
