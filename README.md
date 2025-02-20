# GA4 Data Export to BigQuery
## Stream events directly to BigQuery using the tag template

This template utilizes the BigQuery **Streaming Insert API** to send events directly to a **BigQuery** table.

Before streaming events, you must create a BigQuery table with the correct schema.<br>
Use the schema provided in `schema.txt` to ensure compatibility.

The schema is similar to the GA4 schema for main fields, user_properties, event_params, items.  It supports defining new events without requiring schema modifications in BigQuery.

<!-- Related blog post: -->