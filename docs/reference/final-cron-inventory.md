# Final Cron Inventory

Snapshot date: 2026-06-27.

This inventory records enabled scheduled jobs after optimization. It is sanitized for public sharing: exact cron expressions are replaced with coarse cadence so local operating windows are not exposed.

## Summary

| Metric | Count |
| --- | ---: |
| Total cron jobs | 36 |
| Enabled jobs | 28 |
| Disabled jobs | 8 |
| Enabled command jobs | 4 |
| Enabled agentTurn jobs | 24 |
| Cron-level model overrides | 0 |

## Enabled jobs

| Job | Type | Status class | Cadence |
| --- | --- | --- | --- |
| 定时任务健康巡检 | command | validated | several times per day |
| Dev.to AI + 创意 | agentTurn | pending natural rerun | weekly |
| AI Ideas - 创作风格优化 | agentTurn | validated | weekly |
| AI Ideas - 社区互动 | agentTurn | validated | weekly |
| AI Ideas - 每日运营复盘 | agentTurn | validated | daily |
| 三爪协作日志推送 | agentTurn | validated | daily |
| 三爪协作同步 | command | validated | daily |
| AI Ideas - 创意生成 | agentTurn | validated | daily |
| AI Ideas - Issue 清理 | agentTurn | validated | weekly |
| 每周 GitHub AI 趋势 | agentTurn | pending natural rerun | weekly |
| 公开 AI 热点 + 创意 | agentTurn | pending natural rerun | weekly |
| AI Ideas - 进化研究 | agentTurn | validated | weekly |
| 每周系统健康检查 | command | validated | weekly |
| AI Ideas - 高价值 Issue -> PR | agentTurn | pending natural rerun | weekly |
| AI Ideas - 趋势预测 | agentTurn | validated | weekly |
| Product Hunt AI + 创意 | agentTurn | pending natural rerun | weekly |
| 每周 AI 资讯推送 | agentTurn | pending natural rerun | weekly |
| Hugging Face 趋势 + 创意 | agentTurn | pending natural rerun | weekly |
| AI Ideas - Issue 深度评估 | agentTurn | pending natural rerun | weekly |
| ArXiv AI 论文 + 创意 | agentTurn | pending natural rerun | weekly |
| 2077日报 + AI创意 | agentTurn | validated | weekly |
| AI 应用场景扫描 | agentTurn | validated | weekly |
| 每周 Hacker News 精选 | agentTurn | pending natural rerun | weekly |
| Reddit/HN AI 热点 + 创意 | agentTurn | pending natural rerun | weekly |
| Papers With Code + 创意 | agentTurn | pending natural rerun | weekly |
| Papers - 想法优化 | agentTurn | validated | weekly |
| TechCrunch AI + 创意 | agentTurn | pending natural rerun | weekly |
| AI 订阅账单提醒 | command | pending natural rerun | monthly |

## Interpretation

The inventory is a scheduling map, not a live service-level objective report. A historical `error` value means the last recorded run failed before the repair, or the job has not naturally re-run since the repair. It is intentionally not rewritten by hand.

## Expected drift controls

- New pure shell/reporting tasks should be command jobs.
- New summarization/ideation tasks can be agentTurn jobs.
- New jobs should inherit the global model stack unless there is a documented exception.
- External APIs should use timeout, fallback, and output caps.
