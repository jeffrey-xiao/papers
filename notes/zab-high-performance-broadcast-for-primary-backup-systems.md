---
geometry: margin=3cm
title: "Notes for ``Zab: High-performance broadcast for primary-backup systems''"
---

# Introduction

- Primary process receives client requests, executes, and propagates incremental state changes to
  backup replicas in the form of _transactions_
- When the primary crashes, processes engage in recovery protocol to agree on a consistent state and
  to establish a new primary
- Process must have support of a quorum of processes to become the primary
- Processes change to a new view only when a primary crashes or loses support from a quorum
- State changes are incremental with respect to the previous state
- Cannot be applied in any arbitrary order
- Requirements
  - Multiple outstanding transactions that satisfy incremental state property
  - Efficient recovery using _transaction identification scheme_
    - Transaction identifiers are pairs of values: an instance value and the positive of the
      transaction in the sequence broadcast
    - New primary can decide which transactions to recover and from which process by collecting
      highest transaction identifier from each process

# System Model

- System comprises a set of processes $\Pi = \{p_1, p_2, \ldots, p_n\}$
- Processes can crash and recover an unbounded number of times
- A quorum system is defined _a priori_
  - $\forall Q \in \mathcal{Q} : Q \subseteq \Pi$
  - $\forall Q_1, Q_2 \in \mathcal{Q} : Q_1 \cap Q_2 \neq \emptyset$
- Processes use bidirectional channels to exchange messages; one input and one output buffer
  associated with each process in every channel
