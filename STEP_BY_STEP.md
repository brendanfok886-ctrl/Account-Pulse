# AccountPulse — Step-by-Step Setup and Test Guide

This guide takes you from a fresh n8n workspace to a verified AccountPulse report. No OpenAI or Meta Ads API credential is required.

## 1. Download the project files

Download these files from the repository:

- [`AccountPulse_MVP_GitHub_Public.json`](./AccountPulse_MVP_GitHub_Public.json) — n8n workflow
- [`AccountPulse_Meta_Ads_Test.csv`](./AccountPulse_Meta_Ads_Test.csv) — purpose-built test data

## 2. Import the workflow into n8n

1. Sign in to n8n Cloud or open your self-hosted n8n instance.
2. Create a new workflow.
3. Open the workflow menu and select **Import from File**.
4. Choose `AccountPulse_MVP_GitHub_Public.json`.
5. Confirm that both starting paths appear:
   - **Run AccountPulse Demo** — no file required
   - **Upload Meta Ads CSV** — accepts a CSV through an n8n form
6. Save the workflow.

No credentials should be requested. The optional AI narrative node is disabled.

## 3. Run the built-in demo

Use this path first because it verifies the analytics and rule engine without requiring a CSV upload.

1. Select the **Run AccountPulse Demo** manual-trigger node.
2. Click **Execute Workflow**.
3. Wait until the execution reaches **Display Analysis Report**.
4. Open the final output to view the generated HTML report.

The purpose-built demo should demonstrate:

- An **underspending risk**
- A **CPA deterioration** alert
- A **creative performance deterioration** alert
- A **controlled scaling opportunity**
- An **Insufficient Data** state for a low-volume campaign

If the workflow completes and these states appear, the deterministic calculation and rule paths are working.

## 4. Test the CSV upload path

### Test mode

1. Open **Upload Meta Ads CSV**.
2. Click **Test step** or **Execute step** to open the test form.
3. Upload `AccountPulse_Meta_Ads_Test.csv`.
4. Enter the following values:

| Field | Test value |
|---|---:|
| Monthly Budget | 50000 |
| Target CPA | 250 |
| Analysis Date | 2026-08-18 |
| Currency | HKD |

5. Submit the form.
6. The browser should display the AccountPulse HTML report when the workflow finishes.

### Production form

1. Publish or activate the workflow in n8n.
2. Open the production URL shown by **Upload Meta Ads CSV**.
3. Upload a compatible CSV and provide the account context.
4. Submit the form to generate the report.

Only share the production form when its access and sample-data settings are appropriate for your environment.

## 5. Verify the expected results

With the supplied test CSV and the values above, check the following:

| Check | Expected result |
|---|---|
| Budget pacing | Underspending risk: projected spend is below 90% of HKD 50,000 |
| CPA deterioration | Triggered for `CPA Decline Campaign` |
| Creative deterioration | Triggered for `Creative Deterioration Campaign` |
| Scaling opportunity | Triggered for `Growth Prospecting` |
| Low-volume campaign | Insufficient Data instead of an alert |

### Manual CPA verification

The workflow compares two non-overlapping seven-day windows and calculates each window's CPA from aggregated totals:

```text
CPA = Total Spend / Total Conversions

CPA Change =
(Recent CPA - Previous CPA) / Previous CPA
```

Sample checks:

| Campaign and window | Spend | Conversions | CPA |
|---|---:|---:|---:|
| Growth Prospecting — recent 7 days | HKD 4,200 | 28 | HKD 150 |
| Growth Prospecting — previous 7 days | HKD 4,200 | 21 | HKD 200 |
| CPA Decline Campaign — recent 7 days | HKD 3,780 | 21 | HKD 180 |
| CPA Decline Campaign — previous 7 days | HKD 2,800 | 28 | HKD 100 |

Both comparison windows require at least 15 conversions before the CPA-deterioration rule can fire.

## 6. Test the overspending guardrail

Run the upload test again with:

| Field | Test value |
|---|---:|
| Monthly Budget | 30000 |
| Target CPA | 250 |
| Analysis Date | 2026-08-18 |
| Currency | HKD |

Expected outcome:

- Budget pacing changes to **overspending risk**.
- Controlled scaling is blocked because projected spend exceeds 110% of budget.

This confirms that an efficient campaign is not recommended for scaling when the account is already forecast to overspend.

## 7. Use your own Meta Ads CSV

Your file must contain these required fields, or compatible English Meta Ads column aliases:

```text
date
campaign_name
spend
impressions
clicks
conversions
```

Optional fields:

```text
revenue
ad_name
```

For a meaningful comparison:

- Include at least 14 days of data for the two seven-day windows.
- Include data from the start of the month for a reliable month-end pacing estimate.
- Include `ad_name` if you want ad-level creative deterioration checks.
- Use the account's actual monthly budget and target CPA.
- Set the analysis date to the latest complete date represented in the export.

## 8. Understand the forecast

AccountPulse uses transparent linear pacing:

```text
Projected Month-End Spend =
Month-to-Date Spend / Elapsed Calendar Days
× Total Calendar Days in Month
```

It is an explainable pacing estimate, not a machine-learning forecast. It does not account for:

- Day-of-week spending patterns
- Seasonality or promotional events
- Scheduled budget changes
- Future campaign launches or pauses
- Learning-phase effects

If the CSV does not provide adequate date coverage, the report should show **Insufficient Data for Reliable Forecast** rather than false precision.

## 9. Troubleshooting

### The form does not open

- In test mode, start the form trigger by clicking **Test step** or **Execute step** first.
- For a reusable production URL, publish or activate the workflow.

### Missing required columns

Check that the CSV contains date, campaign, spend, impressions, clicks and conversions fields. The validation output lists missing fields explicitly.

### Insufficient Data appears

This is an intentional safeguard, not an error. Common reasons include:

- Fewer than 15 conversions in either CPA comparison window
- Fewer than 5,000 impressions or 50 clicks in a creative comparison window
- Fewer than 7 days or incomplete month-to-date coverage for pacing
- Missing ad-level names for creative analysis

### CPA or ROAS shows N/A

- CPA requires conversions greater than zero.
- ROAS requires spend greater than zero and the optional revenue column.
- AccountPulse returns `N/A` instead of `Infinity` or `NaN` when a denominator is zero.

### The workflow requests an AI credential

The public MVP should not require one. Confirm that **Optional AI Narrative — Phase 2** remains disabled and that no model node has been connected.

## 10. Safety boundary

AccountPulse is a decision-support tool:

- It does not connect to the live Meta Ads API.
- It does not modify campaigns or budgets.
- Every triggered finding includes evidence and thresholds.
- Any budget change requires human review and approval.

After both test scenarios pass, the project is ready to demonstrate from the built-in demo or a sanitized CSV export.
