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

---

# D5 · Lasna SYSTEM 费用参数实测与经济模型

## 1. VERIFIED · 实时参数查询

查询日期：2026-11-21

网络：Reactive Lasna

Chain ID：5318007

查询开始时记录的区块：8871785

RPC：https://lasna-rpc.rnk.dev/

SYSTEM：0x8888888888888888888888888888888888888888

说明：区块 8871785 是首次批量查询时记录的区块高度。第一项参数在该批次成功读取；后两项在发现 cast to-dec 无法解析方括号辅助显示后，改用不带转换管道的 cast call 重新读取。后两次调用使用 RPC 的 latest 状态，没有固定到区块 8871785。因此，本记录保留实际命令和原始输出，不将三次调用描述为同一固定区块的原子快照。

### 参数一：Gas Price Coefficient

执行命令：

```bash
cast call "$SYSTEM_CONTRACT_ADDR" \
  "_gasPriceCoeffPer1000()(uint256)" \
  --rpc-url "$REACTIVE_RPC" | cast to-dec
```

实际输出：

```text
1000
```

### 参数二：Extra Gas

执行命令：

```bash
cast call \
  0x8888888888888888888888888888888888888888 \
  "_extraGas()(uint256)" \
  --rpc-url https://lasna-rpc.rnk.dev/
```

实际输出：

```text
100000 [1e5]
```

### 参数三：Maximum Charge Gas

执行命令：

```bash
cast call \
  0x8888888888888888888888888888888888888888 \
  "_maxChargeGas()(uint256)" \
  --rpc-url https://lasna-rpc.rnk.dev/
```

实际输出：

```text
50000 [5e4]
```

参数汇总：

| 参数 | 实测值 | 解释 |
|---|---:|---|
| _gasPriceCoeffPer1000() | 1000 | 计费系数为 1000 / 1000，即 1 |
| _extraGas() | 100000 | 固定附加 Gas |
| _maxChargeGas() | 50000 | SYSTEM 调用合约 pay() 结算欠费时可使用的 Gas 上限 |

说明：Cast 输出中的 [1e5] 和 [5e4] 是科学计数法辅助显示，分别等于 100000 和 50000，不是错误。第一次批量查询时，cast to-dec 无法解析这类方括号辅助显示；直接查看 cast call 原始输出即可确认数值。

可复现的变量定义：

```bash
export REACTIVE_RPC="https://lasna-rpc.rnk.dev/"
export SYSTEM_CONTRACT_ADDR="0x8888888888888888888888888888888888888888"
```

## 2. EXPLANATION · 一次 react() 的费用怎么算？

Reactive 官方费用公式：

```text
fee = p_base × C × (g_used + K)
```

其中：

- p_base = max(tx.gasprice, block.basefee)，即交易 Gas Price 与区块基础费中的较大值。
- C = _gasPriceCoeffPer1000 / 1000，本轮 Lasna 实测为 1000 / 1000 = 1。
- g_used 是本次 Reactive 执行实际消耗的 Gas。
- K = _extraGas，本轮实测为 100000。

_maxChargeGas = 50000 不进入费用公式；它限制 SYSTEM 调用合约 pay() 进行欠费结算时可使用的 Gas 预算。

**EXAMPLE（仅为公式示例，不是本轮链上报价）：**

假设 p_base = 100 gwei、g_used = 50000、C = 1、K = 100000：

```text
fee = 100 gwei × 1 × (50000 + 100000)
    = 0.015 lREACT
```

实际费用会随当次 Gas 价格和实际执行 Gas 变化。不能把交易回执的整笔交易 Gas Limit 直接代入 g_used。

## 3. EXPLANATION · Balance / Reserves / Debt

- **Balance**：Reactive 合约自身持有的原生币余额，由合约控制。标准 AbstractPayer.pay() 可在 SYSTEM 请求付款时，从合约 Balance 支付欠费。
- **Reserves**：预先存放在 SYSTEM 账本中的执行费用额度。费用结算优先消耗 Reserves。
- **Debt**：已经产生但尚未结清的执行费用。

执行顺序：

