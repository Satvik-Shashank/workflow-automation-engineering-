# Workflow Automation Engineering — BPMN Process Models

Two business processes modeled in BPMN 2.0, each with a written design (happy path, modeling choices, failure-path register) and a `.bpmn` file.

| | Assignment 1 | Assignment 2 |
|---|---|---|
| **Process** | Student Project Approval & Allocation | Logistics & Shipment Exception Management |
| **Starts when** | A student team submits a proposal | A shipper books a shipment |
| **Ends when** | A faculty guide accepts and the team is notified | Delivered and closed, returned, lost, or cancelled |
| **Lanes** | Student/Team, Coordinator, Review Committee, Faculty Guide, System | Shipper, Dispatcher, Pickup Agent, Hub, Delivery Agent, Recipient, Claims |
| **Happy path** | 12 steps | 10 steps |
| **Failure paths** | 15 (F1–F15) | 18 (E1–E18) |
| **Files** | [`solution.md`](assignment-1-student-project/solution.md) · [`process.bpmn`](assignment-1-student-project/process.bpmn) | [`solution.md`](assignment-2-logistics-shipment/solution.md) · [`process.bpmn`](assignment-2-logistics-shipment/process.bpmn) |

## Approach

**Assignment 1:** proposal → validation → coordinator screening → similarity check → committee decision → rule-based guide ranking → guide acceptance → persisted allocation.
Notable handling:
- Revision and resubmission loops are capped.
- A guide counts as allocated only after they accept *and* the record is saved.
- Withdrawals release the reserved guide slot.

**Assignment 2:** booking and charge validation → pickup → parallel transport and tracking → hub subprocess for each leg → delivery with proof → closure and invoice.
Notable handling:
- Tracking failures never stop the parcel; they go to a reconciliation queue.
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
| Event-based gateway | Wait for a message or a timeout |
| Timer boundary event | SLAs, deadlines, retry windows |
| Error boundary event | Technical failures, with bounded retry and manual fallback |
| Message event | Withdrawal, cancellation, delay notices |
| Distinct end events | Each terminal outcome (rejected, lapsed, lost, returned, cancelled) |

Every retry loop is bounded and ends in escalation or a terminal state, so no case loops forever.

## Viewing the diagrams

Open either `.bpmn` file in [bpmn.io](https://demo.bpmn.io) or Camunda Modeler.

The `.bpmn` files model the **core flow**. The exception handling is specified in the failure-path register in each `solution.md` (trigger, resolution, BPMN construct, outcome), not drawn in full in the XML.
