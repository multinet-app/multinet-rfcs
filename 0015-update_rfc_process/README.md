---
RFC: 0015
Author: Roni Choudhury
Created: 2021-04-26
Last-Modified: 2021-04-26
Supersedes: 0003
---

# RFC 0015: Adopt Python-esque process for creating RFCs

This RFC describes a process for creating RFCs that adapts the Python/Django
methodologies for creating "PEPs" and "DEPs" (Python/Django Enhancement
Proposals). [Django's
process](https://github.com/django/deps/blob/master/final/0001-dep-process.rst)
is detailed via its own DEP.

## Motivation

Our previous process for creating RFCs is tedious and error prone, and ends up
hiding RFC documents in various directories which tend to obscure the history of
the project. One of the goals of writing RFCs is to provide a written history of
development for retrospectives and new developers alike. Another goal is for
people to easily be able to write RFCs (whether in this project, where
willingness to do so is high, or in other projects, where a similar process may
not have as much buy in).

To these ends, we're simplifying the process laid out in 0003 to eliminate some
of the steps involved in writing an RFC. In the future, tooling may help to
support this streamlined process, leaving little or no friction to writing RFCs.

## Proposals

### New Process for RFC Creation/Review/Acceptance/Rejection

1. The author clones this repository, creates a new directory in the top level
   directory with a prospective RFC number based on the highest numbered RFC
   existing so far `0047-new_proposal/`, if `0046-existing_proposal/` is the
   highest-numbered proposal there), and copies the root-level `template.md`
   file there as `README.md`.
2. The author modifies the new `README.md` file to contain their newly proposed
   RFC, taking care to fill in the metadata information in the header block. If
   the new RFC is intended to supersede an older one, its metadata should
   reflect that (see below for details).
3. If supplemental materials are needed (this should be generally rare), they
   can go into the new directory alongside the `README.md` file.
4. The author opens a pull request. The title of the pull request should be the
   RFC title preceded by a prefix of `RFC:`. A PR description is optional, and
   can generally be omitted if the PR is simply introducing a new RFC (since the
   change set itself will detail the reason for opening the PR).
5. Reviewers provide feedback, engage in discussion, etc., just as in a normal
   GitHub-based code review.
6. The RFC becomes "settled". Settlement involves one of several paths of
   action, detailed next.

### Settling an RFC

RFCs can be settled in several ways.

**Acceptance.** An RFC can be accepted as written (which includes mutually
agreed-upon edits).

1. Two reviewers provide a review approval.
2. The author changes the `Status` metadata item to `accepted`.

**Rejection.** The RFC can be rejected if it does not fit into the larger aims
of the project.

1. Discussion ends with a determination that the RFC should be rejected.
2. The author changes the `Status` metadata item to `rejected`.

**Withdrawal.** The RFC can be withdrawn by the author.

1. The author determines that they wish to withdraw the RFC.
2. The author simply closes the PR (leaving no history of the ideas
   being explored), OR they change the `Status` metadata item to `withdrawn`.
   This can be useful in case the withdrawn RFC will be built upon later into
   another RFC, or just to record a good idea that nonetheless was abandoned by
   the author.

### Finalizing the RFC

After the RFC has been settled, the pull request associated with it can be
merged (in most cases; see "Withdrawal" above). To do so, follow these steps:

1. The author fetches the `master` branch, then merges that branch into their
   RFC branch.
2. If there is now a conflict with RFC numbers (e.g., someone else got their RFC
   in while the author is still working on theirs), the author selects a new,
   appropriate RFC number for the directory and metadata of the RFC document.
3. The author pushes their commits, then merges the PR.

Note that, in the case that the RFC settled to rejection or withdrawal, despite
**rejecting** or **withdrawing** the RFC, this procedure **accepts** the pull
request. This is necessary to preserve the history of ideas and information that
flowed through the project. The RFC number must still be assigned to make
reference to this RFC easier in the future.

### Revisiting Settled RFCs

Settled RFCs can be revisited in several ways.

**Revival.** Anyone who wants to bring a `withdrawn` RFC back for
reconsideration can do so by opening a pull request that moves that RFC (with
its original RFC number) back to `draft` status. The same procedures as above
resume from the beginning, leading to settlement of that RFC once more.

**Supersession.** An accepted RFC can be *superseded* (that is, replaced) by a
new RFC. This can be done because the team has changed its collective mind on
something, or because new facts demand an update of old information or
procedures.

1. A new RFC is formulated for review using the usual procedure as detailed
   above.
2. The RFC must indicate which RFC it supersedes in its `Supersedes` metadata
   item.
3. The pull request should add a `Superseded-By` metadata item to the
   appropriate RFC.
4. Review proceeds as usual.

**Modification.** An accepted or final RFC can be modified after the fact by
opening a new PR that makes edits or additions. The edits should include an
update to the `Last-Modified` metadata field. The PR should have a title
describing the nature of the modification, and the description should describe
the motivation for doing so. As with any other RFC PR, discussion followed by
two approvals can lead to a merge.
