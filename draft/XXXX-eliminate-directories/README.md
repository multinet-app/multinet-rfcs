---
RFC: XXXX
Author: Roni Choudhury
Created: 2021-01-26
Last-Modified: 2021-01-26
---

# RFC XXXX: Eliminate directory structure in RFC repository

This proposes to restructure the RFC repository to eliminate the proposal-status
focused directory structure. This will simplify the workflow and has a number of
other benefits. An additional observation: the proposed method of running this
RFC repository will bring it closer to how the [Python PEP
repository](https://github.com/python/peps) works.

## Motivation

The current process for creating a proposal involves starting it out in a
`drafts` folder, then assigning it a number and moving it to an `accepted`
folder (should it actually be accepted), and then possibly moving it to other
folders as its state changes (if it is revoked by the author during publication,
it goes to a `withdrawn` folder, or if it becomes implemented in the associated
project, it moves to `final`, etc.). This has the following drawbacks:

- it is an extra burden on the RFC author
- it duplicates data "out-of-band" that would better be recorded in the RFC
  document itself
- it makes browsing the RFC repository annoying
- it makes it largely impossible to offer stable URLs to accepted RFCs

If, instead, accepted proposals could simply be listed in the top level of the
repository, all of the drawbacks above would be solved. Some immediate effects
would include better "new user" friendliness, and an improved propensity for an
individual author to want to write RFCs in the first place.

## Proposals

The bulk of this proposal is to write a new RFC that will supersede [RFC
0003](https://github.com/multinet-app/multinet-rfcs/tree/master/final/0003-rfc_process).
The process detailed therein would largely be reproduced as-is, with the
following changes to procedure:

1. While RFCs would still begin life in a dedicated `drafts/` folder, once
   accepted they will simply be copied to the top-level.
2. The top-level `template.md` file can be moved to `drafts/`.
3. The folders `accepted/`, `final/`, `rejected/`, `superseded/`, and
   `withdrawn` would disappear. `draft/` would be retained for housing draft
   RFCs as detailed in point 1 above.
4. All existing RFCs will gain a `Status` metadata field which will inherit the
   disappeared folder name under which they used to live.
5. The RFC template will gain a metadata field `Status`, set to `draft` by
   default, but that must be updated prior to acceptance to the appropriate
   status.

## Reference Implementation

Upon acceptance, this RFC will authorize a repo admin (probably Roni Choudhury)
to perform the changes to the repo and instructions needed to implement these
recommendations.

Jake Nesbitt and Roni Choudhury are working on an experimental Python toolkit
intended to provide automation support for various stages in the RFC procedure;
that toolkit will absorb the new recommendations.
