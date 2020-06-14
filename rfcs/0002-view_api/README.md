---
author: Roni Choudhury
date: 2020-06-11
---

# RFC: AQL-Based Table Creation

This RFC describes a proposal for adding API to create new tables from AQL
queries. This supercedes a [rejected RFC for a View API](view_api_rejected.md),
offering a simpler approach.

The proposal is to add a way to run an AQL query as the basis for data to
"upload" to a table, fulfilling the target use cases of the View API idea, and
providing an eventual basis for [virtual tables](https://docs.google.com/document/d/15WMH_FO10qDPtwu5A_uq4Ex2tWoOcVZsGmt1e-nNM3Q/edit)
in Multinet.

This RFC is derived ultimately from a [Google Doc proposal](https://docs.google.com/document/d/1C8VDA867jn6sLnLtOlsZAJVn7nGxg5lGq7OSYcPa848/edit).

## Motivation

Multinet needs a way to provide fuller database-type functionality to end users.
In particular, without such a facility, the user would be forced to write data
processing logic to shuttle data between CSV files outside of Multinet, always
going through a lengthy upload-validate-visualize cycle every time they want to
update something about their tables. Furthermore, this API is built around
ArangoDB's AQL query language, which offers unique powers to perform
computations on the graph structures of the user's data, such as pathfinding,
graph traversal, and more.

The AQL path for creating tables can be used to perform the following tasks,
among others:
- Stitch together two or more tables end to end by concatenating rows from each
  using a series of AQL loops
- "Widen" a dataset by creating a new table containing columns from two or more
  existing tables using AQL’s `RETURN` operator to construct the virtual rows
- Perform traditional database operations such as outer or inner join using AQL
  idioms for doing so
- Carry out other arbitrary operations expressible in AQL

Additionally, this mechanism can be used to quickly reorganize edge tables by
remixing the columns of existing tables, repurposing them as, e.g., the `_from` and
`_to` columns of other network configurations the user wishes to work with.

## Proposals

### API Endpoint for AQL-Based Table Creation

The API receives the following new endpoint:

- `POST /workspaces/<workspace>/tables`: create a new table from AQL text
  (provided in the request body)

Tables created using this endpoint will record the input AQL query as table
metadata.

This endpoint is somewhat parallel to the existing `POST
/workspaces/<workspace>/aql` endpoint. In fact, the UI for this new feature (see
below) will make use of both endpoints to provide a preview feature for
exploring AQL queries before creating the table.

The main difference is that the new endpoint will require the results of the AQL
query to undergo validation (as with data uploaded from, e.g., a CSV file)
before it can succeed, while the AQL endpoint itself has no such restriction.

Access to this API endpoint will be mediated by the `writer` role.

### UI/UX for AQL-Based Table Creation

The Multinet client will have UI for engaging the new functionality, combined
with a mechanism for previewing the data used for creating a new table. The UI
will consist of a text input for writing AQL, a feedback element that shows any
errors encountered upon running the query, a preview of the data coming back
from the query, and an option to create a new table from a previewed AQL query.

The client can furthermore offer some ready-made options for creating a table
from common database operations such as inner/outer join, *n*-hop networks
around a node in a network, etc. These will "compile" transparently to an
appropriate AQL query but will offer an optimized UX for these common
operations.
