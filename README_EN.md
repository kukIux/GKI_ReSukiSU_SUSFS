<div align="center">

# GKI BakaSU SUSFS

Build Android GKI kernels in the cloud with GitHub Actions. The default build includes **BakaSU** (root) and **SUSFS** (hides root traces).

[简体中文](README.md) / English

[![Release](https://img.shields.io/github/v/release/coolzyd9107/GKI_BakaSU_SUSFS?label=Release&style=flat-square&logo=github&logoColor=white&color=2ea44f)](https://github.com/zhuzhuzihan/GKI_BakaSU_SUSFS/releases)
[![Build GKI Kernels](https://github.com/zhuzhuzihan/GKI_BakaSU_SUSFS/actions/workflows/main.yml/badge.svg)](https://github.com/zhuzhuzihan/GKI_BakaSU_SUSFS/actions/workflows/main.yml)
[![Telegram](https://img.shields.io/static/v1?label=Telegram&message=Channel&color=0088cc)](https://t.me/BakaSUKernelBuilds)
[![BakaSU](https://img.shields.io/badge/KernelSU-BakaSU-5AA300?style=flat-square)](https://github.com/Baka-SU/BakaSU)
[![SUSFS](https://img.shields.io/badge/Filesystem-SUSFS-E67E22?style=flat-square)](https://gitlab.com/simonpunk/susfs4ksu)

</div>

> **First things first:** the output is a flashable **AnyKernel3 kernel ZIP**, not a ROM or a full firmware package. Flashing the wrong kernel can prevent your phone from booting. Back up your current boot image and make sure you know how to restore it before flashing.
>
> **In a hurry?** Fork → Actions → Build GKI Kernels → check `build_kernel_version`, enter your full kernel version (for example `6.6.92`; for 5.10/5.15 also choose `kernel_android_version`), clear `build_time` → Run. Download from Artifacts when it finishes. If this is your first time, read the KMI section and the five mistakes below first.

## Six terms in plain English

| Term | What it means |
|---|---|
| **GKI** | Google's Generic Kernel Image. It is what this project builds. |
| **KMI** | Kernel Module Interface: the compatibility level between the kernel and your phone's vendor modules. A wrong KMI can break the camera or Wi-Fi, or stop the phone from booting. |
| **BakaSU** | A KernelSU fork that provides root. |
| **SUSFS** | Kernel patches that hide root and filesystem traces. It does not guarantee passing every detection. |
| **AnyKernel3** | A flashable kernel ZIP format. This is the actual output, not a `boot.img`. |
| **Artifacts** | Files produced by an Actions run. GitHub wraps them in an outer ZIP, so you need to extract that layer first. |

## Before you start: identify your KMI (most important)

A KMI is the combination of an **Android version and a kernel series**, for example `android15-6.6`. Your build target must match the KMI your phone actually uses.

**Do not choose by the Android version alone.** An Android 14 phone may run kernel 5.15 or 6.1, and Android 13 and Android 14 phones can both run a 5.15.x kernel. The reliable clue is the `androidXX` tag and the `5.xx` / `6.xx` series in the kernel version string.

How to check:

1. On the phone: **Settings → About phone → Kernel version**;
2. Or run `uname -r` in a terminal or through adb;
3. Or ask the ROM / device maintainer.

| Android version | Kernel series | Common tag in the kernel version | `build_target` in the main workflow |
|---|---|---|---|
| Android 12 | 5.10 | `android12` | `android12-5.10` |
| Android 13 | 5.10 | `android13` | `android13-5.10` |
| Android 13 | 5.15 | `android13` | `android13-5.15` |
| Android 14 | 5.15 | `android14` | `android14-5.15` |
| Android 14 | 6.1 | `android14` | `android14-6.1` |
| Android 15 | 6.6 | `android15` | `android15-6.6` |
| Android 16 | 6.12 | `android16` | `android16-6.12` |
| Android 17 | 6.18 | `android17` | `android17-6.18` |

Examples:

- `5.15.94-android13-8-g...` → series 5.15, Android 13 → `android13-5.15`
- `5.15.110-android14-11-...` → series 5.15, Android 14 → `android14-5.15`
- `6.6.92-android15-8-...` → series 6.6, Android 15 → `android15-6.6`

> **Note:** both 5.10 and 5.15 map to two different Android KMIs. `5.15.xxx` alone cannot tell you whether the device is Android 13 or 14; look at the `androidXX` part. If a third-party kernel removed that tag, ask the ROM / device maintainer instead of guessing.

## Five common mistakes

| Mistake | What happens | What to do instead |
|---|---|---|
| Leaving `build_target: all` | It expands to every KMI. That is well over a hundred jobs and takes a long time. | For one phone, pick one KMI, or use the version filter below. |
| Not changing `build_time` | The form is prefilled with `Sun Dec 01 08:10:00 UTC 2024`; leaving it unchanged writes that fixed timestamp into the kernel. | Clear the field, or enter `N` to use the current UTC time at run time. |
| Not clearing `revision` in a custom build | The form defaults to `r11`. If the JSON entry has no revision, the build looks for a tag that does not exist and usually fails. | Clear `revision` when the JSON entry has none; otherwise enter the exact value. |
| Checking `clean_build` | The build skips BakaSU and SUSFS, so the result has no root. It is not "cleaner" or "more stable"; it simply has no root. | Leave it unchecked if you want root. |
| Starting another run on the same branch | A new `Build GKI Kernels` run cancels the run that is still in progress. | Wait for the current run to finish, or use another branch. |

## Three ways to build: pick one

| What you want | Where to go | Jobs produced |
|---|---|---|
| Build **one exact kernel version** for your phone (recommended) | **Build GKI Kernels** + version filter (full version) | 1 |
| Build **every patch version** for one KMI | **Build GKI Kernels** + `build_target` | Many |
| Pick a **specific patch month / revision** | **Build Custom Android Kernel** | 1 |

These three options are explained below.

## Option A: build one exact kernel version (recommended)

Use this if you know your full kernel version, for example `uname -r` shows `6.6.92`, and you only want that version.

1. **Fork this repository.** Click **Fork** in the top-right corner and work in your own fork. This avoids occupying the upstream build queue.
2. **Open the workflow.** Go to **Actions** in your fork. If GitHub asks you to enable workflows, enable them first. Select **Build GKI Kernels** on the left, then click **Run workflow**.
3. **Fill in the form as follows.**

| Field | What to enter | Example (device kernel 6.6.92) |
|---|---|---|
| `build_kernel_version` | Check it (true) to use the version filter | Checked |
| `kernel_version_filter` | The full kernel version | `6.6.92` |
| `kernel_android_version` | For 6.x choose `auto`; for 5.10 choose `android12` / `android13`; for 5.15 choose `android13` / `android14` | `auto` |
| `build_target` | Ignored in this mode; leaving the default `all` will not build everything | `all` |
| `build_time` | Clear it, or enter `N` | Empty |
| `release_type` | Keep `Actions` in a fork | `Actions` |
| Other feature switches | Keep the defaults for the first build | Defaults |

4. **Click Run workflow.** This mode produces **exactly one job**, usually named like `6.6.92-android15-2025-07`.
5. **Download the output.** When the run finishes, open **Artifacts** at the bottom of the run page and download `BakaSU_kernel-...`. Extract the outer ZIP and flash the `...-AnyKernel3.zip` inside. See [After the build](#after-the-build-download-extract-flash).

> **Why can `build_target` be ignored?** Once `build_kernel_version` is checked, the main workflow takes the version-filter path and does not use `build_target`. Leaving it at `all` will not trigger a full build.

If you see `kernel version not found in JSON`, the `data/` files do not contain that sublevel yet. Build the whole series with `6.6.X`, or wait for the weekly data sync. Do not guess a version that is not in the data.

If the same sublevel appears more than once in the JSON (common on 5.10 / 5.15), the filter uses the **first** record. To select a later patch month or a specific revision, use Option C.

## Option B: build every patch version for one KMI

Use this if you are not sure which sublevel you want, or you want several kernels for one KMI in one go.

1. Fork this repository, then open **Actions → Build GKI Kernels → Run workflow**.
2. Set `build_target` to your KMI (see the table above). Clear `build_time` or enter `N`. Leave the other fields at their defaults.
3. Click **Run workflow**.
4. When it finishes, download the packages you need from **Artifacts**.

> **Do not choose `all`.** It expands every KMI and currently produces well over a hundred jobs. For one phone, one KMI is enough.

Each patch version is a separate job, and the jobs do not depend on each other. If one fails, the others continue. The 5.10 matrix keeps every monthly record, so it produces more jobs; 5.15 and 6.x keep only the first record for a duplicated sublevel.

If you only need one sublevel, do not use this option. Go back to Option A or use Option C.

## Option C: choose a specific patch month or revision

Use this when:

- the same 5.10 / 5.15 sublevel has multiple revisions and you need a specific one;
- you know the exact `date` (patch month) you need;
- you want `sub_level` to be `X` and follow the current source of the selected branch.

This uses **Build Custom Android Kernel**. It builds one combination per run and does not create a Release.

### Look up the data JSON first

Open the matching file, for example [data/android15/6.6.json](data/android15/6.6.json). Each object under `entries` looks like this:

```json
{ "date": "2025-07", "kernel": "6.6.92" }
```

5.10 / 5.15 entries may also have a `revision`:

```json
{ "date": "2023-06", "kernel": "5.15.110", "revision": "r11" }
```

The top-level `lts` field is a separate LTS target. `date` is the release month of the Google GKI branch; it is not necessarily the full security patch date shown in Android settings.

### Fill in the form

| Target | `android_version` | `kernel_version` | `sub_level` | `os_patch_level` | `revision` |
|---|---|---|---|---|---|
| Android 15 / 6.6.92, 2025-07 | `android15` | `6.6` | `92` | `2025-07` | Leave empty |
| Android 14 / 5.15.110, 2023-06 | `android14` | `5.15` | `110` | `2023-06` | `r11` |

Steps:

1. Open **Actions → Build Custom Android Kernel → Run workflow**.
2. `android_version` and `kernel_version` must be a supported pair, for example `android15` + `6.6`.
3. `sub_level` is only the last component: `6.6.92` → `92`, `5.15.110` → `110`.
4. `os_patch_level` is the JSON `date` in `YYYY-MM` format. For an LTS target, enter `lts`.
5. `revision` must match the JSON exactly. If the entry has no `revision`, clear it, **including the prefilled `r11`**.
6. Enable `supp_op` if you need OnePlus 8E support; enable other features as needed.
7. Click **Run workflow** and download the result from that run's **Artifacts**. This workflow does not create a Release.

> **`revision` is the easiest thing to get wrong.** The form's `r11` is only an example. The current 6.1 / 6.6 / 6.12 / 6.18 data has no revisions, so the field must be empty. A wrong value makes the build look for a non-existent upstream tag and usually fails.

For LTS: set `os_patch_level` to `lts` and normally leave `revision` empty (check `lts_revision` in the JSON; if it is absent, leave it empty).

`sub_level` also accepts `X`, which follows the current source of the selected branch. The artifact name will contain `X`, which is less precise than a numeric version; use a number when possible.

## After the build: download, extract, flash

### Find the right file

Actions artifact names usually look like this:

| Case | Artifact name | After extracting the outer ZIP |
|---|---|---|
| Normal build | `BakaSU_kernel-android15-6.6.92-2025-07` | `android15-6.6.92-2025-07-BakaSU-AnyKernel3.zip` |
| With revision | `BakaSU_kernel-android13-5.10.198-2023-12-r7` | `android13-5.10.198-2023-12-r7-BakaSU-AnyKernel3.zip` |
| Clean build | `Clean_kernel-android15-6.6.92-2025-07` | `android15-6.6.92-2025-07-Clean-AnyKernel3.zip` |

The **Artifacts download is always a ZIP**. Extract only that outer layer; the `...-AnyKernel3.zip` inside is the flashable package. **Do not extract the AK3 ZIP itself.** Artifacts may expire, so download them soon after the run.

`AIO-REJ` is a collection of `.rej` patch-conflict files for maintainers, not a kernel package. The `...-Rejects` artifacts are the same thing and can be ignored.

If `build_bypass` is enabled, the AK3 ZIP contains both the normal `Image` and a `Bypass-Image`. The installer asks you to choose: **Volume Down = Normal mode**, **Volume Up = Bypass mode**. Bypass only relaxes kernel module version checks; it **does not hide root and does not bypass detection**. Normal users do not need it.

### Before flashing

1. Confirm that the KMI and kernel series match the artifact; the same sublevel as your device is best. Do not judge compatibility from a version number such as `6.6.xxx` alone.
2. Back up your current boot image and make sure you can restore it with fastboot or Recovery.
3. Flash the inner `...-AnyKernel3.zip` with Recovery or an AnyKernel3-compatible kernel flasher.
4. Reboot. The first boot may take a little longer than usual.
5. After booting, install the [BakaSU manager](https://github.com/Baka-SU/BakaSU) and confirm root works. If you need the SUSFS module, you can install the [ksu_module_susfs](https://github.com/sidex15/ksu_module_susfs) recommended in the release notes; if you need NoMount, install the [NoMount module](https://github.com/maxsteeel/nomount).

If the phone does not boot, restore the boot image you backed up and do not keep re-flashing. Post the failed job log, device model, and kernel version in [Issues](../../issues).

> **Do not** flash the output as a ROM or a `boot.img`. It only replaces the kernel. User data is normally untouched, but a bad boot image will prevent the phone from starting.

## Input reference

### Build scope and release (Build GKI Kernels only)

| Input | Default | How to use it |
|---|---|---|
| `build_target` | `all` | Select the one KMI for your device; it is single-select, so run the workflow separately for additional KMIs. Do not use `all` for a single phone. This is ignored in version-filter mode. |
| `build_kernel_version` | `false` | Check it to filter by full kernel version and build only one version; `build_target` is then ignored. |
| `kernel_version_filter` | Empty | A full version such as `6.6.92`, or `6.6.X` for the whole series (`X` is case-insensitive). Enter the bare version only: no `v`, no `-android15`, no suffix. It cannot be empty. |
| `kernel_android_version` | `auto` | Use `auto` for 6.x. For 5.10 choose `android12` / `android13`; for 5.15 choose `android13` / `android14`. |
| `release_type` | `Actions` | Keep `Actions` in a fork. `Pre-Release` / `Release` only publish from the release repository and only when version filtering is off; a fork does not create a Release. |
| `change_log` | Empty | Used only in release notification text; it does not affect the build. |

### Common inputs (all workflows)

| Input | Default | How to use it |
|---|---|---|
| `kernelsu_branch` | `main` (blank also means `main`) | Normally leave it blank or enter `main`. Advanced users can enter a BakaSU branch name or a full 40-character commit SHA. |
| `version` | Empty | Optional local kernel version suffix. It is not the kernel version; it changes the `uname -r` suffix. Usually leave it empty. |
| `build_time` | `Sun Dec 01 08:10:00 UTC 2024` | Clear it or enter `N` to use the current UTC time; or enter a timestamp string such as `Sun Dec 01 08:10:00 UTC 2024`. |
| `clean_build` | `false` | Skips BakaSU, SUSFS, and optional feature patches; the artifact is named `Clean_kernel-...`. Do not check it if you want root. `build_bypass` still triggers an extra compilation. |

### Feature switches

| Input | Default | In one sentence | When to enable / notes |
|---|---|---|---|
| `cancel_susfs` | `false` | Checking it **disables** SUSFS | Leave it unchecked to keep SUSFS. Clean builds and 6.18 skip SUSFS. |
| `use_zram` | `false` | Enhanced ZRAM / LZ4KD algorithms | Enable only if you want the ZRAM enhancements. When building 6.12 via `build_target` it is forced off; 6.18 skips it. |
| `use_bbg` | `false` | BBG anti-brick patch | Enable only if needed; not every device requires it. |
| `use_rekernel` | `false` | Experimental Re-Kernel drivers | Usually unnecessary; 6.18 skips it. |
| `cve_2026_43499_patch` | `true` | Applies the CVE-2026-43499 fix series | Recommended to keep on; 6.18 skips it. |
| `droidspaces` | `off` | Droidspaces container support | Leave `off` unless you use containers. For 5.10–6.6 choose `678` / `123` / `345`; different devices may need different slots. For 6.12 / 6.18 any non-`off` value enables it. |
| `droidspaces_ntsync` | `false` | Adds NTSync to Droidspaces | Requires Droidspaces to be enabled. Supported: Android 12/5.10, 13/5.15, 14/6.1, 15/6.6, 16/6.12. Android 14/5.15 fails; Android 13/5.10 and 17/6.18 skip it. |
| `build_bypass` | `false` | Builds an extra Image with relaxed kernel module version checks | Significantly increases build time. It is for diagnosing module compatibility, **not for hiding root**. |
| `use_nomount` | `false` | NoMount mount support | Also requires the separate [NoMount module](https://github.com/maxsteeel/nomount). Clean builds skip it. |
| `supp_op` | `false` | OnePlus 8E support | Both the main and custom workflows expose it; 6.18 skips it. |

> **Android 17 / 6.18 automatically skips:** SUSFS, ZRAM/LZ4KD, NTSync, Re-Kernel, OnePlus 8E, and the CVE-2026-43499 fix series. BBG and Droidspaces remain available. The "build information summary" at the start of each build log lists the features that are actually enabled.

> **Android 16 / 6.12 and ZRAM:** when building 6.12 via `build_target`, ZRAM is forced off; 6.18 skips it. Keeping it off is recommended for 6.12.

## Workflow reference

| Workflow | Manual run? | What it does |
|---|---|---|
| **Build GKI Kernels** | Yes | Recommended entry point. Build one version with the version filter, or pick one KMI with `build_target` and build its full matrix. |
| **Build Custom Android Kernel** | Yes | One exact combination from the form (patch month / revision). |
| **Update GKI Version Data** | Yes (also runs weekly) | Syncs the GKI version data under `data/`; it does not build kernels. |

Notes:

- Full-KMI builds now go through **Build GKI Kernels** and `build_target`; there are no separate fixed-KMI entries. `build_target` accepts one KMI per run.
- **Prepare GKI Build Matrix**, **Build KMI Matrix**, and **Kernel Build Progress** are internal workflows called by the main workflow. They do not need to be run directly and do not appear in the manual workflow list.
- Only **Build GKI Kernels** can create a Release, and only in the release repository with the version filter off. The custom workflow only produces Artifacts.

## Troubleshooting

**The workflow is missing from Actions, or there is no Run workflow button.**  
Make sure Actions is enabled in your fork and the default branch contains the files under `.github/workflows/`. Only **Build GKI Kernels** and **Build Custom Android Kernel** are manual build entry points; the rest are internal workflows or data sync.

**I only wanted one version, but many jobs started.**  
You probably used `build_target` mode, which builds every patch version for that KMI. Use the version filter in Option A instead.

**The version filter says the kernel version was not found.**  
That sublevel is not in `data/` yet. Use `6.6.X` for the whole series, or wait for the weekly data sync.

**A build failed.**  
Open the failed job and expand the red step to read the log. For a custom build, check `revision` against the JSON first. If NTSync is enabled, make sure you did not select Android 14 / 5.15. Other matrix jobs will keep running.

**The main workflow is marked failed, but some versions succeeded.**  
Matrix jobs are independent. Go back to the run summary and download Artifacts from the successful jobs; use the failed job logs to identify the affected version.

**The downloaded ZIP cannot be flashed.**  
Extract the outer ZIP downloaded from GitHub first; the `...-AnyKernel3.zip` inside is the flashable package. Do not extract the AK3 ZIP.

**Should I choose `android13-5.15` or `android14-5.15`?**  
Look at the `androidXX` part of `uname -r`, not the Android version. An Android 14 phone may also use a 6.1 kernel.

**Why is there no Release?**  
Forks, version-filtered builds, and custom builds do not create a Release. Download from **Artifacts** for that run. Only the release repository in non-filtered mode may create one.

**Can I run several builds at once?**  
A new `Build GKI Kernels` run on the same branch cancels the one already running. Wait for it to finish, or use another branch.

**Does this support SukiSU?**  
No. This repository builds BakaSU only and does not expose a SukiSU option.

## Optional: spoof `/proc/config.gz` (Stock Config)

If `config/stock_defconfig` exists in the repository, the build uses it to spoof `/proc/config.gz`; if the file does not exist, the step is skipped. Clean builds do not apply it.

To use it, obtain `/proc/config.gz` from the device's current stock kernel, decompress it, place it in `config/`, and rename it to `stock_defconfig`. Rebuild afterwards. The build log shows whether the step was applied.

## Maintainers: update GKI version data

[Update GKI Version Data](.github/workflows/update-gki-data.yml) runs every Monday at 08:00 UTC and can also be triggered manually. It runs the sync tests, updates the JSON files under `data/android*/`, validates all build matrices, and commits the data changes.

Normal builds do not need to run it. If the version filter keeps reporting a missing version, check whether the data has been synced.

## Credits

- [zzh20188](https://github.com/zzh20188): author of the former upstream GKI build repository. This repository is no longer part of that fork network, and `zzh20188/GKI_KernelSU_SUSFS` is no longer its upstream repository.
- [coolzyd9107](https://github.com/coolzyd9107): maintainer of this repository.
- [zhuzhuzihan](https://github.com/zhuzhuzihan): workflow fixes and Telegram Bot development and maintenance.
- [TanakaLun](https://github.com/TanakaLun): workflow fixes and feature improvements.
- [YC酱luyancib](https://github.com/luyanci): Telegram Bot and build workflow suggestions.
- [AlexLiuDev233](https://github.com/AlexLiuDev233): workflow fixes.
- [cctv18](https://github.com/cctv18): workflow, 6.12 support, and SUSFS troubleshooting suggestions.

Thanks also to the BakaSU, SUSFS4KSU, AnyKernel3, and other upstream projects.

See the [Telegram channel](https://t.me/BakaSUKernelBuilds) for new builds and important updates. The official BakaSU channel is [BakaSU_Grp](https://t.me/BakaSU_Grp).

## License

This project is released under GPL-2.0. See [LICENSE](LICENSE).
