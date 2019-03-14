---
geometry: margin=3cm
title: Notes for ``Paxos Made Simple''
---

# The Consensus Algorithm

## The Problem

- Safety requirements
  1. Only a value that has been proposed may be chosen
  2. Only a single value is chosen
  3. A process never learns that a value has been chosen unless it actually has been
- Broad liveness requirement: some proposed value is eventually chosen
- Three roles
  1. Proposers
  2. Acceptors
  3. Learners

## Choosing a Value

- A proposal is chosen if it is accepted by a majority of the agents
- An acceptor must be allowed to accept more than one proposal
- An acceptor is always allowed to respond to _prepare_ requests, but not always to _accept_
  requests
- Requirements
  1. An acceptor must accept the first proposal that it receives
     a) An acceptor can accept a proposal numbered $n$ if and only if it has not responded to a
     _prepare_ request having a number greater than $n$
  2. If a proposal with value $v$ is chosen, then every higher-numbered proposal that is chosen has
  value $v$
     a) If a proposal with value $v$ is chosen, then every higher-numbered proposal accepted by any
     acceptor has value $v$
     b) If a proposal with value $v$ is chosen, then every higher-numbered proposal issued by any
     proposer has value $v$.
     c) For any $v$ and $n$, if a proposal with value $v$ and number $n$ is issued, then there is a
     set $S$ consisting of a majority of acceptors such that either no acceptor in $S$ has accepted
     any proposal numbered less than $n$, or $v$ is the value of the highest-numbered proposal among
     all proposals numbered less than $n$ accepted by the acceptors in $S$
- Algorithm
  1. A proposer chooses a new proposal number $n$ and sends a _prepare_ request to each member of
     some majority set of acceptors, asking it to response with a promise to never again accept a
     proposal numbered less than $n$ and the proposal with the highest number that it has accepted,
     if any. Acceptors should not respond to any _prepare_ requests numbered $n$ if it has already
     responded to another _prepare_ request numbered greater than $n$.
  2. If proposer receives the requested responses from a majority of the acceptors, then it can
     issue a proposal (by sending _accept_ requests) with number $n$ and value $v$, where $v$ is the
     value of the highest-numbered proposal among the responses or is any value selected by the
     proposer if the responders reported no proposals. Acceptors responds to all _accept_ requests
     unless it has already responded to a _prepare_ request having a number greater than $n$.
- If an acceptor ignores a _prepare_ or _accept_ request, then it has already received a _prepare_
  request with a higher number and it should tell the proposer to abandon the proposal for
  performance

## Learning a Chosen Value

- All acceptors respond with their acceptances to distinguished learner which informs other learners
- In the general case, acceptors could respond to some set of distinguished learners

## Progress

- To guarantee progress, a distinguished proposer must be selected as the only one to try to issue
  proposals
- This proposer must use a proposal with a number greater than any already used
- If it sees a request with a higher proposal number, the distinguished proposer can retry and it
  will eventually choose a high enough proposal number
- Reliable algorithm for electing a proposer must use randomness or real time (E.G. timeouts)

## Implementation

- Each process plays the role of proposer, acceptor, and learner
- Algorithm chooses leader which plays role of distinguished proposer and distinguished learner
- Stable storage is need to maintain information that the acceptor must remember between failures
- Different proposals can choose their numbers from a disjoint set of numbers

# Distributed State Machine

- Distributed system can be thought of as a collection of clients that issue commands to a central
  server
- The state of the server can be described using a deterministic state machine
- We can use multiple rounds of Paxos to determine the sequence of commands to execute
- Since multiple rounds can be executed in parallel, some rounds finish out of order
  - When a new designated proposer is chosen, there may be gaps
  - If phase 1 of a missing round ends with no constrained value, then set it to be a no-op action
- Cost in practice is executing phase 2 of consensus algorithm since failures should be rare events
- Configuration of servers can be changed by allowing a leader to get $\alpha$ commands ahead by
  letting the set of servers that execute instance $i + \alpha$ of the consensus algorithm be
  specified by the state after execution of the $i^{th}$ state machine command
