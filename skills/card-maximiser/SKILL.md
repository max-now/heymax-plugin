---
name: card-maximiser
description: Use the authenticated HeyMax User MCP to analyse a Singapore member's linked-card spending, rewards earned, bonus caps and minimum spend, pick the best linked card for a merchant, category, payment method or overseas spend, explain card terms, exclusions and MCCs, find where the wrong card was used, and compare which cards earn the most miles in a loyalty program.
---

# Card Maximiser

Take every spend, reward, rate, cap and MCC from a tool result. Never infer an MCC from general knowledge.

## Tools

- Linked cards: `get_linked_cards`.
- Spend totals and transactions: `get_card_spend_transactions`.
- Rewards and cap use: `get_card_rewards`. Search transactions with rewards: `search_card_spend_transactions`.
- Rates, caps, minimum spend and current cycle: `get_card_reward_structure`. Full terms and changes: `get_card_rules`.
- Best card: `rank_linked_cards_for_merchant`. MCC meaning: `get_mcc_info`.

`action_required: "unsupported_region"` means card analysis covers Singapore members only.

## Spend and rewards

- Resolve periods against today's Singapore date and pass both `start_date` and `end_date`. "Last month" is the previous calendar month, and "last N months" means the N most recent complete months. A follow-up keeps the earlier period.
- Use the totals the tools return; they already apply caps, minimum spend and exclusions. Do not add up transactions yourself.
- Split settled and pending spend. Pending spend has no reward yet.
- Lead with the period total, then one line per card. Mention a cap only when it is hit or close.
- Name cards by the returned `card_short_name`. UOB PV and UOB PRVI are different cards.
- If every transaction on a tracked card shows no reward, say reward calculation is not supported for that card yet.

## Best card

Call `rank_linked_cards_for_merchant` before answering. Expand shorthand such as dtf (Din Tai Fung) or ntuc (NTUC FairPrice). Apple Pay, Google Pay and tapping a card are `contactless`. For overseas spend, pass `is_overseas: true` and note the 3% to 3.5% foreign-currency fee.

- `needs_choice: "condition"`: answer for `condition_used` if cards are ranked, list up to four `available_conditions`, and call again with the user's pick.
- `needs_choice: "merchant"`: list the `candidates` and ask which one.
- `inferred` match: say the merchant looks similar to `matched_name` and suggest checking the MCC on the first transaction.
- No match: say you do not know its category yet. Do not guess.

Open with "Based on your cards". Give the card, rate, why it earns that rate, any `reward_cap_warning`, and the runner-up. When a rate depends on how they pay, say which method earns it.

For "where did I use the wrong card", rank the merchants behind settled base-rate transactions of S$20 or more and report the five largest gaps.

## Miles into a loyalty program

`rate_per_dollar` assumes the bank's standard miles ratio. Do not multiply it by a route rate. For a program comparison, call `get_transfer_routes_to_program`, keep the top three ranked cards whose bank has a route, and compute program miles per dollar as native points per dollar times the route `rate`. Cashback cards do not transfer.

End every card recommendation or reward analysis with: "Actual rewards may vary based on bank terms and merchant classifications."
