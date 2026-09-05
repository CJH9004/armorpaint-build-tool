任务目标
在当前工作区文件夹下，利用其文件夹名称作为仓库名，在 GitHub 上新建一个仓库。为该仓库配置 GitHub Actions 自动化工作流，实现从 armory3d/armorpaint 官方源码编译构建，并发布到 Releases。参考项目：

https://github.com/Zhang-Chang-Yuan/aseprite-build-tool（同型项目：Aseprite 自动构建）

https://github.com/armory3d/armorpaint（官方仓库，其 .github/workflows 即官方 CI）

功能需求
手动触发构建：工作流支持 workflow_dispatch，提供以下输入：

版本选择：输入 ArmorPaint 官方版本 tag（CalVer「年.月」格式，如 26.09；留空则自动构建最新可构建版本）。

平台选择：下拉选项为 Windows、macOS、Linux (Ubuntu)，实际对应 windows-latest、macos-latest、ubuntu-latest。

build_all：勾选后一次性补齐所有缺失的「版本×平台」组合。

自动定时构建：与参考项目保持一致，默认不启用 schedule；如需每日自动检测新版本可自行添加（cron 例如 UTC 2:00），与 build_all 逻辑完全兼容。

重复构建覆盖：如果同一版本已存在 Release，再次构建（手动或自动）时，若是正式 Release 则先删除（含 Tag）再重建；若是草稿则以 --clobber 覆盖上传产物，确保始终为最新构建。

产物管理：构建后的压缩包（.zip 或 .tar.gz）上传至 Draft Release（草稿），仅仓库所有者可见。ArmorPaint 源码为 zlib 许可（允许再分发），但官方以付费形式分发构建产物以支持开发，为尊重其资助模式，本仓库产物仅以草稿形式发布。

说明文档：在仓库根目录创建 README.md，包含中英文双语的操作步骤、注意事项和常见问题。

关键调研结论（工作流实现的依据）
构建系统：ArmorPaint 自 26.09 起重写为纯 C（iron 引擎，基于 Kore），仓库自带 base/make 构建系统（预编译 amake 工具 + QuickJS 脚本 base/tools/make.js），无外部依赖需手动准备（无需 Haxe/Kinc）。

构建命令：与官方 CI 完全一致 —— cd paint && ../base/make --compile（Release 配置）。

平台工具链：Windows 使用 VS 的 ClangCL 工具集（windows-latest 自带，经 vswhere 定位 vcvars64.bat 后由 MSBuild 编译）；Linux 使用 clang + Makefile；macOS 使用 Xcode（xcodebuild）。仅 Linux 需安装依赖：make clang libvulkan-dev libgtk-3-dev libssl-dev libxi-dev libxrandr-dev libxcursor-dev libasound2-dev（来源：官方 CI 与 base/docs/linux_deps.md）。

产物路径（工作流打包步骤的依据，均以 armorpaint 仓库根为基准）：

Windows / Linux：paint/build/out/ 目录即完整可运行包（ArmorPaint.exe / ArmorPaint + data/ 资源目录）。程序按可执行文件所在目录加载 data（base/sources/iron.h 的 kickstart 与 base/sources/iron_file.c）。

macOS：paint/build/build/Release/ArmorPaint.app，自包含（Xcode Resources 构建阶段会把 paint/build/out 作为文件夹引用复制进 Contents/Resources/out/）。

Windows 上 amake 的 os_exec 会永久 chdir（base/tools/amake/main.c 中 _chdir），故编译后 exe 被复制到 paint/build/out/，与资源同目录。工作流打包时仍做了容错查找（find），以兼容构建脚本行为变化。

版本范围：仅 26.09 及之后（CalVer YY.MM 格式）。更早 tag（23.09、23.08、21.10、20.01、19.08、18.x）为 Haxe + HashLink + armorcore/Kinc 架构，构建体系完全不同，无法用当前流程构建。判定方法：对应 tag 的 paint/project.js 含 add_cfiles 即为纯 C 架构（可构建）；阈值由 MIN_VERSION 环境变量控制。

执行步骤
1. 获取仓库名称
获取当前工作区文件夹的绝对路径，取其最后一级目录名作为新仓库名称（本仓库为 armorpaint-build-tool）。

2. 在 GitHub 上创建仓库
通过 GitHub API（POST /user/repos）使用本机 Git 凭据管理器中已存储的账号凭据（Zhang-Chang-Yuan）创建空仓库（不初始化 README、.gitignore 等），随后由纯 git 操作完成推送，不依赖浏览器。

3. 初始化本地 Git 仓库
在工作区执行：

git init
git remote add origin https://github.com/Zhang-Chang-Yuan/armorpaint-build-tool.git
4. 创建工作流文件
在 .github/workflows/build_and_release.yml 中写入完整 YAML（见仓库内该文件）。

5. 创建 README.md
在仓库根目录创建中英双语 README.md（见仓库内该文件）。

6. 提交并推送
git add .
git commit -m "Initial commit: ArmorPaint auto-builder workflow"
git push -u origin main
7. 启用 Actions
推送后，GitHub Actions 将自动启用。工作流仅含 workflow_dispatch 触发器，无需额外设置。
