# HeyMax MCP tool reference

This reference describes the 28 tools exposed to connected members on 29 October 2026. The MCP server's live `tools/list` response defines the current input schemas, output schemas, annotations, and availability. Clients should use that response when making calls. The descriptions here help members and connector reviewers understand what each tool reads or changes.

Connect to `https://heymax.ai/user-mcp/mcp` with per-member OAuth as described in the [connection guide](mcp.md). Connecting authorizes the public tools listed here for the signed-in member.

All calls use the signed-in member's HeyMax account. "Private" means the result can contain the member's account data, spending, identity status, or trip search. Do not publish or log those results. A "read" call does not change the member's saved account data, though a flight search may query an external provider. The two "write" calls are described separately below.

## Rewards and loyalty accounts

| Tool | Input | Result | Access | Private |
| --- | --- | --- | --- | --- |
| `get_miles_balance` | None | Current redeemable Max Miles balance. | Read | Yes |
| `get_recent_transactions` | Optional `limit` | Recent Max Miles earning and spending transactions. | Read | Yes |
| `get_redemption_history` | Optional `limit`, `status`, `redeem_type` | Past redemption records and their status. | Read | Yes |
| `list_active_redeem_options` | Optional `program_type` | Available Max Miles redemption options. It does not initiate a redemption. | Read | No |
| `get_loyalty_program_accounts` | None | Saved bank, reward, and airline programs with their recorded planning balances. It does not query those programs for a live balance. | Read | Yes |
| `update_loyalty_program_points_balance` | Required `program`, `points_balance_amount` | The program, absolute balance saved in HeyMax, and update time. | Write | Yes |

`update_loyalty_program_points_balance` replaces one recorded balance in private HeyMax planning data. It does not change a bank or airline balance. Call it only when the member explicitly asks to save a new, absolute current balance, and show which program and amount will be saved. It can overwrite a previous recorded value.

## Cards and spending

Linked-card spending tools are available to Singapore members. Card terms and tracking may vary by card.

| Tool | Input | Result | Access | Private |
| --- | --- | --- | --- | --- |
| `get_linked_cards` | None | Linked card names and basic card metadata. | Read | Yes |
| `get_card_reward_structure` | Required `card_short_name` | Earn tiers, exclusions, caps, minimum spend, warnings, and current cycle use when available. | Read | Yes |
| `get_card_rules` | Required `card_short_name`; optional `document` | One card terms or review document and the names of other available documents. | Read | No |
| `get_mcc_info` | Required `mcc_code` | Merchant category code meaning and spending category. | Read | No |
| `get_card_spend_transactions` | Optional `card_name`, `category`, `merchant_name`, `payment_channel`, `start_date`, `end_date` | Matching settled and pending card transactions with spend totals, without reward calculations. Defaults to the current calendar month. | Read | Yes |
| `get_card_rewards` | Optional `card_name`, `start_date`, `end_date` | Card reward totals, reward breakdowns, and bonus cap use. Defaults to the current calendar month. Pending spend counts toward caps but has no earned reward yet. | Read | Yes |
| `search_card_spend_transactions` | Optional `query`, `card_name`, `category`, `payment_channel`, `start_date`, `end_date` | Matching transactions with earned rewards and cap use. Defaults to the last 90 days. | Read | Yes |
| `rank_linked_cards_for_merchant` | Required `merchant_name`; optional `mcc_code`, `payment_method`, `condition`, `is_overseas` | Linked cards ranked by applicable earn rate, or candidates when the merchant or purchase type needs a choice. | Read | Yes |

These tools read HeyMax's card catalogue and the member's linked-card data. They do not make a payment or change a card. Some cards have no tracked spending; the response may say so. A pending transaction is distinct from a settled transaction, and estimated reward rates can depend on category, payment method, cap, and current terms.

## Merchants and deals

