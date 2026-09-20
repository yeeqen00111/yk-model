# yk-model

让 AI 模型在**普通笔记本**上跑得动、跑得快、装得上。

![status](https://img.shields.io/badge/status-全自研-关键)（引擎与模型全自研，暂无第三方运行时）

- 📄 架构蓝图：[docs/architecture-blueprint.md](docs/architecture-blueprint.md)
- 🔬 研究证据库：[docs/research-notes.md](docs/research-notes.md)
- 📏 基准方法论：[docs/benchmark-methodology.md](docs/benchmark-methodology.md)
- 🛠 环境手册：[docs/dev-environment.md](docs/dev-environment.md)
- 🔍 自检工序：[docs/self-check.md](docs/self-check.md)（交付前必过）
- 🤖 Claude Code 引导：[CLAUDE.md](CLAUDE.md)（vibe coding 自动加载）
- 🔬 核心路线：**引擎全自研 + 模型全自研**（三元/低比特自训练；算子、加载、KV、调度全自写，参考算法不抄实现）
- 💻 开发机：T0 档（i7-1165G7 + Iris Xe + 16GB 单通道），本机为最低配验收机

## 仓库结构

```
yk-model/
├── engine/    自研推理引擎源码（YkEngine，C/C++）
├── train/     自研模型训练脚本（YkNet）
├── dist/      获取入口：yk-release.yaml + yk install（自研 CLI）
├── docs/      架构蓝图/证据库/基准口径/环境手册
└── CLAUDE.md  Claude Code 引导
```

发行物：自研引擎二进制（Release）+ 自研模型（ModelScope/HF，开发期本地暂存），`yk install` 一键获取。

## 阶段状态（全自研重排）

- [ ] P0：自研最小推理链骨架（格式 + 加载 + 单设备推理），轻量工具链就绪
- [ ] P1：三元/低比特内核（AVX-512）+ 拓扑探测（DrawShell）+ 自动级配
- [ ] P2：冷热分级驻留 + KV 量化
- [ ] P3：熵路由 / 推测解码 + 模型自训管线（云上）