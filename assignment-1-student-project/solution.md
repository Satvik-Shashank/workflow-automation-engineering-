# Assignment 1 — Student Project Approval & Allocation System

## 1. Process scope and participants
**Start:** student team decides to submit a proposal.  
**Successful end:** a faculty guide accepts the allocation, and the team is notified.  
**Other ends:** rejected, lapsed, or withdrawn.

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

## 4. Failure Path Register (5 failure points, all drawn in process.bpmn)

| ID | Failure point / trigger | Resolution inside the process | BPMN construct / terminal outcome |
|---|---|---|---|
| F1 | Committee asks for a revision | Send actionable feedback; allow up to two revision cycles with a seven-day timer per cycle; lapse if the team never responds | Timer boundary event + bounded loop; `Lapsed` end |
| F2 | Committee rejects the proposal | Send the reason to the team and close the case | Send task + XOR; `Rejected` end |
| F3 | Guide declines or does not respond | Exclude that guide and re-run matching; after three attempts the coordinator allocates manually | XOR + timer boundary event + bounded retry |
| F4 | Team withdraws while the guide response is pending | Release the reserved guide capacity and close the case | Message boundary event + compensation; `Withdrawn` end |
| F5 | Database write fails when saving the allocation | Retry up to three times, then the coordinator sends a manual notice and records a reconciliation task | Error boundary event + retry + manual fallback |

## 5. Important state rules
- A guide is not confirmed until acceptance **and** successful persistence of the allocation.
- Every rejection, lapse, or invalid-team outcome records a reason and notifies the student.
- Retry loops are bounded; repeated failures escalate or terminate instead of looping forever.
- A team withdrawal must release any reserved guide capacity.
