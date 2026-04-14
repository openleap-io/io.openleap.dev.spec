# CO Suite — Open Questions

## [OQ-CO-001] Port assignments for CO services
- **Status:** Open
- **Question:** Ports are TBD in several domain specs (co_abc, co_io). Need formal port assignment from platform-lead.
- **Impact:** OpenAPI server stubs reference localhost:TBD

## [OQ-CO-002] Internal Order vs. Project distinction
- **Status:** Open
- **Question:** ops.prj and co.io have overlapping project-cost concepts. Confirm clear boundary.
- **Impact:** Feature model constraints between CO and OPS suites

## [OQ-CO-003] ABC integration with FI
- **Status:** Open
- **Question:** Does co.abc publish cost object events that FI directly consumes, or does CO aggregate first?
- **Impact:** Event schema design for abc.process-cost.calculated
