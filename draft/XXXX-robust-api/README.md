---
RFC: XXXX
Author: Jacob Nesbitt
Created: 2020-06-29
Last-Modified: 2020-12-12
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

* `dataType` - The data type of the file to upload, which must be one of the supported (or installed, if using a plugin system) data types. E.g. `csv`, `d3_json`, `nested_json`, `newick`, etc. If this parameter does not match one of the supported data types, a `4XX` error is thrown.
* `size` - The size of the file, in bytes. This is needed to determine the number of pre-signed URLs to create for this upload object.

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
  "uploadPartTags": [ // This will be an empty list initially
    "<ETag from the first uploaded part>",
    "<ETag from the second uploaded part>",
    ...
  ],
  "metadata": {
    // metadata like type information goes here
  }
}
```

**NOTE:** The ordering of the `uploadPartUrls` array is significant, as it represents the specific URL that is to be used to upload each corresponding part. These URLs may be called concurrently, but the payload for each URL must be the section of the file which corresponds to its index.


### Upload the individual parts

This is a client side operation, and involves making an HTTP `PUT` request to each URL in the `uploadPartUrls` array. The payload for each of these requests is the section of the file data which corresponds to the index of the signed URL in the array (i.e. if the file is 100MB total, and the `uploadPartUrls` array contains 5 entries, the payload for the first request will be the bytes 1 - 20,000, the payload for the second request will be bytes 20,001 - 40,000, etc.). The response for each `PUT` call is an `ETag`, which must be recorded respective to the part index and used in the finalization call later.


### Finalize the upload

`POST /api/workspaces/{workspace}/upload/finalize`

This is the last step that the client is responsible for. This instructs the server to finish the upload. The payload for the request must contain the following:

```
{
  "uploadPartTags": [
    <ETag returned from the first signed url>,
    <ETag returned from the second signed url>,
    ...
  ]
}
```

**NOTE:** Similar to the `uploadPartUrls` array mentioned in the Upload creation step, the ordering of the `uploadPartTags` array is significant.

At this point, the sever downloads the reconstructed file from S3, and performs data processing, table/graph creation, etc. with the appropriate processor for that data type. This process doesn't technically need to be done asynchronously, as if the data is small enough it's feasible that this could be completed within the time frame of a single request. However, in the interest of scalability and robustness, this HTTP request should initiate an asynchronous task to process the data.

## Additional Changes

There are changes in addition to the endpoints mentioned above, which may not be crucial to the client, but should be detailed.

### Upload Collection

Each workspace should contain its own upload collection, which will be used to store each upload document shown above. This will need to be accompanied by a new `Upload` ORM, which will facilitate operations on uploads.

### Additional Upload Endpoints

`GET /api/workspaces/{workspace}/uploads/{id}`

This endpoint returns the Upload document with the corresponding ID.

## Open Questions

### Changes to data processing procedure

The notion of having various file type "uploaders" will be removed. Instead, these will become data processors, which will take raw uploaded data (from S3), and run validation, table/graph creation, data insertion, etc. When downloading the finalized upload from S3, this processing should be done in a streaming manner, in order to limit the risk of running out of memory in the server. This will involve updating each data processor to handle streamed data, instead of the entire file. This could be done in the following manner:

1. Read in data until a fully formed object can be parsed (a table row, for example).
2. Run validation on this object
3. If validation is passed, insert this object into the target location. If validation fails, save the validation error, and either continue processing more data, or halt data processing.
4. Repeat steps 1-3 until all data has been processed
5. If there are errors resulting from this processing, delete the partially formed tables/graphs, and return the validation errors to the user.

These errors would likely be stored in the upload model (yet to be outlined), however, it may make sense to introduce another model that would track this status more specifically. Feedback is welcome here.


### Tracking upload progress

The current proposal requires that the client upload all parts, and then returns the ETags of these parts all at once when finalizing the upload. However, this poses a couple of possible pitfalls:

1. There is no tracking of upload progress on the server
2. If it's desired to allow for upload pausing and resuming (assuming the page is totally refreshed and all context is lost, or the computer is changed, etc.), there is no way to do this, as the initially acquired ETags are now lost

This could be addressed by creating endpoints for finalizing an individual part upload, as well as the overall finalize call. In this case, the procedure would then require the client to call some endpoint with the following signature:

```
POST /api/workspaces/{workspace}/upload/{id}/finalizePart

{
  "partNumber": integer,
  "ETag": string
}
```

This would add to the `uploadPartTags` array on the upload document, which would allow for tracking the progress of this upload. After all file parts are uploaded, the upload finalize call is made as usual, but in this new approach, it would not contain any payload.


Feedback on this suggested feature is appreciated.

## Backwards Compatibility

This RFC is not backwards compatible with the existing API, as the structure, upload procedure, etc. are all breaking changes.

## Reference
* [Amazon multi-part upload docs](https://docs.aws.amazon.com/AmazonS3/latest/dev/mpuoverview.html)
* [Blog that walks through the upload process](https://www.altostra.com/blog/multipart-uploads-with-s3-presigned-url)
