# D2 · 网络角色与执行费用

## VERIFIED · D3 的两个网络

| 网络 | Chain ID | D3 角色 |
| --- | ---: | --- |
| Ethereum Sepolia | 11155111 | Origin Chain，部署 D3TestToken 并发出 `Transfer` |
| Reactive Lasna | 5318007 | Reactive 执行链，SYSTEM 调用 Monitor 的 `react()` |

Destination Chain 指接收 Callback 的链。D3 只在 Lasna 更新计数器，所以这次没有目标链回调。做后续 Callback 实验时，应按官方列表确认测试网和对应代理。

## ANALYSIS · Omni 的 SYSTEM 和资金账户

Lasna Omni SYSTEM 为 `0x8888888888888888888888888888888888888888`。D3 依赖 `reactive-lib-omni`，使用 `AbstractReactive` 和 `onlySystem`。

| 字段 | 含义 |
| --- | --- |
| Balance | 合约自己的原生币余额。支持 `pay()` 的合约可在被请求时用余额结算 |
| Reserves | 由 SYSTEM 记录的预付资金。结费优先消耗预付资金 |
| Debt | 尚未结清的费用。存在欠费时，SYSTEM 暂停后续 Reactive 调用 |

`react()` 的费用由计费 Gas、附加 Gas、费率系数和 Gas 价格共同决定。官方公式为 `max(tx.gasprice, block.basefee) × (gasPriceCoeffPer1000 / 1000) × (gasUsed + extraGas)`。交易回执里的总 Gas 包含其他处理步骤，不能直接替代计费公式中的执行 Gas。回滚仍可能产生执行费用。预付或余额不足且结算失败时会出现欠费，后续事件处理随之停止。部署 Gas、构造函数转入金额、执行费是不同的账目。

## VERIFIED · 链上参数与费用

在 Lasna 区块 8774856 查询到 `_gasPriceCoeffPer1000 = 1000`、`_extraGas = 100000`、`_maxChargeGas = 50000`。第三个参数限制付款调用可用的 Gas。首次响应通过 SYSTEM 的 `pay()` 支付 `0.0253192 lREACT`，Monitor 余额从 `0.1` 降到 `0.0746808 lREACT`，执行后 Reserves 和 Debt 均为零。具体交易见 [D3 记录](D3-transfer-monitor.md)。

[官方 Origins & Destinations](https://dev.reactive.network/origins-and-destinations) · [官方 Economy & System Interactions](https://dev.reactive.network/economy) · [事件驱动架构](architecture/event-driven-execution.md)
