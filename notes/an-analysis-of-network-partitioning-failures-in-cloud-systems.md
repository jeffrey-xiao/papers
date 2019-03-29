---
geometry: margin=3cm
title: "Notes for ``An Analysis of Network-Partitioning Failures in Cloud Systems''"
---

# Open Questions About System Failures

- Failure Impact
- Ease of Manifestation
- Partial Network Partitions
  - Complete partition: The system is split into two disconnected groups.
    - Can manifest because of ToR switch failure.
    - Loss of connectivity between data centers in geo-replicated systems.
  - Partial partition: The partition affects some, but not all nodes in the system.
    - Loss of connectivity between two data centers, while are reachable by a third center.
    - Due to inconsistencies in switch-forwarding rules.
  - Simplex partition: Traffic flows only in one direction.
    - Inconsistent forwarding rules or hardware failures.
- Testability

# Common Presumptions

- Systems are robust enough to tolerate network partitioning, but 88% of the failures can occur by
  isolating a single node.
- Limiting client access to one side of a network partition will eliminate the possibility of a
  failure, but 64% of the failures required no client access at all or client access to one side of
  the network partition.

# Important Takeaways

- Need end-to-end testing of complete distributed protocols to detect failures instead of unit
  tests with mocks.
- System designers must consider the impact of a network partition fully on all system operations,
  including asynchronous client operations and offline internal operations.
- Testers should pay close attention to timing.
- Top-of-Rack (ToR) switch failure is important.

# Network Partitioning Testing Framework (NEAT)

## Limitations

1. Representativeness of the selected systems.

- Systems studied follow diverse designs.

2. Sampling bias.

- The selection of tickets may bias the results.
- A network partition that isolates a single node can trigger the same failures as caused by a
  single node crash.

3. Observer error.

- All failures reviewed by two team members and discussed in a group meeting before agreement was
  reached.

## General Findings

1. 80% of failures have a catastrophic impact. Data loss most common (27%)
2. 90% of failures were silent.
3. 21% of failures lead to permanent damage to the system that persists even after the network
   partition heals.
4. Leader election, configuration change, request routing, and data consolidation are most
   vulnerable aspects.
5. 64% of failures either do not require any client access or require client access to only one side
   of the network partition.
6. 69% are caused by complete partitions, 29% are caused by partial partitions.
7. Required an additional three or fewer inputs for failure condition to manifest.
8. All failures that involve multiple events only manifest if the events happen in a specific order.

- 84% of manifestation sequences start with network-partitioning fault.
- 27.7% of sequences, the order of the rest of events is not important.
- Probability is still high, since network partitioning can occur for hours and all possible
  permutations of these common events are possible.

9. 88% of failures manifest by isolating a single node, 45% of failures manifest by isolating any
   replica.
10. 80% of failures are deterministic (62%) or have known timing constraints (18%)
11. 47% of failures required redesigning a system mechanism.
12. All failures can be reproduced on a cluster of five nodes with the majority (83%) of the
    failures being reproducible with three nodes only.
13. 93% of failures can be reproduced by using a fault injection framework such as NEAT.
14. Implicit assumptions made in studied systems are untrue.
15. Lack of adequate testing tools.

## NEAT API

- `ISystem` interface - methods to start, obtain the status of, and shut down the target system.
- `Client` - provides wrappers around the client API.
- test workload and verification code.

# Questions

- How does electing bad leader lead to data loss? Leader with incomplete log? (Finding 4)
- Client access before network partition, but race condition? (Finding 5)
- Elaborate on RethinkDB configuration change failure. (Finding 5)
- Isolating one node is equivalent to one node crashing. (Finding 9)
- Are there failures that happen with only a large cluster? (Finding 12)
- How to test non-deterministic failures or with short vulnerability intervals?
