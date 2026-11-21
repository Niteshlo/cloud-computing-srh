# Metrics: delay per lane for SHP04 (Greenleaf Foods)

Source: `freight.bookings` joined to `freight.schedules` on `schedule_id`
(query: `sessions/03-batch-streams-and-architecture-patterns/starter/delay_per_lane.sql`, shipper `SHP04`).
Data: October 2026 batch. Bookings whose sailing departs after October are not in `schedules` and are not counted.
"On time" means actual arrival no more than 24 hours after planned arrival.

Greenleaf Foods reliability target: **65% on time on every lane.**

| Lane | Trade lane | Shipments | Avg delay (h) | On-time % | Target | Gap | Result |
|---|---|---|---|---|---|---|---|
| CNYTN → USLAX | Transpacific | 24 | 47.3 | 45.8% | 65% | -19.2 pts | MISSES |
| CNSHA → NLRTM | Asia-North Europe | 31 | 33.5 | 54.8% | 65% | -10.2 pts | MISSES |
| NLRTM → USHOU | Transatlantic | 30 | 48.4 | 56.7% | 65% | -8.3 pts | MISSES |
| GBFXT → CNSHA | North Europe-Asia | 29 | 38.5 | 79.3% | 65% | +14.3 pts | MEETS |

## Summary

- 3 of 4 lanes miss the 65% target. Only GBFXT → CNSHA meets it.
- Transpacific is the weakest lane (45.8% on time, 47.3 h average delay).
- Sample sizes are small (24 to 31 shipments per lane), so the percentages move a lot with a few bookings.

## Carriers on the lanes that miss the target


Carrier names: ATL = Atlas Ocean, BLC = Blue Crest Shipping, HRZ = Horizon Container Line,
KST = Kestrel Maritime, NWL = Northwind Lines, ORS = Orion Sea Freight.

### CNYTN → USLAX (Transpacific): 45.8% on time, MISSES

| Carrier | Shipments | Avg delay (h) | On-time % |
|---|---|---|---|
| ATL Atlas Ocean | 5 | 105.6 | 0.0% |
| NWL Northwind Lines | 4 | 42.0 | 25.0% |
| KST Kestrel Maritime | 4 | 37.5 | 25.0% |
| BLC Blue Crest Shipping | 3 | 64.0 | 33.3% |
| ORS Orion Sea Freight | 5 | 9.6 | 100.0% |
| HRZ Horizon Container Line | 3 | 16.0 | 100.0% |

### CNSHA → NLRTM (Asia-North Europe): 54.8% on time, MISSES

| Carrier | Shipments | Avg delay (h) | On-time % |
|---|---|---|---|
| NWL Northwind Lines | 9 | 64.0 | 0.0% |
| ATL Atlas Ocean | 6 | 48.0 | 33.3% |
| BLC Blue Crest Shipping | 4 | 12.0 | 75.0% |
| HRZ Horizon Container Line | 7 | 15.4 | 100.0% |
| ORS Orion Sea Freight | 5 | 3.6 | 100.0% |

### NLRTM → USHOU (Transatlantic): 56.7% on time, MISSES

| Carrier | Shipments | Avg delay (h) | On-time % |
|---|---|---|---|
| ORS Orion Sea Freight | 9 | 68.0 | 33.3% |
| KST Kestrel Maritime | 3 | 48.0 | 33.3% |
| ATL Atlas Ocean | 4 | 69.0 | 50.0% |
| NWL Northwind Lines | 7 | 27.4 | 71.4% |
| HRZ Horizon Container Line | 4 | 39.0 | 75.0% |
| BLC Blue Crest Shipping | 3 | 24.0 | 100.0% |

### All four lanes combined, per carrier (totals computed from the per-lane results)

| Carrier | Shipments | On time | On-time % |
|---|---|---|---|
| ATL Atlas Ocean | 21 | 5 | 23.8% |
| NWL Northwind Lines | 27 | 13 | 48.1% |
| KST Kestrel Maritime | 11 | 6 | 54.5% |
| BLC Blue Crest Shipping | 14 | 10 | 71.4% |
| ORS Orion Sea Freight | 23 | 17 | 73.9% |
| HRZ Horizon Container Line | 18 | 17 | 94.4% |

## Findings

- Atlas Ocean and Northwind Lines have 48 of Greenleaf's 114 bookings (42%) but only 18 on-time arrivals (37.5%). They are the main reason the target is missed.
- Horizon (94.4%) and Orion (73.9%) are the strongest carriers overall. Orion is weak on Transatlantic (3 of 9 on time), so carrier choice has to be per lane, not global.
- The other carriers (BLC, KST) are mixed or small samples.
- Samples are small (3 to 9 shipments per carrier per lane), so these are indications, not proof.
- Implication for the design: Greenleaf's lateness comes from specific sailings running late, and its warehouse teams must re-plan the moment a vessel slips. This supports detecting delays quickly (the 60-second freshness target) rather than only reporting them in a daily batch.
