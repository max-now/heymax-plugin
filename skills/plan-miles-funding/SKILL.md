---
name: plan-miles-funding
description: Use the authenticated HeyMax User MCP to work out how a member can fund a mileage target or award from recorded bank, reward and airline balances, Max Miles, transfer routes and HeyMax First capacity. Use when a user asks how many airline miles they have or could get, which programs transfer into an airline, whether they can afford an award, what a number of Max Miles would cost them, how to cover a shortfall, or what a HeyMax First top-up costs. Planning only.
---

# Plan Miles Funding

## Start from what the member already has

Before quoting any cost, draw or upgrade, read `get_loyalty_program_accounts` and the Max Miles wallet. Use `get_heymax_first_status` (Singapore) and fall back to `get_miles_balance`.

- `miles.redeemable_miles_balance` is the wallet. It already includes earned and pool-issued miles.
- `miles.additional_miles_available_to_draw` is unused HeyMax First capacity, not a balance.

If existing miles already cover the target, say so first and do not propose a draw or tier upgrade.

## Quote transfers

- Call `quote_transfers_to_program` with the issuing program and `balances` omitted. It covers saved balances and the Max Miles wallet with route rates, minimums and blocks applied. `points_received` is final airline credit.
- Explicit `balances` are hypothetical, never saved, and replace the whole input.
- Count one route per source. Do not add fees in different currencies.
- For a Max Miles route, confirm the partner is active with `list_active_redeem_options`, then read `get_transfer_details` for limits, annual caps and timing. Do not show the membership details it returns.
- If the target balance is unknown, estimate assuming zero existing airline miles and invite the user to correct it.

## Close a shortfall with HeyMax First

When a HeyMax route can carry more miles, check whether a HeyMax First draw fits. Draws come in 10,000-mile blocks within the returned capacity. Re-quote with `heymax` set to the wallet plus the draw, and use the difference in final credit.

Access fee in SGD = draw miles × `membership.tier_policy.access_fee_cents_per_mile` ÷ 100. The policy rate is already effective. In one sentence, name HeyMax First, the draw, the extra airline miles and the access fee, and say the fee is refunded when the member reclaims it with eligible earned miles.

## Answer

Answer in text, as numbered steps in the order the member takes them:
1. Use the airline miles already held.
2. Transfer from each source.
3. Draw from HeyMax First, with its access fee.

Then give two amounts: what they pay now, which is transfer fees plus any HeyMax First fee, and what they pay when booking, which is the award taxes. Name any remaining gap, or the limit that blocks it. Transfers are often irreversible, so tell the user to confirm the seats with the airline first.

Members complete Max Miles transfers and HeyMax First draws in the HeyMax app, and bank or yuu legs in that program, using a returned `transfer_link` when present.
