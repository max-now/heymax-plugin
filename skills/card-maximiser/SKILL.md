---
name: card-maximiser
description: Use the authenticated HeyMax User MCP to analyse a member's linked-card spending, rewards earned, bonus caps and minimum spend, pick the best linked card for a merchant, category, payment method or overseas spend, explain card terms, exclusions and MCCs (including for cards the member does not hold), find where the wrong card was used, and compare which cards earn the most miles in a loyalty program. Use for any card spend, card reward, cap, or "which card should I use" question from a Singapore member.
---

# Card Maximiser

Answer card questions from HeyMax's card data, never from general knowledge. Max Miles balances and transfers belong to the `plan-miles-funding` skill.

## Tools

Inspect the HeyMax tools exposed in this session and treat their schemas as canonical:

- Linked cards: `get_linked_cards`.
- Spend totals and transactions: `get_card_spend_transactions`.
- Rewards, per-card totals and cap use: `get_card_rewards`.
- Transactions by merchant, category or payment channel, with rewards: `search_card_spend_transactions`.
- Rates, caps, minimum spend, exclusions, warnings and the current cycle: `get_card_reward_structure`.
- Detailed terms and changes: `get_card_rules`. It returns the README and `available_documents`; request another by `document`.
- Best card for a merchant, category, payment channel or overseas spend: `rank_linked_cards_for_merchant`.
- What an MCC means: `get_mcc_info`. A merchant's MCC: `rank_linked_cards_for_merchant`, in its `merchant` result.
- Card points into a loyalty program: `get_transfer_routes_to_program`.

If a card tool requires authorization, ask the user to reconnect HeyMax. If a tool returns `action_required: "unsupported_region"`, say card analysis covers Singapore members only.

Query all cards unless the user names one, and group results by card. `card_name` accepts a card, alias or bank name; a bank matches all its cards. In searches, pass the card in `card_name`, not inside `query`. Use `category` for spend categories such as dining or fuel.

## Ground every number

- Take every spend, reward, count, card rate, cap and MCC from a tool result. Never infer a merchant's MCC from general knowledge. Use the totals a tool returns instead of adding up transactions.
- When a tool reports no transactions for a period, say there were none in that period.
- For a follow-up or a disputed figure, call the tool again with the same dates and card instead of reusing remembered numbers, and show the transactions behind it.
- Rewards from `get_card_rewards` and `search_card_spend_transactions` already apply caps, minimum spend, rounding and exclusions. Use them as returned.
- Confirm a bonus from the card's included MCCs and payment tags or a transaction's `is_bonus`. A broad category name does not mean every MCC in it earns the bonus.
- Wallet top-ups such as GrabPay, YouTrip, ShopeePay and Singtel Dash earn nothing unless a tool result says they do.
- CardUp payments appear as `CARDUP-<payee>` with CardUp's MCC 7399. Name the payee, as in "AIA via CardUp", and keep the computed reward.
- Name cards by the `card_short_name` a tool returned, never by ID. Similar names such as UOB PV and UOB PRVI are different cards.

## Dates

Resolve periods against today's Singapore date and always pass both `start_date` and `end_date`. Without a named period, use this month, or the last 90 days for a transaction search. A month without a year means the latest such month that is not in the future. "Last month" is the previous calendar month, and "this month" runs from the 1st to today. "Last N months" means the N most recent complete calendar months, not a rolling window. A follow-up keeps the earlier period unless the user names a new one. State the range used for a multi-month answer.

## Present spend and rewards

Lead with the period total, then one short line per card from `card_totals`, not a table. Give transaction detail only when asked. Describe categories and exclusions in plain words, name first: "Restaurants (MCC 5812)".

Always split settled and pending spend. Pending spend earns nothing until it settles, so never attach rewards to it. Mention a cap only when it is hit or close. Show a hit cap as the split it caused, such as "S$1,200 at 4 mpd + S$428 at 0.4 mpd". Label the period behind every cap figure and never mix periods.

Illustrative values only; use real results:

- "September: S$3,391 over 112 transactions (107 settled, 5 pending). The settled S$3,201 earned **11,830 miles**; the S$190 pending earns when it settles."
- "**UOB Visa Signature**: S$2,736 settled, 9,394 miles (3.4 mpd), led by Best Denki."
- "FCY cap: S$700 settled + S$200 pending of S$1,200, so S$300 left this statement period (15 Sep to 14 Oct)."

Cards in `untracked_cards` have no transactions, but their terms still work. Mention them at most once. When every transaction on a tracked card has no reward, say reward calculation is not supported for that card yet, not that it earned nothing. With no linked cards, answer card terms and MCC questions, and say that linking cards in HeyMax enables spend tracking and best-card ranking.

