# engine — 自研推理引擎源码（C/C++）

YkEngine，全自研，服务于 YkNet（架构蓝 §6A）。当前为骨架占位，P0 起逐步填充。

分层（对应蓝图 §6A.2）：
- `kern/` — 内核层（三元乘 LUT/VNNI · 分级精度 GEMM · softmax · KV 量化 · 路由）
- `exec/` — 执行器（per-token 动态执行 · 三阶段流水 · 线程池）
- `mem/`  — 内存/驻留层（mmap · 热区/冷区 · 切片迁移 · 预取）
- `kv/`   — KV 管理层（环形窗口 · Q8 打包）
- `ctx/`  — 上下文层（对话 · 采样 · 前缀缓存）
- `format/` — YkFormat 权重格式读写（v0 草案 §6A.4）

工具链：Zig（首选）或 WinLibs MinGW-w64，装 D:，不装 MSVC；构建产物输出 D:\build。
首版范围（P0）：`format/` 解析 + 最小前向/解码跑通 + 内置计时。