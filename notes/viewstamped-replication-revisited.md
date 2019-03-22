---
geometry: margin=3cm
title: Notes for ``Viewstamped Replication Revisited''
---

# Background

## Assumptions

- Handles crash failures; nodes fail by crashing
- Does not handle Byzantine failures
- Handles asynchronous network; messages might be lost, delivered late or out of order, or delivered
  more than once

## Replica Groups

- Groups of size $2f + 1$
- Ensure reliability and availability when no more than a threshold of $f$ replicas are faulty
- Group is referred to as a _quorum_ and correctness of protocol comes from _quorum intersection
  property_

## Architecture

- User code communicates with VR proxy
- Proxy communicates with replicas to cause the operation to be carried out
- Replicas run VR code which accepts requests from clients, carries out the protocol and makes an
  upcall to service code when request is ready to be executed
- Service code executes the call and returns the result to the VR code which sends it in a message
  to the client proxy

# Overview

- VR uses _primary_ replica to order client requests
- Other replicas are _backups_ that simply accept the order of requests selected by primary
- Different replicas can assume role of primary over time
- Backups carry out _view change_ when primary is detected to be faulty
- Primary must wait until at least $f + 1$ replicas know about a client request before executing it
- Three sub-protocols to ensure correctness
  1. Normal case processing of user requests
  2. View changes to select new primary
  3. Recovery of failed replica so that it can rejoin the group

# VR Protocol

- Identity of primary is determined from _view-number_ and the _configuration_
- Replicas are numbered based on their IP addresses and the replica with the smallest IP address is
  replica 1
- Primary is chosen round-robin, starting with replica 1
- _Status_ indicates what sub-protocol replica is in
- Client-side proxy records _configuration_ and current _view-number_
- Each message sent to client contains information about current _view-number_
- A client records its _client-id_ and current _request-number_ which increases

## Normal Operation

- Every message sent from one replica to another contains sender's current _view-number_
- Replicas only process normal protocol messages with the same _view-number_ that they know
- If sender is behind, receiver drops the message
- If sender is ahead, receiver performs _state-transfer and requests information it is missing
  from the other replicas and uses information to bring itself up-to-date
- If client does not receive a timely response, it re-sends the request to all replicas in the case
  of a view-change

### VR State

- Configuration: Sorted array containing IP address of $2f + 1$ replicas
- Replica number: The index into the configuration where this replica's IP address is stored
- View-number: Initially 0
- Status: One of _normal_, _view-change_, or _recovering_
- Op-number: Number assigned to most recently received request number, initially 0
- Log: Array conainting op-number entries
- Commit-number: Op-number of the most recently committed operation
- Client-table: Table that maps clients to its most recent request and the result for the request if
  the request has been executed

### Request Processing Protocol

1. Client sends $\langle \text{REQUEST}, op, c, s \rangle$ to primary where $op$ is the opeartion
   with its arguments, $c$ is the _client-id_ and $s$ is the _request-number_
2. Primary compares _request-number_ with information in the client table and drops if it is not
   greater. It will resend result if it is equal to the most recent request recorded.
3. Primary advances _op-number_, adds request to end of the _log_ and updates the information in the
   _client_table_
4. Primary sends $\langle \text{PREPARE}, v, m, n, k \rangle$ to other replicas where $v$ is the
   current _view-number_, $m$ is the message it received from the client, $n$ is the _op-number_ it
   assigned to the request, and $k$ is the _commit-number_
5. Backups process PREPARE requests in order and won't accept prepare with _op-number_ $n$ until it
   has entries for all earlier requests in its _log_ (performing _state-transfer_ if necessary). It
   also increments its _op-number_, adds request to the end of its _log_, updates the client's
   information in the _client-table_, and sends a $\langle \text{PREPARE\_OK}, v, n, i \rangle$ to
   primary.
