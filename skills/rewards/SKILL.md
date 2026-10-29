---
name: rewards
description: Use the authenticated HeyMax User MCP to inspect a member's Max Miles, recent transactions, redemptions, merchant earning rates, upcoming deals, affiliate offers, tracked shopping links, and saved loyalty-program balances, or to update an explicitly confirmed recorded balance. Use when a user asks about their HeyMax account or Max Miles activity, where or how to earn more miles, where to buy for the best overall outcome, current merchant rates or deals, a HeyMax shopping link, or to remember a current loyalty-program balance.
---

# HeyMax Rewards

HeyMax helps members earn and manage Max Miles through everyday spending and partner rewards. Use the HeyMax User MCP to understand account activity, compare earning options, discover deals, and find tracked shopping links.

`update_loyalty_program_points_balance` is the only account-data write this plugin performs. Members complete Max Miles transfers, redemptions, and HeyMax First draws in the HeyMax app. Never call `request_transfer_otp`, `submit_transfer`, or `draw_heymax_first_pool`, and never ask for or accept a one-time code in chat, even when the session exposes those tools.

## Workflow

1. Inspect the HeyMax tools exposed by the MCP client for the current session and treat those tool definitions as canonical.
2. Identify the smallest set of tools that can answer the request. Preserve merchant names, filters, and conditions exactly as the user gave them.
3. If the HeyMax MCP requires authorization or a permission is missing, ask the user to connect or reconnect HeyMax and approve that permission in the browser OAuth flow. Never ask for passwords, access tokens, one-time codes, or full card numbers in chat.
4. Call tools only after resolving ambiguity that materially changes the lookup. Avoid broad account queries when a single targeted query is enough. Before a balance write, require an unambiguous program, an absolute current amount, and explicit save/update/remember intent.
5. Check tool results for `success: false`, `error`, or `action_required`. Explain the required user action instead of guessing or silently substituting data.
6. Summarize the result in the user's language. Separate data returned by HeyMax from advice or inference, and state when no result was found.

## Common Tasks

### Summarize an account

- Use `get_miles_balance` for the current balance.
- Add `get_recent_transactions` only when recent activity or a balance explanation is relevant.
- Add `get_redemption_history` only when the user asks about redemptions or a transfer's status.
- Omit internal user identifiers from the response.

### Save a recorded loyalty balance

- Use `get_loyalty_program_accounts` first when the existing recorded amount is not already known.
- Use `update_loyalty_program_points_balance` only when the user explicitly asks to save, update, or remember an absolute current balance for one supported program. An imperative containing the program and absolute amount is sufficient confirmation; otherwise ask once before writing.
- Treat the tool as an absolute setter, never a delta operation. For "add 10,000" or "deduct this award," ask the user to confirm the resulting current balance instead of calculating and saving it silently.
- Do not save a hypothetical, expected post-transfer, expected post-booking, or inferred balance.
- After success, state only the program and saved amount. Never repeat membership numbers, account email, internal account IDs, or other fields returned alongside the balance.
- Explain that this updates HeyMax's recorded balance for future planning; it does not change the balance at the bank or airline.
- Never use this tool for Max Miles or HeyMax First balances, which remain system-derived.

### Card questions

Use the `card-maximiser` skill for linked cards, card spend, rewards earned, caps, card terms, and which card to use. Do not assume card rewards and affiliate rewards stack; present them separately unless HeyMax explicitly confirms the combined rate.

### Find ways to earn

- Use `find_merchants` to search by merchant name or category.
- Use `get_upcoming_deals` for time-bounded promotions. Mention the returned end time, and do not call an offer current if its dates do not support that claim.
- After settling the final merchant list, call `render_affiliate_offers` once with up to eight deduplicated `find_merchants` results, ordered for the user. Deals from `get_upcoming_deals` cannot be rendered this way. When a specific conditional rate was chosen, pass its exact name in `selected_rate_names_by_merchant`. Keep the written answer complete in case the client does not display the card.
- Use `get_merchant_tracking_link` only when the user asks for a shopping link or chooses a merchant. Clearly label it as a tracked HeyMax link.

### Choose where to buy

Optimize the user's total outcome, not HeyMax usage or headline miles.

- Start with options that meet the user's hard requirements: product, specification, delivery country, deadline, and budget. Rewards cannot rescue the wrong product or merchant.
- Compare the all-in price, discounts, affiliate and card rewards, caps, minimum spend, exclusions, and payment-method conditions that the tools or public terms support. Use the `card-maximiser` skill for card rates, caps, and terms.
- Treat "up to" rates as conditional. Distinguish guaranteed values from estimates with a short inline label.
- Show cash and miles separately unless the user supplied a valuation; state it when you use it.
- Recommend a non-HeyMax path when it is materially better, and say why.
- Ask whether a purchase is online or in-store only when that could change the rate or earning path.

## Guardrails

- Treat Max Miles balances, transactions, cards, and redemptions as private account data.
- Do not expose raw authentication data or internal identifiers.
- Do not invent missing rates, eligibility, deal dates, card benefits, or redemption status.
- Do not claim a recorded balance was saved unless the update tool succeeds.
- Do not purchase, check out, or claim an order on the user's behalf. HeyMax tools return links and recommendations; the user completes the purchase with the merchant.
