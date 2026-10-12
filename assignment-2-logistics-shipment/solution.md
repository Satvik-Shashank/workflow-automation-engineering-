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
- Use **timer boundary events** for missing scans, customs-document deadlines, delivery attempts, and tracing windows.
- Use **error boundary events** for scanner/tracking failures, with manual recording and later reconciliation.
- Use compensation for cancellation after dispatch: stop movement where possible, return parcel, and refund charges minus applicable fees.

## 4. Failure Path Register (5 failure points, all drawn in process.bpmn)

| ID | Failure point / trigger | Resolution inside the process | BPMN construct / terminal outcome |
|---|---|---|---|
| E1 | Pickup agent arrives but shipper is absent or parcel is not ready | Reschedule up to two times; after that cancel the booking and apply the stated cancellation policy | XOR gateway + bounded loop; `Cancelled` end |
| E2 | No scan event for 48 hours | Start tracing; if the parcel is not found within five days, declare it lost, compensate per policy and notify the customer | Timer boundary event + XOR; `Lost` end |
| E3 | Recipient unavailable at delivery | Retry delivery up to three times; hold at the depot for seven days, then return to sender | XOR + counter loop; `Returned` end |
| E4 | Shipper cancels after dispatch | Intercept the parcel if possible, return it to origin and refund charges minus the applicable fee | Message boundary event + compensation; `Cancelled` end |
| E5 | Invoice generation or shipment closure fails after delivery | Preserve proof of delivery, retry closure and invoicing, then send to operations reconciliation; do not repeat the delivery | Error boundary event + retry + reconciliation |

## 5. Important state rules
- A shipment is not marked delivered without proof of delivery or an explicitly approved exception.
- Tracking failures must not automatically stop physical movement; maintain a reconciliation queue.
- Loss declarations use a defined no-scan timer and tracing window.
- Returns, claims, and cancellation should retain a clear audit trail and charge/refund decision.