6. Primary waits for $f$ PREPARE_OK messages at which point it considers the request to be committed.
   It executes the request by making an upcall, increments it _commit-number_, updates the
   _client-table_ with the result, and sends a $\langle \text{REPLY}, v, s, x \rangle$ to the client
   where $v$ is the _view-number_, $s$ is the number the client provided in the reuqest, and $x$ is
   the result of the upcall. 6.
7. Primary informs backups about the commit when it sends next PREPARE message. If primary does not
   receive a new request in a timely manner, it sends $\langle \text{COMMIT}, v, k \rangle$ message
   where $k$ is the _commit-number_.
8. When backup learns of new commit, it waits until it has the request in its $log$ (may require
   _state-transfer_). Then it executes the operation by performing the upcall, increments it
   _commit-number_, updates its _client-table_, but does not send the reply to client.

## View Change

- If primary is idle, it will send COMMIT messages as heartbeats
- If a timeout expires without a communication from the primary, replicas carry out view change to
  switch to new primary
- Correctness condition: every operation that has been executed by means of an upcall at one of the
  replicas must survive into the new view in the same order selected for it at the time it was
  executed
- View change protocol must obtain information from the logs of at least $f + 1$ replicas
  - Sufficient to ensure that all committed operations will be known
- More than one request can be assigned the same _op-number_, but it is solved by taking the log
  from the latest previous active view

### View Change Protocol

1. Replica $i$ notices the need for a view change, advances its _view-number_, sets it status to
   _view-change_, and sends a $\langle \text{START\_VIEW\_CHANGE}, v, i \rangle$ message to all
   other replicas, where $v$ is the new _view-number_. It notices the need based on its own timer,
   receiving a START_VIEW_CHANGE message or receiving a DO_VIEW_CHANGE message for a view with a
   larger number than its own _view-number_.
2. When replica $i$ receives START_VIEW_CHANGE message for its _view-number_ from $f$ other
   replicas, it sends a $\langle \text{DO\_VIEW\_CHANGE}, v, l, v', n, k, i \rangle$ message to the
   node that will be the primary in the new view where $v$ is its _view-number_, $l$ is its log,
   $v'$ is the view number of the latest view in which its status was _normal_, $n$ is the
   _op-number_, and $k$ is the _commit-number_.
3. When new primary receives $f + 1$ DO_VIEW_CHANGE messages from different replicas (including
   itself), it sets its _view_number_ to that in the messages and selects the log in the message
   with the highest $v'$, breaking ties by the highest $n$. It sets the _op-number_ to latest entry
   in new log, sets its _commit-number_ to the largest number it received, changes its _status_ to
   normal, and informs the other replicas of the completion of the view change by sending $\langle
   START\_VIEW, v, l, n, k \rangle$ message where $l$ is the new log, $n$ is the _op-number_, and $k$
   is the _commit-number_.
4. New primary can start accepting client requests. It also executes any committed operations that
   hasn't been executed, updates its client-table, and sends the replies to the clients.
5. When other replicas receive the START_VIEW change, they replace their _log_, set their
   _op-number_ to the latest entry in the new log, set their _view-number_ to the view number in the
   message, change their _status_ to _normal_ and update the information in their _client-table_. If
   there are non-committed operations in the log, they send a $\langle PREPARE\_OK, v, n, i \rangle$
   message to the primary. They also execute all operations known to be committed that they haven't
   executed previously, advance their _commit-number_ and update the information in their
   _client-table_.

## Recovery

- If nodes record their state on disk, they can immediate rejoin the system as soon as it has
  reinitialized its state by reading from disk
- Can eliminate disk write by using a _recovery protocol_
- When a node comes back, it sets its _state_ to _recovering_ and must learn the configuration
again but waiting for messages from other group members and then fetching the configuration from one
of them
- Configuration information can also be stored on disk
- Nonce is used to determine messages for the current recovery and not for a previous one

### Recovery Protocol

