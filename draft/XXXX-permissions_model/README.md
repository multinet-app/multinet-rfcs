---
RFC: XXXX
Author: Roni Choudhury
Status: draft
Created: 2020-05-22
Last-Modified: 2020-07-01
---

# RFC XXXX: Multinet Permissions Model

This RFC lays out the permissions model used by Multinet. It is a simpler and
coarser grained model than many systems use, designed to offer a measure of data
protection and privacy while still allowing for quick implementation so the team
is able to focus more on research issues than a more complex permissions model
that may not bring major benefits as the project develops.

The model uses Role Based Access Control (RBAC) and Discretionary Access Control
(DAC) to define who can perform what operations on the data in the system. The
approach is coarse-grained because the permissions apply directly to whole
workspaces, rather than individual tables or networks.

## Motivation

Research work on the Multinet system occurs inside of workspaces. Within each
workspace, a given user has uniform access to all resources (i.e., networks and
tables). We have developed a simple role-based access control system that
balances ease of use for the end user, ease of implementation for the core team,
and a measure of data safety and privacy within the system as a whole.

After implementing this model, Multinet will be well suited for wide sharing
with the public, with minimal risk of damage to existing data.

## Proposals

### Permissions

The permissions available in Multinet are as follows:

- **Read**: this allows access to the GET API calls, enabling the knowledge of
  existence of and viewing of a workspace, and viewing of the tables and
  networks within that workspace.
- **Query**: this allows access to non-mutating AQL queries on a workspace
  (i.e., attempts to perform a mutating query under this permission will result
  in an error).
- **Write**: this allows access to various POST and PUT calls on tables,
  networks, and views within a workspace--that is, the ability to upload data to
  create new tables within a workspace, etc.
- **Remove**: this allows access to DELETE calls on tables, networks, and views
  within a workspace.
- **Delete**: this allows access to DELETE calls on workspaces themselves.
- **Grant**: this allows the assigning of roles (other than ownership) on a
  given workspace.
- **Transfer**: this allows transferring ownership to another user.

(There is one putative permission, **execute**, which does not currently
exist--this is the permission to execute mutating queries on a workspace.
Because the current Multinet data model is static, no such permission currently
exists in the system.)

### Roles

Roles are associated with users for each specific workspace. That is, given a
particular workspace *W* and a user *U*, *R(W, U)* gives the role of user *U*
with respect to workspace *W*. In turn, that role specifies what operations *U*
is thereby authorized to perform on *W*.

The roles are as follows:

- **Reader**: *U* may **read** and **query** *W*.
- **Writer**: *U* is a **reader** of *W*, and may also **write** and **remove**
  on *W*.
- **Maintainer**: *U* is a **writer** of *W*, and may also **grant** roles on
  *W* and **delete** *W*.
- **Owner**: *U* is a **maintainer** of *W*, and may also **transfer** ownership
  of *W*.

All roles originate with **ownership**: when *U* creates a new workspace *W*,
*U* immediately becomes the **owner** of *W*. *U* may now **grant** roles on W
to others, up to and including **maintainer**. *U* may also mark *W* as public,
in which case all users implicitly become **readers** of *W*. If *U*
**transfers** ownership to *V*, then *V* becomes the new **owner** of *W*, while
*U* loses all roles on *W*.

Any user who is logged in will be able to create a workspace (and thereby become
the owner of it).

### Implementation

Each API route will gain some gating logic to ensure that the user who is logged
in is allowed to perform the requested action based on the target workspace and
that user’s role within that workspace. For instance, when querying `GET /workspaces`,
the response will be filtered to only contain those workspaces for
which the user is a reader, or that are public workspaces. Similarly, `POST /csv`
calls will fail if the requesting user is not a writer of the workspace, etc.

## Reference Implementation

[PR #402](https://github.com/multinet-app/multinet-server/pull/402) and [PR #412](https://github.com/multinet-app/multinet-server/pull/412) constitute a
reference implementation for this feature.
