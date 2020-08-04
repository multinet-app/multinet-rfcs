---
RFC: XXXX
Author: John Q. Public
Created: 2020-06-29
Last-Modified: 2020-06-29
Supersedes: XXXX (not usually required)
Superseded-By: XXXX (not usually required)
---

# RFC XXXX: RFC Template

This file provides a sample template for creating your own RFC. See [RFC
0003](https://github.com/multinet-app/multinet-rfcs/final/0003-rfc_process)
for a fuller discussion on how to create and submit an RFC.

Note: if you are reading this via the web, you should first grab [the source of
this markdown
file](https://raw.githubusercontent.com/multinet-app/multinet-rfcs/template.md)
in order to complete the steps below.  **DO NOT USE THE HTML FILE AS YOUR
TEMPLATE!**

To get the source this (or any) RFC, look at the top of the Github page
and click "raw".

If you're unfamiliar with GitHub flavored Markdown (the format required for
Multinet RFCs), see [Mastering
Markdown](https://guides.github.com/features/mastering-markdown/) or the [GitHub
Flavored Markdown Spec](https://github.github.com/gfm/).

Once you've made a copy of this template, replace this overview section with a
short (~200 word) description of the technical issue being addressed, fill out
the title and metadata above (leaving RFC numbers as `XXXX`) and the sections
below, then submit the RFC. Follow the [guidelines](guidelines) in [RFC
0003](https://github.com/multinet-app/multinet-rfcs/final/0003-rfc_process).

Note that the `Superseded` and `Superseded-By` metadata entries are only needed
when a new RFC will supersede an existing one (then the new one must say which
RFC it supersedes, and the superseded one must indicate the new RFC that
supersedes it).

## Motivation

Explain any background needed to understand why this RFC is needed. This section
can be as short as a single sentence, or stretch to discuss a difficult problem
area. If the RFC is truly self-evident, this section can be omitted altogether
(but this should be rare).

## Proposals

This section should contain the meat of the RFC. The whole thing can go right
here, or this section can contain "H3" sections to further structure the text.

If the RFC is meant as a guide for implementation, it should be detailed enough
that someone could perform the implementation work with just the RFC in hand.

### This is an example "H3" subsection

This section is just an example; you can delete this subsection, retitle it to
suit your needs, or create more just like it to provide more structure here.

## Backwards Compatibility

If this RFC introduces backwards incompatibilities, you must must include this
section. It should describe these incompatibilities and their severity, and what
mitigation you plan to take to deal with these incompatibilities.

## Reference Implementation

If there's an implementation of the feature under discussion in this DEP, this
section should include or link to that implementation and provide any notes
about installing/using/trying out the implementation.
