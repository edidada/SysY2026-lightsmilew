# SysY 编译器（lightsmilew）— 中文说明

> 上游：`lightsmilew/SysyCompiler`，2026 年全国大学生计算机系统能力大赛编译系统设计赛**国一等奖**（2025 二等奖）。RISC-V 后端，支持扩展文法与 RVV 向量化。本仓库为其镜像，仅新增 CI workflow 与中文说明。

## 版本对齐 SysY2026

- 项目 `docs/设计文档.md`、`docs/扩展文法与中端改造指南.md` 描述前端、IR、优化、后端全链路；`docs/RVV.md` 与测试脚本 `performance2026` 对应 2026 年向量类型目标。
- 目标平台：RISC-V 64GC（BOOM v3），支持 RVV 向量扩展（`-rvv`）。
- 前端：ANTLR **4.13.2**（生成的 `SysYLexer.cpp` / `SysYParser.cpp` 已在 `myCompiler/src/frontend/generate/`），语法文件 `SysY.g4`。

## 构建方法

ANTLR4 C++ runtime 在上游被 `.gitignore` 忽略（`myCompiler/3rd_party/`），需自行下载 4.13.2 runtime 放到 `myCompiler/3rd_party/antlr4-runtime/`。

```bash
sudo apt install -y build-essential cmake ninja-build openjdk-17-jre-headless unzip wget
mkdir -p myCompiler/3rd_party/antlr4-runtime
cd myCompiler/3rd_party/antlr4-runtime
wget -O rt.zip https://www.antlr.org/download/antlr4-cpp-runtime-4.13.2-source.zip
unzip -q rt.zip                      # CMakeLists.txt、runtime/ 等直接在当前目录
cd ../../..

cmake -S myCompiler -B build -DCMAKE_BUILD_TYPE=Release -DANTLR_BUILD_CPP_TESTS=OFF
cmake --build build -j4              # -> build/my_compiler
```

**注意**：上游 `myCompiler/CMakeLists.txt` 链接了过时 target `antlr4_runtime`，而 4.13.2 runtime 定义的是 `antlr4_static`。CI 里用 `sed -i 's/antlr4_runtime/antlr4_static/g' myCompiler/CMakeLists.txt` 修正；本地构建同样需要这一步（或用 `-D` alias）。

运行脚本：`scripts/run.sh -build`、`-ir`、`-riscv -O17 -rvv`、`-qemu-test-rvv`。

## CI 状态

- GitHub Actions：`.github/workflows/build.yml`，装依赖 → 下载 ANTLR runtime → 修正 target 名 → CMake 构建 → 冒烟测试。
- 状态：✅ 通过（run `44905001612`，约 3m30s）。
- Gitee 镜像：`https://gitee.com/edidada/SysY2026-lightsmilew`（`main`）。
- 本地：未编译，全部交给 CI。