1. Recovering replica, $i$, sends a $\langle \text{RECOVERY}, i, x \rangle$ message to all other
   replicas where $x$ is a nonce.
2. A replica $j$ only replies to a RECOVERY message only when its status is _normal_. It sends a
   $\langle \text{RECOVERY\_RESPONSE}, v, x, l, n, k, j \rangle$ message to the recovering replica,
   where $v$ is its _view-number_, $x$ is the nonce in the RECOVERY message. If $j$ is the primary
   of the view, then $l$ is its _log_, $n$ is its _op-number_, and $k$ is the _commit-number_,
   otherwises these values are _nil_.
3. Recovering replica waits for at least $f + 1$ RECOVERY_RESPONSE messages, including one from the
   primary of the latest view it learns of in these messages. Then it updates state using
   information from the primary and changes its status to _normal_.

## Client Recovery

- If a client crashes, it must recover with a _request_number_ larger than what it had before it
  failed; it can fetch its latest number from the replicas and add 2 to this value to be sure that
  the new _request-number_ is big enough.

# Pragmatics

## Efficient Recovery

- Logs can be long in long-lived system
- Can keep prefix of the log on disk to reduce expense
- Log can be pushed to disk in the background
- Replica can fetch prefix from disk and suffix from other replicas
- Better approach is to write application state that represents a prefix of the log
- Can use checkpoints; after every $O$ operations, node makes an upcall to the application to take a
  checkpoint
- Application records a snapshot of its state on disk and records the _op-number_ of the latest
  operation included in the checkpoint
- Operations after the checkpoint are executed using copy-on-write
- When recovering, a node obtains the application state from another replica
- For efficiency, nodes can maintain a Merkle tree of the pages in the snapshot to determine which
  pages are different
- After the recovering node has all the application state of the latest checkpoint at a node, it can
  run the recovery protocol
- Recovering node will inform other nodes its current _op-number_

## State Transfer

- If node is missing requests in its current view, it only needs to learn about requests after its
  _op-number_
- If node has heard about a later view, it must remove requests after its _commit-number_ and set
  its _op-number_ to its _commit-number_ since requests might be reordered in the view change
- A replica sends a $\langle \text{GET\_STATE}, v, n', i \rangle$ message to one of the other
  replicas, where $v$ is its _view-number_ and $n'$ is its _op-number_.
- A replica will only respond to a GET_STATE message if its _status_ is _normal_ and it is in view
  $v$
- Response to GET_STATE message would be $\langle \text{NEW\_STATE}, v, l, n, k \rangle$ where $v$ is
  its _view-number_, $l$ is its log after $n'$, $n$ is its _op-number_, and $k$ is its
  _commit-number_

## View Changes

- Primary of new view must obtain an up-to-date log
- Replicas can include a suffix of their log in their DO_VIEW_CHANGE messages since new primary is
  likely to be up-to-date
- If the information is not enough, new primary must use application state to bring itself up to
  date

# Optimizations

## Witnesses

- Group of $2f + 1$ replicas contains $f + 1$ active replicas and $f$ witnesses
- Primary is always an active replica
- Witnesses are needed for view changes and recovery and fill in for active replicas when they
  aren't responding

## Batching

- Primary collects a bunch of requests and then runs the protocol for all of them at once
- Use of batching can be limited to when primary is heavily loaded

## Fast Reads

### Reads at the Primary

- Primary can unilaterally serve reads using leases because reads do not change state
- If primary holds valid leases from $f$ other replicas, then it can serve reads unilaterally
- New view will only start when leases at $f + 1$ participants have expired
- Read requests no longer have to run through the protocol

### Reads at Backups

- If result can be based on stale information, we can also execute reads at backups
- When client does a write, the primary returns the _op-number_ assigned to that request which the
  client stores as _last-request-number_
- Backups whose _commit-number_ is greater than or equal to the _last-request-number_ can serve the
  read request
  - Backups must also return their _commit-number_ for the client to store as _last-request-number_
    if the read is served

