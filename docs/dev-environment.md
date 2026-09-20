# 开发环境手册（实测）

> 实测日期：2026-09-20。所有路径/版本均为本机真实值，改了先改这里。
> 用途：给人和 Claude Code 共同使用的环境事实 + 命令全集。

## 1. 硬件基线（实测，勿猜）

| 项 | 实测 | 影响 |
|---|---|---|
| CPU | i7-1165G7（Tiger Lake）4C/8T 2.8GHz | 支持 AVX-512 VNNI → 三元/查表内核可吃满 |
| 核显 | Intel Iris Xe（96 EU） | 共享系统内存 |
| 内存 | 16GB 单条 DDR4-3200（**单通道**） | ~25.6GB/s 带宽上限；加一条→双通道~51GB/s |
| 独显 / NPU | 无 | T0 拓扑；T2/T3 需云 GPU 验证 |
| 磁盘 | C: 200GB（余 16.3GB）/ D: 275GB（余 142.5GB） | **一切写 D:** |

## 2. 目录与磁盘纪律

```
D:\work\work_of_own\yk-model\      # 本项目（代码）
D:\application\env\miniconda3\     # conda 根（已存在，勿动）
D:\model-store\                    # 模型统一存放（计划，遵循"大文件一律 D:"）
```

- C: 仅剩 16GB → **禁止**任何工具链/模型/缓存写 C:。
- 工具链缓存位可设：`pip` 缓存指到 D:（`pip config set global.cache-dir D:\pip-cache`）。

## 3. conda（Python 环境）

**现状**（conda 26.1.1；截至 2026-09-20）：
- 根：`D:/application/env/miniconda3`（在 PATH）
- 已有 env：base、GPTSoVits、fay、py310、dy_spide_py310
- **`yk-model` 已创建**＝ Python 3.11.15（因缓存的 3.11.16 损坏，创建时锁定到 3.11.15，见第 11 节决策日志旁的坑）。用前验证：
  ```bash
  D:/application/env/miniconda3/Scripts/conda.exe run -n yk-model python -c "import sys; print(sys.version_info[:2])"
  ```

**注意**：本机有另一套 conda（`D:\application\env\conda`）容易混，一律只用 `miniconda3` 这套。
交互式激活：`source D:/application/env/miniconda3/etc/profile.d/conda.sh && conda activate yk-model`

## 4. 自研引擎工具链（全自研路线：无第三方运行时）

**本机不装 MSVC**（用户明示）。引擎（C/C++ 自研）用轻量工具链之一（均装 D:）：
1. **Zig**（自带 clang，下载 ~50MB）→ 首选；`zig cc`/`zig c++` 即 C/C++ 编译器，能指定 `-mavx512f -mavx512vnni` 等指令集。
2. **WinLibs MinGW-w64**（免 MSYS2，~1GB）→ 备选；含 gcc/g++，同样支持 AVX-512 标志。
- 编译产物输出到 `D:\build\` 等 D: 路径；C: 保持干净。
- Python 控制面（DrawShell/基准/训练脚本）装进 `yk-model` env。
- 外部对照（可选、非本路线）：`D:\model-store\engines\llama.cpp\` 残留的 llama.cpp b11060 二进制**不属本项目**，仅在需要"第三方参照 baseline"时作外部对照并在结果中注明。

## 5. 模型存放（自产权重）

- 统一 `D:\model-store\models\`，按 `{gen}/{name}/` 组织：
  ```
  D:\model-store\models\gen0\                # Gen-0 Dense 三元（0.4–0.7B）
  D:\model-store\models\gen1\                # Gen-1 切片 MoE（远期）
  ```
- 权重为自训/自转产物（YkFormat，§6A.4）；**记录 SHA256**；版本与训练 artifact 关联。
- 体积预期（内存账）：0.4–0.7B 三元 ≈ 0.1–0.25GB（含 8-bit 关键层）。

## 6. Git

- **已 init 并推送**（main@`877d51b`，作者 111lw，远程 github.com/yeeqen00111/yk-model.git）：
  ```bash
  git status   # 保持跟踪；文档改动及时 commit + push
  ```
- 提交身份为本仓库 local（`111lw`），**不影响全局**（全局仍是公司身份）。
- `.gitignore` 已就位（排除模型/缓存/venv），大文件不入库。

## 7. 就绪验证清单（对照当前进度更新）

- [x] `yk-model` conda env 存在（Python 3.11.15）
- [ ] Zig（或 WinLibs MinGW）装到 D:，`zig cc --version` 可用（P0 项）
- [ ] `D:\model-store` 目录结构就位（engines/models 已建）
- [ ] 自研引擎首版可编译（YkFormat 加载 + 最小前向跑通）
- [ ] 按 `benchmark-methodology.md` 跑出自研首条基线
- [x] git 已 init，首 commit 已推送

## 8. 待办/坑（留痕）

- [ ] 装 Zig 到 D:（~50MB，首选 C/C++ 工具链）
- [ ] C: 16GB 告急：可考虑清理临时文件 + Windows 更新残留（hiberfil 等），不作为本项目主线
- [ ] 加一条 16GB DDR4-3200 → 双通道，吞吐近翻倍（强烈建议，硬件级）
- [ ] 缓存的 python-3.11.16 损坏（缺 .lib/.pyc）→ 当前用 3.11.15 绕开；如再建 3.11.x env 需规避（不删任何东西）