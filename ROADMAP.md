# 学习路线

## 已完成的学习与工程阶段

- [x] D0 配置 Mac mini、Foundry 和 Solidity 编译器，核验 Sepolia 与 Lasna。[环境准备](06-reactive-network/reactive-builder-camp-2026/D0-environment-setup.md)
- [x] D1 整理事件、订阅、执行与 Callback 的关系，拆解 Whale Swap Monitor。[用例](06-reactive-network/reactive-builder-camp-2026/D1-use-case.md) · [基础](06-reactive-network/reactive-builder-camp-2026/D1-reactive-fundamentals.md)
- [x] D2 记录 Omni 的网络角色、Balance / Reserves / Debt 和计费机制。[费用模型](06-reactive-network/reactive-builder-camp-2026/D2-network-and-economy.md)
- [x] D3 编译、测试和 lint 通过。完成两条测试网的部署、一次自转账和一次成功的 Lasna 响应。[实验记录](06-reactive-network/reactive-builder-camp-2026/D3-transfer-monitor.md)

## TODO

- [ ] D4 在独立练习中验证从来源事件到目标链 Callback 的完整链路。
- [ ] 在独立测试网实验中累计三条合格日志，复核实际自动退订；当前只有本地测试覆盖。
- [ ] 为 Whale Swap Monitor 选定一个 Pair，核对真实 Swap 日志和阈值，然后验证告警方案。
- [ ] 核对 WCB 的课程任务提交回执。

进展证据和场景边界见 [训练营复盘](reviews/reactive-builder-camp/)。
