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

**现状**（conda 26.1.1）：
- 根：`D:/application/env/miniconda3`（在 PATH）
- 已有 env：base、GPTSoVits、fay、py310、dy_spide_py310
- **`yk-model` 尚未创建** —— P0 第一步建它：

```bash
# Git Bash 一次性（最稳）：创建
D:/application/env/miniconda3/Scripts/conda.exe create -n yk-model python=3.11 -y
D:/application/env/miniconda3/Scripts/conda.exe run -n yk-model python -c "import sys; print(sys.version_info[:2])"
```

**注意**：本机有另一套 conda（`D:\application\env\conda`）容易混，一律只用 `miniconda3` 这套。
交互式激活：`source D:/application/env/miniconda3/etc/profile.d/conda.sh && conda activate yk-model`

## 4. 计算面引擎（预编译，零编译器）

优先顺序（按 AVX-512 利用度）：
1. **llama.cpp 官方 Windows 预编译**（GitHub Releases，文件名为 `llama-*-bin-win-avx512-x64.zip`）
   - 解压到 `D:\model-store\engines\llama.cpp\`
   - 自带 `llama-bench.exe` → P0 基准直接用
2. **llama-cpp-python**（pip wheel，预编译）—— 控制面用 Python 调引擎时选它
   - `D:/application/env/miniconda3/Scripts/conda.exe run -n yk-model pip install llama-cpp-python`
   - 注意 CPU wheel 默认 AVX2；AVX-512 需标志或自定义编译（先不追求，基准要记录启用集）
3. **bitnet.cpp**（三元模型底座）：官方 Windows 需 MSVC 构建，**不急于本机 building**；先评估是否已有 release 二进制，或以 llama.cpp avx512 跑 GGUF 的 BitNet 转换版替代。本机甜点模型先行。

引擎选择要落口：纪录 **引擎名 + commit/版本 + 指令集（avx2/avx512）**。

## 5. 模型存放

- 统一 `D:\model-store\models\`，按 `{family}/{quant}/` 组织：
  ```
  D:\model-store\models\bitnet\                # 1.58-bit 三元系
  D:\model-store\models\qwen\Qwen2.5-1.5B-Q4_K_M.gguf
  D:\model-store\models\qwen\Qwen2.5-7B-Q4_K_M.gguf
  ```
- 模型下载镜像：HuggingFace 直链 / 魔搭（hf-mirror）按网络情况选；**记录 SHA256**。
- 体积预期（内存账）：0.4GB（BitNet-2B）/ ~1.9GB（3B Q4）/ ~4.4GB（7B Q4）

## 6. Git

- **当前未 init**（2026-09-20 实测）。文档已齐 → 建议立刻：
  ```bash
  git init
  git add .
  git commit -m "docs: 蓝图/证据库/基准口径/环境手册 初始冻结"
  ```
- `.gitignore` 已就位（排除模型/缓存/venv），模型不入库。

## 7. 就绪验证清单（vibe coding 开工前）

- [ ] `yk-model` conda env 存在，`s/conda.exe run -n yk-model python -V` 输出 3.11
- [ ] llama.cpp win-avx512 解压到 D:\model-store\engines\
- [ ] 首个模型（BitNet-2B 或 1.5B Q4）到位，SHA256 记录
- [ ] 按 `benchmark-methodology.md` 跑通一条基线
- [ ] git 已 init，首个 commit 完成

## 8. 待办/坑（留痕）

- [ ] C: 16GB 告急：可考虑清理临时文件 + Windows 更新残留（hiberfil 等）为 C: 减压，但不作为本项目主线
- [ ] 加一条 16GB DDR4-3200 → 双通道，吞吐近翻倍（强烈建议，硬件级）
- [ ] bitnet.cpp 的 Windows 预编译二进制状态待确认（P0 验证项）