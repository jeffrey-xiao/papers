# papers

A collection of academic papers, articles, and other resources that I plan to read or have read. The
content has a focus on distributed systems.

I have also included my notes on select resources to summarize important takeaways and to help me
better understand the material.

## Blog Posts and Online Articles

- [Consistency Models](https://jepsen.io/consistency)
- [Jepsen Analyses](https://jepsen.io/analyses)
- [Strong Consistency Models](https://aphyr.com/posts/313-strong-consistency-models)
- [The Log: What every software engineering should know about real-time data's unifying abstraction](https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying)

## Consensus

- [Impossibility of Distributed Consensus with One Faulty Process](consensus/impossibility-of-distributed-consensus-with-one-faulty-process.pdf)
  > Fischer, Michael J, Nancy A Lynch, and Michael S Paterson. 1982. “Impossibility of Distributed Consensus with One Faulty Process.” MASSACHUSETTS INST OF TECH CAMBRIDGE LAB FOR COMPUTER SCIENCE.
- [Viewstamped Replication: A New Primary Copy Method to Support Highly-Available Distributed Systems](consensus/viewstamped-replication-a-new-primary-copy-method-to-support-highly-available-distributed-systems.pdf)
  > Oki, Brian M, and Barbara H Liskov. 1988. “Viewstamped Replication: A New Primary Copy Method to Support Highly-Available Distributed Systems.” In _Proceedings of the Seventh Annual Acm Symposium on Principles of Distributed Computing_, 8–17. ACM.
- [Viewstamped Replication Revisited](consensus/viewstamped-replication-revisited.pdf)
  > Liskov, Barbara, and James Cowling. 2012. “Viewstamped Replication Revisited.”
- [The Part Time Parliament](consensus/the-part-time-parliament.pdf)
  > Lamport, Leslie, and others. 1998. “The Part-Time Parliament.” _ACM Transactions on Computer Systems_ 16 (2): 133–69.
- [Paxos Made Simple](consensus/paxos-made-simple.pdf)
  > Lamport, Leslie, and others. 2001. “Paxos Made Simple.” _ACM Sigact News_ 32 (4): 18–25.
- [Paxos Made Live - An Engineering Perspective](consensus/paxos-made-live-an-engineering-perspective.pdf)
  > Chandra, Tushar D, Robert Griesemer, and Joshua Redstone. 2007. “Paxos Made Live: An Engineering Perspective.” In _Proceedings of the Twenty-Sixth Annual Acm Symposium on Principles of Distributed Computing_, 398–407. ACM.
- [Flexible Paxos: Quorum Intersection Revisited](consensus/flexible-paxos-quorum-intersection-revisited.pdf)
  > Howard, Heidi, Dahlia Malkhi, and Alexander Spiegelman. 2016. “Flexible Paxos: Quorum Intersection Revisited.” _arXiv Preprint arXiv:1608.06696_.
- [Zab: High-Performance Broadcast for Primary-Backup Systems](consensus/zab-high-performance-broadcast-for-primary-backup-systems.pdf)
  > Junqueira, Flavio P, Benjamin C Reed, and Marco Serafini. 2011. “Zab: High-Performance Broadcast for Primary-Backup Systems.” In _2011 Ieee/Ifip 41st International Conference on Dependable Systems & Networks (Dsn)_, 245–56. IEEE.
- [ZooKeeper's Atomic Broadcast Protocol: Theory and Practice](consensus/zookeepers-atomic-broadcast-protocol-theory-and-practice.pdf)
  > Medeiros, André. 2012. “ZooKeeper’s Atomic Broadcast Protocol: Theory and Practice.” _Aalto University School of Science_ 20.
- [Vive la Différence: Paxos vs. Viewstamped Replication vs. Zab](consensus/viva-la-difference-paxos-vs-viewstamped-replication-vs-zab)
  > Van Renesse, Robbert, Nicolas Schiper, and Fred B Schneider. 2015. “Vive La Différence: Paxos Vs. Viewstamped Replication Vs. Zab.” *IEEE Transactions on Dependable and Secure Computing* 12 (4). IEEE: 472–84.
- [In Search of an Understandable Consensus Algorithm](consensus/in-search-of-an-understandable-consensus-algorithm.pdf)
  > Ongaro, Diego, and John Ousterhout. 2014. “In Search of an Understandable Consensus Algorithm.” In _2014 {Usenix} Annual Technical Conference ({Usenix}{ATC} 14)_, 305–19.
- [Consensus: Bridging Theory and Practice](consensus/consensus-bridging-theory-and-practice.pdf)
  > Ongaro, Diego. 2014. “Consensus: Bridging Theory and Practice.” PhD thesis, Stanford University.
- [Raft Refloated: Do We Have Consensus?](consensus/raft-refloated-do-we-have-consensus.pdf)
  > Howard, Heidi, Malte Schwarzkopf, Anil Madhavapeddy, and Jon Crowcroft. 2015. “Raft Refloated: Do We Have Consensus?” _Operating Systems Review_ 49 (1): 12–21.

## Causality

- [Time, Clocks, and the Ordering of Events in a Distributed System](causality/time-clocks-and-the-ordering-of-events-in-a-distributed-system.pdf)
  > Lamport, Leslie. 1978. “Time, Clocks, and the Ordering of Events in a Distributed System.” *Communications of the ACM* 21 (7). ACM: 558–65.
- [Timestamps in Message-Passing Systems That Preserve the Partial Ordering](causality/timestamps-in-message-passing-systems-that-preserve-the-partial-ordering.pdf)
  > Fidge, Colin J. 1987. *Timestamps in Message-Passing Systems That Preserve the Partial Ordering*. Australian National University. Department of Computer Science.
- [Virtual Time and Global States of Distributed Systems](causality/virtual-time-and-global-states-of-distributed-systems.pdf)
  > Mattern, Friedemann, and others. 1988. *Virtual Time and Global States of Distributed Systems*. Citeseer.
- [Logical Physical Clocks and Consistent Snapshots in Globally Distributed Databases](causality/logical-physical-clocks-and-consistent-snapshots-in-globally-distributed-databases.pdf)
  > Demirbas, Murat, Marcelo Leone, Bharadwaj Avva, Deepak Madeppa, and Sandeep Kulkarni. 2014. “Logical Physical Clocks and Consistent Snapshots in Globally Distributed Databases.”

## Consistency

- [Brewer's Conjecture and the Feasibility of Consistent, Available, Partition-Tolerant Web Services](brewers-conjecture-and-the-feasibility-of-consistent-available-partition-tolerant-web-services.pdf)
  > Gilbert, Seth, and Nancy Lynch. 2002. “Brewer’s Conjecture and the Feasibility of Consistent, Available, Partition-Tolerant Web Services.” *Acm Sigact News* 33 (2). ACM: 51–59.

## Data Structures

- [Fast set operations using treaps](data-structures/fast-set-operations-using-treaps.pdf)
  > Blelloch, Guy E, and Margaret Reid-Miller. 1998. “Fast Set Operations Using Treaps.” In *SPAA*, 98:16–26.
- [A Skip List Cookbook](data-structures/a-skip-list-cookbook.pdf)
  > Pugh, William. 1998. “A Skip List Cookbook.”
- [Skip Lists: A Probabilistic Alternative to Balanced Trees](data-structures/skip-lists-a-probabilistic-alternative-to-balanced-trees.pdf)
  > Pugh, William. 1990. “Skip Lists: A Probabilistic Alternative to Balanced Trees.” *Communications of the ACM* 33 (6).
- [Less hashing, same performance: Building a better Bloom filter](data-structures/less-hashing-same-performance-building-a-better-bloom-filter.pdf)
  > Kirsch, Adam, and Michael Mitzenmacher. 2006. “Less Hashing, Same Performance: Building a Better Bloom Filter.” In *European Symposium on Algorithms*, 456–67. Springer.
- [Advanced bloom filter based algorithms for efficient approximate data de-duplication in streams](data-structures/advanced-bloom-filter-based-algorithms-for-efficient-approximate-data-de-duplication-in-streams.pdf)
  > Bera, Suman K, Sourav Dutta, Ankur Narang, and Souvik Bhattacherjee. 2012. “Advanced Bloom Filter Based Algorithms for Efficient Approximate Data de-Duplication in Streams.” *arXiv Preprint arXiv:1212.3964*.
- [Cuckoo filter: Practically better than bloom](data-structures/cuckoo-filter-practically-better-than-bloom.pdf)
  > Fan, Bin, Dave G Andersen, Michael Kaminsky, and Michael D Mitzenmacher. 2014. “Cuckoo Filter: Practically Better Than Bloom.” In *Proceedings of the 10th Acm International on Conference on Emerging Networking Experiments and Technologies*, 75–88. ACM.
- [Don't thrash: how to cache your hash on flash](data-structures/dont-thrash-how-to-cache-your-hash-on-flash.pdf)
  > Bender, Michael A, Martin Farach-Colton, Rob Johnson, Russell Kraner, Bradley C Kuszmaul, Dzejla Medjedovic, Pablo Montes, Pradeep Shetty, Richard P Spillane, and Erez Zadok. 2012. “Don’t Thrash: How to Cache Your Hash on Flash.” *Proceedings of the VLDB Endowment* 5 (11). VLDB Endowment: 1627–37.
- [An improved data stream summary: the count-min sketch and its applications](data-structures/an-improved-data-stream-summary-the-count-min-sketch-and-its-applications.pdf)
  > Cormode, Graham, and Shan Muthukrishnan. 2005. “An Improved Data Stream Summary: The Count-Min Sketch and Its Applications.” *Journal of Algorithms* 55 (1). Elsevier: 58–75.
- [A general-purpose counting filter: Making every bit count](data-structures/a-general-purpose-counting-filter-making-every-bit-count.pdf)
  > Pandey, Prashant, Michael A Bender, Rob Johnson, and Rob Patro. 2017. “A General-Purpose Counting Filter: Making Every Bit Count.” In *Proceedings of the 2017 Acm International Conference on Management of Data*, 775–87. ACM.
- [HyperLogLog: the analysis of a near-optimal cardinality estimation algorithm](data-structures/hyperloglog-the-analysis-of-a-near-optimal-cardinality-estimation-algorithm.pdf)
  > Flajolet, Philippe, Éric Fusy, Olivier Gandouet, and Frédéric Meunier. 2007. “Hyperloglog: The Analysis of a Near-Optimal Cardinality Estimation Algorithm.” In *Discrete Mathematics and Theoretical Computer Science*, 137–56. Discrete Mathematics; Theoretical Computer Science.
- [HyperLogLog in practice: algorithmic engineering of a state of the art cardinality estimation algorithm](data-structures/hyperloglog-in-practice-algorithmic-engineering-of-a-state-of-the-art-cardinality-estimation-algorithm.pdf)
  > Heule, Stefan, Marc Nunkesser, and Alexander Hall. 2013. “HyperLogLog in Practice: Algorithmic Engineering of a State of the Art Cardinality Estimation Algorithm.” In *Proceedings of the 16th International Conference on Extending Database Technology*, 683–92. ACM.
- [LSM Tree](data-structures/lsm-tree.pdf)
  > O’Neil, Patrick, Edward Cheng, Dieter Gawlick, and Elizabeth O’Neil. 1996. “The Log-Structured Merge-Tree (Lsm-Tree).” *Acta Informatica* 33 (4). Springer: 351–85.
- [bLSM: A General Purpose LSM Tree](data-structures/blsm-a-general-purpose-lsm-tree.pdf)
  > Sears, Russell, and Raghu Ramakrishnan. 2012. “BLSM: A General Purpose Log Structured Merge Tree.” In *Proceedings of the 2012 Acm Sigmod International Conference on Management of Data*, 217–28. ACM.
- [A Comprehensive Study of Convergent and Commutative Replicated Data Types](data-structures/a-comprehensive-study-of-convergent-and-commutative-replicated-data-types.pdf)
  > Shapiro, Marc, Nuno Preguiça, Carlos Baquero, and Marek Zawirski. 2011. “A Comprehensive Study of Convergent and Commutative Replicated Data Types.” PhD thesis, Inria–Centre Paris-Rocquencourt; INRIA.
- [Efficient Synchronization of State-based CRDTs](data-structures/efficient-synchronization-of-state-based-crdts.pdf)
  > Enes, Vitor, Paulo Sérgio Almeida, Carlos Baquero, and João Leitão. 2018. “Efficient Synchronization of State-Based Crdts.” _arXiv Preprint arXiv:1803.02750_.

## P2P

- [Chord: A Scalable Peer-to-peer Lookup Service for Internet Applications](p2p/chord-a-scalable-peer-to-peer-lookup-service-for-internet-applications.pdf)
  > Stoica, Ion, Robert Morris, David Liben-Nowell, David R Karger, M Frans Kaashoek, Frank Dabek, and Hari Balakrishnan. 2003. “Chord: A Scalable Peer-to-Peer Lookup Protocol for Internet Applications.” _IEEE/ACM Transactions on Networking (TON)_ 11 (1). IEEE Press: 17–32.
- [Kademlia: A Peer-to-peer Information System Based on the XOR Metric](p2p/kademlia-a-peer-to-peer-information-system-based-on-the-xor-metric.pdf)
  > Maymounkov, Petar, and David Mazieres. 2002. “Kademlia: A Peer-to-Peer Information System Based on the Xor Metric.” In _International Workshop on Peer-to-Peer Systems_, 53–65. Springer.

## Systems

- [Dynamo: Amazon's Highly Available Key-value Store](systems/dynamo-amazons-highly-available-key-value-store.pdf)
  > DeCandia, Giuseppe, Deniz Hastorun, Madan Jampani, Gunavardhan Kakulapati, Avinash Lakshman, Alex Pilchin, Swaminathan Sivasubramanian, Peter Vosshall, and Werner Vogels. 2007. “Dynamo: Amazon’s Highly Available Key-Value Store.” In _ACM Sigops Operating Systems Review_, 41:205–20. 6. ACM.
- [Bigtable: A Distributed Storage System for Structured Data](systems/bigtable-a-distributed-storage-system-for-structured-data.pdf)
  > Chang, Fay, Jeffrey Dean, Sanjay Ghemawat, Wilson C Hsieh, Deborah A Wallach, Mike Burrows, Tushar Chandra, Andrew Fikes, and Robert E Gruber. 2008. “Bigtable: A Distributed Storage System for Structured Data.” _ACM Transactions on Computer Systems (TOCS)_ 26 (2). ACM: 4.
- [Cassandra - A Decentralized Structured Storage System](systems/cassandra-a-decentralized-structured-storage-system.pdf)
  > Lakshman, Avinash, and Prashant Malik. 2010. “Cassandra: A Decentralized Structured Storage System.” *ACM SIGOPS Operating Systems Review* 44 (2). ACM: 35–40.
- [Kafka: A Distributed Messaging System for Log Processing](systems/kafka-a-distributed-messaging-system-for-log-processing.pdf)
  > Kreps, Jay, Neha Narkhede, Jun Rao, and others. 2011. “Kafka: A Distributed Messaging System for Log Processing.” In *Proceedings of the Netdb*, 1–7.
- [Spanner: Google's Globally-Distributed Database](systems/spanner-googles-globally-distributed-database.pdf)
  > Corbett, James C., Jeffrey Dean, Michael Epstein, Andrew Fikes, Christopher Frost, JJ Furman, Sanjay Ghemawat, et al. 2012. “Spanner: Google’s Globally-Distributed Database.” In *OSDI*.
- [Spanner: Becoming a SQL System](systems/spanner-becoming-a-sql-system.pdf)
  > Bacon, David F., Nathan Bales, Nico Bruno, Brian F. Cooper, Adam Dickinson, Andrew Fikes, Campbell Fraser, et al. 2017. “Spanner: Becoming a Sql System.” In *Proc. SIGMOD 2017*, 331–43.

## Testing

- [An Analysis of Network-Partitioning Failures in Cloud Systems](testing/an-analysis-of-network-partitioning-failures-in-cloud-systems.pdf)
  > Alquraan, Ahmed, Hatem Takruri, Mohammed Alfatafta, and Samer Al-Kiswany. 2018. “An Analysis of Network-Partitioning Failures in Cloud Systems.” In _13th Usenix Symposium on Operating Systems Design and Implementation Osdi 18)_, 51–68.
- [Why Is Random Testing Effective for Partition Tolerance Bugs?](testing/why-is-random-testing-effective-for-partition-tolerance-bugs.pdf)
  > Majumdar, Rupak, and Filip Niksic. 2017. “Why Is Random Testing Effective for Partition Tolerance Bugs?” _Proceedings of the ACM on Programming Languages_ 2 (POPL). ACM: 46.

## Textbooks

- Operating Systems: Principles and Practice
  - [Volume 1: Kernels and Processes](textbooks/operating-systems-principles-and-practice-vol-1-kernels-and-processes.pdf)
  - [Volume 2: Concurrency](textbooks/operating-systems-principles-and-practice-vol-2-concurrency.pdf)
  - [Volume 3: Memory Management](textbooks/operating-systems-principles-and-practice-vol-3-memory-management.pdf)
  - [Volume 4: Persistent Storage](textbooks/operating-systems-principles-and-practice-vol-4-persistent-storage.pdf)
- [Designing Data-Intensive Applications](textbooks/designing-data-intensive-applications.pdf)
- [Distributed Systems](textbooks/distributed-systems.pdf)
- [Distributed Systems for Fun and Profit](http://book.mixu.net/distsys)
- [Readings in Database Systems](textbooks/readings-in-database-systems.pdf)
