# Check a job board for expired Greenhouse and Ashby listings

A daily inventory check helps a job board editor find stale vacancies before the next newsletter or feed refresh. Start with jobs that are live today. The checker can then distinguish a missing posting from a temporary feed failure and queue confirmed closures for human review. Your existing listing IDs appear in the output, ready to join to your inventory.

[Open the Expired Job Apply Link Checker](https://apify.com/changeable_peddler/expired-job-apply-link-checker) · [Pricing](https://apify.com/changeable_peddler/expired-job-apply-link-checker/pricing)

## First check

Open the [three-row sample CSV](sample-review.csv) to inspect the output without an account or a run. You can also choose **Free static example** in Apify. For a live check, select **Check my listings** and paste CSV with the columns listingId and applyUrl. Use direct employer posting links from job-boards.greenhouse.io, boards.greenhouse.io, or jobs.ashbyhq.com. Start with one real posting and set a $0.002 run maximum. The fee is $0.002 for each unique verified open or confirmed closed posting. Unknowns and duplicates have zero result fee. Each batch supports up to 1,000 rows from 50 boards.

| Result | Next step for the editor |
| --- | --- |
| open | Keep the listing. Its posting ID appears on the employer's board. |
| unknown | Keep the last confirmed value. Check the reason and revisit after at least 24 hours. |
| closed | Review the source link before removing the listing. A previously verified job has been missing from two complete board responses at least 24 hours apart. |

A job that was already missing at first import stays unknown until the checker has seen it live. A returning job can reopen. These checks establish presence on the published board. They do not submit applications.

## Daily review with Google Sheets and n8n

1. Create an Inventory tab with the headers listingId and applyUrl. Add your first few jobs. Create a separate Review tab with the headers listingId, applyUrl, checkedAt, status, lastConfirmed, action and reason.
2. Import the [n8n workflow](job-link-review.n8n.json). Select your Google Sheets credential, document and tab in both Sheets nodes. Select an HTTP Header Auth credential for Apify with header name Authorization and your own Bearer token stored in n8n's credential manager.
3. Execute manually once and inspect Review. The workflow caps a run at $0.002 times the supplied row count, up to $2. It neither removes source rows nor submits applications. A completed retry returns the saved result and skips a second review-sheet append.
4. After verifying your first run, activate the daily trigger in your own n8n instance. Give the checker at least 24 hours between complete missing observations. Check your n8n timezone. If a request times out, inspect the original Apify run before retrying. If the sheet write alone fails, resume from the failed write in n8n.

## Use with an AI assistant

Connect [Apify's MCP tools](https://mcp.apify.com/?tools=search-actors,fetch-actor-details,call-actor,get-actor-run,get-dataset-items) to your assistant. Ask it to look up **changeable_peddler/expired-job-apply-link-checker** and inspect the schema and pricing. Example request: “Check these two employer job URLs for my review sheet. Keep my listing IDs. Spend up to $0.004 and show me anything I should review.” Supply the two real posting URLs. The tool call below illustrates the input with placeholders for your listings.

```json
{
  "actor": "changeable_peddler/expired-job-apply-link-checker",
  "input": {
    "mode": "live",
    "listings": [
      {"listingId": "board-1", "applyUrl": "https://job-boards.greenhouse.io/example/jobs/123456"},
      {"listingId": "board-2", "applyUrl": "https://job-boards.greenhouse.io/example/jobs/123457"}
    ]
  },
  "callOptions": {"maxTotalChargeUsd": 0.004}
}
```

Use call-actor once, retain the run ID, then read its default dataset. Review unknowns separately. The qualified Actor can also be selected directly in an Apify MCP connection. [Apify MCP documentation](https://docs.apify.com/mcp)
