---
name: search-flights
description: Use the authenticated HeyMax User MCP to search cash fares and award flights, compare them, and recommend one. Use when a user asks what a flight costs in cash or miles, wants to redeem points for a trip, compares paying cash with using miles, explores flexible dates or airports, or looks for premium-cabin seats.
---

# Search Flights

## Get the trip

Use every detail already in the conversation. If something that changes the search is missing, ask once, in one message: route or airports, dates or flexibility, travellers, cabin, and stops. Don't ask again. Search with defaults for anything unanswered (economy, any stops, the whole date window) and state them in the answer.

When the user doesn't say how they pay, search cash and miles.

## Cash fares

Call `search_cash_flights` with uppercase IATA codes, the full party, `cabin_class`, and the member's `currency`, such as `SGD`. Pass `return_date` for a return trip.

## Award flights

1. Call `search_cached_award_flights` for the date window with the full party and cabin.
2. Refresh with `start_award_flight_search`, one direction per task:
   - If the cache has matches, refresh the best three dates.
   - If not, search up to 11 dates per direction: the user's dates, or for a longer window the 11 best, such as the cheapest cash dates.
   - Skip live search for dates beyond the airlines' booking window, about 355 days ahead.
3. Show cached prices while live searches run. Poll `get_award_flight_search_results` with `retry_after_seconds` until each task ends.
4. Live search doesn't cover KrisFlyer (SQ). Say when its prices come only from the cache.

Award search covers Max Miles partners only: Cathay Pacific, Flying Blue, KrisFlyer, Qatar Airways, SAS and Turkish Airlines. Leave `programs` empty unless the user names one of them. If they name another program, say HeyMax can't search it.

Pass `num_stops` only when the user limits stops. On every flight tool it is the maximum number of stops, and 0 means nonstop.

Say "no award seats" only after live search found none. Name the dates you checked and suggest one change.

## Compare

- Put every option on one scale. A cash fare costs the fare. An award costs Max Miles needed × the HeyMax First `access_fee_cents_per_mile` from `get_heymax_first_status` ÷ 100, plus taxes. Without that rate, use (cash fare − award taxes) ÷ miles. Convert to one currency.
- Show party totals, and say whether each price covers the return or one way only.
- Show business with miles only when it costs at most 1.5× the cheapest economy option, unless the user asked for business.
- Call `lookup_award_chart` when available. Treat `benchmark_miles` as an internal anchor and don't quote it.
- Prices marked `price_basis: "per_passenger"` are per person.

## Answer

1. Give a verdict: the option to book and why, and one fallback.
2. Call `render_flight_comparison` once with up to three options, best first:
   - Set `title`, `passengers`, a `label` on the best option, and `trip_coverage` on each option.
   - Add `cash_equivalent` on awards.
   - Put the price to show first in `pricing_options`.
3. Write the full answer as text too.

Don't plan funding here. When the member picks an award, follow `plan-miles-funding`. For a cash fare, the member books on the airline or travel site; `card-maximiser` finds the best card. Prices are observations, not holds.
