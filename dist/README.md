# dist — 获取入口（yk install）

本目录是"别人/未来 clone 仓库后拿到全部可运行物"的入口。

- `yk-release.yaml` — 当前版本的发布清单（版本 / 引擎地址 / 模型 ID / SHA256）
- `yk`（CLI 脚本，P0 实现）— `yk install` 按清单拉取 + 校验 + 归位

> 开发期约定：模型暂不外部发布，先放本地 `D:\model-store\models\`；`yk install` 先走本地 stub。