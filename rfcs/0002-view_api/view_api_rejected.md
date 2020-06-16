---
author: Roni Choudhury
date: 2020-06-11
---

# RFC: View API

This RFC describes a proposal for a Multinet View API. A "view" is essentially a
virtual table computed by an AQL query running against a workspace. It can be
used to explore data within the workspace, prepare database joins across tables,
or remix data in arbitrary ways. The API includes facilities for running AQL
queries, storing an object in the database that linked to one such query, and
"promoting" a view into a full-fledged table that can be consumed in the usual
way by Multinet.

This RFC is derived from a [Google Doc
proposal](https://docs.google.com/document/d/1C8VDA867jn6sLnLtOlsZAJVn7nGxg5lGq7OSYcPa848/edit).
This document lays out the evolution of thinking in arriving at the final
proposal, which is presented in the current RFC.

## Motivation

A View API is needed in order to provide fuller database-type functionality to
users of Multinet. In particular, without such a facility, the user would be
forced to write data processing logic to shuttle data between CSV files outside
of Multinet, always going through a lengthy upload-validate-visualize cycle
every time they want to update something about their tables. Furthermore, this
API is built around ArangoDB's AQL query language, which offers unique powers to
perform computations on the graph structures of the user's data, such as
pathfinding, graph traversal, and more.

The API can be used to perform the following tasks, among others:
- Stitch together two or more tables end to end by concatenating rows from each
  using a series of AQL loops
- "Widen" a dataset by creating a new table containing columns from two or more
  existing tables using AQL’s “RETURN” operator to construct the virtual rows
- Perform traditional database operations such as outer or inner join using AQL
  idioms for doing so
- Carry out other arbitrary operations expressible in AQL

Additionally, this mechanism can be used to quickly reorganize edge tables by
remixing the columns of existing tables, repurposing them as, e.g., the `_from` and
`_to` columns of other network configurations the user wishes to work with.

## Proposals

### Multinet View REST API

The API has the following endpoints:

- `GET /workspaces/<workspace>/views/<view>`: get metadata about a view (e.g.,
  the AQL query used to create it)
- `GET /workspaces/<workspace>/views/<view>/rows`: get the rows of a view (i.e.
  by executing the stored AQL query)
- `POST /workspaces/<workspace>/views`: create a new view in the named workspace
  from an AQL query
- `PUT /workspaces/<workspace>/views/<view>`: update the AQL query, name, etc.
  for a given view
- `DELETE /workspaces/<workspace>/views/<view>`: remove a view from the system

This API will be mediated by a similar (or the same) permissions model governing
workspaces and tables.

### Coordinated Update to the Table API

One last operation remains: the ability to turn a view into a table. This is
necessary due to the way ArangoDB works. Unfortunately, it is not possible to
animate a table using the results of an AQL query, thus creating a kind of
virtual table. In turn, ArangoDB *requires* tables for constructing graphs;
therefore, in order to use the results of a View as the basis for creating a
network in Multinet, it is necessary to first capture the concrete results of a
View as an actual table in the system.

The Table API will have the following new endpoint:

- `POST /workspaces/<workspace>/tables`: create a new table from a view
  (provided as a `viewId` query argument)

### UI for View API

The Multinet client will have UI for creating new views, listing the created
views, executing them, and editing them. This UI will reuse some concepts from
elsewhere in the client; in particular, the row display for tables can be used
to display rows of results from a view. Otherwise, the views themselves will be
treated much like tables and networks: the client will have affordances for
examining them, deleting them, etc. in a special list of the existing views.

The client can furthermore offer some ready-made options for creating a view by
implementing common database operations such as inner/outer join, *n*-hop
networks around a node in a network, etc. These will "compile" transparently to
an appropriate AQL query but will offer an optimized UX for these common
operations.
