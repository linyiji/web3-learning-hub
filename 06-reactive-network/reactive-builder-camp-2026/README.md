# Reactive Builder Camp 2026

这部分记录 D0–D3 的学习顺序、测试网实验，以及可继续验证的设计场景。

| 阅读顺序 | 内容 | 证据等级 |
| --- | --- | --- |
| D0 | [环境准备](D0-environment-setup.md) | VERIFIED 环境与链配置，ANALYSIS 远程执行架构 |
| D1 | [用例拆解](D1-use-case.md) · [Reactive 基础](D1-reactive-fundamentals.md) | ANALYSIS 设计与概念 |
| D2 | [网络与费用](D2-network-and-economy.md) | VERIFIED 系统参数与 D3 费用，ANALYSIS 计费机制 |
| D3 | [Transfer 监控](D3-transfer-monitor.md) · [公开证据](evidence/) | VERIFIED 部署与首次响应，SIMULATION 本地测试 |
| 场景 | [Uniswap V2 Swap 解码](scenarios/uniswap-v2-swap-decoding.md) · [Whale Impact Risk Guard](scenarios/whale-impact-risk-guard.md) | ANALYSIS 和 TODO |
| 架构 | [Reactive 与 Bot](architecture/reactive-vs-bot.md) · [事件驱动执行](architecture/event-driven-execution.md) | ANALYSIS，注明已实测的路径 |

`VERIFIED` 指有命令结果或链上证据；`ANALYSIS` 指机制解释与设计判断；`SIMULATION` 指本地测试或只读模拟；`TODO` 指待执行的验证。

[返回知识库](../../README.md) · [训练营复盘](../../reviews/reactive-builder-camp/) · [课程模板](https://github.com/ethpanda-org/reactive-builder-camp)
