Weekly Case Report — Reusable Prompt Template

Variables:
- `{{CUSTOMER}}` — customer/account name. Required.
- `{{CASE_LIST}}` — list of case numbers to be included in the report. Required.
- `{{REPORT_DATE}}` — today's date in `YYYY-MM-DD` (UTC). Calculated.
- `{{MARKDOWN_FORMAT}}` — flag that determines the output format (default: true). 

---
Task:
Prepare a weekly active cases report for `{{CUSTOMER}}` covering the cases in `{{CASE_LIST}}`. For each case, fetch metadata and the full comment history. Analyze the entire timeline to derive accurate state before generating the report using the structure and rules below.

---
Case Analysis Procedure (MANDATORY per case, before writing any field):
1. READ ALL COMMENTS — Scan the entire comment timeline. Identify:
   - The actual current problem state (resolved? in progress? blocked? hypothesis confirmed or still unconfirmed?)
   - Who is actively working the case (may differ from metadata owner/contact)
   - What was the most recent meaningful exchange (skip auto-reminders)
   - Whether any root cause or diagnosis is CONFIRMED vs HYPOTHESIZED vs UNCONFIRMED
2. VERIFY OWNER — Cross-check the `owner` metadata field against who is actually responding in comments.
   - If the case was transferred (FTS handoff, NTSE reassignment, TAM routing), use the CURRENT active engineer.
   - If multiple engineers contributed, list the current owner first, then note key contributors.
3. VERIFY CONTACT — Cross-check the `contact` metadata field against who is actively communicating.
   - If the opener differs from the active technical contact, list the active contact first, then note the opener.
   - For multi-person customer teams, list the primary active contact and key participants.
5. VERIFY STATUS — Use the actual case status field from metadata. Do not infer or override it.
6. QUALIFY CLAIMS — For every technical assertion in the summary:
   - If confirmed by evidence/logs/reproduction: state as fact
   - If hypothesized but untested/unconfirmed: use "suspected", "hypothesized", "initial theory"
   - If disproven by later evidence: do NOT include as current state; note it was ruled out if relevant
   - Never present a hypothesis as a confirmed diagnosis
7. CHECK RISK SIGNALS — Flag any of:
   - Case is customer-escalated or CSM-escalated (from metadata)
   - Auto-close reminders have been sent with no customer response
   - Case has been waiting for customer response for >5 days
   
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
- latest_update: Bulleted list covering the 2 most recent MEANINGFUL comments (skip auto-close reminders and system-generated comments unless they are the only activity).
   - Each bullet: who said/did what, and when (days ago / yesterday / today).
   - Compute "days ago" from `{{REPORT_DATE}}`, not from when the report is generated.
- next_steps: Bulleted actions derived from the current case state and recent comments, prefixed `{{CUSTOMER}}` ([contact_first_name]) or Support ([owner_first_name]).
    - Include only actions that are actually pending — not completed or superseded actions.
    - If no next steps, write "No next steps".
- risk_flags: (optional) Only include if risk signals detected per step 7 above. Prefix with warning emoji.
    - Examples: "⚠ Escalated", "⚠ Approaching auto-close (N reminders sent, no response since DATE)", "⚠ No customer response in N days"

---
Output Structure:

Header (H1)
# `{{CUSTOMER}}` Cases Report - `{{REPORT_DATE}}`

Per-case block (repeat, one per case)
- First line bolded; remaining lines as an indented bulleted list.

**N. [[case_number]]([case_hub_url]) [case_title]**\
[severity_icon] [severity] | [status] | Opened [age] days ago
   - Summary: [summary]
   - Contact: [contact]
   - MongoDB Owner: [owner]
   - Latest Update: [latest_update]
   - Next Steps: [next_steps]
   - [risk_flags] (omit this line entirely if no risk signals)
***

---
Ordering Rules:
Sort cases by severity ascending (S1 → S2 → S3 → S4). Tie-breakers in order: status priority `In Progress` > `Waiting for Customer` > `Waiting for Development` > `Closed` > `Resolved`

---
Formatting Rules:
- Output Format:
  - If `{{MARKDOWN_FORMAT}}` is true (or not specified): Wrap the entire report in a single markdown code block (using triple backticks and the `markdown` identifier, like ```markdown ... ```) to make it easy to copy and transfer to external editor tools.
  - If `{{MARKDOWN_FORMAT}}` is false: Output the report directly as inline markdown without any wrapping code fences.
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
- At the bottom of the report, add a table summarizing counts of cases by severity; counting only cases with status "In Progress", "Waiting for Customer", "Waiting for Development"; name this section "Active Cases Count". Combine `severity_icon` and `severity` into first column.