# Reconfiguration

- Used to replace failed nodes or change the group size
- Triggered by special client request that goes through the normal case protocol
- When request commits, the system moves to a new _epoch_
- The new group cannot process requests until all replicas are up-to-date

## Reconfiguration Details

- A replica sets its status to _transitioning_ (new status) at the beginning of the next epoch
- Replicas that are members of the replica group for the new epoch change their status to _normal_
  when they have received the complete log up to the start of the epoch
- Replicas that are being replaced shut down once they know their state has been transferred
- Every message contains _epoch-number_
- Replicas only process messages that match the epoch they are in
- If they receive a message with a later epoch, they move to that epoch
- If they receive a message with an earlier epoch, they discard the message and inform the sender of
  the new epoch
- Reconfigurations are requested by a client $c$ via $\langle \text{RECONFIGURATION}, e, c, s,
  \text{new-config} \rangle$ where $e$ is the current _epoch-number_ known to $c$, $s$ is $c$'s
  _request-number_, and _new-config_ provides the IP addresses of all members of the new group
- The primary will only accept if $s$ is large enough and $e$ is its current _epoch-number_
- The new group size must be greater than 2

### Steps to Process Reconfiguration Request

1. Primary adds request to its log, sends PREPARE messages to backups, and stops accepting client
   requests
2. Backups handle the PREPARE requests in the usual way and return PREPARE_OK responses
3. When primary receives $f$ of these responses, it increments its _epoch-number_, sends COMMIT
   messages to old replicas and sends $\langle \text{START\_EPOCH}, e, n, \text{old-config},
   \text{new-config} \rangle$ messages to replicas being added to the system where $e$ is the new
   _epoch-number_ and $n$ is the _op-number_
4. Primary executes all client requests ordered before the reconfiguration that it hasn't executed
   and sets its state to _transitioning_

### Processing in the New Group

1. When replica learns of the new epoch (E.G. by receiving a START_EPOCH or COMMIT message), it
   initializes its state to record old and new configurations, the new _epoch-number_, and
   _op-number_, sets its _view-number_ to 0, and sets its _status_ to _transitioning_
2. If replica is missing requests from its log, it brings its state up-to-date by sending state
   transfer to the old replicas and also to other new replicas
3. Once a replica in the new group is up-to-date with respect to the start of the epoch, it sets its
   status to _normal_, processes any requests it hasn't already and sends $\langle
   \text{EPOCH\_STARTED}, e, i \rangle$ messages to the replicas that are being replaced.

### Processing at Replicas Being Replaced

1. Replica changes its _epoch-number_ and sets its _status_ to _transitioning_
2. If it does not have the reconfiguration request, it performs state transfer from other old
   replicas
3. Replica will respond to state transfer requests until they receive $f' + 1$ EPOCH_STARTED
   messages, at which point the replica shuts down
4. If replica does not receive EPOCH_STARTED messages in a timely way, it sends START_EPOCH messages
   to new replicas

## Other Protocol Changes

- Replica does not accept messages for a epoch earlier than what it knows
- In a view change, the new primary needs to recognize a reconfiguration is in process and to stop
  accepting new client requests
- If a replica that is recovering is informed of a new epoch, it will shut down if it is not part of
  the new configuration. Otherwise it will continue with the recovery process

## Shutting Down Old Replicas

- Administrator must wait for reconfiguration to be complete before shutting down nodes being
  replaced
EW- Administrator can run $\langle \text{CHECK\_EPOCH}, e, c, s \rangle$ after getting the reply to the
  RECONFIGURATION request, and upon getting a reply, determine that the reconfiguration is complete

## Locating the Group

- Old replicas that receive a client request with an old epoch number and inform the client about
  the new group
- Can use out-of-band mechanism to inform clients

## Discussion

- Reduce delay by "warming up" new nodes
- Only send RECONFIGURATION request when new nodes are almost up-to-date
