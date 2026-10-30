---
name: search-award-flights
description: Collect a concise trip brief, use the authenticated HeyMax User MCP to search cached award prices while an exact live search completes, and compare up to three itineraries using the user's recorded bank and airline balances, verified transfer routes, current Max Miles, and clearly separated HeyMax First capacity. Use when a user asks to find award flights, redeem points or miles for a trip, compare award and cash fares, explore flexible dates or airports, or look for premium-cabin availability.
---

# Search Award Flights

Treat award travel as scarce-inventory search. Show useful cached observations immediately, refresh the strongest exact dates with a live search, and rank no more than three distinct itineraries for value, comfort, convenience, and realistic funding.

## Non-Negotiable Rules

- A live search is still a live search when cached prices are shown first. Keep those cached leads visible while the live task completes, then replace or augment them with the completed result.
- A completed live search is a live-search result, not a hold or booking guarantee. If the result projection does not expose price provenance, do not upgrade every returned price to a freshly observed availability claim.
- Use balances to assess affordability, but never expose membership numbers, account email addresses, user IDs, or other account identifiers.
- Keep current Max Miles separate from additional HeyMax First capacity. Never add the components of a reported balance together again.
- Never invent a transfer relationship, ratio, bonus, fee, passenger-price basis, or currency conversion.

## Workflow

1. Inspect the current MCP definitions for the relevant tools and treat their discovered schemas as canonical. Use the cache search, live award start/poll, loyalty-account, HeyMax First, transfer quote and route, active redemption-option, cash-search, and flight-comparison display tools only when available.
2. If HeyMax needs authorization, ask the user to connect or reconnect HeyMax in the browser. Never request passwords, access tokens, one-time codes, or loyalty account numbers.
3. Extract every usable trip detail from the conversation. Ask one compact grouped question only for missing information that materially changes the search.
4. Privately load the user's recorded airline balances and Max Miles position. Ask only for important balances that HeyMax cannot retrieve, such as bank or card points. Persist a supplied airline balance only when the user explicitly asks to save it.
5. Search cached award observations across the approved date window and airports so the user gets useful leads immediately.
6. Shortlist the strongest exact route/date/cabin combinations and start the live award search for them. Poll each task according to `retry_after_seconds`; do not busy-poll or create duplicate tasks.
7. When live results complete, discard non-observed estimates when the response exposes their provenance, reapply the user's hard constraints, and reassess the shortlist. If provenance is absent, label the completed result conservatively. If live refresh fails or times out, retain cached leads and label the failure clearly.
8. Quote verified transfer paths and calculate affordability without double counting. Optionally obtain a comparable cash reference when it materially improves the value comparison; use the `card-maximiser` skill when card earning, caps, or overseas rates on that cash fare matter.
9. Rank independently rather than trusting tool response order. Present at most three distinct options, display them with the comparison tool when available, and finish with a verification step.

## Tell Us About Your Trip

Use this short intake when details are missing:

- **Route:** origin and destination, plus acceptable nearby airports
- **Dates:** outbound and return windows, or one-way, plus real flexibility
- **Travelers:** adults, children, and infants
- **Cabin:** preferred cabin and acceptable fallback
- **Unrecorded points:** approximate bank, card, or airline balances not already recorded in HeyMax; account details are unnecessary
- **Preferences:** maximum stops and whether value, comfort, or convenience matters most

Do not make beginners translate their request into loyalty-program jargon. Do not ask them to repeat balances already returned by HeyMax. Never assume bank points transfer to a program unless a tool result or the user establishes that relationship.

## Load and Interpret Balances

Use `get_loyalty_program_accounts` for personalized searches when it is available. Retain only the program, recorded points balance, balance currency, expiry schedule, and `updated_at` freshness needed for the decision.

- Treat these as user-recorded balances, not live airline-login balances.
- Say when a decisive balance is missing or old. Do not fabricate a freshness threshold; show its recorded date when it affects the recommendation. If its age could change the funding label, ask the user to confirm it. An old timestamp alone lowers confidence but does not force **Unknown** when the user has confirmed the amount.
- Give useful priority to points expiring soon, but treat expiry buckets as components of the recorded balance, never extra points. Do not let expiry override a hard trip preference or a materially worse itinerary without explaining the trade-off.
- Do not restrict broad discovery to linked programs. Search broadly, then use balances and confirmed transfer paths to rank.

