---
geometry: margin=3cm
title: Notes for ``In Search of an Understandable Consensus Algorithm''
---

# Abstract

- Overlapping majorities to guarantee safety
- State reduction
- Key elements of consensus
  - Leader election
  - Log replication
  - Safety
- Key features
  - Strong leader: Log entries flow form leader to other servers
  - Leader election: Randomized timers to elect leaders
  - Membership changes: Majorities of two configurations overlap during transitions
  - Reduced number of states to consider: System is more coherent

# Replicated State Machines

- State machine on a collection of servers compute identical copies of same state
- Used to solve fault tolerance problems in distributed systems
- Replicated state machine for leader election
- Configuration information to survive leader crashes
- Replicated State Machine is implemented as a replicated log -- contains commands in
  deterministic order
- Consensus algorithm keeps logs consistent
  - Should be safe under non-Byzantine conditions
  - Should be functional as long as any majority of servers are operational and can communicate
    with each other and with clients
  - Should not depend on timing
  - A command can complete if majority of cluster has responded

# Raft Consensus Algorithm

- Algorithm for managing a replicated log
- First elect a leader
  - Manages replicated log
  - Accepts log entries
  - Replicates them on other servers
  - Tells servers when it is safe to apply log entries to state machine

# Raft basics

- Server can be in three states: leader, follower, candidate
- Normal operation: One leader, all followers
- Candidate is used to elect new leader
- Terms are logical clocks in Raft
- Servers communicate via Remote procedure calls (RPCs)
  - `RequestVote` are initiated by candidates during elections
  - `AppendEntries` are initiated by leaders to replicate log entries

# Leader election

- Every server is initially a follower
- Leaders send periodic heartbeats (`AppendEntries` that carry no log entry)
- If a follower receives no communication, then timeout and new election begins
  - Increments term
  - Issues `RequestVote` in parallel to other servers
  - Continues until it wins, another leader is established, or timeout
  - Wins election if it receives from majority of servers
  - Each server will vote for at most one candidate on first-come-first-served basis
  - Once candidate wins election, it sends heartbeat messages to establish authority
  - Election could timeout, end with split votes, another round of elections would begin
    - Randomly timeout with intervals from 150-300ms to prevent infinite cycles

# Log replication

- Leader takes client request and appends the command to its log as a new entry
- Issues `AppendEntries` RPC in parallel to other servers
- When entry has been safely replicated, leader applies entry to its state machine and returns
  result to client
  - A log entry is called commitment when it is safe to apply it to the state machines
  - The leader decides if a log entry will be committed and it will happen when the leader
    successfully replicates the log entry to a majority of the servers
  - The leader includes the highest index it knows can be committed to `AppendEntries` RPC
- If followers crash or run slowly, or if network packets are lost, leader retries `AppendEntries`
  RPCs indefinitely until all followers store log entries
- Leaders force follows to duplicate its log
  - Conflicting entries will be overwritten
  - Leader must find latest log entry where two logs agree, delete log entries after that point and
    send follower all the leader's entries after that point

# Log Matching Property

- If two entries in different logs have the same index and term, then they store the same command
  - Fact follows that entries are never reordered and leaders only commit one log for a given
    index and term
- If two entries in different logs have the same index and term, then the logs are identical in all
  preceding entries
  - Prove via induction (when sending `AppendEntries` RPC, also send index and term of preceding
    entry)

# Committing Entries from Previous Terms

- A leader will never commit an entry from a previous term, only entries from current term
- If entry of current term is committed, then entries of previous terms are also committed

# Election Restriction

- Voters will reject votes if voter is more "up-to-date" than the candidate
