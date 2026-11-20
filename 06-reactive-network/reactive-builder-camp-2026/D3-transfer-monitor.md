# D3 · 监控 Sepolia 上的代币转账

## VERIFIED · 实验目标与范围

在 Ethereum Sepolia 部署 D3TestToken，在 Reactive Lasna 为该 Token 的 `Transfer(address,address,uint256)` 注册订阅，最后用一次 2 D3T 的测试自转账验证 SYSTEM 实际调用 `react()`。实现工程独立维护，本页记录公开证据和验证结果。

## VERIFIED · 已部署合约

| 网络 | 合约 | 部署交易 |
| --- | --- | --- |
| Ethereum Sepolia | [D3TestToken · `0x3C120b65E5CD6cDBeB9dDcB0741665F0b6a1fEa6`](https://sepolia.etherscan.io/address/0x3C120b65E5CD6cDBeB9dDcB0741665F0b6a1fEa6) | [`0xb9d463ebef22b3efae8a3b4b9bad66058b43b36e4cbeed12c1d263283e90d9ec`](https://sepolia.etherscan.io/tx/0xb9d463ebef22b3efae8a3b4b9bad66058b43b36e4cbeed12c1d263283e90d9ec) |
| Reactive Lasna | [D3TransferMonitor · `0x851bDdaf549F62f2973D8B3a3322642F6b0AA086`](https://lasna.reactscan.net/address/0x851bDdaf549F62f2973D8B3a3322642F6b0AA086) | [`0x0fe5f60540c271a4e8964247c2d1a48418d836cb9cac13c9d3a0bdb2b3e41a80`](https://lasna.reactscan.net/tx/0x0fe5f60540c271a4e8964247c2d1a48418d836cb9cac13c9d3a0bdb2b3e41a80) |

Token 全名为 Reactive D3 Test Token，符号 D3T，18 位小数，初始供应量 1,000 D3T。Monitor 将来源链固定为 11155111，并用上表 Token 地址与 Transfer topic 0 做精确过滤。Topic 1–3 使用通配符。链上过滤器标识为 `0x6b16b74849f092532a428871b8db4d67`。

Monitor 验证链、来源合约、事件签名、opCode、数据长度及地址 topic 的高 96 位，并对已处理日志去重。金额达到 1 D3T 才增加 `acceptedCount`。有效格式且不重复的日志都会增加 `observedCount` 并消耗一个名额；第三条将触发退订。

## VERIFIED · 首次执行证据

| 阶段 | 交易 | 回执和日志 |
| --- | --- | --- |
| 源链自转账 | [`0x7430d281e3c972b0baf66688f158c8f7053bc69f9156d2295c4939c8be9378bc`](https://sepolia.etherscan.io/tx/0x7430d281e3c972b0baf66688f158c8f7053bc69f9156d2295c4939c8be9378bc) | 状态 `0x1`，区块 12172615，Transfer 日志索引 325，金额 2 D3T |
| Reactive 响应 | [`0x38d8ce22cd0576ed829911318be2f003a237346c4810c2f04faae06c2cf7e90a`](https://lasna.reactscan.net/tx/0x38d8ce22cd0576ed829911318be2f003a237346c4810c2f04faae06c2cf7e90a) | 状态 `0x1`，区块 8779196，`TransferAccepted` 日志索引 1 |

Lasna 调用追踪确认 SYSTEM 调用 `react(LogRecord)`，来源交易哈希和索引 325 与 Sepolia 回执一致。SYSTEM 的 `pay()` 调用和执行前后的合约余额共同核验了 0.0253192 lREACT 的执行费。

| 字段 | 执行前 | 执行后 |
| --- | ---: | ---: |
| `acceptedCount` | 0 | 1 |
| `observedCount` | 0 | 1 |
| `remaining` | 3 | 2 |
| Monitor Balance | 0.1 lREACT | 0.0746808 lREACT |
| Reserves | 0 lREACT | 0 lREACT |
| Debt | 0 lREACT | 0 lREACT |

源链转账消耗 24,837 Gas，实际交易费为 422,229 wei。自转账只用于生成可核验的 Transfer 日志，测试代币净余额保持不变。本实验未使用真实资产交易。

## SIMULATION · 本地测试

使用 Foundry 1.9.0 和 Solc 0.8.29。实际 `forge build` 通过，`forge test` 为 8 PASS、0 FAIL，`forge lint` 通过。测试覆盖订阅、onlySystem、异常输入、地址 topic 校验、去重、阈值和三次后的退订。只读自转账模拟用于广播前估算，链上响应以实际回执为准。

## TODO · 验证边界

三条事件自动退订已有本地测试，链上只验证了首次响应。D3 没有 Callback 实现，目标链交付留待 D4。

[公开证据索引](evidence/) · [D2 费用模型](D2-network-and-economy.md) · [Reactive Omni 库](https://github.com/Reactive-Network/reactive-lib-omni/tree/3ade0dcf1d27e67e5783ef807e73a1b9e0c0cce8)
