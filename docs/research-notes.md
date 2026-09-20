# 研究笔记（检索证据库）

> 蓝图第 3 节证据地图的原始出处层。改蓝图结论前，先回这里看证据。
> 收录格式：来源 / 关键数字 / 对设计的意义 / 置信度。

## 1. BitNet b1.58 / 2B4T —— 三元权重的现实底座

- **来源**：
  - bitnet.cpp（微软）：github.com/microsoft/bitnet
  - BitNet b1.58 论文 / Technical Report（Wang et al. 2025）
- **关键数字**：
  - 权重三值化 {-1,0,+1}，存 2-bit（log₂3≈1.58，故称 1.58-bit）
  - `BitNet-b1.58-2B-4T`：2.4B 参数、4T token 训练、**全模型 0.4GB**、CPU 解码延迟 29ms、能耗 0.028J
  - x86 推理提速 2.37–6.17×、能耗降 71.9–82.2%；ARM 提速 1.37–5.07×
  - **100B 模型单 CPU 5–7 tok/s**（接近人类阅读速度）
  - GSM8K 58.38（vs Llama3.2-1B 38.21、Qwen2.5-1.5B 56.79）；WinoGrande 71.90
- **实现事实**：bitnet.cpp **骨架基于 llama.cpp，内核基于 T-MAC 查表法**。官方支持 CPU/GPU，NPU 在路。
- **对设计的意义**：三元路线不是假说，是有商用级开源底座的路。我们的运行时层正是它没做的部分。✅ 高置信
- **风险线索**：小规模模型下 BitNet 会退化（见 §5）。

## 2. BitNet a4.8 —— 1-bit 模型的稀疏化

- **来源**：《BitNet a4.8: 4-bit Activations for 1-bit LLMs》（微软亚研，2024.11）
- **关键数字**：
  - 注意力/FFN 输入用 **4-bit 激活**，中间状态稀疏化后 8-bit 量化
  - **仅激活 55% 参数**（稀疏化内建）
  - 支持 **3-bit KV cache**
  - 性能与 b1.58 相当，但启用了 INT4/FP4 内核 → 更快
- **训练方法**：两阶段——先在 8-bit 激活下训练，逐步切到 4-bit 激活。
- **对设计的意义**："低比特 + 稀疏化"是 BitNet 官方路线，与我们的"2-bit 专家 + 稀疏激活"主张同向。✅ 高置信

## 3. MoQE —— 专家层对比重层更耐低比特（捡到宝的结论）

- **来源**：《Mixture of Quantized Experts: Complementary Effect of Low-bit Quantization and Robustness》（微软，2023.12）
- **关键数字**：
  - **只把专家权重量化到超低比特（最低 2-bit）**，共享层保持高精度
  - 专家层比稠密 FFN 更鲁棒 → 量化不掉点
  - 2-bit 专家模型性能优于同数据训练的稠密模型
  - **模型体积 -79.6%**，A100 上推理 1.24× 加速
  - **不需要退训**（即插即用的量化）
- **对设计的意义**：支撑"细粒度混合级配：专家 2-bit + 共享 8-bit"。是"2-bit MoE 装进 8GB"的论文底子。✅ 高置信

## 4. T-MAC —— 查表（LUT）代替反量化+乘加

- **来源**：《T-MAC: CPU Renaissance via Table Lookup for Low-Bit LLM Deployment on Edge》（微软/中科大，2024.7，arXiv:2407.00088）
- **关键数字**：
  - mpGEMM 改为**位级查表**：不解量化、消除乘法、减少加法；表存在片上减少访存
  - **比 llama.cpp 吞吐 +4×、能耗 -70%**
  - Surface AI PC：3B BitNet-b1.58 **48 tok/s**、2-bit 7B **30 tok/s**、4-bit 7B **20 tok/s**（**超过 NPU**）
  - M2 Ultra 单核 30 / 八核 71 tok/s；Raspberry Pi 5：11 tok/s
  - 计算量随位宽线性扩展
