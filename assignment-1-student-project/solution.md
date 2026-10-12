# Assignment 1 — Student Project Approval & Allocation System

## 1. Process scope and participants
**Start:** student team decides to submit a proposal.  
**Successful end:** a faculty guide accepts the allocation, and the team is notified.  
**Other ends:** invalid team, rejected, lapsed, or withdrawn.

**Lanes:** Student / Team; Project Coordinator; Review Committee (including HoD); Faculty Guide; Project Management System.

## 2. Happy path
1. **Student — User Task:** Submit project proposal and team details.
2. **System — Service Task:** Validate required fields, abstract, roll numbers, and team rules.
3. **Coordinator — User Task:** Screen proposal for completeness and suitability.
4. **System — Service Task:** Check topic similarity against prior projects.
5. **Committee — User Task:** Evaluate feasibility, scope, novelty, and resources.
6. **XOR gateway — Committee decision:** approved / revise / rejected.
7. **System — Business Rule Task:** Rank eligible guides by domain match, availability, and workload.
8. **System — Service Task:** Allocate the highest-ranked eligible guide.
9. **System — Send Task:** Send allocation request to guide.
10. **Guide — Receive/User Task:** Accept or decline the request.
11. **System — Send Task + Service Task:** Notify student and persist the confirmed allocation.
12. **End Event:** Project allocated and confirmed.

## 3. BPMN design choices
- Use **user tasks** for proposal submission, coordinator screening, committee evaluation, and guide acceptance because a person makes the decision or supplies information.
- Use **service tasks** for deterministic validation, similarity checking, notifications, and database updates.
- Use a **business rule task** for guide matching because allocation follows policy (domain fit, workload limits, availability).
- Use an **exclusive gateway** for mutually exclusive committee outcomes and guide acceptance/decline.
- Use **timer boundary events** for submission deadlines, committee SLAs, guide response time, and revision windows.
- Use **error boundary events** for system/notification failures; retry with a limit before manual fallback.
- Use a **message boundary event** on the allocation subprocess for withdrawal or a guide going on leave; compensate by releasing the guide slot.
- Limit revision/resubmission loops to avoid cases cycling indefinitely.

## 4. Failure Path Register

| ID | Failure point / trigger | Resolution inside the process | BPMN construct / terminal outcome |
|---|---|---|---|
| F1 | Proposal fields missing, abstract absent, invalid roll number | Return an explicit validation report; allow resubmission twice, then escalate to coordinator | XOR gateway + bounded loop + escalation |
| F2 | Team size or membership rule violated; member belongs to another team | Notify team to reform membership; do not send invalid proposal to review | XOR gateway; error end `Invalid team` if unresolved |
| F3 | Topic duplicates or similarity exceeds threshold | Coordinator checks match; request a changed topic or reject with reason | User task + XOR; loop to proposal |
| F4 | Submission deadline missed | Notify team and flag late submission; coordinator decides whether to accept an exception | Timer boundary event + XOR |
| F5 | Committee requests revision | Send actionable feedback; allow up to two revision cycles with a seven-day timer per cycle | Timer + bounded loop; force approve/reject after limit |
| F6 | Committee rejects proposal | Send reason; allow one new-topic submission; otherwise close case | Message/send task + XOR; `Rejected` end |
| F7 | Committee review exceeds SLA | Send reminder; if second SLA expires, escalate to HoD | Non-interrupting timer boundary events + escalation |
| F8 | No suitable guide available or all matching guides are at capacity | Coordinator manually allocates, appoints co-guide/external guide, or places team on a monitored waiting list | Business rule result + XOR + timer |
| F9 | Guide declines or does not respond | Exclude guide and rerun matching; after three attempts, coordinator manually allocates | XOR + timer + bounded retry |
| F10 | Notification service or database write fails | Retry up to three times; then coordinator sends manual notice and records a reconciliation task | Error boundary event + retry loop + manual fallback |
| F11 | Team withdraws after approval or guide becomes unavailable | Release reserved guide capacity and return case to allocation or close it if team withdrew | Message boundary event + compensation |
| F12 | Coordinator screening backlog | SLA timer sends reminder and then escalates to programme lead; proposal remains queued rather than silently lost | Non-interrupting timer + escalation |
| F13 | Similarity service unavailable | Retry; if still unavailable, queue manual similarity review instead of treating the proposal as unique | Error boundary event + manual task |
| F14 | Guide accepts but allocation persistence fails | Keep allocation pending, retry database write, reconcile against guide response, and notify only after durable confirmation | Error boundary event + retry + reconciliation |
| F15 | Student does not respond to revision request | Reminder before deadline; lapse proposal after the defined response window | Timer boundary event; `Lapsed` end |

## 5. Important state rules
- A guide is not confirmed until acceptance **and** successful persistence of the allocation.
- Every rejection, lapse, or invalid-team outcome records a reason and notifies the student.
- Retry loops are bounded; repeated failures escalate or terminate instead of looping forever.
- A team withdrawal must release any reserved guide capacity.
