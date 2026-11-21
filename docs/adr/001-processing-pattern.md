
# ADR 001: Choose Lambda Architecture for Greenleaf Foods

## Status
Accepted

Date: 2026-11-21

## Context

Shipper: Greenleaf Foods (SHP04).

Greenleaf imports and exports packaged food with limited shelf life. Warehouse teams need to react quickly when vessels are delayed.

The platform has three targets:
- **Freshness:** Shipment updates must be available within 60 seconds.
- **Reliability:** At least 65% of shipments must arrive no more than 24 hours after their planned arrival on every lane.
- **Budget:** Maximum $30 per month.

The October 2026 BigQuery analysis joined bookings and schedules and measured 114 shipments across four ocean lanes.

Results:
- CNYTN → USLAX (Transpacific): 45.8% on time, 19.2 percentage points below target; average delay 47.3 hours.
- CNSHA → NLRTM (Asia-North Europe): 54.8% on time, 10.2 percentage points below target; average delay 33.5 hours.
- NLRTM → USHOU (Transatlantic): 56.7% on time, 8.3 percentage points below target; average delay 48.4 hours.
- GBFXT → CNSHA (North Europe-Asia): 79.3% on time, meeting the target by 14.3 percentage points; average delay 38.5 hours.

Three of the four lanes miss the reliability target. The platform must provide fresh shipment information so warehouse teams can identify delays and re-plan.

The data sources include live shipment tracking events and scheduled carrier files. These have different processing requirements.

## Options considered

1. **Lambda architecture:** Use a live streaming path for shipment events and a separate batch path for carrier files and historical reporting. This supports different source types and freshness requirements, but requires two processing paths.

2. **Kappa architecture:** Process all data through one streaming pipeline and replay historical data when needed. This reduces duplicated processing logic, but requires carrier files to be converted into events and a reliable event log to be maintained.

## Decision

We chose **Lambda architecture** because Greenleaf receives both live shipment events and scheduled carrier files. The live path supports the 60-second freshness target, while the batch path supports historical carrier analysis and executive reporting.

The proposed live path uses Pub/Sub, Dataflow and BigQuery. The batch path uses Cloud Storage and BigQuery load jobs and SQL transformations.

## Consequences

- **Cost:** The budget is $30 per month. Actual costs for Pub/Sub, Dataflow, Cloud Storage and BigQuery must be checked using the Google Cloud pricing calculator. The design must be simplified or adjusted if the estimate exceeds the budget. No claim of budget compliance is made until this check is completed.
- **Freshness:** The live path is designed to make shipment updates available within 60 seconds. End-to-end latency must be tested. The batch path is scheduled and does not guarantee 60-second freshness.
- **Complexity:** Two processing paths must be built and monitored. Event-time processing, late events, duplicate events and failed notifications need attention.
- **What gets harder:** Some business logic and monitoring may be duplicated across the live and batch paths. Changes must be kept consistent. The system can detect and communicate delays, but cannot guarantee vessel arrival performance.
