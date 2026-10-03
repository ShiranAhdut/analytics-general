# `SUPERSONICSTUDIO.AGGREGATIONS.RMS_FULL_DATA`

Daily user-acquisition (UA) data per install cohort in Snowflake: revenue and users (`cohort` rows) plus spend (`uff` rows).

- **Size:** about 134B rows, roughly 100M per `EVENT_DATE`.
- **History:** event dates from 2023-01-08; updated daily.
- **Sibling table:** `RMS_FULL_DATA_ICEBERG` (about 376B rows). Its revenue and cost totals match this table, but it has more rows (more granular breakdowns) and is slower to query.

> The table has no column comments in Snowflake. The descriptions below are inferred from column names and data profiling (Oct 2026), not from official documentation. **Fill** = share of rows with a value, measured on `EVENT_DATE = 2026-10-01`. "Not checked" = fill wasn't measured.

## Usage notes

- **Always filter on `EVENT_DATE`** (and an app or game). Aggregate in the query; don't pull raw rows.
- **Use `INSTALL_DATE_CALC`, not `INSTALL_DATE`.** About 12.5% of cohort rows have `INSTALL_DATE = 1970-01-01`; the real date is in `INSTALL_DATE_CALC`, and `AGE` is calculated from it.
- **Revenue and cost are on different rows.** Revenue exists only on `DATA_SOURCE = 'cohort'` rows and cost/installs only on `'uff'` rows. For ROAS or CPI, aggregate across both sources. Only about 290 of ~10K apps have spend data.
- **Filter on `APP_ID` or `PRODUCT_CODE`, not `GAME_NAME`.** `GAME_NAME` is NULL on about 740K rows per day.
- **Campaign/ad breakdowns are often missing on cohort rows** (`CAMPAIGN_ID` NULL on 62%, `AD_ID` on 82%), so campaign-level revenue covers well under half the total.
- **Unverified:** cohorts are split into `REVENUE_TYPE` rows (`ads`, `iap`, NULL). Check whether `COHORT_USERS` repeats across these rows before summing it, to avoid double counting.

## Columns

### Dates and grain
| # | Column | Type | Description | Fill |
|---|---|---|---|---|
| 1 | `INSTALL_DATE` | DATE | Cohort install date. Set to 1970-01-01 on ~12.5% of cohort rows; use `INSTALL_DATE_CALC`. | all |
| 2 | `INSTALL_DATE_CALC` | DATE | Corrected install date; `AGE` is calculated from it. | all |
| 3 | `AGE` | NUMBER | Days since install (`EVENT_DATE − INSTALL_DATE_CALC`). | cohort only |
| 53 | `DOWNLOAD_DATE` | DATE | Probably the store download date. | empty |
| 62 | `EVENT_DATE` | DATE | The day the activity, revenue or spend happened. | all |
| 63 | `DATA_SOURCE` | TEXT | `cohort` = revenue and user rows; `uff` = spend rows (cost, installs, clicks). | all |

### App, game and platform
| # | Column | Type | Description | Fill |
|---|---|---|---|---|
| 5 | `APP_ID` | TEXT | Store app ID (bundle ID or Apple ID). The most reliable key. | ~all |
| 6 | `APP_NAME` | TEXT | App name. | not checked |
| 7 | `SW_GAME_ID` | TEXT | Supersonic internal game ID (UUID). | not checked |
| 8 | `GAME_NAME` | TEXT | Game name. NULL on ~740K rows/day. | partial |
| 9 | `PRODUCT_CODE` | TEXT | Short game code, e.g. `SNAK`. | not checked |
| 10 | `OS` | TEXT | `android` / `ios`. | not checked |
| 19 | `OS_VERSION` | TEXT | OS version. | `uff` only |
| 20 | `APP_VERSION` | TEXT | App version. | partial |
| 11 | `COUNTRY_CODE` | TEXT | 2-letter country code. | 99.9% |

