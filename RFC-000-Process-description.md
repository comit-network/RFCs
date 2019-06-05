# Process description

- RFC-Number: 000
- Status: Draft
- Discussion-issue: -
- Created on: 25 Jan. 2019

**Table of contents**
- [Description](#description)
- [Lifecycle](#lifecycle)
    - [Everything starts as `Draft`](#everything-starts-as-draft)
    - [On to stabilization](#on-to-stabilization)

## Description

This document describes the RFC (Request For Comments) process that is used to
document and specify the COMIT protocol.  We are using an RFC process because we
want the documentation and the process around it to be open-source and
accessible to anyone.

## Lifecycle

### Everything starts as `Draft`

Anybody is welcome to submit an RFC to extend or change the COMIT protocol by
opening a pull-request to this repository.  All accepted changes will have
status `Draft`.  RFCs in `Draft` status MUST have a discussion issue that can be
used to discuss the current content of the RFC.  This will be the place where
anybody can voice opinions or point out possible flaws or inconsistencies.  As
instructed by the [issue template](./.github/ISSUE_TEMPLATE/discussion_issue.md),
please add a checklist for unresolved problems or open questions at the end of
the discussion issue.  As long as an RFC is in `Draft`, new PRs can change it
accordingly to reflect the newest findings of the discussions.

### On to stabilization

Once all open issues have been resolved and no other objections are raised
within a month, a stabilization PR can be opened.  This PR MUST:
- close the discussion issue via `Resolves #...`
- apply the changes the PR documents to the [registry](./COMIT-registry.md)
- change the status of the RFC to `Final` in the [README](./README.md) and the
  RFC itself
 