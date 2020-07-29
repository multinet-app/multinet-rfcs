---
RFC: XXXX
Author: Roni Choudhury
Status: draft
Created: 2020-07-09
Last-Modified: 2020-07-09
---

# RFC XXXX: Autopublishing for MultinetJS

One of the active pieces of software driving the Multinet project is
`multinetjs`, a TypeScript library providing clientside bindings and utility
functions for the Multinet API. This software grows with the needs of the
project, specifically of the [Multinet
client](https://github.com/multinet-app/multinet-client). This means that often,
a new feature in the client requires a new feature in MultinetJS, and to support
this use case, this RFC lays out a plan for more easily publishing prerelease
and development versions of the library for use in client development.

## Motivation

MultinetJS does not currently have an automated, official release process.
Releases are done by hand after a pull request is approved, and before that step
happens, developers are expected to use `yarn link` to provide development
linkages between MultinetJS and the client codebase.

While this procedure works well in local development, it leaves CI out. In
particular, since the Netlify build previews of the client rely on published
versions of MultinetJS, build previews depending on a development version of
MultinetJS simply cannot be built.

A procedure for automated or semi-automated publishing of MultinetJS, even for
development versions, would fix this issue. It would also eliminate the manual
step currently needed to perform releases to NPM.

## Proposals

The proposals here use GitHub Actions to drive publishing pipelines to NPM on
both pull request branches, and on master. Details for each follow.

### Development Releases

The idea here is to automatically publish development versions based on a PR
branch to NPM automatically using CI. On each push from a PR branch, the GH
Actions workflow can run the tests, perform a build, and if successful, publish
a prerelease version to NPM. The version number can be taken from the current
value in `package.json`, appended with the first six characters of the current
commit's Git SHA--for example `multinet-0.47.3-a1c0ef` ([Clause
9](https://semver.org/#spec-item-9) of the semver spec stipulates that
prerelease versions can be indicated by appending alphanumeric strings in this
manner). The publishing would be done using an NPM token stored in a GitHub
secret, arranging the process so that any developer can effectively create
releases simply by pushing to a branch (or master).

Alternatively, instead of automatically pushing these releases, they could
instead be done manually via an NPM script. However, this would have the
drawback of requiring distribution of credentials to the different developers.
GitHub Actions can also be used to trigger such actions manually; see below.

### Production Releases

Tools such as
[semantic-release](https://github.com/semantic-release/semantic-release) can be
used to perform automatically versioned releases of NPM projects. However, that
approach requires a specific, machine-readable commit message format that can be
used by the tooling to compute the correct release version.

In place of that heavier-handed method, which introduces a specific burden on
all developers, we can simply perform a release to NPM on every merge to master,
based upon the version number listed in `package.json`. This essentially
requires human intervention to both compute the correct semantic version for a
given branch, and update it manually in the source. However, this also allows a
bit of extra control, allowing for *not* updating the version number in order to
avoid creating a release.

### Manual GitHub Actions via Workflow Dispatch

GitHub Actions includes a
[workflow_dispatch](https://github.blog/changelog/2020-07-06-github-actions-manual-triggers-with-workflow_dispatch/)
mechanism that allows for a pushbutton to execute a workflow. We can use this to
set up several workflows: one for publishing a prelease version from a branch;
one for publishing release versions from master; and maybe one for deleting (or
"unpublishing") prerelease versions that were published from merged branches.

This more manual approach would prevent too many prerelease versions from being
published, while avoiding having to distribute credentials to developers who
would otherwise want to do such publishing from their local setup.

### Implementation Notes

Determining what version string to use should be fairly straightforward: on a
branch, it would be the `version` property of `package.json` (extractible via a
utility such as `jq`), with the result of the first six characters of `git
rev-parse HEAD` appended thereto (with a hyphen).

For a production release, it is much the same. However, it would be good to
check that the version number from `package.json` does not lie in the `versions`
field of the result of `npm show multinet --json` first. This avoids an error in
CI for the case when the version number has not changed (`yarn publish` fails in
this case with a message about not being able to overwrite an existing package
version); simply ignoring this error code is not an option, since an actual
failure to publish for some other reason *should* be reported as a CI failure.
