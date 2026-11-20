# D0 · 环境准备

记录实际使用的 Mac mini、开发工具和测试网配置，供后续独立复现实验。

## VERIFIED · 实际环境

| 项目 | 核验结果 |
| --- | --- |
| 主机 | Mac mini，Apple Silicon `arm64`，macOS 27.0（26A425） |
| 工作区 | 已挂载的外置 APFS SSD，业务源码与本学习仓库分开保存 |
| Git | 2.54.0（Apple Git-157） |
| Foundry | `forge`、`cast`、`anvil` 1.9.0 |
| Solidity | D3 实际用 Solc 0.8.29 编译，Foundry 配置的 EVM 目标为 Cancun |
| Codex | Mac 本机 Codex 应用内置 CLI，实测 `codex-cli 0.167.0-alpha.3.2` |
| 测试钱包 | 使用独立测试钱包及本地加密 Keystore，已核验具备 Lasna 测试资产 |

Mac mini 采集的版本输出如下。

```text
$ uname -m
arm64
$ git --version
git version 2.54.0 (Apple Git-157)
$ forge --version
forge Version: 1.9.0
Commit SHA: 6056c5845d34205a83cf7108042adb137325c9df
Build Timestamp: 2026-11-02T20:16:54.701302000Z (1793650614)
Build Profile: dist
```

## ANALYSIS · Codex Remote SSH 架构

控制端可通过 Tailscale 私网连接 Mac mini 的 SSH 服务，在 Mac mini 上运行 Codex 和 Foundry，工作目录指向已挂载的外置 SSD。Tailscale 提供网络可达性，SSH 仍需单独验证远端主机及密钥认证。本次记录中的命令实际由 Mac mini 本机 Codex 执行。本机可见 Tailscale 接口，但非交互 SSH 探测未通过认证，远程执行链路仍待独立验证。

## 测试网与 lREACT faucet

- **Ethereum Sepolia**（11155111）用于部署和触发测试 ERC-20 事件。
- **Reactive Lasna**（5318007）用于部署 Monitor 和执行 `react()`，原生测试币为 lREACT。
- **Faucet 流程**（官方文档）在 Sepolia 取得测试 ETH 后，核对 Reactive 官方 faucet 地址和当前兑付规则。使用测试钱包发起兑换，到账后在 Lasna 查询该钱包原生余额。公开记录核验了持有余额，本轮未重新发起兑换交易。

[官方 Lasna 与 Faucet 说明](https://dev.reactive.network/reactive-mainnet#lasna-testnet) · [训练营速查表](https://github.com/ethpanda-org/reactive-builder-camp/blob/main/docs/cheatsheet.md) · [D2 网络与费用](D2-network-and-economy.md)
