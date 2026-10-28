# CustomerCare AI Voice Agent

An n8n customer-support backend that lets a voice assistant register complaints and retrieve their status through a JSON webhook, with Google Sheets as a lightweight complaint store.

**Portfolio scope:** this repository contains the nine-node n8n integration workflow. Speech recognition, speech generation, and the conversational AI/voice-provider configuration are external and are not included. The exported workflow does not contain an LLM node.

## What it does

- Accepts `POST` requests at the `customercare-voice` webhook path.
- Routes `create_complaint` and `check_status` requests to separate branches.
- Checks that a new complaint has a customer name, order number, and issue description.
- Stores complaints with a generated reference, `Open` status, creation date, and `Voice` channel.
- Returns JSON messages that a connected voice assistant can read back to the caller.

## Workflow

```text
POST webhook → Switch on action
  ├─ check_status → Read Google Sheets rows → Filter by complaint ID → JSON response
  └─ create_complaint → Validate required fields
                         ├─ valid → Append Google Sheets row → Success response
                         └─ invalid → Missing-fields response
```

This project demonstrates webhook integration, conditional routing, expression-based field mapping, input checks, and persistent storage across a conversational support flow.

## Repository contents

| File | Purpose |
| --- | --- |
| `workflows/customercare-voice.json` | Sanitized n8n workflow export |
| `examples/create-complaint.json` | Fictional complaint creation request |
| `examples/check-status.json` | Status request template |
| `examples/invalid-complaint.json` | Missing-fields test request |
| `SANITIZATION.md` | Privacy changes and verification scope |

## Setup

1. Import `workflows/customercare-voice.json` into your n8n instance. Use a version supporting the exported node versions listed below. The workflow is deliberately inactive.
2. Create a Google spreadsheet with a tab named `Voice_Complaints`. Add these exact column headers in its first row:

   ```text
   Complaint_ID | Customer_Name | Order_ID | Issue_Description | Status | Created_Date | Channel
   ```

3. In both **Get row(s) in sheet** and **Append row in sheet**, replace `YOUR_GOOGLE_SPREADSHEET_ID` with your spreadsheet ID. Select your own Google Sheets OAuth2 credential and confirm that it can access the spreadsheet. If needed, reselect the `Voice_Complaints` tab and refresh the column mapping.
4. Keep complaint IDs as text in the spreadsheet. Creation currently generates the ID with `Date.now().toString()` and uses the n8n workflow timezone for the creation date.
5. Open **Webhook**, start listening for a test event, and copy the test URL shown by your own n8n instance.
6. Send one of the example requests. For status lookup, first create a complaint and replace the example's placeholder with the returned `complaint_id`.
7. Connect your external voice agent's HTTP/tool action to this endpoint with the same request fields. After testing and adding appropriate access controls, activate/publish the workflow and use the production URL shown in n8n.

No credential, API key, account-specific endpoint, or real customer data is included here. Configure credentials inside n8n, not in this repository.

### Exported node versions

| Node type | Version |
| --- | --- |
| Webhook | 2.2 |
| Respond to Webhook | 1.5 |
| Google Sheets | 4.7 |
| Switch | 3.5 |
| If / Filter | 2.4 |

The original n8n application version was not recorded in the export; compatibility must be confirmed on import.

## Try it

From the repository folder, substitute the webhook URL shown in your n8n instance:

```sh
curl --request POST 'https://YOUR_N8N_HOST/webhook-test/customercare-voice' \
  --header 'Content-Type: application/json' \
  --data-binary @examples/create-complaint.json
```

On Windows PowerShell, use `curl.exe` and enter the command on one line. Test webhooks must be listening before each test as required by your instance.

### Request contract

| Action | Fields |
| --- | --- |
| `create_complaint` | Non-empty string values for `customer_name`, `order_id`, `issue_description` |
| `check_status` | `complaint_id`, preferably a string matching a stored reference |

Example creation response (illustrative; the ID changes on each request):

```json
{
  "success": true,
  "complaint_id": "1790000000000",
  "status": "Open",
  "message": "Your complaint has been registered successfully. Your complaint reference number is 1790000000000"
}
```

A matching status response includes `success`, `complaint_id`, `status`, and a message containing the stored issue description. The response expression also provides a not-found message with null ID/status; confirm that path in your own n8n runtime, including an empty sheet. Missing creation fields return `success: false` and a prompt to provide the required information. Inspect the JSON `success` field; the workflow does not set separate error HTTP status codes.

## Verification checklist

- Create a complaint and confirm exactly one row appears with `Open` status and `Voice` channel.
- Look up the returned complaint reference and confirm the status and issue match.
- Try an unknown reference and an empty sheet; verify that a not-found response is delivered.
- Send `examples/invalid-complaint.json`; confirm no row is added.
- Test from the external voice service after replacing its test URL with the production endpoint.

The repository preparation checked JSON parsing, all connection endpoints, preservation of all nine nodes, and removal of identified private references. No live n8n execution or Google Sheets access was performed during repository preparation.

## Current limitations and next steps

This is a portfolio prototype. Its logic has been preserved from the supplied export:

- Webhook authentication and complaint ownership checks are not configured. Add both before using real customer data: knowledge of a complaint ID alone should not grant access to its issue description.
- Unsupported actions have no explicit fallback response. Add a default route and structured error response.
- Required-field checks assume string inputs; add type checks, length limits, and validation for malformed requests.
- Status lookup reads sheet rows before filtering. Use a server-side lookup or database as volume grows.
- Timestamp IDs can collide under concurrent requests, and retries can create duplicate rows. Add unique IDs and idempotency handling.
- Google Sheets failures do not have a dedicated error-handling branch. Add error responses, monitoring, and retry policies.
- Voice-provider configuration and end-to-end voice behavior need separate setup and testing.

## بالمصري

المشروع ده بيربط المساعد الصوتي بخدمة العملاء عن طريق n8n. المساعد يقدر يبعت بيانات شكوى جديدة، فتتسجل في Google Sheets ويرجع للعميل رقم يتابع بيه. ولو العميل معاه رقم شكوى، يقدر يسأل عن حالتها. الموجود هنا هو الـ workflow اللي بينفّذ الطلبات؛ إعداد المساعد الصوتي نفسه بيتعمل بشكل منفصل.
