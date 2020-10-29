---
RFC: XXXX
Author: Jacob Nesbitt
Status: draft
Created: 2020-10-29
Last-Modified: 2020-10-29
---

# RFC XXXX: Table/Graph Metadata

Currently, Multinet doesn't store any metadata about tables (or graphs), chiefly type information. This RFC outlines a way to specify, store and retrieve not just type information, but generalized metadata about a table/graph.

## Motivation

The main motivation for this is storing type information for each column in a table. We now have at least two use cases for storing this information.

1. Basic computation/comparison. If we attempt to use these tables for creating graphs and deriving useful data, it's necessary that we store type information, so that we are able to properly compare and compute certain values. For example, if you have a column of numbers, but we store them as strings (the current behavior), then if you try to do comparisons of these column values between different entries, you either will be doing incorrect string comparisons, or will simply not be able to do the comparisons/computations you need to.

2. Providing this information to other applications, so that they do not have to do this data inspection themselves, or if this information is required by those applications. Specifically, [UpSet](https://github.com/VCG/upset) is the first application for which this will be used.

## Proposals

#### Data Model
There are several way we could go about this, but in my opinion, the cleanest way is to store this information in a special collection, in the workspace that the table resides in. This special collection would be named something like `multinet_meta`, and each entry would correspond to the metadata on a specific graph or table. I think the data model would look something like this:


```
{
  item_id: str                 // The ID of the table/graph this data belongs to
  item_type: "table" | "graph" // The major type of the item
  table?: {                    // Only present if the major type is "table"
    type: "edge" | "node"      // The table type
    columns: [
      {
        key: str               // E.g. {"foo": "bar"}, "foo" is the key
        type: "number" | "label" | "category" | "date" | "bool"
      }
    ]
  }
  graph?: {                    // Only present if the major type is "graph"
    // TBD
  }

}
```

#### API
There are two main operations that are necessary for this functionality.

1. Specifying the metadata on table creation
2. Retrieving this metadata

Ideally, our API would allow for the creation of a table/graph record, with more versatile options for specifying metadata. However, our API isn't currently structured this way. Instead, in order to conform to our existing API, we would likely just add this metadata object as a query parameter onto each upload endpoint. For retrieving this metadata, the API should resemble the following:

```
GET /api/workspaces/{workspace}/tables/{table}/metadata
```

##### Specifying Metadata on Table Creation

The data model of the API would differ slightly from that of what we store internally, to allow for a single request to describe metadata on multiple entities (mainly in the case of graphs. The data format of this API may resemble the following:

```
{
  graph?: {
    // TBD, any relevant graph metadata
  }
  tables: [  // For a single CSV, this array must contain only one item.
    {
      // Unchanged from the table field above
    }
  ]
}
```

However, handling the case of graphs may not be necessary in the first approach. Feedback on how to best implement this is welcome.

## Backwards Compatibility

All changes may be backwards compatible to the server, as this new query parameter would ideally be an "optional" parameter. However, if it's desired that this field be mandatory, then this breaks backwards compatibility for all upload endpoints.