Use `get_heymax_first_status` when available. Interpret it exactly as follows:

- `miles.redeemable_miles_balance` is the user's **current Max Miles** available for redemption. It already includes earned and pool-issued miles.
- `miles.earned_miles_balance` and `miles.pool_issued_miles_balance` are components of that current balance. Never add either to `redeemable_miles_balance`.
- `miles.additional_miles_available_to_draw` is unused HeyMax First capacity. It is potential funding, not a current balance.
- `miles.miles_available_to_reclaim` is not additional spendable funding.
- Drawing First miles is Singapore-only, comes in 10,000-mile blocks, can require identity verification and payment, incurs the returned tier-policy access fee, and increases pool liability. It does not itself transfer miles to an airline.

If HeyMax First status is unavailable or inapplicable, use `get_miles_balance` as the current Max Miles fallback. Do not call both and add their balances. A failed First lookup must not block an otherwise valid award search.

The skill needs only these decision-relevant HeyMax First mechanics. Do not turn the answer into a general membership explainer unless the user asks.

### Save a confirmed balance for future searches

- Use a balance supplied in the conversation for the current search even when the user does not want it saved. Do not block the search on an optional write.
- Call `update_loyalty_program_points_balance` only when the user explicitly asks to save, update, or remember an absolute current balance for one unambiguous supported program. If that intent is absent, optionally offer to save it after presenting the search results.
- If the current recorded value is not already known, call `get_loyalty_program_accounts` before overwriting it. An explicit command containing the program and absolute amount is sufficient confirmation; otherwise show the program and proposed amount and ask once.
- Treat the tool as an absolute setter. For a delta, expected transfer, expected booking deduction, or inferred balance, ask the user to provide or confirm the resulting current balance instead of writing.
- After success, say that HeyMax saved the recorded planning balance. Do not imply the airline account changed, and do not expose membership numbers, emails, or internal account IDs returned by the tool.
- Never use this setter for Max Miles or HeyMax First state.

## Search the Cache, Then Refresh Live

Build the smallest useful calls from the discovered tool schemas.

- Translate named airports to uppercase IATA codes. Confirm ambiguous cities or airport-region expansions.
- Use the same start and end date for an exact cache search. Keep flexible windows within the tool's limit.
- Pass nearby origins or destinations together only when the user accepts them.
- Search for the full party first and require at least one traveler; never send an all-zero passenger request.
- Pass the cabin explicitly to every cached and live call because defaults can differ.
- Omit program restrictions for broad discovery. Restrict only when the user requests an issuing program or a focused refresh needs it.
- Set the maximum stops only when the user states one. Translate stop semantics separately for award and cash tools rather than reusing an integer blindly.
- For round trips and open jaws, search each direction independently. Start a separate live task for each one-way leg; do not rely on a live tool's `return_date` unless its discovered contract confirms round-trip award behavior.

After cached discovery:

1. Choose up to three promising exact route/date/cabin cohorts, not one task for every returned pricing option. If the cache is empty, still start a live search for the exact requested date; for a flexible window, use up to three disclosed dates closest to the user's stated preference rather than treating an empty cache as the end of the search.
2. Call `start_award_flight_search` for each selected one-way cohort and retain only the task IDs returned in this workflow.
3. While a task is pending or fetching, continue to present the cached prices as preliminary leads. Active polls may omit partial itineraries even when a result count is nonzero.
4. Poll with `get_award_flight_search_results` using the returned retry hint. Stop on completed, failed, timeout, skipped, maximum-retry, or an unknown terminal state; do not poll forever. Live search does not cover every program the cache covers; KrisFlyer (`SQ`) and EVA Air (`BR`) are currently cache-only. When the live search cannot cover a requested program, say so plainly, keep its cached observations as cached leads, and do not retry. An empty cache for a cache-only program means no recent observation, not a live result.
5. On completion, locally enforce the requested programs and hard constraints. If a source field is exposed, exclude `static_miles_pricing` from observed-availability claims. If source is omitted, do not pretend the client can distinguish observed and estimated rows.
6. Describe a completed result with explicit recent non-static provenance as **Fresh search observation — verify with the program**. When provenance is absent, use **Live search completed — price provenance unavailable; verify with the program**. Describe retained cache-only results as **Cached lead — live refresh incomplete**.

