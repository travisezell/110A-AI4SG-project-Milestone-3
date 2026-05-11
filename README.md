# Campus Accessibility 311 Assistant

## 1. Problem — who is affected and what breaks today

Our project focuses on disabled students and staff on campus who rely on accessible routes, doors, elevators, and classrooms to get to class and work on time.[page:1] Today, when an automatic door fails, an elevator is out of service, or an accessible ramp is blocked, there is no simple, structured way for them to report the issue and track whether it is being fixed.[page:1]

The current path is fragmented: people might email facilities, tell a professor, or just avoid that building altogether, which means many accessibility failures are invisible in the maintenance queue.[page:1] The exact failure point is that real accessibility barriers experienced by disabled users do not reliably become structured, actionable tickets that reach the right facilities team quickly.[page:1]

This breakdown disproportionately affects wheelchair users, people with mobility impairments, and others who cannot "just take the stairs" or "use another entrance," turning minor facility failures into missed classes, safety risks, and exclusion from parts of campus.[page:1]

## 2. AI Capability — what we used and why it fits

We use the **structured** data extraction capability from Lab 2 to turn messy accessibility complaints into a consistent, machine‑readable record.[page:1] A disabled student might write "The button for the south entrance of the Engineering building doesn't work again, I had to wait for someone to open it," which is clear to a human but not in the format a ticketing system expects.[page:1]

By applying the Lab 2 pattern, our model fills in a fixed schema with fields like `building`, `entrance_type`, `issue_type`, `urgency_level`, `accessibility_impact`, and `recommended_team` based on the user's free‑text description.[page:1] This fits the failure point because the problem is not lack of awareness that barriers exist; the problem is that those reports never become structured data that facilities can prioritize and track over time.[page:1]

Optionally, we also use **text** generation (Lab 1) to draft an acknowledgment email or SMS in clear, plain language that confirms the report and sets expectations for follow‑up, which matters for users who are used to their complaints disappearing.[page:1]

## 3. Workflow — what goes in, what the AI does, what comes out, who acts (with screenshots)

At a high level, the system takes free‑text accessibility complaints (plus optional photos), converts them into structured tickets, and routes them to campus facilities with an accessibility impact flag.[page:1]

### Inputs

- A disabled student or staff member opens an "Accessibility 311" web form.
- They submit:
  - A short description of the problem in their own words  
  - Optional photo (e.g., blocked ramp, broken door button)  
  - Location fields (building, floor, entrance if known)[page:1]

> Screenshot: `screenshots/01_accessibility_form.png` — campus accessibility 311 form showing text box, location fields, and optional photo upload.[page:1]

### AI processing

- Our Colab notebook sends the text (and, in future versions, images) to the model with a schema inspired by Lab 2:  
  `building`, `location_detail`, `issue_type` (door, elevator, ramp, signage), `urgency_level` (LOW/MEDIUM/HIGH), `accessibility_impact` (BLOCKED_ACCESS, PARTIAL_ACCESS, INCONVENIENCE), and `recommended_team`.[page:1]
- The AI:
  - Parses the free‑text description into this schema  
  - Classifies urgency based on whether alternative accessible routes exist  
  - Suggests which facilities team should handle it (electrical, elevator vendor, grounds, etc.)[page:1]

> Screenshot: `screenshots/02_schema_prompt.png` — notebook cell showing the Gemini prompt and JSON schema used for extraction.[page:1]

### Outputs

- A JSON‑like record representing a ticket that could be ingested by an existing facilities system.
- A human‑readable summary for facilities staff.
- An optional acknowledgment message to send back to the reporter.[page:1]

> Screenshot: `screenshots/03_sample_output.png` — example model output with all fields populated plus the generated summary.[page:1]

### Who acts on it

- Facilities or Disability Resource Center staff review the structured ticket in a simple dashboard, adjust routing or urgency if needed, and then create or update a work order.[page:1]
- Over time, aggregated tickets help identify buildings or routes with repeated accessibility failures for capital planning.[page:1]

> Screenshot: `screenshots/04_staff_dashboard_mock.png` — mock view of a staff dashboard listing accessibility tickets with impact and urgency flags.[page:1]

## 4. Failure Case — one realistic error tied to a lab output

In an edge‑case test inspired by Lab 2, we submitted a complaint: "The elevator in the Library is out again but there's another elevator at the other end, I was late but still made it," and marked the user as a wheelchair user.[page:1] The AI extracted fields correctly but labeled `urgency_level` as `LOW` and `accessibility_impact` as "INCONVENIENCE," because it recognized that an alternative route technically existed.[page:1]

For a wheelchair user who has to cross the entire building or go outside and back in, this is more than an inconvenience; it can mean arriving late to exams or avoiding the library entirely.[page:1] In a real deployment, under‑classifying this ticket would push it to the bottom of the queue, prolonging the barrier for the people the system is designed to help.

We saw a similar pattern in Lab 2, where the model could fill the schema but made questionable decisions on severity and department when the text contained mixed signals.[page:1] That lab output showed that even when extraction is formally correct, the model's judgment about impact and urgency can understate the real effect on vulnerable users.[page:1]

## 5. Oversight and Tradeoff — human review and the one change

**Oversight decision.** Our position is that any ticket with `accessibility_impact` other than "INCONVENIENCE" (for example, "BLOCKED_ACCESS" or "PARTIAL_ACCESS") must be reviewed by Disability Resource Center or facilities staff before it is prioritized or closed.[page:1] We ground this in the labs, where we saw that the model's severity labels can be systematically off in cases that affect people, not just infrastructure.[page:1]

**The one change.** To reduce the harm from the under‑classified elevator case, we added a rule that if the reporter identifies as using a wheelchair or mobility aid, all elevator and ramp issues are automatically escalated at least one urgency level above what the model assigns.[page:1] The tradeoff is that this increases the number of "HIGH" or "MEDIUM" tickets that require faster response and may cause some non‑critical issues to compete with other facilities work, potentially slowing down maintenance for lower‑impact problems.[page:1]

We accept this tradeoff because the cost of occasionally over‑prioritizing an accessibility issue is lower than the cost of leaving a critical barrier unresolved for disabled students and staff who cannot use alternative routes easily or safely.[page:1]