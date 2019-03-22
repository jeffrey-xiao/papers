---
geometry: margin=3cm
title: Notes for ``Viewstamped Replication - A New Primary Copy Method to Support Highly-Available Distributed Systems''
---

# Introduction

- Assumptions
  - Failures are not byzantine
  - Nodes will eventually recover from crashes
  - Partitions are eventually repaired
  - Computations run as atomic transactions
  - Model of computation where a distributed program consists of _modules_, each of which resides at
    a single node of the network
    - Modules can recover with some of their state intact
    - Modules cannot access the data objects of another module directly and must use _remote
      procedure calls_
- Based on primary copy technique
  - One replica is designated as the primary and others as backups
  - Primary is responsible for processsing transactions and notifying backups
  - When a replica crashes/recovers, replicas are reorganized and a new primary is selected if
    necessary
  - Reorganization is called a _view_
  - Primary copy techinque only works if node failures are distinguishable from network failures,
    but it is not true in the general case

# Overview

- Replicate individual modules to obtain _module groups_ which consists of several copies of the
  module called _cohorts_
- The set of cohorts is the group's _configuration_
- Each cohort has a unique name called an _mid_
- The group has a unique `groupid`
- One cohort is designed the _primary_ and remaining cohorts are _backups_
- Each cohort runs in a _view_ which is a set of cohorts that are capable of communicating with each
  other
  - It is a a subset of the configuration and must contain a majority of the group members
- A group switches to a new view by executing a _view change protocol_
- Each view is identified by a unique `viewid` (totally ordered)
- If a majority of cohorts accept the new view, cohorts switch to the new active view, else they
  remain in their old views
  - Transactions are only processed in active views
- Primary generates a timestamp (unique within a view and totally ordered) every time it needs to
  communicate information to its backups referred to as an _event_
- Primary maintains a communication buffer where it writes _event records_
  - An event record identifies the type of event and other relevant information about the event
  - Information in the buffer is sent to backups in timestamp order
- Timestamps are used as an inexpensive way of determining what information survives a view change
- _Viewstamps_ are timestamps concatenated with the `viewid` of the view in which the timestamp is
  generated
- Each cohort maintains a _history_ of viewstamps

# Running Transactions

- Viewstamps are used to determine if a transaction can commit
- State of cohort:
  - `status`: A cohort is _active_ if it can participate in transaction processing, otherwise it is
    _inactive_ and involved in a view change
  - `gstate`: Set of objects that consistute the group state
  - `groupid`: Name of the module group
  - `cur_viewid`: Current `viewid`
  - `cur_view`: Current view
  - `history`: Sequence of events known to cohort
  - `timestamp`: Timestamp generator
  - `buffer`: Sequence of event records (communication buffer)
- Transactions are synchronized by strict 2-phase locking
- Transactions modify _tentative_ version which is discarded if transaction is aborted and becomes
  base version if it is committed
- Communication buffer actions:
  - `add`: Takes an event record, atomically assigns it a timestamp and adds the event record to the
    buffer
  - `force-to`: Takes viewstamp $v$ has an argument. Returns immediately if $v$ is not for the
    current view and otherwise waits until a sub-majority of backups know about all events in the
    current view with timestamps less than or equal to $v.ts$
- Clients create transactions, make remote calls, and act as coordinators of two-phase commit
- Servers process remote calls and participate in two-phase commit
- Assumes that a highly-available location server that maps `groupids` to configurations exists

## Active Primaries of Clients

- When a server finishes processing a remote call on behalf of transaction, it assigns the call a
  viewstamp
- When transaction is created, it receives a unique transaction id `aid` and an empty `pset` that
  keeps track of remote calls done in the transaction
- To make a remote call, system looks up primary and `viewid` for the group and then sends a call
  message which contains a unique call id to the primary
  - Reply returns a `pset` of `<groupid, viewstamps>` pairs
  - If there is no reply, then abort transaction and send abort messages to all participants
  - If reply indicates that view has changed, update cache, retry call and if most recent view
    cannot be discovered, then abort the transaction
- Client's primary acts as coordinate of two-phase commit protocol
  - Determines who the participants are from the `pset`
  - Sends prepare messages to participants of `pset`
  - Adds committing record to buffer if all participants agree to ensure that commit will be known
    across view change of coordinator
  - Sends commit messages and when all are acknowledged, adds done record to buffer
  - If transaction aborts, then coordinator sends abort messages to participants and adds an aborted
    record to buffer

## Active Primaries of Servers

- When remote call completes, server's primary assigns it a viewstamp and returns this information
  in the reply message
- Primary can agree to prepare only if it knows about all remote calls its group has done on behalf
  of preparing transaction
- Rejects call if call's `viewid` does not equal to `cur_viewid`
- Otherwise, it creates empty `pset` and runs the call, possibly making nested calls
- Writes a completed call record to buffer that identifies each atomic object that was read or
  written in processing the call with the type of lock obtained and the tentative version if any
- When primary receives a prepare message, it must check that all calls made by transaction to its
  group is known and compatible with its history
- If prepare is successful, then it takes the viewstamp of most recent completed call record and
  forces the buffer
- If it receives a commit message, the primary forces a committed record and then sends
  acknowledgement to the coordinator
- If it receives an abort message, it adds an aborted record to the buffer

## Other Processing at Cohorts

- Cohorts that are not active primaries reject messages sent to them by other module group (except
  for queries described in next section) and the response contains `cur_viewid` and the id of the
  primary if the cohort knows it.
- Active backups process event records in timestamp order and update the state accordingly

## Queries

- Any cohort can response to a query whenever it knows the answer (E.G. a cohort that is not a
  primary may know about the abort of a transaction because it received the aborted event record
  from the primary)

## Replicated Clients

- It is desirable for coordinator to be highly available
- If client is not replicated, we can still use a replicated coordinator-server
- Client takes to such a server to start a transaction and the coordinator-server carries out
  two-phase commit on client's behalf

## Nested Transactions

- Use nested transactions as a check-pointing mechanism
- Minimizes effects of view change
- If there is no reply, then just abort the subaction and do the call again

# Changing Views

- If every view has at least a majority of cohorts, then it contains at least one cohort that knows
  about any event that was forced to a majority of cohorts
- Must ensure that state of new view includes what that cohort knows
- Can take state of cohort with highest viewstamp for the previous view
- Event records are sent to backups in timestamp order
- Cohorts send periodic heartbeats to other cohorts in the configuration
- Cohort initiates view change if it notices that it is not communicating with some other cohort or
  if it notices that it is communicating with a cohort that it could not communicate with previously
- Cohorts generate unique new `viewid` using their `mid` and a number greater than `max_viewid` and
  then it sends invitations to other cohorts
- Cohorts will accept invitations from `viewids` that are greater than its largest recorded one
- Cohorts send normal acceptance containing its current viewstamp and an indication of whether or
  not it is the primary in the current view if it is up to date, else it sends a crash acceptance
- Rule for view formation: A majority of cohorts have accepted and
  1. A majority of cohorts accepted normally
  2. `crash_viewid` < `normal_viewid` (Ignore crashed acceptances from old views)
  3. `crash_viewid` = `normal_viewid` and the primary of view `normal_viewid` has done a normal
     acceptance of the invitation (Ignore crashed acceptance if we havve information from the
     primary of its view)
- Cohort with the largest viewstamp is selected as the new primary
- There can be several active primaries, but old primary cannot force effects to backups
