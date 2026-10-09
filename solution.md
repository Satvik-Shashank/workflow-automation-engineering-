# Assignment 2 — Logistics & Shipment Exception Management

## 1. Process scope and participants
**Start:** shipper books a shipment.  
**Ends:** delivered and closed, returned to sender, lost, or cancelled.

**Lanes / participants:** Shipper / Customer; Control Tower / Dispatcher; Pickup Agent; Hub / Warehouse; Delivery Agent; Recipient; Claims Department. Customs may be modeled as an external pool for international shipments.

## 2. Happy path
1. **Shipper — User Task:** Create shipment booking.
2. **System — Business Rule Task:** Validate address, parcel weight/dimensions, service level, and calculate charges.
3. **Dispatcher — Service Task:** Schedule pickup and assign an agent.
4. **Pickup Agent — User/Manual Task:** Collect parcel and scan it.
5. **Parallel gateway:** Start physical transport and tracking/shipper status notification concurrently.
6. **Hub — Subprocess:** Sort, scan, and move parcel through each line-haul leg; use a multi-instance subprocess for multiple hubs.
7. **Dispatcher/System — Send Task:** Mark out for delivery and notify recipient with ETA.
8. **Delivery Agent — User Task:** Hand over parcel and capture proof of delivery.
9. **System — Service Task:** Close shipment, store proof of delivery, and generate invoice.
10. **End Event:** Shipment delivered.

## 3. BPMN design choices
- Use **business rule tasks** for service validation, charges, routing, and surcharge policies.
- Use **service tasks** for assignment, tracking updates, notifications, invoice generation, and reconciliation.
- Use **user/manual tasks** for pickup, inspection, delivery handover, and proof capture.
- Use an **AND gateway** to model independent physical movement and tracking/notification updates.
- Use an **event-based gateway** when the process must wait for either an incoming delay/status message or a timeout.
- Use **timer boundary events** for missing scans, customs-document deadlines, delivery attempts, and tracing windows.
- Use **error boundary events** for scanner/tracking failures, with manual recording and later reconciliation.
- Use a **claims subprocess** for damaged or insured lost parcels.
- Use compensation for cancellation after dispatch: stop movement where possible, return parcel, and refund charges minus applicable fees.

## 4. Failure Path Register

| ID | Failure point / trigger | Resolution inside the process | BPMN construct / terminal outcome |
|---|---|---|---|
| E1 | Pickup agent arrives but shipper is absent or parcel is not ready | Reschedule up to two times; after that cancel booking and apply the stated cancellation policy | Timer/escalation boundary + bounded loop; `Cancelled` end |
| E2 | Address incomplete or invalid at booking | Return booking to shipper for correction before pickup is scheduled | XOR gateway + loop to booking |
| E3 | Address is wrong or recipient cannot be located at delivery | Agent contacts recipient; correct address and recalculate any surcharge before reattempting | Message event + XOR + service task |
| E4 | Parcel is damaged at pickup or hub scan | Inspect and record evidence; if insured, route to claims for refund/replacement/reshipment; otherwise notify shipper and record liability | Inspection task + XOR + claims subprocess |
| E5 | No scan event for 48 hours | Start tracing subprocess and check last known hub/vehicle; if not found within five days, declare lost, compensate per policy, notify customer | Timer boundary event + tracing subprocess; `Lost` end |
| E6 | Weather, strike, vehicle breakdown, or hub overload delays transit | Wait for delay message or SLA timer, notify customer, reroute or assign alternate vehicle, and update ETA | Event-based gateway + message/timer + reroute loop |
| E7 | International parcel held by customs due to missing/incorrect documents | Request corrected documents; allow five days; if received, resubmit; otherwise return to origin | Send/receive tasks + timer; return-to-sender path |
| E8 | Recipient unavailable | Retry delivery up to three times; hold at depot for seven days, then return to sender | Counter loop + timer; `Returned` end |
| E9 | Recipient refuses parcel or cash-on-delivery payment fails | Record refusal/payment failure, adjust charges, and initiate return to sender | XOR gateway + compensation/return subprocess |
| E10 | Scanner offline or tracking data mismatch | Hub supervisor records scan manually; reconcile once systems recover and preserve an audit trail | Error boundary event + manual task + reconciliation |
| E11 | Shipper cancels after dispatch | Stop or intercept parcel if possible, return it to origin, refund charges minus applicable fee, and close the case | Message boundary event + compensation + `Cancelled` end |
| E12 | Parcel weight/dimensions differ from booking | Reweigh and recalculate charge; request shipper approval for surcharge or cancel/return if rejected | Business rule task + XOR |
| E13 | Pickup scan succeeds but tracking update fails | Continue physical movement, queue event for retry, alert operations if unresolved, and reconcile scan history | Error boundary event + asynchronous retry + reconciliation |
| E14 | Parcel routed to wrong hub | Detect route mismatch at scan, create corrective transfer, update ETA, and notify customer if SLA changes | XOR gateway + corrective subprocess |
| E15 | Delivery agent cannot safely complete delivery or parcel is inaccessible | Record exception, return parcel to depot, dispatch a new attempt or arrange collection | Exception task + bounded retry |
| E16 | Proof of delivery missing or invalid | Do not auto-close shipment; request agent correction and reconcile recipient confirmation before invoicing | XOR gateway + user task + timer/escalation |
| E17 | Invoice generation or shipment closure fails after delivery | Preserve proof of delivery, retry closure/invoice, then send to operations reconciliation; do not repeat the physical delivery | Error boundary event + retry + reconciliation |
| E18 | Parcel suspected lost but later found | Cancel lost-claim finalization if not paid, verify parcel identity, resume delivery or return as directed, and notify stakeholders | Message event + XOR + compensation/reopen subprocess |

## 5. Important state rules
- A shipment is not marked delivered without proof of delivery or an explicitly approved exception.
- Tracking failures must not automatically stop physical movement; maintain a reconciliation queue.
- Loss declarations use a defined no-scan timer and tracing window.
- Returns, claims, and cancellation should retain a clear audit trail and charge/refund decision.
