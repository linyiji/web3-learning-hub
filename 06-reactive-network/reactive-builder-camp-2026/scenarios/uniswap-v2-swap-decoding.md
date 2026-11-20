# 场景笔记 · Uniswap V2 Swap 解码

## ANALYSIS · 如何解释一条 Swap 日志

Whale Swap Monitor 计划使用已确认的 Pair 地址监听 `Swap(address,uint256,uint256,uint256,uint256,address)`。标准 Pair 将 `sender` 和 `to` 放在索引 topic 中，data 含 `amount0In`、`amount1In`、`amount0Out`、`amount1Out`。读取 Pair 的 `token0()`、`token1()` 与对应 Token decimals 后才能确定单位与方向。

1. 校验来源 Chain ID、Pair 地址、topic 0、topic 数和 data 长度。
2. 按 Pair 合约 ABI 解码字段，并区分四个数量各自所属的 Token。
3. 对单向交换，从流入及流出字段确定方向，并按 decimals 换算。复杂路径、Flash Swap 和两侧都非零的布局需要结合完整交易解释。
4. 保留交易哈希和日志索引以复核和去重。索引地址可能是路由合约，无法仅凭 `sender` 判断最终用户。

从 Swap 可以计算两侧成交数量。要估算价格冲击，还需要方向一致的成交前基准，例如同一 Pair 的可核对成交前储备，或明确定义的外部参考。应报告基准、观测时点和费率口径。

## TODO · 待验证

选择一个 Pair 和一条真实事件，与区块浏览器的 ABI 解码对照，再把结论反馈给 [风险告警设计](whale-impact-risk-guard.md)。D3 的实验证据来自 ERC-20 Transfer。

[Uniswap V2 Pair 官方源码](https://github.com/Uniswap/v2-core/blob/master/contracts/UniswapV2Pair.sol) · [D1 用例](../D1-use-case.md)
