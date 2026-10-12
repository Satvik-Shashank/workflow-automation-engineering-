# Workflow Automation Engineering — BPMN Process Models

Two business processes modeled in BPMN 2.0, each with a ready-to-open `.bpmn` diagram and a written design (happy path, modeling choices, 5 failure points).

| | Assignment 1 | Assignment 2 |
|---|---|---|
| **Process** | Student Project Approval & Allocation | Logistics & Shipment Exception Management |
| **Starts when** | A student team submits a proposal | A shipper books a shipment |
| **Ends when** | A faculty guide accepts and the team is notified | Delivered and closed, returned, lost, or cancelled |
| **Lanes** | Student/Team, Coordinator, Review Committee, Faculty Guide, System | Shipper, Control Tower, Pickup Agent, Hub, Delivery Agent |
| **Happy path** | 12 steps | 10 steps |
| **Failure paths** | 5 (F1–F5) | 5 (E1–E5) |
| **Files** | [`solution.md`](assignment-1-student-project/solution.md) · [`process.bpmn`](assignment-1-student-project/process.bpmn) | [`solution.md`](assignment-2-logistics-shipment/solution.md) · [`process.bpmn`](assignment-2-logistics-shipment/process.bpmn) |

## Approach

**Assignment 1:** proposal → validation → coordinator screening → similarity check → committee decision → rule-based guide ranking → guide acceptance → persisted allocation.
Notable handling:
- Revision and resubmission loops are capped.
- A guide counts as allocated only after they accept *and* the record is saved.
- Withdrawals release the reserved guide slot.

**Assignment 2:** booking and charge validation → pickup → parallel transport and tracking → multi-instance hub step for each leg → delivery with proof → closure and invoice.
Notable handling:
- Closure or invoice failures are retried and then reconciled; the delivery is never repeated.
- A shipment cannot be marked delivered without proof of delivery.
- A 48-hour no-scan timer starts tracing, and a parcel that stays unfound is declared lost.
- Cancellation after dispatch is handled by compensation (return the parcel and refund minus fees).

## Modeling conventions

| Element | Used for |
|---|---|
| User / manual task | Work done by a person |
| Service task | Automated system actions |
| Business rule task | Policy-driven decisions (guide matching, charges, validation) |
| Exclusive gateway (XOR) | One outcome among several |
| Parallel gateway (AND) | Concurrent branches |
| Timer boundary event | SLAs, deadlines, retry windows |
| Error boundary event | Technical failures, with bounded retry and manual fallback |
| Message event | Withdrawal or cancellation |
| Distinct end events | Each terminal outcome (rejected, lapsed, lost, returned, cancelled) |

Every retry loop is bounded and ends in a terminal state, so no case loops forever.

## Viewing the diagrams

Open either `process.bpmn` in [Camunda Modeler](https://camunda.com/download/modeler/) (File > Open File) or drag it onto [demo.bpmn.io](https://demo.bpmn.io). Each file has lanes, the full happy path and its 5 failure points drawn with timer, error and message boundary events.
