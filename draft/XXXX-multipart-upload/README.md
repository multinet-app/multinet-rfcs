---
RFC: XXXX
Author: Roni Choudhury
Status: draft
Created: 2020-08-04
Last-Modified: 2020-08-04
---

# RFC XXXX: Multipart Table Upload

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
database. Then the client issues one or more "chunk" uploads, each sending a
range of bytes, up to some size limit, to the server. The server saves each
chunk as a document in the database collection associated with this file upload.
Finally, the client issues a "finalize" call, which marks the database
collection as being complete.

The server can then ask for a file-like object associated with the upload. The
server can then use this object to perform validation, retrieve table rows, and
settle them in their final home in the server (as is done currently, directly
with the file contents).

Once the work is completed, the server can delete the upload collection from the
database.

### API design

This RFC adds an `uploads` section to the Multinet API, which contains the
following endpoints:

- `POST /api/uploads` - creates a new upload object by selecting a random,
non-preexisting SHA string, then creating an Arango collection keyed by the
string. This endpoint returns the ID string.
- `POST /api/uploads/<upload-id>/chunk?sequence=<sequence-number>` - uploads one
chunk of data to the named upload object. This endpoint returns the sequence
number, and may raise various status codes to indicate problems.
- `POST /api/uploads/<upload-id>/finalize` - finalizes the IDed upload by
preventing further calls to `POST /api/uploads/<upload-id>/chunk` and performing
a checksum on the settled data.
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

## Questions

1. How do we detect upload interruptions? What is the right way to resume such
   an interrupted upload?
2. What do we do with "stale" upload objects? One answer is to have a `vaccum`
   endpoint that looks for upload objects older than a specific cutoff age and
   remove them, then run that endpoint periodically. Better still, it could be
   installed as a cronjob on the server directly.

## Backwards Compatibility

This RFC deprecates the existing uploader endpoints found in the blueprints for
the various file types Multinet supports. Instead of using those "one-shot"
endpoints, clients would instead need to follow the multi-stage approach
described above.
