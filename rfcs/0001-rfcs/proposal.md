---
author: Roni Choudhury
date: 2020-06-10
---

# RFC: Process for Creating RFCs

This is the first official Multinet RFC. It lays out the procedure for creating
RFCs, soliciting feedback, and making them official.

## Motivation

We have historically made use of Google Docs to put together proposals for
technical work that is either contentious, non-obvious, or both. These written
proposals enable the author to clarify their thinking on a topic, write down at
least minimal detail about the topic to transmit to others on the team, and
formulate an action plan once the proposal is accepted.

However, Google docs have some drawbacks: they are not particularly transparent
and trackable; they are difficult to comment on for deeper discussions; and they
are less pleasant to read and write than simple markdown files.

Therefore, this proposal lays out a procedure for creating markdown-based RFCs,
getting feedback and generating discussions for them, and settling on accepted
versions for use in creating execution plans.

## Proposals

### Definitions

- **RFC.** A "request for comments", though the term is used to mean both a
  proposal for how something should be done, and as a de facto standard for how
  that thing is done. We are avoiding the term **proposal** since that word is
  overloaded to mean "grant proposal" in an academic context.

### Procedure for Creating a new RFC

1. Clone the RFCs repository with a command such as `git clone https://github.com/multinet-app/multinet-rfcs`.
2. Create a new branch naming your RFC: `git checkout -b rfc/my-awesome-proposal`.
3. Push your branch: `git push -u origin rfc/my-awesome-proposal`.
4. Create a subfolder for your RFC in the `rfcs` folder, using the next
   available four-digit number (left-padded with zeroes) and a descriptive name,
   separated by a hyphen: `mkdir rfcs/0047-my_awesome_proposal`.
5. Move into your folder: `cd rfcs/0047-my_awesome_proposal`.
6. Create a `proposal.md` file, and optionally a `README.md` file.

### Writing the RFC in `proposals.md`

The `proposal.md` file will contain the substantive body of the RFC. The
following parts are required in this document (this RFC itself illustrates these
requirements):

- **Metadata.** Insert a section at the very top of the file, delimited by `---`
  on both sides; include a line reading starting with `author: ` and appending
  your name, and another starting with `date: ` and the current date in ISO format
  (i.e., `2020-06-10`).
- **Title.** The title should begin with `RFC: ` and include a
  newspaper-headline style capitalized title that is descriptive and concise.
- **Overview.** The text immediately following the title should provide a very
  high-level overview of what the RFC is about.
- **Motivation.** Include an "H2" section called `Motivation` that describes the
  motivation for creating this RFC--what problem it solves, what kinds of
  problems that causes for the project, etc.
- **Proposals.** Include an "H2" section called `Proposals` which will contain
  one or more "H3" sections laying out individual proposal items.

### Including Context in `README.md`

The `README.md` file can be used to describe context, background, history, and
other "out-of-band" notes that are not necessary or desired within the RFC
itself.

### Other Files

Other support materials as needed can be included in the RFC folder, and
referenced from `proposal.md` or `README.md`. These may include images,
supplemental text, or any other material deemed useful.

### Soliciting Feedback

When an initial draft of the RFC is complete, push all changes and open a pull
request at the `multinet-rfcs` repository on GitHub. Copy the title of the RFC
into the PR title, and put some useful description in the PR description field
(at a minimum, this can be a copy-paste of the RFC Overview). If you believe
feedback is particularly useful from specific team members, request their
reviews explicitly using GitHub's mechanism for doing so.

The feedback phase can involve threaded discussions on the PR page as well as
comments on individual lines or blocks of text--much like a regular code review.
This phase may include adding text to the `README.md` to record certain ideas or
conclusions; edits to `proposal.md` to extend, remove, or change parts of the
draft; addition of supplemental material; etc. In general, this phase is meant
to surface disagreements and constructive feedback and in turn, improve the RFC
to a point where everyone has reached something like [rough
consensus](https://tools.ietf.org/html/rfc7282).

### Accepting the RFC

Once discussions have resolved and review approvals are granted, the original
author of the RFC can, when they are satisfied with the document, merge the PR.
This signifies that the RFC is now accepted, and will appear on the main code
listings on GitHub (as part of the `master` branch).

### Updating Accepted RFCs

Any time an accepted RFC must be updated for any reason, a PR may be opened that
makes edits to the RFC materials. It is required in this case to add an
`updated: ` line to the metadata section, including both a date and a name
separated by a comma. For example:

   updated: 2021-06-10, Roni Choudhury

After changes are made and the PR opened, the feedback and acceptance phases are
the same as detailed above.

### Use of Singular They

Computer science as a field, both in industry and academia, is
disproportionately represented by men. In a small effort to recognize and
correct this inherent imbalance, RFCs should make use of [singular
they](https://en.wikipedia.org/wiki/Singular_they) in order to explicitly avoid
excluding any person on the basis of their gender identity.
