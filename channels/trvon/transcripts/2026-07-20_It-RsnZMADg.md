---
video_id: It-RsnZMADg
channel: trvon
title: "The NEXT BIG THING is Coming! ($FUEL Whitepaper)"
published: 2026-07-20
duration: 00:22:41
url: https://youtube.com/watch?v=It-RsnZMADg
---

[00:00] Welcome to the Fuel white paper,
[00:02] presented by moretokens.com.
[00:04] Fuel,
[00:05] a mint-to-earn protocol where
[00:07] participation is the product, promotion
[00:09] is the incentive, and every fee burns
[00:12] forever.
[00:14] Abstract.
[00:15] Fuel is a fair launch mint-to-earn token
[00:18] whose reward function makes every
[00:20] participant financially long the
[00:22] protocol's own growth. Tokens are
[00:24] created only by public minting. Each
[00:26] minter's payout scales with the
[00:28] logarithm of how many mints begin after
[00:30] theirs, converting every holder into a
[00:32] self-interested promoter without
[00:35] referral codes, trust, or off-chain
[00:37] accounting. Every mint action, opening
[00:40] and claiming, pays an ETH protocol fee
[00:42] approximately equal to its own gas cost,
[00:45] and the built-in claim and remint cycle
[00:47] makes paying it a self-renewing habit.
[00:50] Fees route immediately, 45% into a
[00:52] 1,000-day one-way accumulator, the pump
[00:55] fund, 25% into an immediate perpetual
[00:58] fuel burn, 30% into perpetual more
[01:00] burns. All execution is permissionless
[01:03] and keeper-incentivized.
[01:05] This paper specifies the mechanisms,
[01:07] quantifies the incentive structure, and
[01:09] states assumptions and risk plainly.
[01:12] Design thesis: Most token launches
[01:14] purchase attention, marketing budgets,
[01:17] coal allocations, liquidity incentives.
[01:19] Fuel inverts this. The protocol pays for
[01:22] attention with math instead of treasury.
[01:25] The minting reward function is
[01:26] deliberately shaped so that the single
[01:28] most profitable action any existing
[01:30] participant can take, beyond minting
[01:33] again, is causing more people to mint.
[01:36] Growth expenditure is thereby
[01:37] crowdsourced to the entire holder base
[01:40] in the same activity that grows the
[01:41] network simultaneously funds the burning
[01:44] engines that concentrate its value.
[01:46] Three properties anchor the design: fair
[01:48] creation, no presale, no team
[01:50] allocation, no admin mint. The public
[01:53] mint function is the sole source of the
[01:55] supply. Immutable routing.
[01:58] The destinations and splits are welded
[01:59] at deployment. No key can redirect them.
[02:03] Permissionless operation. Every
[02:05] recurring protocol action is a public
[02:07] function with the caller reward. So, the
[02:09] system runs without any operator.
[02:11] Minting mechanics. A participant opens a
[02:14] mint by committing to a term of days, 1
[02:16] to 100 days at launch. The ceiling grows
[02:19] with an adoption towards 1,000. The
[02:21] protocol records their position in the
[02:23] global mint counter. Mint number M mint
[02:27] number NM of all mints are ever opened.
[02:30] At maturity, the participant claims and
[02:32] receives, where M is the number of mints
[02:34] opened after theirs during the term,
[02:37] lowered at two. Amp is a reward
[02:39] amplifier that starts at 3,000 and
[02:40] declines by one per day towards a floor
[02:43] of one. And EAA is an early adoption
[02:46] multiplier starting at plus 10% and
[02:48] decaying as global mints accumulate.
[02:50] Claims must occur within 7 days of
[02:52] maturity.
[02:53] Lateness is penalized with an escalating
[02:55] curve to 99%.
[02:57] The 100-day launch ceiling is not fixed.
[03:00] Once the global mint counter passes
[03:02] 5,000, the maximum term grows with the
[03:05] logarithm of adoption, roughly 349 days
[03:08] at 100,000 global mints,
[03:10] 449 at 10 million,
[03:13] 549 at 1 billion, and it's capped at
[03:16] 1,000.
[03:17] Commitment capacity is therefore itself
[03:19] a network good. The community's growth
[03:22] unlocks longer terms,
[03:23] and longer terms are the highest
[03:26] leverage positions on further growth.
[03:28] Long horizon conviction becomes
[03:30] available exactly in the proportion to
[03:32] demonstrated adoption.
[03:34] Long horizon conviction becomes
[03:35] available exactly in proportion to
[03:37] demonstrated adoption.
[03:39] In this figure, the term calling is a
[03:42] function of global mints on the log
[03:44] x-axis. Growth unlocks at 5,000 mints
[03:46] and compounds slowly.
[03:48] Each doubling of adoption adds 15 days
[03:50] of maximum commitment. The late penalty.
[03:53] Maturity is not a deadline to claim by.
[03:56] It is the start of a 7-day claim window.
[03:58] Claiming on the day maturity hits cost
[04:00] nothing extra.
[04:01] Waiting longer inside the window costs a
[04:03] small and rapidly escalating amount. And
[04:06] claiming after that window closes caps
[04:08] the penalty at its ceiling.
[04:10] The curve roughly doubles every day. 0%
[04:12] the first day, then 1 3 8 17 35 72
[04:18] capping at 99% from day seven onward.
[04:21] There is no way to lose the position
[04:23] entirely.
[04:24] Some reward is always claimable, but the
[04:26] design makes procrastination expensive
[04:28] fast. Which keeps matured positions from
[04:31] sitting idle and gives the claim and
[04:33] remint flow, which we'll go over in
[04:35] section five. A real behavioral nudge
[04:38] behind it, not just a convenience.
[04:41] In the figure above, the exact on-chain
[04:43] penalty curve. Claiming inside the first
[04:45] day is free. The cost of waiting
[04:47] compounds quickly enough that most
[04:49] rational holders claim well before the
[04:51] window closes.
[04:52] Your reward is a function of everyone
[04:54] who mints after you. In the figure
[04:56] above, claim payout versus mints open
[04:58] after yours per the on-chain formula.
[05:01] The x-axis is the protocol's growth
[05:03] during your term. Your reward is a
[05:05] direct increasing function of adoption.
[05:07] Two structural consequences follow.
[05:10] First, the logarithm rewards breadth
[05:12] over timing games.
[05:14] Payouts grow smoothly with adoption
[05:15] rather than clip jumping.
[05:17] So, there is no single moment to snipe.
[05:19] Second, term length is a conviction
[05:21] lever.
[05:22] Reward scales linearly in T. So, a
[05:25] 100-day commitment earns 100x the per
[05:28] mint growth payout on a one-day flip,
[05:31] but exposes the minter to 100 days of
[05:33] adoption, which is precisely the
[05:34] protocol recruiting long horizon
[05:36] promoters. Two clocks that permanently
[05:39] favor early participants. Reward
[05:41] amplifier, early adoption bonus. The two
[05:44] early advantage clocks. MP decays with
[05:46] calendar time. The early adopter bonus
[05:49] decays with global mint count. Both are
[05:51] permanent. A mint open today lock
[05:53] today's multipliers forever.
[05:56] The marketing engine. Consider a
[05:57] participant holding an open mint. Their
[05:59] claim value is equal to R, where every
[06:02] variable is fixed at open except M minus
[06:05] global adoption during their term.
[06:07] The marginal value of one additional
[06:09] recruited mentor is this calculation
[06:11] greater than zero, always. Every open
[06:14] mint is therefore a live financial
[06:16] position on protocol growth, and the
[06:18] holder's rational behavior is promotion.
[06:21] This differs from referral programs in
[06:23] three load-bearing ways.
[06:25] It is trustless. No codes, no
[06:28] attribution disputes. The global counter
[06:30] is the only ledger. Uncapped and
[06:33] self-funding. The flywheel closes on
[06:35] itself.
[06:37] The self-funding marketing engine. You
[06:39] mint fuel.
[06:41] You are now paid to grow global mints.
[06:43] You promote new users mint. Every mint
[06:47] pays fees.
[06:48] The starting claim fee.
[06:50] Claim and remint auto restarts the mint.
[06:54] The self-funding growth loop. Note the
[06:56] highlighted note. Claim and remint
[06:58] automatically reopens each mint at a
[07:00] claim. So, the fee paying step recurs by
[07:03] default rather than by decision.
[07:06] Promotion is individually rational at
[07:08] every node.
[07:10] A quantified example. A participant
[07:12] opens 50 mints at a 30-day term on day
[07:15] one. If the protocol averages 1,000 new
[07:18] mints per day, 30,000 and each mint
[07:21] claims 1.4 million fuel. If their
[07:24] promotion helps push the pace to 3,000
[07:26] per day, 90,000 and each claims equals
[07:29] 1.6 million. An 11% raise on their
[07:33] entire position for growing the network.
[07:35] The gradient never inverts. More
[07:37] adoption is always personally
[07:39] profitable.
[07:40] Promoter's payoff. Your open position
[07:42] appreciates with the pace you help
[07:44] create. The promoters' payoff curve
[07:46] claim value per open mint as a function
[07:48] of the protocol's daily mint pace
[07:51] for three term lengths. The shaded band
[07:53] is the worked example because holders
[07:55] choose the x-axis with their own
[07:57] promotion.
[07:58] Every open position is a self-interested
[08:01] growth campaign.
[08:03] The fee engine. Every mint action pays
[08:06] an ETH protocol fee once at open, once
[08:09] at claim. Solo mint gas benchmarks the
[08:12] gas of one unbashed mint, so the fee
[08:15] equals the gas the participant already
[08:17] paid.
[08:18] Participation costs roughly two times
[08:21] gas
[08:22] gas
[08:22] gas with the second half captured by the
[08:24] protocol.
[08:25] The fee floats with network conditions
[08:27] and is floored so it cannot be gained
[08:29] towards zero.
[08:31] Parameters are only owner turnable only
[08:34] inside hard-coded bounds. Calibration is
[08:37] possible, weaponization and zeroing are
[08:40] not. The batcher penalty. The fee is per
[08:43] mint per action with no batch or remint
[08:45] discount. Batching 100 mints amortizes
[08:48] real gas by an order of magnitude. It
[08:51] amortizes fees not at all. Industrial
[08:54] minters who captured disproportionate
[08:56] reward through scale thereby fund the
[08:59] burn engines disproportionately.
[09:02] The perpetual cycle. Batch minting and
[09:04] claim and remint. The batch minter
[09:06] contract lets one wallet operate up to
[09:09] 100 mints per transaction. Each held at
[09:11] its own deterministic proxy address that
[09:14] is reusable forever.
[09:15] Its centerpiece is claim and remint. A
[09:18] single transaction that claims a matured
[09:20] batch delivering the fuel to the wallet
[09:23] and in the same breath reopens every
[09:25] position for a fresh term. One click,
[09:28] the claim fee and the new open fee are
[09:30] both paid. The participant's exposure to
[09:33] future adoption never lapses. The
[09:35] rationality condition. A participant
[09:37] continues cycling as long as the
[09:39] expected claim value exceeds the round
[09:40] trip cost. Because the fee is calibrated
[09:43] to equal one transaction's gas, the
[09:46] right side is small, on the order of a
[09:48] single swap, while the left side scales
[09:50] with AMP, term, and global adoption. For
[09:54] any position open while the protocol
[09:56] grows,
[09:57] the inequality holds by a wide margin,
[10:00] and recycling is simply the
[10:01] profit-maximizing move. The consequence
[10:04] is structural.
[10:05] Rational self-interest produces a
[10:07] standing population of perpetually
[10:09] renewed mints,
[10:11] each paying two fees per cycle into the
[10:13] engines indefinitely. The pump fund's
[10:15] 1,000-day accumulation is not fed by a
[10:18] launch spike that decays. It is fed by a
[10:21] habit that the reward function itself
[10:22] keeps renewing.
[10:24] Recycle cadence, shorter terms, fuel the
[10:27] engine harder. Recycle cadence,
[10:29] shorter terms, fuel the engines harder.
[10:32] Feed throughput per perpetually recycled
[10:34] mint by chosen term. A 1-day recycler
[10:38] contributes 730 fees per mint per year.
[10:42] Even a conservative 30-day recycler
[10:44] contributes 24.
[10:47] Multiply by batch size and both fees per
[10:49] recycle.
[10:51] The engine inflow is a function of
[10:53] standing participation, not launch hype.
[10:56] Note the interplay with section three.
[10:58] Each recycle also reopens a life
[11:00] position on future growth, refreshing
[11:03] the holders' incentive to promote. The
[11:05] marketing engine and fee engine share
[11:07] the same crank. Flow of funds: 45, 25,
[11:12] 30.
[11:13] 30.
[11:13] 30. 45%
[11:15] 45%
[11:15] 45% the 1,000-day pump fund.
[11:17] One-way accumulator
[11:19] fills for 1,000 days. The entire balance
[11:22] then converts to fuel by pressure. Cycle
[11:24] restarts immediately, forever. 25%
[11:28] Fuel buy and burn
[11:30] enters the drip engine immediately.
[11:32] begins burning the same day.
[11:34] 30% more burner. Market buys more
[11:37] continuously, proceeds to 0x debt. Fees
[11:40] pool in an ownerless fee distributor.
[11:43] Anyone may trigger the split once 0.01
[11:45] ETH accumulates. The 1,000-day pump
[11:48] fund. The pump fund is a one-way
[11:50] accumulator. For 1,000 days from the
[11:52] protocol's first mint, 45% of every fee
[11:55] enters and nothing exits. No owner path,
[11:58] no emergency valve. At cycle end, the
[12:01] permissionless sweep moves the entire
[12:03] balance into the burn drip engine, and
[12:06] the next cycle begins the same block.
[12:08] Pump fund accumulation, sweep, and drip
[12:11] release.
[12:12] Accumulation under several fee and flow
[12:14] scenarios and the post-sweep release.
[12:17] Swept ETH deploys through the drip
[12:19] engine, converting day 1,000 into a
[12:22] month-long buying campaign rather than a
[12:24] single absorbable spike. The convergence
[12:27] thesis. The sweep arrives precisely when
[12:29] fuel is at maximum distribution. Three
[12:31] years of fair minting. Thousands of
[12:34] independent wallets, zero insider
[12:36] overhang. The fund size is a pure mirror
[12:39] of the cumulative usage. At a mean
[12:42] inflow of just one ETH per day of fees,
[12:44] it exceeds 450 ETH. The data is public
[12:47] from block one, and the market may price
[12:49] it accordingly. Release through the drip
[12:51] engine prevents single block absorption
[12:54] by arbitrage. And cycle two begins
[12:56] filling immediately, making the event
[12:58] periodic rather than terminal. This is a
[13:01] design description with illustrative
[13:03] arithmetic, not a projection. Inflows
[13:05] depend entirely on adoption. Price
[13:08] depends on far more than buy pressure.
[13:10] Staking the fund wallet weights.
[13:13] A balance that sits untouched for up to
[13:15] 1,000 days is, by construction, idle
[13:18] capital. And Ethereum offers a native
[13:20] permissionless way to stop it from being
[13:22] idle. The pump fund's design holds its
[13:24] balance as wrapped staked ETH,
[13:27] Lido's non-rebasing liquid staking
[13:28] token, rather than raw ETH. Incoming
[13:31] fees are staked the moment they arrive,
[13:33] and the balance earns Ethereum consensus
[13:35] layer staking yield for every day it
[13:37] waits, on top of the fee inflow the
[13:40] split already guarantees, not instead of
[13:42] it. Wrapped staked ETH specifically, not
[13:45] staked ETH, matters here. It is
[13:47] share-based rather than rebasing, so a
[13:50] contract holding it doesn't need special
[13:52] accounting for a balance that changes on
[13:54] its own. The exchange rate to ETH moves
[13:56] instead, which is the property that
[13:58] makes it safe for a permissionless,
[14:00] ownerless contract to simply hold.
[14:03] Staking the pump fund, the yield gap
[14:05] widens for the whole cycle. Pump fund
[14:07] accumulation at 45% of one ETH a day of
[14:11] fees staked versus unstaked across a
[14:13] range of illustrative Lido APRs,
[14:16] historically roughly 2 to 5% itself
[14:19] variable and not guaranteed.
[14:21] At a typical 3.5% a day, 1,000 lands
[14:25] meaningfully above the unstaked baseline
[14:27] yield the protocol did nothing to
[14:29] generate beyond choosing to stake
[14:31] instead of holding cash.
[14:33] Exiting without eating the gain, yield
[14:35] captured on the way in is only real if
[14:37] it survives the way out. A multi-hundred
[14:39] or multi-thousand ETH balance dumped
[14:41] through a decentralized exchange in one
[14:44] transaction would move price against
[14:45] itself, the same slippage mechanics that
[14:47] govern the burn engines' own drip design
[14:51] in section 8, just larger in scale. The
[14:53] fund avoids this by exiting through
[14:55] Lido's native withdrawal queue instead
[14:57] of a DEX.
[14:58] Wrapped staked ETH redeems for ETH at
[15:00] its true value, one-to-one backing, with
[15:03] no pool, no price impact, and no size
[15:06] limit beyond patience.
[15:08] The unwind runs in three permissionless
[15:10] steps, request, claim, sweep, timed to a
[15:14] window that opens automatically 14 days
[15:17] before each cycle ends,
[15:19] comfortably ahead of Lido's typical
[15:21] processing time, even under load. The
[15:24] result, every ETH the fund earned by
[15:26] staking arrives at the burn engine
[15:28] intact at full value, regardless of how
[15:31] large the balance has grown. Earn
[15:33] engines and keeper economics.
[15:35] Each burn engine drips its ETH pool at
[15:37] 1% per day.
[15:39] Owner tunable, 1 to 10%. Sliced into 144
[15:43] 10-minute intervals.
[15:45] Any address may execute the current
[15:47] interval, swapping ETH for fuel through
[15:50] the protocol owned pool,
[15:52] or for more through its market, and
[15:54] burning the proceeds,
[15:56] earning 1.5% of the interval in ETH.
[15:59] Swaps are guarded by a 15-minute TWAP
[16:01] with bounded slippage. At drip scale,
[16:04] sandwich extraction is unprofitable.
[16:06] Missing intervals accumulate and execute
[16:08] together,
[16:10] so the schedule batches rather than
[16:11] falls behind.
[16:13] The instant leg, steady-state burning.
[16:16] The 25% leg deserves its own analysis.
[16:19] It receives fee inflow continuously and
[16:21] drips out at a fixed percentage.
[16:24] Classic leaky bucket system with a
[16:26] provable equilibrium. The pool grows
[16:28] until rate times pool equals inflow, at
[16:31] which point daily burn equals daily
[16:33] inflow exactly.
[16:35] At 1 ETH per day
[16:37] of protocol fees, the default 1% day
[16:40] drip, the pool converges towards 25 ETH,
[16:43] and thereafter burns 0.25 ETH of fuel
[16:47] per day,
[16:48] 100% of its inflow,
[16:50] forever,
[16:51] while retaining a 25 ETH war chest
[16:54] that smooths burning through any lull in
[16:56] activity.
[16:58] The drip rate does not change how much
[16:59] ultimately burns. Everything does. It
[17:02] only tunes the size of the buffer and
[17:04] speed of convergence.
[17:06] The self-burn pool self-balances. At
[17:09] equilibrium, it burns everything it
[17:11] receives.
[17:12] The instant pool converging to
[17:14] equilibrium under constant inflow
[17:16] at two trip settings.
[17:18] Daily burn dotted
[17:20] asymptotes to the inflow line in both
[17:23] cases.
[17:24] The leg burns everything it is fed with
[17:27] the drip rate setting only the buffer
[17:28] size.
[17:30] Keeper economics, burns become
[17:32] self-executing as pools grow. Protocol
[17:35] on liquidity, the fuel buy and burn
[17:37] creates and permanently holds the fuel
[17:39] WETH pool seeded from early fee ETH
[17:41] against a one-time LP mint. LP fees
[17:44] earned in fuel are burned. The WETH side
[17:47] funds ecosystem operations. Pairing
[17:49] against WETH, not more, is deliberate.
[17:53] Fuel supply is inflationary by design
[17:55] and a more pairing would transmit that
[17:57] inflation into more sell pressure. WETH
[17:59] pairing quarantines the inflation while
[18:02] 30% leg hands more pure supply blind buy
[18:05] pressure.
[18:06] Architecture and trust model, fuel token
[18:09] ERC-20 mint, stake, burn mechanics
[18:13] collects the fee on every open and
[18:14] claim.
[18:15] On chain, the open function retains its
[18:18] lineage, name, claim rank. This paper's
[18:21] mint number is that counter.
[18:23] Batch mentor deterministic proxy factor
[18:26] up to 100 mints per transaction.
[18:30] Full fee per proxy, claim and remit. Fee
[18:33] distributor, ownerless
[18:35] 45-25-30 splitter
[18:38] destinations immutable.
[18:40] Pump fund
[18:41] ownerless 1,000 day accumulator on chain
[18:44] mint vault
[18:45] exit only via sweep owned to the burn
[18:48] engine.
[18:49] Fuel buy and burn
[18:50] drip engine plus permanent protocol
[18:52] owned fuel WETH liquidity.
[18:55] More burner, drip engine, more to 0x
[18:58] dead owner powers bounded
[19:01] fee benchmark and gas floor inside hard
[19:03] limits. Drip rate 1% 1 to 10% a day.
[19:07] Swap slippage tolerance
[19:09] staking independent of minting, any fuel
[19:12] holder may stake their tokens directly
[19:14] with the token contract for a fixed term
[19:16] and a protocol-wide APY.
[19:18] Staking is fee-free. No protocol fee is
[19:20] charged to open or close a stake.
[19:23] Only ordinary network gas. And
[19:25] collateral simple. Stake fuel is burned
[19:28] on entry and re-minted principal plus
[19:30] reward on exit. So, the position
[19:32] requires no token approval and cannot be
[19:35] re-hypothecated elsewhere while locked.
[19:38] Mechanics.
[19:39] A wallet holds one stake position at a
[19:41] time. Opening a new stake while one is
[19:43] active is not permitted.
[19:45] The existing position must be withdrawn
[19:47] first.
[19:48] Reward accrues only if the stake is
[19:50] withdrawn at or after maturity.
[19:53] APY is fixed at the protocol-wide rate
[19:55] in effect the moment the stake opens,
[19:58] not the rate on the day it matures.
[20:00] There is no early exit penalty in the
[20:02] punitive sense.
[20:04] Withdrawn before maturity simply returns
[20:06] principal only, zero reward. Since the
[20:09] reward formula of APY is to zero before
[20:11] the term completes. Capital is never at
[20:14] risk from staking itself.
[20:16] Only the yield is conditional on
[20:17] patience.
[20:19] A protocol-wide decaying rate.
[20:21] Unlike the mint rewards per position
[20:23] amplifier, APY is a single global value
[20:25] that decays on a fixed calendar
[20:27] independent of any individual stake. 25%
[20:30] at launch, stepping down by 1% on every
[20:33] 90 days, floor at 2%. Every stake opened
[20:37] at a given moment locks that moment's
[20:39] rate for its full term. So, as with
[20:42] minting, early stakers are structurally
[20:45] favored.
[20:47] The same 90-day commitment opened on day
[20:49] one earns materially more than the
[20:51] identical commitment opened two years
[20:54] later.
[20:55] Staking, a fee-free burn and re-mint
[20:57] reward system. Relationship to the fee
[21:00] engine. Staking does not itself route
[21:02] ETH into 45 25 30 split. It is a
[21:05] fuel-dominated yield mechanism, not a
[21:07] fee event. Its economic role is
[21:09] complementary. Minting and claim and
[21:12] remint generate the ETH that fuels the
[21:15] burn engines, while staking gives
[21:16] holders a reason to remove freshly
[21:18] minted fuel from circulating supply and
[21:21] hold it through us through a lock,
[21:23] tempering the sell pressure that purely
[21:25] inflationary mint schedule would
[21:26] otherwise create.
[21:28] Risk disclosures. Unaudited code. No
[21:31] third-party audit has been performed.
[21:33] Contract bugs can permanently destroy
[21:35] funds. Unbounded supply. Fuel supply
[21:38] grows with every claim indefinitely.
[21:41] Burn engines are not a counterweight,
[21:43] not a promise of price behavior.
[21:45] Reflexivity cuts both ways. The same
[21:48] loop that amplifies growth amplifies
[21:50] contraction. Falling adoption shrinks
[21:52] rewards, fees, and burns together.
[21:55] The recycled inequality can flip,
[21:57] pausing fee flow until conditions
[21:59] recover.
[22:00] Thin early liquidity.
[22:02] The protocol seeded pool starts small.
[22:06] Early prices are trivially movable. And
[22:08] illustrative figures assume price that
[22:10] large scales would themselves
[22:12] invalidate. Long horizons. The pump
[22:15] funds first sweep is 3 years out.
[22:18] Nothing guarantees any outcome of any
[22:20] timeline.
[22:22] Testnet status. Current deployment is
[22:24] Sepolia with accelerated 7-day cycles.
[22:28] Mainnet parameters and addresses are not
[22:30] final.
[22:32] Nothing herein is financial advice or an
[22:35] offer of securities. Participate only
[22:37] with capital you can lose entirely.
