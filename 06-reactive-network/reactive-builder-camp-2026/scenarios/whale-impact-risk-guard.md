# 场景笔记 · Whale Impact Risk Guard

## ANALYSIS · 在监控基础上加风险条件

这个候选场景沿用 [Whale Swap Monitor](../D1-use-case.md) 的大额 Swap 观察定位。观察指定 Pair 的成交，识别可能对低流动性池造成显著冲击的事件，并留下可追溯的告警。触发发生在源链交易产生日志之后。

| 部分 | 设计点 |
| --- | --- |
| 输入 | Pair 地址、Swap 日志、Token 元数据及可核对的成交前价格基准 |
| 筛选 | 配置最低成交规模和价格冲击阈值，先排除格式异常、来源不符和重复事件 |
| 输出 | 记录原因、方向、金额、价格基准及来源交易。可选 Callback 写入目标链告警状态 |
| 保护 | 按事件键去重，限制次数与费用，缺少可靠价格基准时标记待复核并由人工决定 |

D3 仅验证了 ERC-20 Transfer 触发和 Lasna 响应。大额识别、价格冲击估算、跨链告警都仍需独立验证。

## TODO · 验收

用一条已解码的真实 Swap 和一条未超过阈值的样本验证误报。若增加目标链动作，单独核验授权、状态和实际费用。

[Uniswap V2 Swap 解码](uniswap-v2-swap-decoding.md) · [事件驱动执行](../architecture/event-driven-execution.md) · [Reactive Contracts](https://dev.reactive.network/reactive-contracts)
