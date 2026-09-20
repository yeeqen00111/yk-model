# yk-model

让 AI 模型在**普通笔记本**上跑得动、跑得快、装得上。

![status](https://img.shields.io/badge/status-蓝图阶段-blue)

复用了没有设计意义的标准件，自研承载设计逻辑的调度层。

- 📄 架构蓝图：[docs/architecture-blueprint.md](docs/architecture-blueprint.md)
- 🔬 研究证据库：[docs/research-notes.md](docs/research-notes.md)
- 📏 基准方法论：[docs/benchmark-methodology.md](docs/benchmark-methodology.md)
- 🛠 环境手册：[docs/dev-environment.md](docs/dev-environment.md)
- 🤖 Claude Code 引导：[CLAUDE.md](CLAUDE.md)（vibe coding 自动加载）
- 🔬 核心路线：三元/低比特权重（bitnet.cpp / T-MAC / llama.cpp 骨架）+ 自研「拓扑感知 · 冷热分级驻留 · 熵路由」运行时调度层
- 💻 开发机：T0 档（i7-1165G7 + Iris Xe + 16GB 单通道），本机为最低配验收机

## 阶段状态

- [ ] P0：本机工具链 + 基准 rig 跑通
- [ ] P1：拓扑探测（DrawShell）+ 自动级配
- [ ] P2：冷热分级驻留
- [ ] P3：熵路由 / 推测解码