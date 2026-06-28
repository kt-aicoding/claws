# Claws

OpenClaw 运维实践、定时任务修复经验和多机器治理手册。

这个仓库用于沉淀真实 OpenClaw 使用过程中的排障方法、修复思路和复用模板。它不是运行态备份仓库，也不保存本机或其他机器的原始配置、数据库、会话记录、密钥、token、精确排期或私有输出。

## 适合谁看

- 正在使用 OpenClaw 管理个人或团队自动化任务的人。
- 需要排查 OpenClaw cron、Gateway、模型配置、GitHub CLI 鉴权问题的人。
- 想把单机经验扩展到多机器巡检和治理的人。
- 想公开分享实践，但又不想泄露本地运行数据的人。

## 文档入口

| 文档 | 用途 |
| --- | --- |
| [2026-06-27 OpenClaw 定时任务修复复盘](docs/case-studies/2026-06-27-openclaw-cron-repair.md) | 记录一次真实修复：失败原因、Ark 切换、cron 优化、验证和保留项 |
| [Cron 运维 Runbook](docs/runbooks/cron-operations.md) | 单机定时任务排障、修复和验证步骤 |
| [多机器 OpenClaw 运维 Runbook](docs/runbooks/fleet-openclaw-operations.md) | 多台机器的安全巡检、对比、修复顺序和数据最小化规则 |
| [最终 Cron 清单](docs/reference/final-cron-inventory.md) | 脱敏后的任务结构、类型和粗粒度频率 |
| [Ark 模型配置说明](docs/reference/ark-model-configuration.md) | 如何避免 cron 任务模型漂移，统一继承 Ark 默认模型栈 |
| [机器盘点模板](docs/reference/machine-inventory-template.md) | 用别名和聚合指标记录多台 OpenClaw 机器 |
| [OpenClaw Fleet Optimization Goal](docs/goals/openclaw-fleet-optimization.md) | 长时间运行的 fleet 探索、修复、验证和沉淀目标 |
| [2026-06-28 Fleet Inventory](docs/reference/fleet-inventory-2026-06-28.md) | 当前脱敏机器健康画像 |
| [2026-06-28 Completion Audit](docs/audits/2026-06-28-openclaw-fleet-completion-audit.md) | 当前目标完成度审计 |

## 核心原则

1. 不要盲目关闭失败任务。先区分“历史最后状态失败”和“当前配置仍然失败”。
2. 定时任务必须有边界：短超时、有限输出、明确 fallback、失败时可 no-op。
3. 尽量避免 cron 级模型覆盖。默认让任务继承全局模型栈，减少迁移和排障成本。
4. 纯报告、巡检、提醒类任务优先用 `command` cron；需要判断、总结、生成内容的任务再用 `agentTurn`。
5. 公开分享只分享方法、模板和脱敏结论，不分享原始运行态数据。
6. 多机器治理只发布别名和聚合健康状态，不发布真实主机名、IP、用户名、路径、token、精确 cron 表达式或 raw output。

## 当前沉淀的实践结果

一次真实 OpenClaw 修复后的参考结果：

- cron 总数：36
- 启用任务：28
- 禁用任务：8
- `command` 任务：4
- `agentTurn` 任务：24
- cron 级模型覆盖：0
- 默认模型栈：继承 Volcengine Ark

其中部分任务保留了历史 `error` 状态。这个状态没有被手工清除，因为它应该通过后续真实成功运行自然转为 `ok`，而不是通过隐藏证据来制造“看起来正常”。

## 推荐排障顺序

单机排障时，优先按这个顺序做：

1. 只读查看 `openclaw status --deep`、`openclaw doctor`、`openclaw cron list --all --json`。
2. 确认 Gateway 是否 reachable，LaunchAgent 或系统服务是否 loaded/running。
3. 确认默认模型栈和 cron 级 `payload.model` 覆盖数量。
4. 检查失败任务是当前仍失败，还是历史状态未更新。
5. 把纯命令型任务从 `agentTurn` 转成 `command`。
6. 给外部数据源增加 timeout、fallback 和输出截断。
7. 用 no-delivery 临时任务或只读命令验证调度链路。
8. 对会写 GitHub 的任务，不批量触发；优先等待自然运行或只触发一个明确范围的任务。

## 多机器治理方式

多台机器不要直接汇总 raw output。推荐把每台机器映射成脱敏记录：

```md
| Alias | Role | Version | Gateway | Cron summary | Model overrides | Main action |
| --- | --- | --- | --- | --- | --- | --- |
| machine-01 | workstation | 2026.x | healthy | 28 enabled / 8 disabled | 0 | monitor natural reruns |
| machine-02 | workstation | unknown | unknown | not collected | unknown | run read-only audit |
```

机器别名只表达用途，不暴露真实主机名、位置、用户名、IP 或设备信息。更完整的字段见 [机器盘点模板](docs/reference/machine-inventory-template.md)。

## GitHub CLI 鉴权注意事项

如果环境变量里的 `GH_TOKEN` 已过期，它会覆盖有效的 keyring 登录，导致 `gh` 报 `Bad credentials`。在本地巡检或 cron 脚本里，可以显式清掉环境 token：

```sh
env -u GH_TOKEN -u GITHUB_TOKEN gh api user
```

定时任务里调用 GitHub CLI 时，也推荐使用同样模式：

```sh
env -u GH_TOKEN -u GITHUB_TOKEN gh issue list --repo OWNER/REPO --limit 20
```

这样可以避免坏的环境变量破坏可用的 keyring 认证。

## 公开分享边界

这个仓库可以公开：

- 排障方法。
- 修复顺序。
- 脱敏后的指标。
- 机器别名和聚合健康状态。
- 可复用命令模板。
- 风险判断和经验复盘。

这个仓库不应公开：

- `openclaw.json` 原文。
- SQLite 数据库。
- transcript `.jsonl` 文件。
- auth profile、token、secret、私钥。
- Telegram ID、私聊 ID、群 ID。
- 真实主机名、IP、用户名、绝对路径。
- 精确 cron 表达式。
- raw `openclaw status --deep` 或 raw `openclaw doctor` 输出。

## 维护建议

- 新增案例时，优先写“问题 -> 原因 -> 修复 -> 验证 -> 保留项”。
- 新增机器记录时，只使用 `machine-XX` 这类别名。
- 新增命令时，确认不会打印 token、路径、IP 或 raw 配置。
- 每次推送前运行 secret scan 和自定义敏感规则扫描。
- 如果误提交了敏感内容，不要只做新提交删除；需要清理可达历史，并轮换相关凭据。
