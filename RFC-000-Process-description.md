# Process description

- RFC-Number: 000
- Status: Draft
- Discussion-issue: -
- Created on: 25 Jan. 2019

<!-- TOC GOES here -->

## Description

This document describes the RFC (Request-for-comments) process that is used to document and specify the COMIT procotol.
The reason for using an RFC process for this is that we would like the documentation itself and the process around it to be open-source and accessible to anyone.

## Lifecycle

### Everything starts as `Draft`

Anybody is welcome to submit an RFC to extend or change the COMIT protocol by opening a pull-request to this repository.
All accepted changes will have status `Draft`.
RFCs in status `Draft` MUST have a discussion issue that can be used to discuss the current state of the RFC.
This will be the place where anybody can voice opinions or point out possible flaws or inconsistencies.
As instructed by the [issue template](./.github/ISSUE_TEMPLATE/discussion_issue.md), please add a checklist for unresolved problems or open questions at the end of the discussion issue.
As long as an RFC is in `Draft`, new PRs can change it accordingly to reflect the newest findings of the discussions.

### On to stabilization

As soon as all the contributors are happy with the RFC, a stabilization PR can be opened.
This PR MUST:
- close the discussion issue via `Resolves #...`
- apply the changes the PR documents to the [registry](./COMIT-registry.md)
- change the status of the RFC to `Final` in the [README](./README.md) and the RFC itself
