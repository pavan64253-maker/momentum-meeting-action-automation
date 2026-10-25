# Meeting Notes -> Action Items -> Reminders

Flow: Frontend (index.html) -> n8n Webhook -> Gemini (free tier) extracts summary/tasks/owners/deadlines -> Google Sheet + Gmail recap -> results shown on page.

## Folder
- `n8n/workflow.json`  : import into n8n (backend + AI agent)
- `frontend/index.html`: open in browser (no install needed)
- `sample/notes.txt`   : test notes
- `docker-compose.yml` : optional local n8n

## Setup
1. Start n8n: n8n.cloud (free trial) OR `docker compose up -d` -> http://localhost:5678
2. Workflows -> Import from file -> `n8n/workflow.json`
3. Node "AI Extract (Gemini)": create credential *Header Auth*, Name `x-goog-api-key`, Value = free key from https://aistudio.google.com/apikey
4. Use your existing Google Sheet headers: `Date, Meeting, Task, Owner, Deadline, Priority, Status, Email`. No new columns are required.
   In node "Save to Google Sheet" replace `PASTE_SHEET_ID` with your sheet ID and connect Google account
5. Connect Gmail in both "Send Recap Email" and "Send Owner Reminder". Recap email is skipped when the request has no recipient.
6. Toggle workflow **Active**, copy the **Production URL** of the Webhook node. It must contain `/webhook/meeting-actions`, not `/webhook-test/meeting-actions`.
7. Open `frontend/index.html`, paste the URL, paste notes, click "Extract action items". The task tracker reads saved rows from the `meeting-tasks` GET webhook after the workflow is imported and active.

## Secrets and credentials
- Never put Gemini API keys, Google OAuth tokens, service-account JSON, or n8n credential exports in this folder.
- Configure credentials inside n8n using its credential store. The workflow export intentionally contains placeholders instead of secrets.
- The frontend webhook URL is a public endpoint needed by the browser; it is not an API key. Rotate/deactivate the n8n webhook if it is exposed or no longer needed.

## Free model
Uses Gemini free tier (no card). Free limits are small but fine for testing. The workflow uses `gemini-3.8-flash`; if Google changes model availability, select an available model from AI Studio and update the HTTP Request node URL.
Other free options: Groq (free key, Llama models) or Ollama (fully local, needs a good laptop).

## Troubleshooting
- 404 from webhook: workflow not Active, or you used the Test URL instead of Production URL. The Test URL only works for one request after clicking **Execute workflow** in the n8n editor; use the Production URL for this page.
- 400/403 from Gemini: wrong key, or header name must be exactly x-goog-api-key.
- JSON parse error: re-run; JSON mode is forced. Very long transcripts may be cut by the token limit.
- Task tracker error: import the updated workflow, replace `PASTE_SHEET_ID`, connect Google Sheets, and activate the workflow. The tracker uses `GET /webhook/meeting-tasks`.
- Owner reminders: use the existing `Email` column for the recipient. The daily schedule runs at 09:00 in the configured timezone and reminds tasks due today or tomorrow. With the existing sheet layout, reminders repeat daily until the task is marked Completed.
- Node version warning after import: delete that node and add a fresh one.

## Extend
Notion instead of Sheets, Slack recap, reminder 1 day before deadline (Schedule Trigger), audio -> transcript (Whisper).
