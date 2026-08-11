---
date: 2026-07-20
video_id: It-RsnZMADg
channel: trvon
title: "The NEXT BIG THING is Coming! ($FUEL Whitepaper)"
duration: 00:22:41
published_at: 2026-07-20T15:17:33Z
video_type: regular
url: https://youtube.com/watch?v=It-RsnZMADg
named_assets: [fuel, more, weth, eth]
mentioned_assets: [eth]

## TL;DR
TrVon reads through his own Fuel protocol's whitepaper — a fair-launch "mint-to-earn" token on Ethereum with a 1,000-day one-way "pump fund" — framing it as the next big project he's building, currently live only on Sepolia testnet.

## Main points
1. Fuel is a fair-launch mint-to-earn ERC-20: no presale, no team allocation, no admin mint — tokens are created only via public minting, with routing/splits immutably welded at deployment.
2. Minting mechanics: participants commit to a 1-100 day term at launch (ceiling grows toward 1,000 as global mint count rises — ~349 days at 100K mints, 449 at 10M, 549 at 1B); payout scales with a reward amplifier starting at 3,000 (decaying 1/day to a floor of 1) and an early-adoption multiplier starting at +10% (decaying as mints accumulate), plus the log of how many mints happen after theirs.
3. Claims must happen within 7 days of maturity or face an escalating late penalty (0% day one, roughly doubling daily, capping at 99% from day seven); a "claim and remint" function auto-recycles positions, and he argues the math means recycling is always profit-maximizing as long as the protocol keeps growing.
4. Every mint/claim action pays an ETH fee approximately equal to its own gas cost, split 45% into a 1,000-day one-way "pump fund" accumulator, 25% into immediate perpetual Fuel buy-and-burn, and 30% into perpetual MORE burns (buys MORE and sends to a dead/burn address).
5. The pump fund holds its balance as wrapped staked ETH (Lido) to earn staking yield (illustrated at ~3.5% APR) while it accumulates for 1,000 days, then sweeps entirely into a month-long drip-release buying/burn campaign; he frames this as arriving "when fuel is at maximum distribution" after three years of fair minting with no insider overhang — illustrative math assumes ~1 ETH/day average fee inflow, which would put the fund over 450 ETH.
6. Burn engines (Fuel buy-and-burn and MORE burner) drip their ETH pools at 1%/day (owner-tunable 1-10%) in 144 ten-minute intervals, with keepers earning 1.5% of each interval for executing it; at equilibrium (e.g., 1 ETH/day inflow, 1% drip) the pool converges to 25 ETH and burns 100% of daily inflow (0.25 ETH/day) forever.
7. Separate fixed-term staking exists for Fuel holders (fee-free, burn-on-entry/remint-on-exit): APY starts at 25% and steps down 1% every 90 days to a floor of 2%, with early exit forfeiting all reward (principal-only) and rate locked at open, favoring early stakers.
8. Risk disclosures stated directly: unaudited code, unbounded/inflationary fuel supply, reflexivity that "cuts both ways" (falling adoption shrinks rewards/fees/burns together, can pause fee flow), thin early liquidity, the pump fund's first sweep is 3 years out with no guaranteed outcome, and current deployment is Sepolia testnet only (accelerated 7-day cycles) — mainnet parameters/addresses are not final.

## Outlook
| Asset | months (long) | weeks (mid) | days (short) |
|---|:-:|:-:|:-:|
| fuel |  |  |  |

## Named assets and stance
- Fuel — bullish (as builder), conviction 2 — his new mint-to-earn protocol; still on Sepolia testnet only, "nothing guarantees any outcome of any timeline."
- MORE — bullish, conviction 1 — 30% of Fuel's fee revenue is routed to continuous market-buy-and-burn of MORE.
- WETH/ETH — neutral, conviction 1 — Fuel's liquidity is deliberately paired against WETH rather than MORE "to quarantine" Fuel's inflation from hitting MORE.

## Levels and time-sensitive items
- Fuel launch terms: 1-100 day mint commitments at launch, ceiling rises toward 1,000 days as adoption grows (349 days at 100K global mints, 449 at 10M, 549 at 1B).
- Claim window: 7 days post-maturity; late penalty escalates (0%, 1%, 3%, 8%, 17%, 35%, 72%, capping 99% from day 7).
- Reward amplifier starts at 3,000, decays 1/day toward floor of 1; early-adoption bonus starts +10%, decays with global mint count.
- Fee split: 45% pump fund / 25% Fuel burn / 30% MORE burn; fee distributor triggers once 0.01 ETH accumulates.
- Pump fund: 1,000-day one-way accumulator from protocol's first mint; sweep window opens automatically 14 days before cycle ends (request/claim/sweep via Lido withdrawal queue); illustrative math at 1 ETH/day fees implies >450 ETH by sweep.
- Burn engine drip: 1%/day default (owner-tunable 1-10%), sliced into 144 ten-minute intervals; keeper reward 1.5% per interval; 15-minute TWAP guard on swaps.
- Staking APY: 25% at launch, -1% every 90 days, floor 2%.
- Current deployment: Sepolia testnet with accelerated 7-day cycles; mainnet parameters/addresses not yet final.
- Pump fund's first real sweep event is stated as "3 years out" from mainnet launch.

## Thesis shifts vs previous video
No arrows to compare — this video contains no scored Outlook rows (Fuel's cell is blank; pls, hex, and more from the prior brief are not addressed here). Note: prior brief had Fuel at ↑2 (months) as an upcoming launch "targeting end of July 2026"; this video confirms Fuel exists but only on testnet, with no live market outlook yet.
