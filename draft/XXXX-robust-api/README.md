---
RFC: XXXX
Author: Jacob Nesbitt
Created: 2020-12-12
Last-Modified: 2021-02-01
---

# RFC XXXX: Robust Upload System

This RFC outlines a robust way to handle table/graph creation and upload, allowing for arbitrarily large datasets to be uploaded in any of the supported formats.

## Motivation

As the size of our datasets has grown, our limited approach to handling data upload and table/graph creation has proved to be inefficient. The need has arose for an API that can handle larger data, with more operations. The proposals outlined below will make that possible.

# Proposal

## Upload
Currently, our upload system deals with upload, data processing, and table creation, all at once and in the same request. Since we deploy on heroku, and we need to keep our requests under 30 seconds (as we should regardless), this severely limits both the size of the data we can upload. In order to address this, we can make use of Amazon S3. The new upload process would follow this rough path:


### Create an upload

`POST /api/workspaces/{workspace}/uploads`

Create an upload, returning an `Upload` object that (among other things) will contain a list of pre-signed URLs, that are to be used for uploading each part of the file to S3. **Note:** This means that the amount of parts / size of each part is determined by the server. The parameters for this call will include the following

* `size` - The size of the file, in bytes. This is needed to determine the number of pre-signed URLs to create for this upload object.
* `dataType` - The data type of the file to upload, which must be one of the supported (or installed, if using a plugin system) data types. E.g. `csv`, `d3_json`, `nested_json`, `newick`, etc. If this parameter does not match one of the supported data types, a `4XX` error is thrown.
* `objectType` - The type of object being uploaded (table or graph).
* `metadata` - Any metadata that should be associated with the upload

The response will be an upload object, which will have the following structure

```
{
  "_id": string (uuid),
  "user": string (uuid),
  "started": string (datetime),
  "finished": string (datetime) | null,
  "dataType": string (must be one of supported data types),
  "objectType": "table" | "graph",
  "status": "pending" | "processing" | "finished",
  "uploadId": string (uuid),
  "uploadPartUrls": [
    "<signed url for first part>",
    "<signed url for second part>",
    ...
  ],
  "uploadPartTags": [ // Will be populated later
    null,
    null,
    ...
  ],
  "metadata": {
    // metadata like type information goes here
  }
}
```

**NOTE:** The ordering of the `uploadPartUrls` array is significant, as it represents the specific URL that is to be used to upload each corresponding part. These URLs may be called concurrently, but the payload for each URL must be the section of the file which corresponds to its index.


### Upload the individual parts

This is a client side operation, and involves making an HTTP `PUT` request to each URL in the `uploadPartUrls` array. The payload for each of these requests is the section of the file data which corresponds to the index of the signed URL in the array (i.e. if the file is 100MB total, and the `uploadPartUrls` array contains 5 entries, the payload for the first request will be the bytes 1 - 20,000, the payload for the second request will be bytes 20,001 - 40,000, etc.). The response for each `PUT` call is an `ETag`, which must be used in a subsequent request to convey the status to the server. The structure is shown below.

```
POST /api/workspaces/{workspace}/uploads/{id}/finalizePart

{
  "partNumber": integer,
  "ETag": string
}
```

Each request populates its `ETag` field into the `uploadPartTags` field on the upload document, replacing the value of `null` at its index of `partNumber`.

### Finalize the upload

`POST /api/workspaces/{workspace}/uploads/{id}/finalize`

This is the last step that the client is responsible for. This instructs the server to mark the upload as finished. At this point, the sever downloads the reconstructed file from S3, and performs data processing, table/graph creation, etc. with the appropriate processor for that data type. This HTTP request should initiate an asynchronous (celery) task to process the data.

## Additional Changes

There are changes in addition to the endpoints mentioned above, which may not be crucial to the client, but should be detailed.

### Upload Collection

Each workspace should contain its own upload collection, which will be used to store each upload document shown above. This will need to be accompanied by a new `Upload` ORM, which will facilitate operations on uploads.

### Additional Upload Endpoints

`GET /api/workspaces/{workspace}/uploads/{id}`

This endpoint returns the Upload document with the corresponding ID.


`GET /api/user/uploads`

This endpoint returns all in-progress uploads that the user has created. A parameter may be included that will additionally include finished uploads in the response.

### Changes to data processing procedure

The notion of having various file type "uploaders" will be removed. Instead, these will become data processors, which will take raw uploaded data (from S3), and run validation, table/graph creation, data insertion, etc. When downloading the finalized upload from S3, this processing should be done in a streaming manner, in order to limit the risk of running out of memory in the server. This will involve updating each data processor to handle streamed data, instead of the entire file. This could be done in the following manner:

1. Read in data until a fully formed object can be parsed (a table row, for example).
2. Run validation on this object
3. If validation is passed, insert this object into the target location. If validation fails, save the validation error, and either continue processing more data, or halt data processing.
4. Repeat steps 1-3 until all data has been processed
5. If there are errors resulting from this processing, delete the partially formed tables/graphs, and return the validation errors to the user.

These errors would likely be stored in the upload model (yet to be outlined), however, it may make sense to introduce another model that would track this status more specifically. Feedback is welcome here.


## Backwards Compatibility

This RFC is not backwards compatible with the existing API, as the structure, upload procedure, etc. are all breaking changes.

## Reference
* [Amazon multi-part upload docs](https://docs.aws.amazon.com/AmazonS3/latest/dev/mpuoverview.html)
* [Blog that walks through the upload process](https://www.altostra.com/blog/multipart-uploads-with-s3-presigned-url)
