# BPMN Project Assignments — Two Solutions

This submission models **Assignment 1: Student Project Approval & Allocation** and **Assignment 2: Logistics & Shipment Exception Management** from the provided assignment brief.

## Contents
- `assignment-1-student-project/solution.md` — happy path, BPMN design decisions, failure-path register.
- `assignment-1-student-project/process.bpmn` — BPMN 2.0 XML process model for the core approval/allocation flow.
- `assignment-2-logistics-shipment/solution.md` — happy path, BPMN design decisions, failure-path register.
- `assignment-2-logistics-shipment/process.bpmn` — BPMN 2.0 XML process model for the core shipment flow.

## Open the BPMN diagrams
Import either `.bpmn` file into a BPMN 2.0 modeler such as Camunda Modeler or bpmn.io. The XML provides the core executable-flow structure; the Markdown documents specify exception handling and detailed failure paths to incorporate into the full visual model.

## Modeling conventions
- **User task:** work performed by a person.
- **Service task:** automated system operation.
- **Business rule task:** policy/rule-based decision or allocation.
- **Exclusive gateway (XOR):** exactly one outcome is selected.
- **Parallel gateway (AND):** branches execute together and synchronize.
- **Timer boundary event:** deadline/SLA/timeout handling.
- **Error boundary event:** technical failure handling.
- **Message event:** external participant message or cancellation/withdrawal.
- **End events:** successful completion and distinct terminal outcomes.

The failure-path registers intentionally include more than the obvious happy-path exceptions, with bounded retries, escalation, compensation, and terminal states.