- Algorithm proceeds in iterations
  - $\sigma^{i, j}_{k, k'}$ is the sequence of messages $p_i$ sends to $p_j$ during iteration $k$ of
    $p_i$ and $k'$ of $p_j$
  - **Integrity**: Process $p_j$ receives a message $m$ from $p_i$ only if $p_i$ has sent $m$
  - **Prefix**: If process $p_j$ receives a message $m$ and there is $m'$ such that $m' \prec m$ in
    $\sigma^{i, j}_{k, k'}$, then $p_j$ receives $m'$ before $m$
  - **Single iteration**: The input buffer of a process $p_j$ for channel $c_{i, j}$ contains
    messages from at most one iteration
  - In practice, use new TCP connections for each iteration

# Problem Statement

- At most one primary is active at any time
- Primary only starts generating state updates once the Zab layer has indicated that recovery has
  completed; Zab layer uses $ready(e)$ call to signal that it can start broadcasting changes
- _Transactions_ are state changes a primary propagates to backup processes
  - A transaction is defined as $\langle v, z \rangle$ where $v$ is the transaction value and $z$
    is the transaction identifier
  - A transaction identifier is defined as $\langle e, c \rangle$ where $e$ is the epoch and $c$ is
    the counter
  - $epoch(z) = \mathtt{instance} = e$
  - Upon each new transaction, counter $c$ is incremented
- Once a primary has a transaction to broadcast it calls, $abcast(\langle v, z \rangle)$
- Processes commit a transaction $\langle v, z \rangle$ by calling $abdeliver(\langle v, z \rangle)$

## Core Properties

- **Integrity**: If some process delivers $\langle v, z \rangle$, then some process has broadcast
  $\langle v, z \rangle$
- **Total order**: If some process delivers $\langle v, z \rangle$ before $\langle v', z' \rangle$,
  then any process that delivers $\langle v', z' \rangle$ must also deliver $\langle v, z \rangle$
  and deliver $\langle v, z \rangle$ before $\langle v', z' \rangle$
- **Agreement**: If some process $p_i$ delivers $\langle v, z \rangle$ and some process $p_j$
  delivers $\langle v', z' \rangle$, then either $p_i$ delivers $\langle v', z'$ or $p_j$ delivers
  $\langle v, z \rangle$
- **Primary ordering**
  - **Local primary order**: If a primary broadcasts $\langle v, z \rangle$ before it broadcasts
    $\langle v', z' \rangle$, then a process that delivers $\langle v', z' \rangle$ must also
    deliver $\langle v, z \rangle$ before $\langle v', z' \rangle$
  - **Global primary order**: If a primary $\rho_i$ broadcasts $\langle v, z \rangle$ and a primary
    $\rho_j$, $\rho_i \prec \rho_j$ broadcasts $\langle v', z' \rangle$, and if a process $p_i \in
    \Pi$ delivers both $\langle v, z \rangle$ and $\langle v', z' \rangle$, then $p_i$ must deliver
    $\langle v, z \rangle$ before $\langle v', z' \rangle$
- **Primary integrity**: If a primary $\rho_e$ broadcasts $\langle v, z \rangle$ and some process
  delivers $\langle v', z' \rangle$ such that $\langle v', z' \rangle$ has been broadcast by
  $\rho_e'$, $e' < e$ then, $\rho_e$ must deliver $\langle v', z' \rangle$ before it broadcasts
  $\langle v, z \rangle$

## Comparison With Casual Atomic Broadcast

- PO atomic broadcast and casual atomic broadcast are not comparable
- _Global primary order_ permits the delivery of $\langle v', z' \rangle$ but not $\langle v, z
  \rangle$
- **Casual order**: If $\langle v, z \rangle \prec_c \langle v', z' \rangle$ and a process $p$
  delivers $\langle v', z' \rangle$, then a process $p$ must also deliver $\langle v, z \rangle$ and
  deliver $\langle v, z \rangle$ before $\langle v', z' \rangle$
- **PO casual order**: Same as casual order except with $\prec_{po}$
- **Strict casuality**: If some process delivers $\langle v, z \rangle$ and $\langle v', z'
  \rangle$, then either $\langle v, z \rangle \prec_{po} \langle v', z' \rangle$ or $\langle v', z'
  \rangle \prec_{op} \langle v, z \rangle$
- PO atomic broadcast repects _PO causal order_ and _strict casuality_

# Algorithm Description

- Three phases: discovery, synchronization, broadcast
- Processes can either by _leaders_ or _followers_
- A leader concurrently executes the primary role and proposes transactions according to the order
  of broadcast calls of the primary
- Followers accept transactions according to the steps of the protocol
- Each process implements leader oracle which provides identifier of prospective leader $l$
- If leader oracle of a process determines that it is the leader, it will execute the leader steps
  of the protocol
- It also needs to complete the synchronization phase to establish leadership
- Persisted variables
  - $f.p$: Last new epoch proposal follower $f$ has acknowledged
  - $f.a$: Last new leader proposal follower $f$ has acknowledged
  - $h_f$: History of follower $f$
  - $f.zxid$: Last accepted transaction identifier in $h_f$

## Discovery

1. (F) A follower sends to the prospective leader $l$ its last promise in a $CEPOCH(f.p)$ message
2. (L) Upon receiving $CEPOCH(e)$ messages from a quorum $Q$ of followers, the prospective leader
   $l$ proposes $NEWEPOCH(e')$ to the followers in $Q$. Epoch number $e'$ is later than any $e$
   received in a $CEPOCH(e)$ message
3. (F) Upon receiving $NEWEPOCH(e')$ message from prospective leader, it updates its $f.p$ and sends
   out an acknowledgement message $ACK-E(f.a, h_f)$ if $f.p < e'$
4. (L) Upon receiving acknowledgements from each follower in $Q$, it selects the history, $I_{e'}$
   of the follower with the largest $f.a$, breaking ties by increasing $f.zxid$

## Synchronization

1. (L) The prospective leader $l$ proposes $NEWLEADER(e', I_{e'})$ to all followers in $Q$
2. (F) Upon receiving the $NEWLEADER(e', T)$ message from $l$, the follower starts a new iteration
   of the protocol if $f.p \neq e'$. Else the follower performs the following actions:

   a. Atomically set $f.a = e'$
   b. For all $\langle v, z \rangle \in T$, it accepts $\langle e', \langle v, z \rangle\rangle$ and
   makes $h_f = T$

   It also acknowledges the $NEWLEADER(e', T)$ message from the leader and starts accepting
   transactions.

3. (L) Upon receiving acknowledgements to $NEWLEADER(e', I_{e'})$ from a quorum of followers, the
   leader sends a commit message to all followers.
4. (F) Upon receiving a commit message from the leader, it delivers all transactions in the initial
   history $I_{e'}$

## Broadcast

1. (L) Leader $l$ proposes to all followers in $Q$ in increasing order of zxid such that for each
   proposal $\langle e', \langle v, z \rangle$, $epoch(z) = e'$ and $z$ succeeds all zxid values
   previously broadcast in $e'$
2. (L) Upon receiving acknowledges from a quorum of followers to a given proposal $\langle e',
   \langle v, z \rangle\rangle$, the leader sends a commit $COMMIT(e', \langle v, z\rangle)$ to all
   followers
3. (F) Follower $f$ initially invokes $ready(e')$ if it is ready
4. (F) Follower $f$ accepts proposals from $l$ following reception order and appends them to $h_f$
5. (F) Follower $f$ commits a transaction $\langle v, z \rangle$ by invoking $abdeliver(\langle v, z
   \rangle)$ once it receives $COMMIT(e', \langle v, z \rangle)$ and it has committed all
   transactions in previous epochs
6. (L) Upon receiving a $CEPOCH(e)$ message from follower $f$ while in broadcast phase, leader $l$
   proposes back $NEWEPOCH(e')$ and $NEWLEADER(e', T)$
7. (L) Upon receiving an acknowledgement from $f$ of the $NEWLEADER(e', T)$ proposal, it sends a
   commit message to $f$ and also makes $f$ part of $Q$

# Zab in Detail

- Process has three states: $ELECTION$, $FOLLOWING$, or $LEADING$
- Delivery protocol is similar to two phase commit
- Co-locate primary and leader on same process
- Receiving $NEWEPOCH$ message will prevent followers in $Q$ from accepting proposal proposals from
  earlier epochs
- Transaction history could be long, so we can optimize by sending only $zxid$ and only copying
  transactions that are needed to get leader up-to-date
- Can further optimize by selecting leader that has the latest epoch and has accepted the
  transaction with the highest zxid among a quorum of processes
- Leader and followers exchange periodic heartbeats
- If leader loses does not receive heartbeats from a quorum of followers within a timeout interval,
  it starts a new iteration of the algorithm
