---
RFC: XXXX
Author: Jacob Nesbitt
Status: withdrawn
Created: 2020-07-08
Last-Modified: 2020-07-08
---

# RFC XXXX: Advanced and Robust Upload System

Uploading a large dataset can be both timeconsuming and problematic. This RFC proposes a method for handling uploads that both facilitates ease of use, as well as reliability and fault tolerance. Uploading data is one of the most important parts of this system, and as such should be robust and featured.

## Motivation

Our current system of dealing with uploads is very simple and unfeatured. We send the data through a single request, with no track of progress or anything. This exposes a couple of key issues:

* If the upload takes too long, the connection times out (due to a current gunicorn setting)
* If the connection is interrupted, the upload must start over from the beginning (very frustrating on large datasets)
* There's no progress/indication to the user. This can be implemented on the _client_ by simply tracking the HTTP request progress, but as a matter of exposing this functionality through our API, it does not exist.

## Proposals

### Chunked Uploads
Instead of dealing with a single request to upload data, this data could instead be sent in chunks, allowing for many smaller HTTP requests. This would allow the following things:
* Retain a timeout on requests, while also allowing large datasets to be uploaded with ease.
* Fault tolerance and upload pausing/resuming
* Progress tracking

This would be accomplished through the following additions.

### Hidden Tables/Networks
To allow for uploaded chunks to be stored somewhere, when starting an upload, the desired table/network would be created in a "hidden" mode. This would prevent the table from being returned by any table listing endpoints. This table should be assumed to be volatile, as data is still being uploaded to it in this state. Whether a table is hidden or not would be tracked by a `_meta` collection on each workspace. This would contain, among other things, the information of any hidden tables in that workspace. Each document in this collection will have a structure like so:

```
{
  collection: id,
  hidden: bool,
}
```

The `collection` key would point to the ID of the collection to which this document refers.

Other information could be added to this collection in subsequent RFCs, making this a general purpose collection metadata storage mechanism.

#### Alternate Approach
Instead of modifying the table/network listing endpoints to omit these "hidden" tables, these endpoints could instead be modified to return objects (dictionaries), which along with the table/networks names, would include if they are temporary (i.e. being actively uploaded to). The benefit to this would be that users wouldn't be able to accidentally create duplicate tables of the same name, as they could see all tables which may exist. However, this would allow for any user (who has access to the workspace) to see an in-progress table, which may contain garbage data, and is likely unvalidated. Furthermore, if an upload fails validation after the fact, it may once again disappear from the list of tables/networks, possibly causing more confusion.

### System-wide Upload Collection
To keep track of current uploads, a `_system` collection called `uploads` would be created, which would keep track of any ongoing uploads. This collection would store, among other things, the progress of the upload, the "hidden" table(s)/graph(s) to which it belongs, and the user who initiated the upload. The schema for each document in this collection would have a structure like so:

```
{
  current: int (bytes),
  total: int (bytes),
  done: bool,
  graph: bool (indicates that a graph is being uploaded),
  workspace: string,
  target: {
    tables: string[],
    graphs: string[],
  },
  user: user_id (sub value),
  _id: id (arangodb, used by clients to actually send chunks)
}
```

### Initializing an Upload
To initialize an upload, new endpoints will be needed, which will replace the existing upload endpoints. To initialize an upload, a request of the following format is made (endpoint for each upload format specified):

```
POST /api/upload/csv/{workspace}/{table}
POST /api/upload/d3_json/{workspace}/{graph}
POST /api/upload/nested_json/{workspace}/{graph}
POST /api/upload/newick/{workspace}/{graph}
```

These endpoints replaces the original upload endpoints for each data type we currently support. This returns a JSON response, containing the upload information, in the format referenced above.

A new endpoint for uploading data chunks themselves will have the following format (endpoint for each upload format specified):

```
PUT /api/upload/csv/{workspace}/{table}
PUT /api/upload/d3_json/{workspace}/{graph}
PUT /api/upload/nested_json/{workspace}/{graph}
PUT /api/upload/newick/{workspace}/{graph}

Body: A chunk of the original data

Query Parameters: {
  upload: id (returned by the first call),
}
```

The `upload` query parameter is part of the information returned by the first call, and is used to denote the upload object to which the sent chunk of data belongs to. Since an upload is tied to a specific user, this endpoint is gated behind user login. Furthermore, a `PUT` request (by a logged in user) to an upload endpoint targeting an upload that does not belong to this user, should return `403 Forbidden`.


New endpoints for getting upload information are also required. These would be of the following formats:

```
GET /api/upload/uploads
```

This endpoint is tied to the currently authenticated user, and returns all of the uploads which belong to the user. If the need for a global upload listing endpoint arises, this may have to be revised.


### Intermediate Processing of Chunks

Each data type uploader (csv, newick, etc.) will be responsible for specific logic regarding how chunked uploads are handled and stored in the "hidden" arangodb collection. Given a chunk of data, the uploader needs to be able to process/keep track the following information:

* How many rows/documents are fully contained in that chunk
* How to convert/transform a row/document from its original format, into the format required for arangodb storage
* The current position or "context" in the full document (in graphs for example, are we currently looking at nodes?, links?, etc.)

More than likely, a chunk will not often start/end exactly on the boundary between rows/documents. In this case, any "incomplete" rows/documents will need to stored in a specific location (perhaps on the upload document) until the data which completes that record arrives. Storing this data in memory is not an option, as compatibility across multiple workers must be ensured.

The approach above assumes that these chunks of data are being sent **in order**. If there is a need for sending these chunks out of order, more logic would be required to keep track of "incomplete" rows/documents, and ensuring that in the future, they are able to be completed properly.

#### Alternate Approach
Another way to handle chunked uploads would be to explicitly split chunks along the boundaries of rows/documents, instead of doing this on arbitrary bytes. This would reduce the logic of chunk processing for each uploader. However, this approach would add increased complexity to, as well as place the burden of data processing on, the client application. Specific processing for each data type would be required, in order to determine the boundaries for each row/document. Furthermore, these rows/documents may be (and in practice often are) individually small, which could increase the overhead of the upload system, decreasing the overall upload speed.


## Backwards Compatibility

This RFC introduces breaking changes in the following ways:

* All upload endpoints are now at a different URL, and operate in an entirely different fashion
