---
RFC: XXXX
Author: Jack Wilburn
Created: 2021-03-10
Last-Modified: 2021-03-10
---

# RFC XXXX: Getting User Permissions

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

To satisfy the requirements of this feature, I can conceive of two ways that the work
might be done. 

1. Modify the existing permissions endpoint, `/api/workspaces/{workspace}/permissions?user=<user>`, to accept a query string parameter to filter the permissions to just the information pertinent to that user.
2. Add a new endpoint, `/api/user/permission?workspace=<workspace>`, that returns just that users permissions for the workspace.

I'd like some suggestions here, since I'm not sure which would be best. I'll
update this section after a round of review.

The return should list which permission category the user falls into. That should be 
one of owner, maintainer, writer, reader. If the workspace is public we should return 
reader for all users except those with explicit permissions. For security reasons,
we should consider return "workspace not found" if the user doesn't have permissions
to view that workspace. This would stop a user from guessing valid workspace names,
in case we ever have a vulnerability allows reading from any valid workspace name.
Again, I'm open to feedback on whether this extra step is worth the time to implement.

## Backwards Compatibility

Since this is a new addition, there should be no breaking changes or incompatibilities.

## Reference Implementation

N/A.