- **对设计的意义**：实证"量化数据流重写"而非"卸载补丁"才是快的原因。我们复用它做内核层。✅ 高置信

## 5. MH-MoE —— 1-bit + MoE 可行，但小模型会退化

- **来源**：《MH-MoE: Multi-Head Mixture-of-Experts》（2024.11，arXiv:2411.16205）
- **关键发现**：
  - **BitNet 设置下 MoE 依然有效**："MH-MoE 与 BitNet 结合，MoE 模型可更轻量部署且不掉性能"
  - **警示**：模型规模较小时 BitNet 性能退化（与 BitNet 原论文结论一致）
- **对设计的意义**：路线可行，但 **1–3B 档要留 Q4 稠密退路**。🔶 开放，需实测

## 6. 内存带宽铁律（全篇设计的物理基础）

- **来源**：多篇汇总（华为云 SAIL、财通证券 LPU 报告、Hathora 博客、Cerebras）
- **关键数字**：
  - H100 上单 token 数学运算 < 0.1ms，实际生成 30ms/token → **decode 是带宽瓶颈，不是算力**
  - 2012–2022 算力 +80×，内存带宽仅 +17×（内存墙）
  - decode 占推理耗时 **90%+**；prefill 是计算瓶颈、decode 是带宽瓶颈
  - 单请求 decode = GEMV（矩阵×向量），权重 I/O 是摊销成本
- **推论**：吞吐 ≈ 内存带宽 / 量化后模型字节数。所有设计第一判据。✅ 铁证

## 7. 引擎异构现实

- **来源**：llama.cpp llama.h（`split_mode`/`tensor_split`/设备列表，PR #10497）、多篇实测博客
- **支持**：`--split-mode layer|row` + `--tensor-split` + `-ngl`（GPU 层数）+ 设备列表
- **实测**：Qwen2.5-32B 在 RTX 4060 8GB 上 IQ4_XS + GPU/CPU 分层，10.8 tok/s；Vulkan 后端 AMD 680M 核显可全量 offload 7B
- **墙**：MUX 会关核显；row mode 的 PCIe 同步开销在笔记本上现实地压不低 → **iGPU+dGPU 联合降级为可选优化，不做主线** ✅ 高置信

## 8. 端侧生态趋势（2025）

- MoE + 动态量化 + 蒸馏成为端侧标准动作：智谱 GLM-Edge-V-5B（4.86B）、百度 ERNIE-Thinking 端侧版、Mixtral 8x7B 手机端 14 tok/s
- 前兆架构：Qwen3 有 30B 总参 / 3B 激活的稀疏配置
- 社区公开论题：*"All LLMs Will Be Sparse BitNet Hybrids"*（jackson.dev）
- **意义**：三元+MoE 是行业运动方向，我们从窄门进入有先手。🔶 方向成立

## 9. 本机硬件基线（实测，2026-09-20）

| 项 | 值 | 对推理的含义 |
|---|---|---|
| CPU | i7-1165G7（Tiger Lake）4C/8T | 支持 **AVX-512 VNNI**，三元/查表内核可吃满 |
| 内存 | 16GB 单条 DDR4-3200（**单通道**） | ~25.6 GB/s；加一条→双通道 ~51.2 GB/s，吞吐近翻倍 |
| 核显 | Iris Xe 96EU | 共享内存，与 CPU 争带宽 |
| 逐档预期 | 7B Q4≈5 tok/s；3B Q4≈12 tok/s；**BitNet-2B（0.4GB）30–48 tok/s** | 本机甜点 = 三元/低比特小模型 |

---

## 汇总：哪些是"可信结论"，哪些是"待验证方向"

- ✅ 可信：带宽铁律 / 三元权重可行 / 专家层耐低比特 / 查表内核是提速来源 / llama.cpp 异构能力边界 / 本机硬件基线
- 🔶 开放：1-bit MoE 的路由与热迁移 / 小模型下 1-bit 退化幅度 / 三元路线在"数学/代码类任务"的真实掉点