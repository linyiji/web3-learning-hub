# 架构笔记 · Reactive 与 Bot

## ANALYSIS · 各自解决的问题

| 组件 | 执行位置 | 核心职责 |
| --- | --- | --- |
| RPC / WebSocket | 客户端与链外节点之间 | 读链、订阅流式数据或提交交易；客户端要处理断线和补读 |
| 传统 Bot / Keeper | 自有或托管链外进程 | 监听、决策、签名、重试和对账；运行与密钥由团队负责 |
| Indexer | 链外服务和数据库 | 将链上数据转换为可检索的历史记录、聚合数据和查询接口 |
| Oracle | 链外采集和链上验证 / 发布 | 将外部状态传给合约，信任与验证规则取决于预言机设计 |
| Reactive Network | 网络链外摄取与 Reactive 链上执行 | 网络摄取已匹配日志，SYSTEM 触发合约内的 `react()`；需要时请求目标链 Callback |

Reactive 应用可以把明确的链上判断放进 Solidity，并使用网络执行服务。项目仍需选择可靠的来源、处理重复事件、管理预算，并监控失败情况。Indexer 与 Oracle 可以补足历史查询或外部输入，也可以与 Reactive 配合。

对于源交易必须同步满足的限制，在源链合约做原子检查。Reactive 的订阅在事件产生后处理，适合反馈、告警和后续动作。若决策依赖大量链外数据或人工审批，保留链外流程更容易表达这些规则。

[事件驱动执行](event-driven-execution.md) · [Reactive Contracts](https://dev.reactive.network/reactive-contracts) · [Ethereum JSON-RPC](https://ethereum.org/en/developers/docs/apis/json-rpc/)