Treat outbound and return results as independently observed one-way leads. A paired option is a planning combination, not a single priced or bookable round-trip itinerary. Show each leg's program and price separately.

## Calculate Funding Without Double Counting

Calculate only when the mileage requirement's passenger basis is known or can be clearly labelled as an estimate. Use the price for the full party when explicitly supplied; otherwise show the quoted amount and mark any multiplication as an estimate because child and infant prices can differ.

For a multi-leg trip, first sum the selected legs' requirements by issuing program. Bank, airline, and Max Miles balances are shared across the whole trip; never spend the same balance independently on both directions. Then, for each issuing program:

1. Subtract the user's recorded balance in that same program from the required miles. Never treat miles in an unrelated airline as interchangeable.
2. If a shortfall remains, call `quote_transfers_to_program` for the issuing program with `balances` omitted. It quotes saved bank and reward balances plus the current Max Miles wallet with route rates, minimums, and blocks applied. Follow the `plan-miles-funding` skill to verify Max Miles first hops with `list_active_redeem_options` and `get_transfer_details`, and to apply partner restrictions the quote does not.
3. When two issuing programs draw on the same sources, allocate each source once and re-quote the remainder with explicit `balances`.
4. The quote's `heymax` source already is the Max Miles wallet, converted at the route rate; do not add the wallet again. Only when the quote tool is unavailable, use a `transfer_options_by_program` route whose source explicitly identifies HeyMax or Max Miles and convert current Max Miles from `redeemable_miles_balance`, or from the fallback balance tool, through that route's rate, minimum, and block. If that route's minimum or block is missing and could change the result, use **Unknown** rather than claiming exact affordability.
5. Never add earned or pool-issued components to the wallet again.
6. If a shortfall remains, find the smallest whole 10,000-mile HeyMax First draw that closes it within `additional_miles_available_to_draw`. Explicit `balances` replace the whole quote input, so re-quote all sources together with `heymax` set to the wallet plus that draw, and use the difference in usable final credit. Do not silently draw it and do not imply it is already transferable.
7. When the tier policy returns `access_fee_cents_per_mile`, estimate the access fee in SGD (HeyMax First is Singapore-only) as `draw miles × access_fee_cents_per_mile ÷ 100` and label it an estimate. If the needed draw exceeds available capacity, do not price the impossible draw as if it were feasible; optionally show the fee for drawing the available maximum and the shortfall that would remain.

Assign exactly one funding label:

- **Ready now:** the recorded program balance and quoted transfer credits, including the Max Miles wallet, cover the known whole-trip requirement through a confirmed transfer path with enough detail to apply its constraints.
- **Possible with HeyMax First:** current resources do not cover it, but the minimum additional draw fits within the returned unused First capacity. Show the draw needed, estimated fee when calculable, liability implication, and returned transfer constraints.
- **Shortfall:** the known requirement exceeds current resources plus available First capacity, violates transfer limits, or has no usable route.
- **Unknown:** a decisive balance is missing or unconfirmed, or the passenger-price basis, transfer match, rate, or required constraint is ambiguous enough to prevent the calculation.

Do not automatically rank the fewest miles first. Prefer options the user can fund now, then those possible with an explicit First draw, while still weighing comfort and convenience. Keep a materially better but unfunded option visible when it helps the user understand the trade-off.

## Assess and Rank the Itineraries

Discard results that violate hard constraints. Group duplicates by `flight_id`, or by segment sequence when no ID exists. Treat different loyalty-program prices for the same flights as funding choices, not separate itineraries.

For each candidate:

- Distinguish the operating airline in `segments` from the issuing loyalty program in the miles price.
- Inspect cabin on every segment and call out mixed cabins.
- Treat known `seats_left` as an observed itinerary-level signal, not price-specific proof. If unknown, say so.
- Use each pricing option's `last_seen` value for freshness. Do not invent a result-level refresh time.
- Prefer miles-pricing options for an award search. Show cash pricing only as secondary context and never infer award availability from it.
- Treat a zero tax amount as **zero reported by the source**, not proof that no taxes or fees apply.
- Do not combine taxes in different currencies or invent currency conversions.

