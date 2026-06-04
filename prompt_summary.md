# Reusable Prompt Templates Summary

This repository contains two primary reusable prompt templates used by the AI agent to interact with support case databases and generate structured case summaries and reports.

---

## 1. List Recent Account Cases (`pr-list-account-cases-recent.md`)

* **Purpose:** Queries the MongoDB case summary store to find active and recently updated support cases for a specific company/account, and compiles them into a structured status overview.
* **Key Inputs / Variables:**
  * `{{ACCOUNT_NAME}}` (e.g., `"Apple Inc."`)
  * `{{N_DAYS}}` (default: 14)
* **How It Works:**
  1. Computes the current UTC time and calculates two ISO cutoff date strings: a main cutoff date (`{{CUTOFF_ISO_DATE}}` based on `{{N_DAYS}}`) and a 3-month cutoff date (`{{CUTOFF_3_MONTHS_ISO_DATE}}`).
  2. Runs a vector-store query `find_summaries_by_filter` to fetch cases matching the account name that are either:
     * Updated within the last `N` days, OR
     * Are currently active (status is not `"Closed"` or `"Resolved"`) and have had activity within the past 3 months.
  3. Fetches the case `Subject` lines in parallel via `get_mongodb_case_comments` for all unique matched cases.
  4. Generates a compact markdown table of recent cases followed by comma-separated lists of active and closed case numbers.

### Example Use Case:
```markdown
PR_LIST_ACCOUNT_CASES_RECENT for Apple Inc. with N_DAYS = 14
```
**Expected Output:**
> ### Recent Cases Table
> | Case # | Status | Severity | Subject | Last Updated (UTC) |
> | :--- | :--- | :--- | :--- | :--- |
> | [01584501](https://support.mongodb.com/case/01584501) | In Progress | S3 | app failed after mongo upgrade from 7.0.28 to 7.0.32 | 2026-06-01 |
> | [01583932](https://support.mongodb.com/case/01583932) | Closed | S2 | incremental refresh takes a lot of time | 2026-06-01 |
>
> ### Active Cases
> 01584501
>
> ### Closed Cases
> 01583932

---

## 2. Weekly Case Report (`pr-weekly-case-report.md`)

* **Purpose:** Performs a deep timeline and sentiment analysis of the comment history for a designated list of cases to produce a polished, high-fidelity customer-facing weekly active cases report.
* **Key Inputs / Variables:**
  * `{{CUSTOMER}}` (e.g., `"Apple"`)
  * `{{CASE_LIST}}` (e.g., `"01581877, 01584501"`)
  * `{{REPORT_DATE}}` (calculated)
  * `{{MARKDOWN_FORMAT}}` (default: `true`, outputs raw markdown in code fences for easy copy-pasting)
* **How It Works:**
  1. Executes a mandatory case analysis procedure: reads all comments to identify current status, actual active engineers/contacts (handling handoffs), diagnoses status (confirmed vs. hypothesized), and key risk signals.
  2. Synthesizes a case summary, sanitizing customer references to `{{CUSTOMER}} Team` and fixing name capitalization.
  3. Groups the most recent 2 meaningful updates with relative time references (e.g. `today`, `yesterday`, `N days ago`) and details pending action items/next steps.
  4. Flags risk alerts (e.g. CSM escalation, auto-close warning, inactivity >5 days).
  5. Orders cases by severity ascending (`S1` -> `S4`) with tie-breakers, and appends an "Active Cases Count" summary table at the bottom.

### Example Use Case:
```markdown
PR_WEEKLY_CASE_REPORT for Apple cases 01581877, 01584501 (both Active and Resolved)
```
**Expected Output:**
> ````markdown
> # Apple Cases Report - 2026-06-03
> 
> **1. [[01584501](https://support.mongodb.com/case/01584501)] app failed after mongo upgrade from 7.0.28 to 7.0.32**\
> 🟡 S3 | In Progress | Opened 2 days ago
>    - Summary: Apple Team experienced application crashes immediately following a minor MongoDB upgrade. The issue is suspected to be related to driver compatibility.
>    - Contact: John Doe
>    - MongoDB Owner: Jane Smith
>    - Latest Update: 
>      - Jane Smith requested application logs for review yesterday.
>      - John Doe uploaded driver debug logs today.
>    - Next Steps:
>      - Support (Jane) to analyze the provided logs and confirm driver compatibility.
> 
> ***
> 
> ### Active Cases Count
> | Severity | Count |
> | :--- | :--- |
> | 🟡 S3 | 1 |
> ````

---

## Direct Comparison & Workflow Integration

The prompts form a two-step support reporting pipeline:

| Aspect | `pr-list-account-cases-recent` (Discovery) | `pr-weekly-case-report` (Synthesis) |
| :--- | :--- | :--- |
| **Primary Goal** | Identify case numbers & verify broad metadata. | Detail case status, progression, and actions. |
| **Scope** | Account-wide scan (last 14 days or active in 3 months). | Fixed list of specified case numbers. |
| **Depth** | Fetches high-level AI summaries + case Subject. | Timeline analysis of all historical comments. |
| **Target Audience**| Internal tracking / quick status check. | High-fidelity update shared with the customer. |

### Typical Workflow:
1. Run **`pr-list-account-cases-recent`** to locate all cases for the account.
2. Select desired active and resolved cases from the output lists.
3. Run **`pr-weekly-case-report`** with those case numbers to produce the weekly customer update.
