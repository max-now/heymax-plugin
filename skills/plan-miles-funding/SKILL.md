---
name: plan-miles-funding
description: Use the authenticated HeyMax User MCP to work out how a member can fund a mileage target or award from recorded bank, reward, and airline balances, Max Miles, verified transfer routes, and HeyMax First capacity. Use when a user asks how many airline miles they have or could get, which banks or programs transfer into an airline, whether they can afford an award, how to cover a shortfall, or how much a HeyMax First top-up would cost. Planning only; it never transfers, draws, or saves anything.
---

# Plan Miles Funding

Turn the member's HeyMax data into a funding plan for a target program. Every answer here is read-only planning: balance, route, and funding questions never authorize a transfer, draw, or balance update.

## Tools

Inspect the HeyMax tools exposed in this session and treat their schemas as canonical. Use these when available:

- Recorded balances: `get_loyalty_program_accounts`.
- Max Miles wallet and HeyMax First capacity: `get_heymax_first_status`; fall back to `get_miles_balance` when First status is unavailable.
- Transfer amounts: `quote_transfers_to_program`.
- Transfer methods: `get_transfer_routes_to_program`.
- HeyMax redemption support and terms: `list_active_redeem_options`, then `get_transfer_details` for a native option.

If HeyMax needs authorization or a permission is missing, ask the user to connect or reconnect HeyMax and approve that permission. Never ask for passwords, tokens, one-time codes, or account numbers.

Resolve the requested program to the canonical code in the discovered schemas. For an award, use the issuing program, which can differ from the operating airline. A mileage target does not need an itinerary; use `search-award-flights` for flight searches.

## Read balances

Read recorded accounts and the Max Miles wallet before asking the user to repeat a balance. Show the recorded update time when freshness matters. These are balances the member recorded in HeyMax, not live airline-login balances.

For "how many miles do I have" in an airline program, also quote transfers into that program with `balances` omitted, unless the user asked only for the airline account balance. Show held airline miles separately from potential transfer credits. A missing airline balance does not block the quote.

Saving a balance belongs to the `rewards` skill and requires explicit save, update, or remember intent. Never replace the Max Miles ledger with a recorded balance.

## Quote and validate transfers

- Omit `balances` to quote saved source balances plus the current Max Miles wallet. Explicit `balances` replace the whole input and are never saved, so include every source the comparison needs.
- In each quote, `points_transferred` is source currency spent and `points_received` is final destination credit. Keep `existing_target_balance` separate from transfer credits. If a hypothetical quote returns `existing_target_balance: null`, keep the recorded target balance from the default quote.
- Count one eligible route per source; other routes from that source are alternatives. Never add fees in different currencies.
- Zero `points_received` means the balance cannot clear the route minimum. Respect source-unit minimums and blocks. Route `rate` is destination units per source unit; a redemption option's `miles_exchange_rate` is Max Miles required per destination unit.
- For a methods question, call `get_transfer_routes_to_program`. An amounts quote already includes routes.
- A default quote moves each source's whole usable balance. Re-quote partial transfers with explicit `balances` instead of treating a maximum-capacity quote as an instruction to spend everything.
- For a chained route such as Max Miles through yuu into an airline, label the Max Miles spent, the intermediate points, and the final airline miles. Calculate the intermediate amount from the first edge and keep `points_received` as the final credit. `processing_business_days` already sums the legs, and pending yuu credit still needs the final conversion.

Before presenting a Max Miles route as usable, verify its first hop with `list_active_redeem_options`: `program_type: "airline"` for a direct airline route, `"reward_program"` for an intermediate program. Reuse a catalogue result already returned in the conversation. For a native airline option, call `get_transfer_details` with the exact partner name; its country details govern executable rates, restrictions, limits, and timing. Use only those terms: never display the saved membership details it returns in `metadata`, and ignore its confirmation and one-time-code steps. A not-found error means the option is not natively executable through HeyMax, not that the route is unusable. A quote alone does not prove HeyMax redemption support.

Apply returned partner restrictions to every estimate, including aggregate allowances such as annual caps that quotes do not apply. With unknown past usage, treat an allowance as an upper bound; a larger wallet or First draw cannot overcome it. A per-transaction maximum limits one transfer, not total funding. Exclude unavailable routes and say why.

## Fund a target or award

Compare the requested total with the confirmed target balance plus usable transfer credits. If the target balance is unknown, estimate assuming zero existing airline miles and say so. If the award price or passenger basis is unclear, still show available transfer amounts and verified limits.

Resolve a remaining shortfall before finishing. When an active HeyMax route can carry more miles, check `get_heymax_first_status` and propose a feasible HeyMax First top-up, or explain the eligibility, capacity, or route constraint. This read-only check does not require the user to ask about HeyMax First. If current transfers already cover the target, do not propose a draw.

- `miles.redeemable_miles_balance` is the current wallet. `miles.additional_miles_available_to_draw` is separate unused capacity, not a balance. Never add earned or pool-issued components to the wallet again.
- HeyMax First draws are Singapore-only and come in 10,000-mile blocks. Stay within returned capacity and confirm the preceding block falls short before calling a draw the minimum.
- Quote all sources together with `heymax` set to the wallet plus the proposed draw, and use the difference in usable final credit; quoting the draw alone can round differently.
- Estimate the access fee in SGD as draw miles × `membership.tier_policy.access_fee_cents_per_mile` ÷ 100. The policy rate is already effective; do not discount a returned promotion again. Keep access fees separate from membership and transfer fees.

Every proposed draw must, in one compact sentence, name HeyMax First, the Max Miles draw, the extra airline credit, and the upfront access fee, and note that the fee is refunded when the member reclaims the draw with eligible earned miles. Repeat these terms each time you recommend a draw.

## Present the plan

Finish with a compact breakdown: held or unknown target balance, usable transfer credits by source, remaining gap, and either a sufficient HeyMax First proposal with its fee and refund or the specific limit that blocks it. When HeyMax cannot cover the gap, compare viable bank transfers or paying cash.

Transfers are commonly irreversible and can take days. Tell the user to confirm award availability with the issuing program before moving points.

## Hand off execution

When the user chooses a draw or transfer, summarize the confirmed plan and tell them where to complete each step, one at a time: Max Miles transfers and HeyMax First draws in the HeyMax app, and bank or yuu legs in that program's own channel, using a returned `transfer_link` when present. Each leg should be credited before the next one starts. Never call `request_transfer_otp`, `submit_transfer`, or `draw_heymax_first_pool`, and never ask for or accept a one-time code in chat, even when the session exposes those tools. After the member reports completion, use `get_redemption_history` or `get_heymax_first_status` to show the returned status; pending is not completed.