Rank using:

- **Affordability:** funding label, balance freshness, transfer minimums and timing, First draw and access-fee burden, and useful expiring balances
- **Value:** lower returned miles and fees and, when comparable, a defensible award-versus-cash reference
- **Comfort:** requested cabin on important segments and fewer mixed-cabin compromises
- **Convenience:** fewer stops, shorter duration, and fit with timing or airport preferences
- **Confidence:** fresher non-static observations and known seat counts

Apply the user's stated priority. Otherwise balance affordability, value, comfort, and convenience. Return fewer than three when only one or two strong distinct options exist; never pad the list.

## Present the Result

Use this compact structure:

### Search brief

Summarize the routes, date windows, party, cabin, flexibility, and balance sources actually considered. Mention the recorded date of a decisive loyalty balance without exposing account identifiers.

### Best options

Show up to three options with:

- A label such as **Best overall**, **Best ready-now value**, or **Best comfort**
- Date, flight numbers, route, stops, and duration
- Operating airline and issuing loyalty program
- Cabin on every segment, including mixed-cabin warnings
- Seats observed, or **unknown**
- Returned miles and cash fees with passenger basis, available provenance, and last-observed time
- One of the four funding labels, followed by a compact funding breakdown: recorded program miles, transfer credits by source with the Max Miles spent for the HeyMax route, and minimum First draw if any
- Estimated First access fee, its refund when the draw is reclaimed with eligible earned miles, and any returned transfer minimum, block, and transfer fee
- **Fresh search observation — verify with the program**, **Live search completed — price provenance unavailable; verify with the program**, or **Cached lead — live refresh incomplete**
- One sentence explaining the ranking

After settling the final order, call `render_flight_comparison` once with the ranked itineraries and the relevant transfer routes, preserving the search-result flight and price fields. Keep the written answer complete in case the client does not display the comparison.

### Recommendation

Recommend one option and explain its main funding and itinerary trade-off in plain language. If no option dominates, say which traveler preference should decide.

### Verify before transferring

Tell the user to reproduce the exact date, flights, cabin, passenger count, miles, and fees with the issuing loyalty program. Warn that point transfers are commonly irreversible or delayed and availability can disappear. A First draw and a partner transfer are separate actions. The member completes Max Miles transfers and First draws in the HeyMax app, and bank or yuu legs in that program's own channel, using a returned `transfer_link` when present.

## Recover from No Matches or Failed Refresh

Say **No matching result was found for these parameters**, not **No award seats exist**.

An empty cache does not skip live search. First run the exact-date live request, or the disclosed representative dates for an approved flexible window. If no live result remains and the user allowed flexibility, retry by changing one dimension in this order:

1. Widen the dates within the agreed window.
2. Add accepted nearby airports.
3. Allow one additional stop.
4. Add an accepted fallback cabin.
5. Remove an explicit program restriction.

Explain exactly what changed. A one-passenger diagnostic may identify party size as the constraint, but never present that result as suitable for the full party. If live refresh fails, preserve the cached shortlist with the correct warning. If no useful lead remains, give the precise next search parameters instead of inventing an itinerary.

## Guardrails

- Never label an observation as verified, held, guaranteed, or bookable.
- Never claim coverage of a fixed number of programs; describe only programs represented in the results.
- Never recommend transferring points before live full-party verification.
- Never draw First miles, transfer points, or ask for a one-time code, even when the session exposes those tools. Direct the member to the HeyMax app for Max Miles transfers and draws, and to the source program for bank or yuu legs.
- Do not log in to an airline, book, purchase, or create alerts. HeyMax tools do not book awards.
- Do not update a stored loyalty balance from an inference, completed search, expected transfer, or expected booking deduction.
- Use `update_loyalty_program_points_balance` only for an explicitly requested, user-confirmed absolute current balance; do not call it automatically during search.
- Surface authentication, validation, rate-limit, provider, and server errors instead of guessing around them.
