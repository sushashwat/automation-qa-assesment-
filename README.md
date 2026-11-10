# Automation & QA Assessment - Shashwat Gupta

## Contents

| File | What it is |
|---|---|
| `Task1_QA_Report_ShashwatGupta.pdf` | Task 1: QA & debug report (11 issues + root-cause analysis) |
| `Task2_Workflow_ShashwatGupta.json` | Task 2: n8n workflow export |
| `screenshots/` | Evidence for Task 1 and Task 2 |

## Task 2: HackerNews Morning Brief (n8n)

**Flow:** Schedule Trigger (every 1 hour, for testing) > HTTP Request (HackerNews front page) > Code node (top 5) > HTTP Request (top story details) > IF (points > 500) > Discord.

### APIs used and why
- **HackerNews Search API (Algolia)**, `https://hn.algolia.com/api/v1/search?tags=front_page`: returns the 30 front-page stories with points and comment counts in a single call. No API key is needed, which keeps the workflow simple and avoids storing another secret. (HackerNews's own API only returns IDs, so points would need 30 extra calls.)
- **HackerNews Items API (Algolia)**, `https://hn.algolia.com/api/v1/items/{id}`: second endpoint of the same API. It enriches the top story with its full comment tree, and the digest shows how many top-level comments it has.

### What the transformation does
The Code node takes the 30 stories, drops any without a title or points, sorts by points (highest first), keeps the top 5, and keeps only the fields needed: title, url, points, comments, author, objectID. If a story has no url (for example Ask HN), it builds the HackerNews link from its ID.

### IF threshold
If the top story has **more than 500 points**, a "Hot Brief" goes to Discord (top story, points, top-level comment count, and the top 5 list). Otherwise a shorter "Quiet Day" message goes out. I chose 500 because the top story on a normal day is usually near or above it, so both branches stay meaningful. Changing it means editing one number.

### Error handling
Both HTTP Request nodes use **On Error: Continue (using error output)**. A failed call does not crash the workflow silently:
- Search API fails > Error output > Discord node "Alert: Search API Failed".
- Details API fails > Error output > Discord node "Alert: Details API Failed".

Each alert says which API failed and includes the error details (first 300 characters). I tested this by changing the first URL to a wrong path; the workflow stopped and the alert arrived in Discord (see `screenshots/task2-error-test.png`).

### Credentials
The Discord webhook URL is stored in n8n's **Credentials store** ("Discord Webhook account"), not in any node. The exported JSON contains only the credential's name and ID. I searched the file for the webhook URL before committing and found nothing.

### Screenshots
- `task2-a-workflow-canvas.png`: workflow canvas
- `task2-b-execution-output.png`: output of the Code node (top 5 stories)
- `task2-error-test.png`: error path test with the Discord alert

### How to run
Import `Task2_Workflow_ShashwatGupta.json` into n8n, create a "Discord Webhook" credential with your own webhook URL, select it on the Discord nodes, and click Execute.