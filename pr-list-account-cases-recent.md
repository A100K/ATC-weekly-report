List Recent Account Cases — Reusable Prompt Template

Variables:
- `{{ACCOUNT_NAME}}` — Account name. Required.
- `{{N_DAYS}}` — Number of days of activity to check (default: 14).
- `{{CUTOFF_ISO_DATE}}` — (computed) cutoff date.
- `{{CUTOFF_3_MONTHS_ISO_DATE}}` — (computed) 3-month cutoff date.

---
Task:
List in a compact markdown table all cases for account `{{ACCOUNT_NAME}}` with activity in the past `{{N_DAYS}}` days (defaulting to 14 days if not specified). Fetch case summaries matching the company name within the cutoff date, then generate the output using the structure and rules below.

---
Output Structure:
### Recent Cases Table
| Case # | Status | Severity | Subject | Last Updated (UTC) |
| :--- | :--- | :--- | :--- | :--- |
| [{{CASE_NUMBER}}](https://support.mongodb.com/case/{{CASE_NUMBER}}) | {{STATUS}} | {{SEVERITY}} | {{SUBJECT}} | {{LAST_UPDATED}} |
### Active Cases
[Comma-separated list of active case numbers, or "None"]
### Closed Cases
[Comma-separated list of closed/resolved case numbers, or "None"]

---
Field Rules:
- Case #: Verbatim case number. Render each case number as a clickable markdown link to its Support Hub page: `[{CASE_NUMBER}](https://support.mongodb.com/case/{CASE_NUMBER})`.
- Status: Verbatim case status.
- Severity: Verbatim case severity.
- Subject: The case Subject line — see step 5 under "Execution Steps & Filtering Rules" below.
- Last Updated (UTC): Date of last activity in `YYYY-MM-DD` UTC format.
- Active Cases: Comma-separated list of active case numbers (status is matched to "In Progress", "Waiting for Customer", or "Waiting for Development").
- Closed Cases: Comma-separated list of all other case numbers (status is matched to "Closed", "Resolved").

---
Execution Steps & Filtering Rules:
1. Call `get_time_utc` to retrieve the current UTC time.
2. Call `add_time_to_ISODate` to compute two cutoff dates:
   - With seconds = `-(N_DAYS * 86400)` (using 14 if `{{N_DAYS}}` is not provided) to compute the main cutoff date. Store as `{{CUTOFF_ISO_DATE}}`.
   - With seconds = `-7776000` (90 days) to compute the 3-month cutoff date. Store as `{{CUTOFF_3_MONTHS_ISO_DATE}}`.
3. Call `find_summaries_by_filter` with:
   - vector_set: "summaries_vector_case_summary_text"
   - query_filter: {
       "metadata.companyName": "{{ACCOUNT_NAME}}",
       "$or": [
         { "updatedAt": { "$gte": "{{CUTOFF_ISO_DATE}}" } },
         {
           "metadata.status": { "$nin": ["Closed", "Resolved"] },
           "updatedAt": { "$gte": "{{CUTOFF_3_MONTHS_ISO_DATE}}" }
         }
       ]
     }
   - top_k: 100
4. If the query in Step 3 returns zero results, retry with `createdAt` as a fallback:
   - query_filter: {
       "metadata.companyName": "{{ACCOUNT_NAME}}",
       "createdAt": { "$gte": "{{CUTOFF_ISO_DATE}}" }
     }
5. Retrieve the original Subject for each unique case number via `get_mongodb_case_comments`:
   - The case Subject is in the case metadata header returned by the tool (look for the "Subject" field).
   - Make all `get_mongodb_case_comments` calls in parallel (batch independent calls in a single function call block).
   - Only fetch comments for unique case numbers that will appear in the final table.
6. "updatedAt" is a native datetime field (auto-converts from ISO string in filter).
7. "metadata.companyName" is indexed — use exact match (case-sensitive).
8. "createdAt" is also indexed but reflects document creation, not last case activity — use "updatedAt" for recency unless using the step 4 fallback.

---
Formatting Rules:
- Use inline markdown (no code fences around the table or lists).
- Do not invent data. If a field is unavailable, write `N/A`.
- If no cases are found, output "No recent cases found." for the table, and write "None" for both the Active and Closed case lists.

---
Ordering Rules:
Sort cases by `updatedAt` descending.
