# yk-model — Claude Code 项目引导（环境就绪用）

**目标**：让 AI 模型在普通笔记本上跑得动、跑得快、装得上。本机（T0 档）即最低配验收机。
**当前阶段**：蓝图/证据/基准口径已冻结，P0 未开工。vibe coding 从这里推进。

## 必读文档（按需进，别重复加载）

- `docs/architecture-blueprint.md` — 怎么做 + 决策日志（**改设计先改这里，再动代码**）
- `docs/research-notes.md` — 为什么信（证据库：哪些可信、哪些开放）
- `docs/benchmark-methodology.md` — 怎么证明（**性能结论必须过它的口径**）
- `docs/dev-environment.md` — 环境实测手册（命令全集、验证方法）

## 环境铁律（违反即坑）

- 平台：Windows 11，Shell = **Git Bash**（POSIX 语法，不用 cmd/PowerShell 语法）。
- C: 仅剩 ~16GB：**绝不往 C: 写任何东西**。代码、conda、模型、缓存全在 D:。
- **本机没有任何 C/C++ 编译器，且用户明示不装 MSVC。** 需要编译时用 Zig 或 MinGW（装 D:）；默认路径 = 预编译引擎，**只调不编**。
- Python 3.11 由 **conda** 管理。Miniconda 位于 `D:/application/env/miniconda3`（在 PATH 中）。本项目用独立 env `yk-model`（P0 创建，遵循蓝图 9.3）。
- 计算面 = 预编译引擎（llama.cpp 官方 win-avx512 发布包，或 llama-cpp-python wheel）。**AVX-512 是本机 CPU i7-1165G7 的本命指令集**，选型优先它。

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
2. 模型文件大且易混：报数必须带 **引擎版本 + 模型 SHA256 + 量化档 + 上下文长度** 四件套。
3. 别用 hermes 的 venv、别污染其他 conda env；本项目的一切装进 `yk-model`。
4. 用户决策要补记到蓝图第 10 节决策日志。

## 协作约定（vibe coding）

- 开工状态不明时先 `git status` 与看 README 进度；代码未提交前保持在 D: 项目内。
- 与用户以中文交流。