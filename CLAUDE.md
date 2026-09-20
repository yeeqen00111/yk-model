# yk-model — Claude Code 项目引导（环境就绪用）

**目标**：让 AI 模型在普通笔记本上跑得动、跑得快、装得上。本机（T0 档）即最低配验收机。
**路线**：**引擎全自研 + 模型全自研**（不依赖 llama.cpp / bitnet.cpp 等现成推理框架与第三方发布权重；论文算法可参考，代码自写）。模型架构 = **YkNet**（§6），推理引擎 = **YkEngine**（§6A），由接口契约驱动。详见蓝图第 0、5、6、6A、11 节。
**当前阶段**：蓝图已按全自研改向，P0（自研最小推理链骨架）待开工。

## 必读文档（按需进，别重复加载）

- `docs/architecture-blueprint.md` — 怎么做 + 决策日志（**改设计先改这里，再动代码**）
- `docs/research-notes.md` — 为什么信（证据库：哪些可信、哪些开放）
- `docs/benchmark-methodology.md` — 怎么证明（**性能结论必须过它的口径**）
- `docs/dev-environment.md` — 环境实测手册（命令全集、验证方法）
- `docs/self-check.md` — **自检工序**（每次改动后交付前必过）

## 交付前必做：自检工序

- **改动完成 → 提交前 → 过一遍 `docs/self-check.md`** 全清单（残留扫描/入库验证/交叉引用/决策对齐/事实一致/提交纪律）。
- self-check 里抓到的真 bug（如 .gitignore 误伤源码目录）**必须修复后提交**，不允许带着已知不一致交付。

## 环境铁律（违反即坑）

- 平台：Windows 11，Shell = **Git Bash**（POSIX 语法，不用 cmd/PowerShell 语法）。
- C: 仅剩 ~16GB：**绝不往 C: 写任何东西**。代码、conda、模型、缓存全在 D:。
- **本机没有任何 C/C++ 编译器，且用户明示不装 MSVC。** 引擎是自研 C/C++，需编译环境 → 用 **Zig**（自带 clang，~50MB）或 **WinLibs MinGW-w64**（~1GB）装 D:，编译产物输出到 D:。**AVX-512 是本机 CPU i7-1165G7 的本命指令集**，内核实现优先吃满它。
- Python 3.11 由 **conda** 管理。Miniconda 位于 `D:/application/env/miniconda3`（在 PATH 中）。本项目用独立 env `yk-model`（遵循蓝图 9.3）。
- **自研边界**：不引用任何第三方推理框架代码与发布权重作为运行时；调研中的好消息（查表内积、三值化等）只作为算法参考落地为自己的实现。

## 命令速查（Git Bash）

```bash
# 一次性执行（不依赖 activate，最稳）
D:/application/env/miniconda3/Scripts/conda.exe run -n yk-model <cmd>

# 交互式进入 env
source D:/application/env/miniconda3/etc/profile.d/conda.sh && conda activate yk-model

# 验证确实进了项目 env（必须指向 env 内 python）
python -V && python -c "import sys; print(sys.executable)"
```

## 硬约束 / 雷区

1. 性能陈述必须以 `benchmark-methodology.md` 口径，自带自检：**单通道带宽 ≤25.6GB/s → decode 上限 ≈ 25.6/模型字节数 (GB/s→tok/s)**。测出超上限先怀疑指标错，不是机器变强。
2. 模型文件大且易混：报数必须带 **自研引擎 commit + 模型 SHA256 + 量化档 + 上下文长度** 四件套。
3. 别用 hermes 的 venv、别污染其他 conda env；本项目的一切装进 `yk-model`。
4. 用户决策要补记到蓝图**第 11 节决策日志**。

## 协作约定（vibe coding）

- 开工状态不明时先 `git status` 与看 README 进度；代码未提交前保持在 D: 项目内。
- 与用户以中文交流。