### Attribution
| # | Column | Type | Description | Fill |
|---|---|---|---|---|
| 4 | `MEDIA_SOURCE` | TEXT | Ad network (AppsFlyer media source). NULL = organic or unattributed. | cohort 47%, `uff` 100% |
| 12–13 | `CAMPAIGN_ID`, `CAMPAIGN_NAME` | TEXT | UA campaign. | cohort 38%, `uff` 95% |
| 44–45 | `ADSET_ID`, `ADSET` | TEXT | Ad set or ad group. | not checked |
| 15–16 | `AD_ID`, `AD_NAME` | TEXT | Ad or creative. | cohort 18%, `uff` 93% |
| 17 | `AD_MIN_EVENT_DATE` | DATE | Probably the first date the ad had activity (its launch date). | not checked |
| 18 | `CHANNEL` | TEXT | Sub-channel within a network (e.g. `ACI_Youtube`, `Facebook`, `Instagram`, `TikTok`, `rv`). Not standardised: spelling varies (`REWARD`/`reward`, `VUNGLE`/`vungle`) and some values have trailing spaces. | ~12% cohort, ~13% `uff` |
| 14 | `CLIENT_ID` | NUMBER | Probably the client or account ID. | not checked |

### Revenue (cohort rows)
| # | Column | Type | Description |
|---|---|---|---|
| 21 | `REVENUE_TYPE` | TEXT | `ads` (~$267K/day), `iap` (in-app purchases, ~$91K/day), or NULL (rows with users but no revenue). |
| 24 | `DAILY_REV` | FLOAT | Net revenue on `EVENT_DATE`. |
| 25 | `CUM_REV` | FLOAT | Cumulative revenue from install up to `EVENT_DATE`. |
| 60–61 | `DAILY_GROSS_REV`, `CUM_GROSS_REV` | FLOAT | Gross versions, probably before store or platform fees. |
| 40 | `ORGANIC_REVENUE` | FLOAT | Revenue credited to organic users. |
| 41 | `UA_REVENUE` | FLOAT | Revenue credited to paid UA. |
| 42 | `UNKNOWN_REVENUE` | FLOAT | Revenue with no attribution. |
| 43 | `EXCLUDE_UNKNOWN_REVENUE` | FLOAT | Revenue excluding the unattributed part. |
| 22–23 | `ORGANIC_UPLIFT`, `UNKNOWN_UPLIFT` | FLOAT | Factors that credit paid UA with part of the organic or unattributed revenue. Set on 66% of cohort rows and all `uff` rows. |

### Users and purchases (cohort rows)
| # | Column | Type | Description |
|---|---|---|---|
| 26 | `COHORT_USERS` | NUMBER | Users installed on `INSTALL_DATE_CALC` (cohort size). |
| 27 | `APP_OPEN_USERS` | NUMBER | Cohort users active on `EVENT_DATE`; use for retention. |
| 28 | `PAYING_USERS` | NUMBER | Users who paid on `EVENT_DATE`. |
| 29–30 | `PURCHASES`, `CUM_PURCHASES` | NUMBER | Purchase count, daily and cumulative. |
| 31–32 | `FTD_USERS`, `CUM_FTD` | NUMBER | First-time depositors, i.e. new payers (daily and cumulative). |

### Spend (`uff` rows)
| # | Column | Type | Description |
|---|---|---|---|
| 35 | `COST` | FLOAT | Ad spend in USD. |
| 34 | `INSTALLS` | NUMBER | Paid installs reported by the network. |
| 33 | `CLICKS` | NUMBER | Clicks. |
| 36 | `IMPRESSIONS` | NUMBER | Impressions. |
| 37 | `BUDGET` | NUMBER | Campaign daily budget (~10% filled). |
| 38 | `TARGET` | FLOAT | Campaign bid or ROAS target (~2% filled). |
| 39 | `CONVERSIONS_VALUE` | NUMBER | Conversion value reported by the network (~2% filled). |

### LTV prediction and goals (mostly empty)
| # | Column | Type | Description | Fill |
|---|---|---|---|---|
| 46 | `STATIC_LTV` | FLOAT | Predicted lifetime value (LTV) per user, fixed model. | empty |
| 47–52 | `DYNAMIC_LTV_30/45/60/90/120/180` | FLOAT | Predicted LTV at day N. | empty on cohort rows; `uff` not checked |
| 54 | `MODEL_ID` | TEXT | LTV model ID. | empty |
| 55 | `ORIGINAL_BID` | FLOAT | Original campaign bid. | empty |
| 56 | `ROAS_GOAL` | FLOAT | Target ROAS. | empty |
| 57 | `ROAS_GOAL_ORGANIC` | FLOAT | Target ROAS including organic uplift. | not checked |
| 58 | `NEW_CPI` | FLOAT | Recommended cost per install (CPI) or bid. | empty |
| 59 | `PERFORMANCE_MARGIN_GOAL` | FLOAT | Target profit margin. | not checked |
