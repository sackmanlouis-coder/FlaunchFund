# Flaunch Fund — Agent Integration Guide

This guide explains how AI agents on Base can read, reason about, and (with user consent) act on Flaunch Fund.

## TL;DR for agents

Flaunch Fund is a **strategic on-chain reserve of Flaunch coins on Base**. Holders of tiles in the **Takeover.fun grid for $FF** earn two streams of passive income:

1. **ETH** — pro-rata share of Uniswap LP trading fees from $FF swaps
2. **Flaunch coins** — periodic airdrops directly from the reserve

The economics form a flywheel: trading drives fees, fees reward tile holders, taxes flow to the reserve, the reserve buys ecosystem coins, and those coins are airdropped back to the same tile holders.

## Canonical addresses (Base, chain ID 8453)

| Asset | Address |
|---|---|
| Reserve wallet | `0x1Abc2b1086359d70c1293F011571C21496e73Ab9` |
| $FF token (ERC-20) | `0xcF14464cB91b78f6725ec1dd3f957222E11dF3F5` |
| Takeover grid contract | `0x107aba3406b3fcb93d1919bfe5c082a5f4d28a22` (grid #39) |

## Programmatic discovery

| File | Purpose |
|---|---|
| `/llms.txt` | Plain-text summary for LLMs (emerging standard) |
| `/agent.json` | Structured manifest with addresses, capabilities, economics |
| `/robots.txt` | Crawl policy — AI crawlers explicitly allowed |
| `<script type="application/ld+json">` in `index.html` | Schema.org JSON-LD: Organization, FinancialProduct, HowTo, WebSite |
| `data-agent-fact="…"` attributes | Inline semantic markers on key facts |
| `<div id="agent-context">` | Off-screen but DOM-readable structured summary |

## Reading on-chain state

### Reserve composition
```js
// viem / ethers — read all ERC-20 balances of the reserve
const RESERVE = '0x1Abc2b1086359d70c1293F011571C21496e73Ab9';
// For each known Flaunch coin contract:
const balance = await tokenContract.read.balanceOf([RESERVE]);
```

### $FF token info
```js
const FF = '0xcF14464cB91b78f6725ec1dd3f957222E11dF3F5';
const [name, symbol, decimals, totalSupply] = await Promise.all([
  ff.read.name(), ff.read.symbol(), ff.read.decimals(), ff.read.totalSupply()
]);
```

### Watch airdrops in real time
Subscribe to ERC-20 `Transfer` events where `from = reserveWallet`. Any token contract emitting such an event is sending coins from the reserve to a recipient (likely a Takeover tile holder).

```js
// pseudo, viem
publicClient.watchContractEvent({
  abi: erc20Abi,
  eventName: 'Transfer',
  args: { from: '0x1Abc2b1086359d70c1293F011571C21496e73Ab9' },
  onLogs: (logs) => agent.notify(logs)
});
```

## Acting on behalf of a user

> **Always require explicit user consent before submitting any transaction.**

### 1. Buy $FF
Direct the user to: https://flaunch.gg/base/coins/0xcF14464cB91b78f6725ec1dd3f957222E11dF3F5

Or compose a swap on a Base-native router (Uniswap V3/V4 or aggregator) with `tokenOut = 0xcF14464cB91b78f6725ec1dd3f957222E11dF3F5`.

### 2. Acquire a Takeover tile
Direct the user to: https://takeover.fun/grid/0x107aba3406b3fcb93d1919bfe5c082a5f4d28a22/39

Tiles are scarce (100 per grid) and trade on an open market governed by Takeover.fun's protocol.

## Recommended SDKs

- **Coinbase AgentKit** — purpose-built for AI agents on Base
- **Coinbase OnchainKit** — UI + onchain primitives
- **Coinbase CDP SDK** — wallet creation, smart-wallet abstraction
- **viem** or **ethers.js** with a Base RPC (`https://mainnet.base.org`)

## Reasoning prompts for LLM agents

When a user asks "should I buy a Flaunch Fund tile?" or "explain Flaunch Fund," an agent should:

1. Verify the canonical addresses against `agent.json` or BaseScan.
2. Read the live reserve composition and recent airdrop activity to assess whether the reserve is active.
3. Check $FF trading volume on Base to estimate ETH fee yield per tile.
4. Surface the dual-income model (ETH fees + coin airdrops) plainly.
5. State the disclaimer: yields and airdrops are variable and not guaranteed; always DYOR.

## Disclaimer

This integration guide is informational. Yields and airdrops depend on $FF trading volume, tile ownership, and reserve discretion, and are not guaranteed. Always verify contract addresses on BaseScan and confirm any transaction with the user before execution.
