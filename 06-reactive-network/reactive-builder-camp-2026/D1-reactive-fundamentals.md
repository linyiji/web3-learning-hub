# D1 · Reactive 原理

## ANALYSIS · 四个概念

| 概念 | 职责 |
| --- | --- |
| Event | 源链交易执行时由合约发出日志。索引参数进入 topics，其他参数进入 data |
| Subscribe | Reactive 合约在 Reactive 链上向 SYSTEM 注册来源链、合约及 topic 条件 |
| `react(LogRecord)` | 匹配后由 SYSTEM 调用，合约核对输入并对条件作确定性的链上判断 |
| Callback | 可选后续动作，Reactive 合约提出请求，由网络通过目标链代理投递 |

源链交易生成日志后，Reactive 网络才有事件可处理。源链的签名、撮合与确认可以先完成，因此观察、记账或回调属于异步后续流程。需要同笔交易原子结算的校验应由源链业务合约执行。

## ANALYSIS · 适用范围

按公开链上日志触发、条件可用合约表达、允许后续异步响应的任务适合此模型。跨链通知、观察和后续状态更新属于这一类。依赖未上链的行情、社交数据或主观判断时，需要可信数据引入或链外决策。依赖历史检索、复杂聚合和仪表盘的功能适合交给 Indexer。

传统 Bot 可以灵活整合链外环境，但需要自己维护监听、签名、提交和重试。RPC 与 WebSocket 提供连接，Indexer 提供查询，Oracle 为合约引入可核验的数据。详见 [Reactive 与 Bot](architecture/reactive-vs-bot.md)。

## VERIFIED · D3 范围

[D3 记录](D3-transfer-monitor.md) 核验了匹配订阅、SYSTEM 调用和 Lasna 状态变化。目标链 Callback 留待后续实验。

[Reactive Contracts](https://dev.reactive.network/reactive-contracts) · [Callback Contracts](https://dev.reactive.network/callback-contracts) · [用例拆解](D1-use-case.md)
