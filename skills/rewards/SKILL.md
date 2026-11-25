---
name: rewards
description: Use the authenticated HeyMax User MCP for a member's Max Miles balance, transactions, redemptions, merchant earning rates, deals, affiliate offers, tracked shopping links, and recorded loyalty-program balances. Also use when the user wants to buy or book something, compares where to buy, or shares a Shopee, Lazada or Taobao product link, even if they do not mention HeyMax or miles. Skip purely informational product questions.
---

# HeyMax Rewards

Answer from HeyMax tool results, not general knowledge. Optimize the member's total outcome: recommend a non-HeyMax path when it is clearly better.

## Account

- Balance: `get_miles_balance`. Add `get_recent_transactions` or `get_redemption_history` only when activity or a redemption is relevant.
- Card questions belong to `card-maximiser`. Transfers, shortfalls and HeyMax First belong to `plan-miles-funding`.

## Where to buy

1. When the user intends to buy or book, call `find_merchants` before recommending a store. Pass `merchant_name` only for a named store; otherwise pass the relevant `categories` in one call. If web research finds a store, check it too.
2. Pick the rate from each merchant's `mpd_rates` that fits the purchase. `mpd` is Max Miles per dollar; `flat` is a fixed amount. "Up to" rates are conditional.
3. Call `render_affiliate_offers` once with the final ordered list, passing chosen rates in `selected_rate_names_by_merchant`.
4. In the written answer, link each recommended HeyMax merchant with its returned `tracking_url`, labelled as a tracked HeyMax link.

## Shopee, Lazada and Taobao products

When the user shares a Shopee, Lazada or Taobao product link, or you recommend a product page on one of them, call `get_product_tracking_link` with that URL. Give the product name, `earn_rate_mpd` and the returned `tracking_url`, labelled as a tracked HeyMax link. If it returns `message` instead of a link, pass that message on.

Card rewards and affiliate rewards are separate. Do not add them together unless HeyMax confirms they stack. Use `get_upcoming_deals` for dated promotions and give their end dates.

## Save a recorded balance

`update_loyalty_program_points_balance` is the only write this plugin makes. Call it only when the user asks to save, update or remember an absolute current balance for one program. It replaces the value; for "add 10,000" ask for the resulting balance. Say it updates HeyMax's planning record, not the airline or bank. It never applies to Max Miles or HeyMax First.

## Always

- Never show membership numbers, account emails or internal IDs.
- Lead with the answer. Do not narrate tool calls or name internal tools.