1. SYSTEM 在处理下一次 Reactive 通知前先检查 Debt。
2. 只要 Debt 大于零，SYSTEM 就不会调用该合约的 react()，后续事件处理暂停。
3. Debt 为零时，SYSTEM 才继续投递 Reactive 执行。
4. 执行结束后，费用优先从 Reserves 扣除。
5. Reserves 不足时，SYSTEM 调用合约的 pay()，尝试使用 Balance 偿还本次欠费。
6. 如果付款成功，欠费在同一笔交易中结清；如果 Balance 不足或 pay() 调用失败，Debt 保留，合约进入 inactive 状态。

因此，Reserves 用尽不一定立即产生 Debt：标准 AbstractPayer.pay() 可以在同一笔执行交易中使用足够的 Balance 结清短缺。真正阻断后续执行的是未偿 Debt 大于零。

通过 SYSTEM 的 depositTo(address) 充值时，会先清偿 Debt，剩余金额再进入 Reserves。官方账务规则下，Debt 清偿优先于增加 Reserves，因此合约不会同时保留正数 Debt 和正数 Reserves。

直接向合约转账只增加 Balance，不会自动增加 Reserves，也不会自动清偿 Debt；若合约没有公开 coverDebt()，这笔直接转账可能无法用于恢复执行。

## 4. EXPLANATION · react() 回滚是否收费？

**会收费。**

SYSTEM 捕获 react() 的回滚，记录 ReactiveContractReverted，然后继续结算执行费用。失败执行不会退款，合约仍需支付该次执行产生的费用。

这里的收费不是按整个 Gas Limit 直接扣款，而是按实际执行消耗的 Gas、固定附加 Gas 和计费系数计算。

对合约设计的影响：

1. 普通过滤条件使用 return，不要使用 require 或 revert 排除不匹配事件。
2. 在复杂解码和计算之前，先检查来源链、来源合约、Topic、opCode 和数据长度。
3. 对地址 Topic、数据编码和外部调用做好边界检查，减少可避免的回滚。
4. 保持 Reserves 与可用于 pay() 的 Balance 充足，并监控 Debt。

## 5. VERIFIED · D4 真实执行费用案例

D4 的 Lasna Reactive 实际执行数据：

| 项目 | 数值 |
|---|---:|
| 执行前 Reserves | 0.018239280000182393 lREACT |
| react() 实际储备扣费 | 0.0152004 lREACT |
| 执行后 Reserves | 0.003038880000182393 lREACT |
| 执行后 Balance | 0 |
| 执行后 Debt | 0 |

D4 已真实完成 Sepolia → Lasna → Base Sepolia 跨链 Callback。

Lasna Reactive 的执行费用由 Lasna SYSTEM 结算；Base Callback 的投递费用由目标链 Callback Proxy 使用目标链原生币独立结算，两者不能混为一笔费用。

- [D4 Lasna react() 交易](https://lasna.reactscan.net/tx/0x95e57061219c0adeadcb2f04c39c95d6a015159a7f78658e3319ef31a0fea2af)
- [D4 Base Callback 交易](https://sepolia.basescan.org/tx/0xe51ce0ac20c52553c1a9d79a04e679ad1e37fa907918bc7222f562e94d674901)

## 6. 学习结论

Reactive 的执行成本由网络 Gas 价格、计费系数、实际执行 Gas 和固定附加 Gas 共同决定。

合约能否持续响应事件，不仅取决于 Subscription 是否活跃，还取决于 Reserves、Balance 和 Debt 的状态。

因此，设计 Reactive 产品时，需要同时考虑：

- 订阅过滤效率。
- 单次 react() 执行成本。
- 无效事件比例。
- 执行资金储备。
- 回滚与异常处理。

## 官方资料

- [Reactive Network · Economy & System Interactions](https://dev.reactive.network/economy)
- [Reactive Network · Reactive Contracts](https://dev.reactive.network/reactive-contracts)
- [Reactive Network · Debugging](https://dev.reactive.network/debugging)
- [D3 Transfer Monitor 实验记录](D3-transfer-monitor.md)
- [事件驱动架构](architecture/event-driven-execution.md)
