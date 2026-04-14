# PPS Suite — Open Questions

## [OQ-PPS-001] MES vs MRP event ownership
- **Status:** Open
- **Question:** When MRP triggers a production order, does pps.mrp or pps.mes own the `production-order.created` event?
- **Impact:** Event schema routing and consumer registration

## [OQ-PPS-002] Warehouse management system (WMS) integration boundary
- **Status:** Open
- **Question:** Does pps.wm manage physical warehouse locations, or does it delegate to a third-party WMS via pps.si?
- **Impact:** Feature scope of F-PPS-003 (Inventory & Warehouse)

## [OQ-PPS-003] EAM maintenance order relation to OPS work orders
- **Status:** Open
- **Question:** Does pps.eam.maintenance-order map to ops.ord.work-order, or are these separate entities?
- **Impact:** Cross-suite event schema and feature gating
