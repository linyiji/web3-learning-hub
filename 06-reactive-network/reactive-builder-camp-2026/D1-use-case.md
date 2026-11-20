# D1 · Whale Swap Monitor 用例拆解

## ANALYSIS · 业务定位

**Whale Swap Monitor** 面向关心大额交易和池子风险的观察者。监听指定 DEX Pair 的 `Swap` 事件，解释成交方向和规模，筛出值得人工关注的交易，再产出可以追溯到源交易的告警。后续的 Whale Impact Risk Guard 以同一观察链路为基础，追加价格冲击风险阈值和受限响应规则。

| 决策项 | 候选设计 |
| --- | --- |
| 监听什么 | 支持的来源链、指定 Uniswap V2 Pair、`Swap` 事件 |
| 如何筛选 | 先核对事件来源与格式，再评估标准化成交量；价格冲击需要另行定义基准 |
| 如何防误报 | 使用链 ID、合约、交易哈希和日志索引标记重复，按 Token decimals 归一化 |
| 输出什么 | 记录源交易和触发理由，供观察者复核。可选 Callback 写入目标链告警状态 |
| 适用边界 | 告警发生在源交易产生事件之后，不能用于阻止该笔交易的签名或结算 |

## VERIFIED · 与实验的关系

[D3 Transfer Monitor](D3-transfer-monitor.md) 核验了指定 ERC-20 的真实转账触发 Lasna `react()`。Swap 解码、大额识别和告警阈值仍为方案，后续需用实际 Pair 日志验证。

[Swap 解码](scenarios/uniswap-v2-swap-decoding.md) · [风险告警设计](scenarios/whale-impact-risk-guard.md) · [Reactive Contracts 官方文档](https://dev.reactive.network/reactive-contracts)
