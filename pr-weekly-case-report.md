Weekly Case Report — Reusable Prompt Template

Variables:
- `{{CUSTOMER}}` — customer/account name. Required.
- `{{CASE_LIST}}` — list of case numbers to be included in the report. Required.
- `{{REPORT_DATE}}` — today's date in `YYYY-MM-DD` (UTC). Calculated.
- `{{MARKDOWN_FORMAT}}` — flag that determines the output format (default: true). 

---
Task:
Prepare a weekly active cases report for `{{CUSTOMER}}` covering the cases in `{{CASE_LIST}}`. Fetch each case's data (metadata + comments), then generate report using the  structure and rules below.

---
Field Rules:
- N: Incremented numbered list starting from 1; add period (.) after the number; No surrounding brackets. 
- case_number: 8-digit, zero-padded if 7-digits case numbers provided (1573208 -> 01573208).
- case_hub_url: `https://support.mongodb.com/case/` + case_number.
- case_title: Verbatim from case metadata.
- severity: Verbatim case severity (e.g., S1, S2, S3, S4).
- severity_icon: S1 -> 🔴, S2 -> 🟠, S3 -> 🟡, S4 -> 🟢.
- status: Verbatim case status.
- age: Whole days from open to `{{REPORT_DATE}}` (UTC).
- summary: 2-4 sentence case narrative; replace "customer" or "Customer" refs with `{{CUSTOMER}}` Team; keep nouns.
- contact: Contact name from metadata. 
- contact_first_name: First name of contact person. 
- owner: Owner name from case metadata. 
- owner_first_name: First name of owner.
- latest_update: Bulleted list covering summary of 2 most recent comments, including how many days ago update was made; skip internal/auto comments unless none else exist.
- next_steps: Bulleted actions from recent comments, prefixed `{{CUSTOMER}}` ([contact_first_name]) or Support ([owner_first_name]); else "No next steps".

---
Output Structure:

Header (H1)
# `{{CUSTOMER}}` Cases Report - `{{REPORT_DATE}}`

Per-case block (repeat, one per case)
- First line bolded; remaining lines as an indented bulleted list.

**N. [[case_number]]([case_hub_url]) [case_title]**<br>
[severity_icon] [severity] | [status] | Opened [age] days ago
   - Summary: [summary]
   - Contact: [contact]
   - MongoDB Owner: [owner]
   - Latest Update: [latest_update]
   - Next Steps: [next_steps]

---
Ordering Rules:
Sort cases by severity ascending (S1 → S2 → S3 → S4). Tie-breakers in order: status priority `In Progress` > `Waiting for Customer` > `Waiting for Development` > `Closed` > `Resolved`

---
Formatting Rules:
- Output Format:
  - If `{{MARKDOWN_FORMAT}}` is true (or not specified): Wrap the entire report in a single markdown code block (using triple backticks and the `markdown` identifier, like ```markdown ... ```) to make it easy to copy and transfer to external editor tools.
  - If `{{MARKDOWN_FORMAT}}` is false: Output the report directly as inline markdown without any wrapping code fences.
- Only the first line of each case block is bold, and it must end with a `<br>` tag to force a line break before the metadata line.
- Do not add section headers between cases.
- Do not invent data. If a field is unavailable, write `N/A` (for Next Steps use `No next steps`).
- Fix name capitalization for all customer contact names.
- If the case [age] is 1, render the age line as "Opened 1 day ago" instead of "Opened 1 days ago".
- Render the age of each Latest Update comment as follows: if the comment is from today, use the literal word `today`; if from yesterday, use `yesterday`; otherwise use `N days ago`.
- Do not include internal comments, internal notes, or internal-only timestamps in Latest Update or Summary.
- If a case was auto-closed, add a Next Steps action item for `{{CUSTOMER}}` to confirm the resolution.
- Do not use em-dashes.
- Do not reference internal ticket metadata details (for example, HELP-012345).

---
Summary of cases:
- At the bottom of the report, add a table summarizing counts of cases by severity; use severity_icon; counting only cases with status "In Progress", "Waiting for Customer", "Waiting for Development"; name this section "Active Cases Count".

