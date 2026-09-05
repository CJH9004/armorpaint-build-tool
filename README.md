# 🛠️ armorpaint-build-tool

## 🇨🇳 中文

### 📖 项目简介

本项目通过 **GitHub Actions** 从 [armory3d/armorpaint](https://github.com/armory3d/armorpaint) 官方源码自动编译 ArmorPaint（3D PBR 纹理绘制工具），并将构建产物发布为本仓库的 **Draft Release（草稿发布）**。

- 支持手动触发构建指定版本 / 指定平台
- 支持一键批量补齐所有缺失的「版本×平台」组合
- **仅构建 26.09 及之后的版本**（原因见下方「版本范围说明」）
- 产物仅以草稿形式发布，仅仓库所有者可见

### ✨ 徽章

> 请将下方链接中的用户名/仓库名替换为你的实际仓库。

![Build and Release](https://github.com/Zhang-Chang-Yuan/armorpaint-build-tool/actions/workflows/build_and_release.yml/badge.svg)

### 🎯 版本范围说明（重要）

本项目**仅构建 [26.09](https://github.com/armory3d/armorpaint/releases/tag/26.09) 及之后发布的版本**，更早的版本（23.09、23.08、21.10 等）一律忽略，原因如下：

1. **构建体系完全不同**：26.09 起 ArmorPaint 已重写为**纯 C 语言**（基于内置的 iron 引擎），仓库自带 `base/make` 构建系统（预编译 `amake` 工具 + QuickJS 构建脚本），仅凭编译器即可完成构建，官方 CI 也仅执行 `cd paint && ../base/make --compile`；
2. **旧版本依赖 Haxe/armorcore 工具链**：23.09 及更早的 tag 为 Haxe + HashLink + armorcore/Kinc 架构，需要 Haxe 编译器、Kha/armorcore 构建流程及对应版本的依赖补丁，与当前流程不兼容，无法在 CI 上直接构建；
3. 版本号采用「年.月」（CalVer）格式，工作流中通过检测对应 tag 的 `paint/project.js` 是否包含 `add_cfiles` 来自动判定版本是否可构建，官方未来发布的新版本（如 26.10）会自动被识别并纳入构建。范围阈值由工作流中的 `MIN_VERSION` 环境变量控制，可自行修改。

### 🚀 快速开始

#### 手动触发

1. 进入仓库主页，点击 **Actions** 标签页；
2. 在左侧工作流列表中选择 **Build and Release ArmorPaint**；
3. 点击右侧 **Run workflow** 按钮；
4. 填写参数：
   - **ArmorPaint 版本 tag**（可选）：如 `26.09`（仅支持 26.09 及以上）。⚠️ 需手动输入版本 tag；**留空则自动构建最新可构建版本**；
   - **平台**：下拉选择 `windows-latest` / `macos-latest` / `ubuntu-latest`，默认 `windows-latest`；
   - **构建全部缺失版本（build_all）**：勾选后一次性把所有缺失的「版本×平台」组合全部构建；
5. 点击 **Run workflow** 开始构建。

#### 批量构建（build_all）

在手动触发时勾选 **build_all**，工作流会对比 [armory3d/armorpaint](https://github.com/armory3d/armorpaint) 官方 tags（仅 26.09 及之后的版本）与本仓库已有 Release（含草稿），把所有缺失的「版本×平台」组合一次性构建补齐。已构建过的组合不会重复构建；再次构建同一产物时会以覆盖方式上传，保证始终是最新构建。

> ℹ️ 本项目**未启用定时构建**（schedule），一切构建均由手动触发。如需每日自动检测新版本，可在 `.github/workflows/build_and_release.yml` 中自行添加 `schedule` 触发器（与 build_all 逻辑完全兼容）。

### 📦 产物说明

| 平台 | 产物 | 内容 |
| --- | --- | --- |
| Windows | `ArmorPaint-<版本>-windows-x64.zip` | `ArmorPaint.exe` + `data/` 资源目录 |
| macOS (Apple Silicon) | `ArmorPaint-<版本>-macos-arm64.zip` | 自包含的 `ArmorPaint.app` |
| Linux | `ArmorPaint-<版本>-linux-x64.tar.gz` | `ArmorPaint` + `data/` 资源目录 |

构建为 Release 配置，与 ArmorPaint 官方 CI 一致（Windows 使用 Visual Studio ClangCL 工具链，Linux 使用 clang，macOS 使用 Xcode）。

### ⚠️ 注意事项

- 📝 **Draft Release 仅仓库所有者/协作者可见**，其他人无法访问；
- ⚖️ ArmorPaint 源码遵循 [zlib 许可证](https://github.com/armory3d/armorpaint/blob/main/license.md)（允许再分发），但**官方以付费形式分发构建产物**以支持项目开发（详见[官方说明](https://github.com/armory3d/armorpaint#readme)）。为尊重其资助模式，本仓库产物仅以草稿形式发布，**请勿公开分发**；
- ⏱️ 注意 GitHub Actions 的分钟数限制（免费额度公共仓库无限，私有仓库有限额）；
- 🕐 单次构建耗时约 **20–60 分钟**（视平台而定）；
- 🪟 Windows 产物动态链接 VC 运行库，若提示缺少 `vcruntime140.dll` 等，请安装 [Visual C++ Redistributable](https://aka.ms/vs/17/release/vc_redist.x64.exe)；
- 🐧 Linux 版为动态链接，运行需要较新发行版及以下库：Vulkan、GTK 3、OpenSSL、ALSA、X11（`libxi` / `libxrandr` / `libxcursor`）；
- 🍎 macOS 版未做公证签名，首次打开若被 Gatekeeper 拦截，请在终端执行 `xattr -cr /Applications/ArmorPaint.app` 后再启动。

### ❓ FAQ

**Q: 为什么只支持 26.09 及以上版本，旧版本能构建吗？**
A: 不能。26.09 起 ArmorPaint 为纯 C 架构并自带 `base/make` 构建系统，可直接用编译器构建；23.09 及更早版本为 Haxe/armorcore 架构，需要完全不同的工具链，无法用当前流程构建。详细原因见「版本范围说明」。

**Q: 构建失败了怎么办？**
A: 可在 Actions 页面对应运行中点击 **Re-run failed jobs** 重试失败的任务；也可直接手动重新触发一次（同名 Release 会先删除再重建，草稿产物则以覆盖方式更新）。

**Q: 为什么版本号需要手动输入，而不是下拉选择？**
A: GitHub Actions 原生不支持在 `workflow_dispatch` 输入中提供动态下拉列表（choice 选项必须写死），因此版本 tag 采用字符串输入，留空即自动使用最新的可构建版本。

**Q: 下载的压缩包解压后怎么运行？**
A: Windows / Linux：将压缩包解压到任意目录，保证 `ArmorPaint(.exe)` 与 `data/` 目录同级，直接运行可执行文件即可；macOS：解压得到 `ArmorPaint.app`，可拖入「应用程序」文件夹后启动。

**Q: 如何下载草稿（Draft）Release？**
A: 登录 GitHub 后，进入仓库主页右侧的 **Releases** 栏，或直接访问 `https://github.com/<用户名>/armorpaint-build-tool/releases` 页面，草稿条目标记为 *Draft*，点击进入后可下载 Assets 中的产物，也可手动将其发布为正式 Release。

### 📄 许可证说明

本仓库**仅包含 CI 脚本与说明文档**，不包含、不分发任何 ArmorPaint 源码或二进制文件。ArmorPaint 源码版权归其原作者所有，遵循 [zlib 许可证](https://github.com/armory3d/armorpaint/blob/main/license.md)。

---

## 🇬🇧 English

### 📖 About

This project uses **GitHub Actions** to automatically build ArmorPaint (a 3D PBR texture painting tool) from the official [armory3d/armorpaint](https://github.com/armory3d/armorpaint) source code and publish the artifacts as **Draft Releases** in this repository.

- Manual trigger with custom version / platform
- One-click batch fill of all missing version × platform combinations
- **Only versions 26.09 and later are built** (see "Supported Version Range" below)
- Artifacts are published as drafts, visible only to the repository owner

### ✨ Badge

> Replace the username/repository in the URLs below with your own.

![Build and Release](https://github.com/Zhang-Chang-Yuan/armorpaint-build-tool/actions/workflows/build_and_release.yml/badge.svg)

### 🎯 Supported Version Range (Important)

This project **only builds versions [26.09](https://github.com/armory3d/armorpaint/releases/tag/26.09) and later**; older versions (23.09, 23.08, 21.10, etc.) are ignored, for the following reasons:

1. **Completely different build system**: since 26.09, ArmorPaint has been rewritten in **pure C** (on top of the built-in iron engine). The repository ships its own `base/make` build system (a prebuilt `amake` tool + QuickJS build scripts), so a plain compiler is all that is needed — the official CI simply runs `cd paint && ../base/make --compile`;
2. **Older versions require the Haxe/armorcore toolchain**: tags 23.09 and earlier use the Haxe + HashLink + armorcore/Kinc architecture with version-specific patches, which is incompatible with the current pipeline and cannot be built on CI directly;
3. Versions follow the year.month (CalVer) scheme. The workflow auto-detects buildable versions by checking whether `paint/project.js` at the tag contains `add_cfiles`, so future official releases (e.g. 26.10) are picked up automatically. The threshold is controlled by the `MIN_VERSION` environment variable in the workflow file.

### 🚀 Quick Start

#### Manual Trigger

1. Go to the repository's **Actions** tab;
2. Select **Build and Release ArmorPaint** from the workflow list;
3. Click **Run workflow**;
4. Fill in the inputs:
   - **ArmorPaint version tag** (optional): e.g. `26.09` (26.09 and later only). ⚠️ Type the tag manually; **leave empty to build the latest buildable version**;
   - **Platform**: choose `windows-latest` / `macos-latest` / `ubuntu-latest` (default: `windows-latest`);
   - **Build all missing versions (build_all)**: check this to build every missing version × platform combination in one run;
5. Click **Run workflow** to start.

#### Batch Build (build_all)

When triggering manually with **build_all** checked, the workflow compares the official [armory3d/armorpaint](https://github.com/armory3d/armorpaint) tags (26.09 and later only) with existing Releases (including drafts) in this repository, then builds every missing version × platform combination in one run. Already-built combinations are not rebuilt; re-built artifacts are uploaded with overwrite so they are always the latest build.

> ℹ️ This project does **not** use scheduled builds (`schedule`) — everything is triggered manually. If you want daily automatic detection of new versions, add a `schedule` trigger to `.github/workflows/build_and_release.yml` yourself (fully compatible with the build_all logic).

### 📦 Artifacts

| Platform | Artifact | Contents |
| --- | --- | --- |
| Windows | `ArmorPaint-<version>-windows-x64.zip` | `ArmorPaint.exe` + `data/` assets folder |
| macOS (Apple Silicon) | `ArmorPaint-<version>-macos-arm64.zip` | Self-contained `ArmorPaint.app` |
| Linux | `ArmorPaint-<version>-linux-x64.tar.gz` | `ArmorPaint` + `data/` assets folder |

Builds use the Release configuration and mirror the official ArmorPaint CI (Visual Studio ClangCL on Windows, clang on Linux, Xcode on macOS).

### ⚠️ Notes

- 📝 **Draft Releases are visible only to the repository owner/collaborators**;
- ⚖️ ArmorPaint source is [zlib-licensed](https://github.com/armory3d/armorpaint/blob/main/license.md) (redistribution is permitted), but the **official binaries are paid** to support the project's development (see the [official note](https://github.com/armory3d/armorpaint#readme)). Out of respect for that funding model, artifacts here are published as drafts only — **please do not distribute them publicly**;
- ⏱️ Be aware of GitHub Actions usage minute limits;
- 🕐 A single build takes about **20–60 minutes** depending on the platform;
- 🪟 The Windows build links the VC runtime dynamically — if `vcruntime140.dll` is reported missing, install the [Visual C++ Redistributable](https://aka.ms/vs/17/release/vc_redist.x64.exe);
- 🐧 The Linux build is dynamically linked and requires a reasonably modern distribution with Vulkan, GTK 3, OpenSSL, ALSA and X11 (`libxi` / `libxrandr` / `libxcursor`);
- 🍎 The macOS build is not notarized. If Gatekeeper blocks the first launch, run `xattr -cr /Applications/ArmorPaint.app` in a terminal.

### ❓ FAQ

**Q: Why are only 26.09+ supported? Can older versions be built?**
A: No. Since 26.09, ArmorPaint is pure C and ships its own `base/make` build system, so it builds with a plain compiler. Versions 23.09 and earlier use the Haxe/armorcore architecture requiring a completely different toolchain. See "Supported Version Range" above.

**Q: What if a build fails?**
A: On the Actions run page, click **Re-run failed jobs** to retry, or simply trigger a manual run again (a published Release with the same tag is deleted and recreated; draft artifacts are overwritten in place).

**Q: Why do I have to type the version instead of picking it from a dropdown?**
A: GitHub Actions does not support dynamic option lists in `workflow_dispatch` inputs (choices must be static), so the version is a string input. Leaving it empty builds the latest buildable version automatically.

**Q: How do I run the downloaded archive?**
A: Windows / Linux: extract anywhere, keep `ArmorPaint(.exe)` and the `data/` folder side by side, and run the executable. macOS: extract `ArmorPaint.app` and optionally move it to the Applications folder.

**Q: How do I download a Draft Release?**
A: Sign in to GitHub, then open the **Releases** section on the repository page or visit `https://github.com/<user>/armorpaint-build-tool/releases`. Draft entries are labeled *Draft*; open one to download the assets, or publish it as a full release manually.

### 📄 License Note

This repository **contains only CI scripts and documentation** — no ArmorPaint source code or binaries are included or distributed. ArmorPaint source code is copyrighted by its authors and licensed under the [zlib license](https://github.com/armory3d/armorpaint/blob/main/license.md).
