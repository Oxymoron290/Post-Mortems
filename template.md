# Incident title

**Date**: MM-DD-YYYY start of incident

**Status**: A status of the incident

Status suggestions:

```none
❗ Major Outage - Issue ongoing with significant impact
❌ Unresolved - Issue ongoing/workaround exists/not a priority
⚠️ Mitigated - crisis eliminated, outstanding action items
✅ Resolved - All action items completed
```

A sentence or two that describe the incident. Start with the initial issue reported.

## Overview

One or two paragraphs that summarize the incident. Do not include any technical details.

## Impact

Describe what systems were affected by the incident, the users and individuals who were impacted, and any revenue loss as a result of outages.

## Root Cause

Describe the problem, or problems that manifested in the incident taking place.

## Detection

Describe how the problem came to be realized by the team.

## Resolution and Recovery

Describe what steps (and missteps) were taken to correct the problem.

### Action Items

Describe any outstanding tasks that need to be addressed as a result of this incident. Whether tactical (i.e. data cleanup and reactions) or strategic (i.e. prevention and proaction)

Status | Action Item | Type | Owner | Link
---: | --- | :---: | ---: | ---
🔄 | Sanitize and normalize data in `schema.table` | Tactical | John Doe | [Github Issue #3](./)
✔️ | Create data validator to prevent erroneous data | Strategic | Mary Smith | [Github Issue #7](./)

## Lessons Learned

### What went well

- item
- item
- item

### What went wrong

- item
- item
- item

### Where we got lucky

- item
- item
- item

## Timeline

> All times in a defined TimeZone, Central time preferred

Time | Description
--- | ---
16:34 | This event happened
16:39 | That event happened
15:07 | The following logs were recorded - [Logs](http://example.com/)

## Supporting information

- links to slack chats
- links to log files
- links to research material used for resolution
- etc.
