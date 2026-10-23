# GA4 Data Export to BigQuery
## Stream events directly to BigQuery using the tag template

This template utilizes the BigQuery **Streaming Insert API** to send events directly to a **BigQuery** table.

Before streaming events, you must create a BigQuery table with the correct schema.<br>
Use the schema provided in `schema.txt` to ensure compatibility.

The schema is similar to the GA4 schema for main fields, user_properties, event_params, items.  It supports defining new events without requiring schema modifications in BigQuery.

### Files

| File | Purpose |
|---|---|
| `template.tpl` | The server-side GTM tag template – import it into the Templates section of the server container |
| `schema.txt` | Schema of the main events table |
| `error_logs_schema.txt` | Schema of the optional error logs table |
| `Authentication.txt` | How to grant the server container access to BigQuery |
| `metadata.yaml` | Community Template Gallery metadata |

### How items are written

Item keys that match a column in the `items` schema are written to that column; every other key goes to `item_params` as a key/value pair.

- Numeric columns are cast to the schema type before insert: `quantity` and `item_list_index` as INTEGER; `price`, `price_in_usd`, `item_revenue`, `item_revenue_in_usd`, `item_refund`, `item_refund_in_usd` as FLOAT. A value that is not a number (for example "abc") is written as NULL instead of failing the whole row. FLOAT columns keep the fractional part. A fractional value for an INTEGER column is not truncated: the column stays NULL and the original value is written to `item_params` as float_value.
- The GA4 item key `index` is written to the `item_list_index` column (as in the native GA4 export). If `item_list_index` is sent explicitly, it takes priority over `index`.

### Optional identifier fields

- **original_fpid**: Cookie value (FPID) variable. Written to the "original_fpid" event parameter, with the "FPIDx.y." prefix removed and URL-encoded characters decoded.
- **original_cid_ss**: Cookie value (_ga) variable. Written to the "original_cid_ss" event parameter.

### Editing event parameters in the tag

Similar to the GA4 tag in server-side GTM, event_params can be adjusted per tag (applies to event_params only, not to items or user_properties):

- **Event parameters to exclude** – parameter names that are not written. A name ending with * is a prefix match, so "x-ga-*" removes all technical x-ga- parameters.
- **Event parameters to add / edit** – name and value pairs. An existing parameter with the same name is replaced; rows with an empty name or value are ignored. Applied after the exclusions.

### Error logs

If **BigQuery Table Id For Error Logs** is filled in, a failed insert into the main table is logged there (schema in `error_logs_schema.txt`). The tag reports failure to GTM only if that fallback insert also fails.

### Changelog

#### Version 2
- Fix: **original_fpid** no longer depends on a hardcoded "FPID2.2." prefix. The prefix is now removed dynamically (everything up to and including the second dot), so other FPID versions such as "FPID2.3." are processed correctly. Wrapping quotes are removed as well.
- New optional field **original_cid_ss**: enter the Cookie value (_ga) variable to write the original _ga cookie value to the "original_cid_ss" event parameter.
- user_properties: values of an unexpected type are now written to string_value instead of being dropped.
- New optional tables **Event parameters to exclude** (supports a "*" prefix, e.g. "x-ga-*") and **Event parameters to add / edit**.
- Help texts of the project and table fields link to `Authentication.txt` and `schema.txt`.
- Items: `index` is now mapped to `item_list_index` (previously it was missing from the table or ended up in `item_params`).
- Items: numeric columns are cast to the schema type (INTEGER / FLOAT); non-numeric values become NULL. FLOAT columns keep the fractional part. A fractional value for an INTEGER column (quantity, item_list_index) is never truncated: the column is NULL and the original value is written to item_params as float_value.
- Fix: when the main insert failed, the tag called `gtmOnFailure` immediately after starting the fallback insert, so the result of the error log write was ignored and the tag always reported failure. It now reports the result of the fallback insert.
- No schema changes: the existing table works as is.

#### Version 1
- Initial version.

## Learn more

For a full walkthrough with screenshots, see our guide: [GA4 data export to BigQuery via Server-Side GTM](https://proanalytics.team/blog/export-data-ga4-from-ss-gtm-directly-to-bigquery?utm_source=github&utm_medium=referral&utm_campaign=ga4-to-bq&utm_content=readme-guide).

## About

This template was built by [ProAnalytics Team](https://proanalytics.team/?utm_source=github&utm_medium=referral&utm_campaign=ga4-to-bq&utm_content=readme-about-team), a web analytics agency, after we saw significant data loss with GA4's native BigQuery export on client projects.

We also run [ProAnalytics Academy](https://proanalytics.academy/?utm_source=github&utm_medium=referral&utm_campaign=ga4-to-bq&utm_content=readme-about-academy), with practical courses and webinars on GA4, GTM and BigQuery.

Questions or ideas for improvements? [Open an issue](https://github.com/proanalytics-team/proanalytics-ga4-to-bq/issues) or read more on our [blog](https://proanalytics.team/blog?utm_source=github&utm_medium=referral&utm_campaign=ga4-to-bq&utm_content=readme-about-blog).
