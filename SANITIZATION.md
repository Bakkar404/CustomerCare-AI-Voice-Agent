# Sanitization and verification

The published workflow is derived from the supplied export. Its business logic, node names, positions, expressions, and connections are preserved.

## Removed or replaced

- Removed the entire `pinData` object, including recorded request headers, IP addresses, host information, webhook URL, and sample execution data.
- Removed node credential objects, including credential IDs and account labels.
- Replaced both Google Sheets document references with `YOUR_GOOGLE_SPREADSHEET_ID` in ID mode.
- Replaced the private numeric sheet reference with `Voice_Complaints` in name mode; removed cached spreadsheet names and URLs.
- Removed top-level workflow ID, version ID, and metadata containing the instance ID.
- Removed the webhook node's `webhookId`.
- Set `active` to `false` so the imported workflow requires deliberate configuration and activation.

Internal node and condition IDs remain: these are structural identifiers, not credentials or deployment endpoints. Column schema IDs remain because they are the spreadsheet field names used by the workflow.

## Checks performed

- Parsed the sanitized file as JSON.
- Confirmed nine nodes and valid connection source/target names.
- Compared sanitized content against the removed private identifiers.
- Checked repository content for credential/token patterns and original private metadata.
- Used fictional data and explicit placeholders in the examples.

Static inspection is not an n8n import or execution test. Credentials, spreadsheet access, node-version compatibility, and end-to-end responses must be verified in the importing user's environment. The repository contains no execution history, and no live customer service was contacted during preparation.
