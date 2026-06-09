---
locale: en
title: Claude test article
summary: A test article created via the Fin-as-Code GitHub sync to verify the create flow.
author_id: 10804128
published: false
chatbot_availability: true
copilot_availability: true
sales_agent_availability: true
with_table_of_contents: false
---

# Claude test article

This article was created from git to exercise the Fin-as-Code sync flow.

**Commit 2 edit:** this paragraph was added in a second commit, before the
outbound sync echoed the assigned `id` back into `meta.yaml`. If the inbound
ApplyService drops this edit on the default branch, that is the `identity_skip`
(`id_mismatch`) race we are testing.

## What this verifies

- A new article folder under a collection is picked up by the connector.
- A blank `id` triggers entity creation on merge.
- An edit made inside the pre-echo window resolves identity correctly.
- Standard CommonMark renders as expected.

:::callout
This is a test article. It is published as a draft (`published: false`) so it stays out of the live help center.
:::
