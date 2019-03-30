---
geometry: margin=3cm
title: "Notes for ``ZooKeeper's Atomic Broadcast Protocol: Theory and Practice''"
---

# Implementation

- Fast Leader Election (FLE): attempt to elect the peer that has the most up-to-date history as the
  leader from a quorum of processes
- No clear distinction between phase 1 (discovery) and phase 2 (synchronization)

## Recovery Phase

1. When leader receives $FOLLOWERINFO(f.zxid)$ from a follower, it sends a $NEWLEADER$ response with
   the leader's most recent transaction id
2. If follower is too out-of-date, leader sends $SNAP$ message with the complete snapshot of the
   state. If the follower is a bit out-of-date, the leader will send a $DIFF$ message containing
   only state the client is missing. Else, the follower is ahead and the leader will send a $TRUNC$
   to truncate its history.
3. After processing $SNAP$, $DIFF$, or $TRUNC$, the follower sends a $ACKNEWLEADER$ message back to
   the leader to acknowledge it.
4. Once the leader has received acknowledgements from a quorum of followers, it proceeds to phase 3.

## Fast Leader Election

- Assume that peer with most recent proposed transaction must also have the most recent committed
  transaction
- FLE aims to elect a leader with the highest last $zxid$ among a quorum
- Peers exchange notifications about their votes and update their vote when peer with more recent
  history is discovered
- Different executions of FLE are distinguished by a round number which is incremented when FLE
  restarts
- Vote for peer $p_i$ is represented by $(z_i, i)$ where $z_i$ is the last $zxid$ in $p_i$
- A notification is represented by $(\textrm{vote}, \textrm{id}, \textrm{state}, \textrm{round})$
- Steps
  1. A peer starts by voting for itself and sends notification of its vote to all other peers
  2. Upon receiving a notification, it handles it according to the state of the peer that sent it.
     - If the state is _election_, current peer updates its view of the other peers' votes and
       update its own vote in case the received vote is better
     - If the state is not _election_, current peer updates its view of follower-leader
       relationships
     - Notifications from previous rounds are ignored