| Tool | Input | Result | Access | Private |
| --- | --- | --- | --- | --- |
| `find_merchants` | Optional `merchant_name`, `categories`, `limit` | Matching HeyMax merchants, earning rates, and available earning paths. | Read | Yes |
| `get_upcoming_deals` | Optional `limit` | Upcoming dated HeyMax deals in the member's market. | Read | Yes |
| `get_merchant_tracking_link` | Required `merchant_name` | A HeyMax tracked merchant URL and any earning instructions. | Read | Yes |
| `render_affiliate_offers` | Required `offers`; optional `selected_rate_names_by_merchant` | An offer-card display built from offers already returned in the conversation. | Read | Yes |

These tools do not open a merchant site or make a purchase. A tracking link can credit HeyMax or the member for a qualifying purchase. Identify it as a tracked link when presenting it. The member chooses whether to open it.

## Flights and transfer planning

Flight prices and availability are observations, not reservations. A live search can contact external flight data providers; no tool below holds, books, or buys a seat.

| Tool | Input | Result | Access | Private |
| --- | --- | --- | --- | --- |
| `search_cached_award_flights` | Required `origin`, `destination`, `start_date`, `end_date`; optional `num_adults`, `num_children`, `num_infants`, `cabin_class`, `num_stops`, `programs` | Recently observed award itineraries and prices, with observation time and source where available. | Read | Yes |
| `search_cash_flights` | Required `origin`, `destination`, `departure_date`; optional `return_date`, `num_adults`, `num_children`, `num_infants`, `cabin_class`, `num_stops`, `currency` | Current cash fare observations in the shared flight result format. It queries external providers. | Read | Yes |
| `start_award_flight_search` | Required `origin`, `destination`, `departure_date`; optional `return_date`, `num_adults`, `num_children`, `num_infants`, `cabin_class`, `num_stops`, `programs` | Private search `task_id` and initial `status`. It starts an external award search. | Write | Yes |
| `get_award_flight_search_results` | Required `task_id` | Current task status and award results when the search completes. | Read | Yes |
| `render_flight_comparison` | Required `flight_data`; optional `transfer_options_by_program` | A comparison display built from already returned itineraries. | Read | Yes |
| `get_transfer_routes_to_program` | Required `target_program` | Transfer routes into that program with rates, fees, and timing. | Read | No |
| `quote_transfers_to_program` | Required `target_program`; optional hypothetical `balances` | Transfer quotes using saved balances and Max Miles, or supplied hypothetical balances. Supplied balances are not saved. | Read | Yes |
| `get_transfer_details` | Required `partner_name` | Partner requirements, rate, limits, processing time, and saved membership details when available. | Read | Yes |

`start_award_flight_search` creates a private task and queries outside data sources. Poll `get_award_flight_search_results` with its `task_id` until the task completes or fails. A static award-chart price or an older cached observation does not prove a seat is bookable. Transfer routes and quotes are plans only; members complete transfers in the HeyMax app.

## HeyMax First and identity

These tools are available only to Singapore members.

| Tool | Input | Result | Access | Private |
| --- | --- | --- | --- | --- |
| `get_heymax_first_status` | None | Membership, billing state, redeemable balance, unused draw capacity, and reclaimable amount. | Read | Yes |
| `get_identity_verification` | None | Verification status and, when needed, a HeyMax verification URL for the member. | Read | Yes |

Unused draw capacity is not redeemable Max Miles. The assistant should give a verification URL to the member to open themselves. These tools do not start verification or draw miles. Members complete draws in the HeyMax app.

## Errors, limits, and support

The server validates inputs and authentication before returning a result. An invalid input or unavailable feature returns a tool error or an action-needed message; do not treat it as a successful empty result. An expired or revoked connection requires the member to reconnect. A flight search can finish without available results or fail because an external provider is unavailable. Report that state without inventing prices or availability.

Each member can call each tool up to 30 times per minute and 500 times per day. A limit error means the client should wait before trying again. Do not repeatedly retry a failed write. The live MCP schema and server response give the precise error and result fields for each call.

For help, visit the [HeyMax help centre](https://help.heymax.ai/en/) or email support@heymax.ai.
