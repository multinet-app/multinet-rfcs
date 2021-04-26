---
RFC: 0008
Author: Roni Choudhury
Created: 2020-08-04
Last-Modified: 2020-08-19
Status: accepted
---

# RFC 0008: Multipart Table Upload

This RFC describes how large table data can be safely and efficiently uploaded
to Multinet, using a variation of the common multipart upload strategy common to
many web server technologies.

## Motivation

Multipart upload is needed for the following reasons:

1. As a practical matter, our hosting platform Heroku does not allow for
   requests that take longer than thirty seconds to complete. Some moderately
   sized CSV files already break this threshold, and so called "big data" tables
   most certainly will. In order to upload such data files, we will need a way
   to upload them in chunks.
2. A long-running one-shot upload does not give the end user much feedback on
   what is going on. Multipart upload would allow us to provide a more detailed
   progress meter that can improve the user experience.
3. If a long-running upload is interrupted for any reason, a multipart upload
   strategy allows for resuming such uploads without losing the time already
   spent in uploading one more initial chunks of the file.

## Proposals

### Summary of Approach

The basic idea is as follows: the client initiates a new upload session for a
given file; this action causes the server to create a new collection in the
`uploads` ArangoDB database. Then the client issues one or more "chunk" uploads,
each sending a range of bytes, up to some size limit, to the server. The server
saves each chunk as a document in the database collection associated with this
file upload.  Finally, the client issues a "finalize" call, which marks the
database collection as being complete.

The server can then ask for a file-like object associated with the upload. The
server can then use this object to perform validation, retrieve table rows, and
settle them in their final home in the databse (as is done currently, directly
with the file contents) as a Multinet table in a workspace.

Once the work is completed, the server can delete the upload collection from the
database.

### API design

This RFC adds an `upload` section to the Multinet API, which contains the
following endpoints:

- `POST /api/uploads` - creates a new upload object by selecting a random,
  non-preexisting UUID, then creating an Arango collection keyed by the string.
  This endpoint returns the ID string.
- `POST /api/uploads/<upload-id>/chunk?sequence=<sequence-number>` - uploads one
  chunk of data to the named upload object. This endpoint returns the sequence
  number, and may raise various status codes to indicate problems.
- `POST /api/uploads/<upload-id>/finalize?uploader=<uploader-name>&...` -
  finalizes the IDed upload by preventing further calls to `POST
  /api/uploads/<upload-id>/chunk`, performing a checksum on the settled data,
  and invoking an uploader.
- `DELETE /api/uploads/<upload-id>` - removes the database collection associated
  with the given ID.

### File-like interface to upload collection

A Python class named `UploadReader` which implements the file-like interface
for reading data from an upload collection will be used by the serverside to
interact with the data uploaded to the database using the `/api/uploads`
endpoints. It should be constructed with a string ID referring to an upload, and
it should implement the `seek()`, `read()`, and `readline()` methods as though
the various documents in the collection constitute a regular file as opened by
Python's `open()` function.

### Usage

Currently, file uploads are done by file type using the `uploaders` API section.
There, a file of the specified type is sent to a specific `POST
/api/uploaders/<file-type>` endpoint, which in turn handles the full file data
including validation and ingestion to an Arango database and collection.

Under this RFC, file upload becomes detached from file ingestion. Now, the
correct sequence of `/api/uploads` calls must be made in order to settle the
data in the database in an intermediate form.

Next, the client can invoke one of the uploader paths, which will in turn
instantiate an `UploadReader` object. This file-like object can be used to
perform validation and bulk ingestion into a Multinet data table proper. The
main advantage of this usage is that an `UploadReader` does not exhaust its data
iterator after one pass; it can be rewound by using the `.seek()` method. This
means that both validation and ingestion can be performed without a memory
blowup caused by exhausting an HTTP request body iterator into a string in
memory.

Finally, the client (or the uploader methods) can invoke the `DELETE` endpoint
to remove the intermediate upload data, leaving the server in a clean state.

## Future Work

This section tracks some ideas that will be fleshed out in future RFCs. They are
not critical to the core feature of multipart upload, and are thus listed here
with fewer details.

### Resumable Uploads

One advantage to this approach is that it enables *resumable uploads*. Since a
single chunk is now the atomic unit of uploading, we can design the
upload infrastructure to track which chunks of a file have been uploaded and
which ones remain. Then, a client can use this data to resume an interrupted
upload in the right state.

Metadata such as the following would be useful in implementing this idea:

```javascript
{
  "id": <uuid>,
  "collection": <string>,  # Collection that stores the upload data
  "filename": <string>,    # The name of the uploading file client-side (only used as a hint to the client) 
  "size": <int>,           # The size of the total file in bytes (as reported by the client)
  "chunk_size": <int>,     # The size of each chunk in bytes
  "progress": {
    [chunk_seq: int]: true
  }
}
```

The `progress` value represents a set of completed chunk uploads that the client
can use to determine which chunks remain to be uploaded.

### Stale Upload Cleanup

If an upload is abandoned for any reason, we need a way to free the temporary
space that was used for that upload. This could be done via an admin-only
`vacuum` endpoint that filters the upload objects based on various criteria,
such as ones older than a certain age, for which no active chunk upload exists.
This is a database maintenance action, so we must decide on how it is to be
deployed and when and how it should be run.

## Backwards Compatibility

This RFC deprecates the existing uploader endpoints found in the blueprints for
the various file types Multinet supports. Instead of using those "one-shot"
endpoints, clients would instead need to follow the multi-stage approach
described above.
