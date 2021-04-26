---
RFC: 0012
Author: Roni Choudhury
Created: 2021-01-26
Last-Modified: 2021-01-26
Status: accepted
---

# RFC 0012: PUT endpoint for appending/modifying data in a table

This RFC proposes a new `PUT` endpoint for table data, enabling appending of new
rows, and modification of rows already in the table.

## Motivation

Up until now, Multinet's data model has been *static*; that is, once data is
uploaded to the system, it never changes. This property has some nice effects on
system design, especially for envisioned (but, as of now non-existent) features
such as canned views of specific tables, or of certain network operations based
on static underlying table data.

However, we now have a specific use case for which dynamic data is necessary.
Because the predicted benefits of a purely static data model never materialized,
we are relatively free to make this change now to address actual needs that have
arisen.

The use case is from the Marclab, where data collection is a long, ongoing
process. There are two modes of data update the lab needs:

1. Sometimes, new data is computed and the appropriate rows must simply be
   appended to an existing data table. This is the case of conceptually
   incomplete data following a gradual procedure of completion.
2. Other times, existing data must be updated due to new knowledge discovery.
   This is a case of conceptually incorrect data undergoing a process of
   correction.

Finally, there is a third ancillary use case solved by the `PUT` endpoint: our
infrastructure can currently only support a maximum time of 30 seconds per
request, which is not long enough to support the uploading of very large
datasets (on the order of a 30MB CSV file). Though an in-flight RFC is
addressing this problem organically, the `PUT` endpoint nonetheless provides a
workaround: simply slice the CSV file into smaller tranches of data, then use
the `PUT` endpoint to append these tranches to reconstruct the whole table.

## Proposals

### `PUT` endpoint

We are proposing to add a new endpoint:

```
PUT /api/workspaces/{workspace}/tables/{table}/rows
```

the body of which will contain data formatted as "JSON lines". For each such
data row, the semantics are as follows:

- If the row's `_key` field matches the `_key` field of an existing row in the
  table, the database will perform an update operation, replacing the existing row
  with the new one.
- If the `_key` field does not match any row in the table, or if there is no
  `_key` field, then the database will append the row to the end of the table.

Put another way, the `PUT` endpoint represents a database "upsert" operation.

### `DELETE` endpoint

Though not strictly necessary for the use case detailed above, the `PUT`
endpoint does not enable a method for deletion of rows. To that end, a `DELETE`
endpoint is proposed as well:

```
DELETE /api/workspaces/{workspace}/tables/{table}/rows
```

Note that because a `DELETE` endpoint on `tables` already exists, with the usual
RESTFUL semantics of deleting an entire table, this endpoint must take a
different form. (See next subsection for a deeper discussion of the implications
hereof.)

The body of this request will be a collection of rows, formatted as "JSON
lines". For each row's `_key` field, that row in the table will be deleted.

### RESTFUL structure

The conflict between the two kinds of `DELETE` endpoints raises an issue with
the API: technically, the `GET /api/workspaces/{workspace}/tables/{table}`
endpoint should *not* return data, but rather information about the table
itself. In parallel with the `DELETE` endpoints, there should be a separate `GET
/api/workspaces/{workspace}/tables/{table}/rows` endpoint that behaves as the
currently existing `GET` endpoint does. A further proposal made here is to
perform this change for the sake of API consistency.

## Backwards Compatibility

The third proposal to change the route for the "get rows" endpoint is a backward
incompatibility. The severity of the change is low, provided the multinetjs
library and the client applications all update their code to continue working.
