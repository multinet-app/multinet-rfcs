---
RFC: 0013
Author: Jack Wilburn
Created: 2021-03-10
Last-Modified: 2021-03-10
Status: accepted
---

# RFC 0013: Getting User Permissions

## Motivation

With our current API design, it's impossible for a client application (or a
user) to check if a user has access to write/read on a specific workspace. The
only currently supported permissions check is to check if a user is a maintainer,
and if they are, it allows the user to see all the permissions for every reader,
writer, maintainer, and the owner.

Given this limitation, it's impossible for us to disable the add network/table
buttons on the main client. We should add another endpoint or modify the existing
endpoint to allow an application to check if the user has permissions to write to/
read from the workspace without exposing the list of others who have access.

## Proposals

To allow client applications to access a users permissions on a specific workspaces,
we should add a new endpoint:

`GET /api/user/permission?workspace=<workspace>`

that returns just that users permissions for the workspace if they have any. For
public workspaces that a user doesn't have explicit permissions on, that'd be
reader. For private workspaces that a user doesn't have permission on, a 404 
Workspace Not Found (for security). Finally, for workspaces where a user has
explicit permissions (owner, maintainer, writer, reader) return just the permissions
level that the user has.

## Backwards Compatibility

Since this is a new addition, there should be no breaking changes or incompatibilities.
