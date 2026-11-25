---
name: search-flights
description: Use the authenticated HeyMax User MCP to search cash fares and award flights, compare them, and check what the member can fund from airline miles, Max Miles and HeyMax First. Use when a user asks what a flight costs in cash or miles, wants to redeem points for a trip, compares paying cash with using miles, explores flexible dates or airports, or looks for premium-cabin seats.
---

# Search Flights

## Get the trip

Use every detail already in the conversation. Ask once, in one message, only for what changes the search: route, dates or flexibility, number of travellers, and cabin.

Search what the user asked for: cash, miles, or both. When they ask what a flight costs without saying how they pay, search both.

## Cash fares

Call `search_cash_flights` with uppercase IATA codes, the full party and the cabin. It defaults to business class and USD, so pass `cabin_class` and the member's `currency`, such as `SGD`, explicitly. It searches a return trip in one call when you pass `return_date`. Its `num_stops` is a filter code: 0 any, 1 nonstop, 2 at most one stop.

## Award flights

1. Call `search_cached_award_flights` for the date window, with the full party and the cabin. Omit `programs` unless the user names one. Its `num_stops` is a maximum, so 0 means nonstop.
2. Pick up to three promising one-way route/date/cabin options, or the exact requested date if the cache is empty. Call `start_award_flight_search` for each. Search each direction of a return trip separately.
3. Show the cached prices while the live searches run. Poll `get_award_flight_search_results` with the returned `retry_after_seconds` until each task ends.
4. Live search does not cover KrisFlyer (SQ) or EVA Air (BR). For those, use cached results only.

## Compare

- When available, call `lookup_award_chart` for the route and cabin. Use `benchmark_miles` only as an internal anchor: at or below it is a good price; above it needs a stronger reason. Do not quote the benchmark.
- Prices marked `price_basis: "per_passenger"` are per person. Multiply by the passengers for the party total.
- The operating airline can differ from the program that issues the award. Funding depends on the issuing program; follow `plan-miles-funding` for each award you recommend. Do not spend the same balance on both directions.
- Flag mixed cabins, unknown taxes and the observation time in `last_seen`.

### Put cash and miles on one scale

Compare every option by its cash-equivalent cost per person:

- **Cash fare:** the fare.
- **Award:** Max Miles needed × the member's HeyMax First `access_fee_cents_per_mile` ÷ 100, plus taxes. Take the Max Miles needed from the funding plan's transfer rate. Use this rate even for miles the member already holds, so options compare fairly however they are funded.

Use the value the user gives per mile instead, when they give one. Without a HeyMax First rate, compare by value per mile: (cash fare − award taxes) ÷ miles. Put amounts in one currency. When the currencies differ, convert at an approximate rate and state it.

Rank by this cost, then state the member's actual out-of-pocket under the funding plan: taxes, plus any draw access fee, which is refunded when they reclaim it with eligible earned miles. Example: "Award ≈ S$2,950 equivalent (85,000 Max Miles at 3¢ + S$400 taxes) vs S$6,794 cash; with your current miles you pay only the S$400 taxes."
- For a cash fare the member may book, use `card-maximiser` for the best card and `rewards` to check whether the airline or a travel site earns Max Miles.

## Answer

Recommend up to three distinct options, best first, mixing cash and award when both were searched. For each, give the date, flights, stops, cabin, and the price per person: cash with currency, or miles plus taxes with the issuing program, plus the cash-equivalent cost when cash and award are both shown. For an award, say whether the member can fund it now, with a HeyMax First draw, or not. Then call `render_flight_comparison` once with the ranked results and keep the written answer complete.

Say "no matching result for these parameters", not "no seats exist", and suggest one change, such as nearby dates. Prices are observations, not holds. Tell the user to confirm an award seat with the airline before transferring points. The member books on the airline or travel site.