## Caps, minimum spend and terms

For cap, minimum-spend or "how much is left" questions, call `get_card_reward_structure` for the rules and current cycle, then `get_card_rewards` for that card over that cycle. Give the rule, progress and what remains, with the period. Say when a card tracks by statement cycle rather than calendar month.

Prefer `get_card_reward_structure` for rates, caps and minimum spend, and `get_card_rules` for detail and changes. Mention `critical_warnings` that affect the answer. Compare dated changes with today. Say a past change is already in effect and how far away a future one is, prioritizing changes due within three months.

## Best card

Call `rank_linked_cards_for_merchant` before answering. Expand common shorthand first, such as dtf (Din Tai Fung), ntuc (NTUC FairPrice), sbux (Starbucks) or gv (Golden Village); otherwise pass the user's words. Pass `mcc_code` when an earlier transaction showed the merchant's MCC. When the user says how they pay, pass `payment_method`: Apple Pay, Google Pay and tapping a card are `contactless`. For spending abroad, pass `is_overseas: true` on every call and note the foreign-currency fee, about 3% to 3.5% on most Singapore cards. A cashback rate below the fee loses money.

Act on the result:

- `needs_choice: "condition"` with ranked `cards`: answer for `condition_used` first, then say the best card can change with the purchase type and list up to four `available_conditions`. Call again with the `condition` the user picks.
- `needs_choice: "condition"` with no `cards`: the merchant is a payment platform with no default. List up to four `available_conditions` and ask which applies, unless the user already said what they are paying for; then call again with that `condition`.
- `needs_choice: "merchant"`: list the `candidates` with their categories as numbered options and ask which one. Do not guess.
- `category` match: answer for the category and name relevant `excluded_merchants`. Do not ask which merchant.
- `payment_channel` match: rank every card for that channel and say the merchant's category can still change the rate.
- `inferred` match: in two sentences, say the merchant is likely similar to `matched_name`, give its category, card and rate, and suggest checking the MCC on the first transaction.
- No match: say you do not have that merchant's category yet and that its MCC appears on the statement after a first transaction. Do not guess an MCC or suggest similar merchants.

Each card's `rates_by_payment_method` gives its rate online, contactless and in store. When the user did not say how they pay and a card's rate depends on the method, name the method that earns it, such as "4 mpd when you pay online, 0.4 mpd in store".

Open with "Based on your cards", since the ranking covers only linked cards. Give the card, rate, reward type (miles or cashback), the bonus and merchant categories behind it, any `reward_cap_warning`, and the runner-up. Keep the returned order.

To maximize rewards across categories, rank dining, groceries, fuel, transport, online and travel, and give one line per category with the card and rate.

For "where did I use the wrong card", get the period's rewards with `get_card_rewards`. For settled transactions of S$20 or more that earned the base rate, rank the merchant with its `mcc_code`. The gap is the rate difference times the spend. When the better card has a `reward_cap_warning`, the gap is the lesser of spend times its rate and the remaining cap, minus the actual reward. Report only gaps above 1 mpd or 50 miles. Give the five largest with why the better card wins, then the total, and offer the rest.

Card rewards and HeyMax affiliate rewards are separate. Do not combine them unless HeyMax confirms they stack; the `rewards` skill covers affiliate offers and tracked links.

## Loyalty program miles

`reward_per_dollar` and `rate_per_dollar` are miles per dollar at the bank's standard points-to-miles ratio. They are not native bank points, and they equal a program's miles only when the bank converts to that program at its standard ratio. HSBC and OCBC ratios differ by program. Never multiply these rates by a route rate.

For a program question, make one `rank_linked_cards_for_merchant` call for the spend in question, or for `online` when none is named, and one `get_transfer_routes_to_program` call with the program's canonical code. Keep at most the top three ranked miles cards whose bank has a route, matching the card's bank points to a route's `from_program`. Program miles per dollar are native points per dollar times that route's `rate`. For each kept card only, take native points per dollar from its `get_card_rules` README, or from a reward transaction's `native_reward` divided by its spend. Compare those cards by program miles per dollar, with each route's fee and processing time. If none transfer, name the banks that do. Cashback cards do not transfer. For balances, quotes or Max Miles routes, use the `plan-miles-funding` skill.

## Disclosure

End every card recommendation or reward analysis with this required line: "Actual rewards may vary based on bank terms and merchant classifications." Omit it for card terms, clarifying questions and refusals.
