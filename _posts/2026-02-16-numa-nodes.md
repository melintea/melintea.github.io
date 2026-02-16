---
layout: post
title: NUMA performance notes 
---

- NUMA nodes false sharing is extensive:
  - normal false sharing is of concern for cache lines
  - on NUMA nodes, false sharing is also of concern for pages as pages have to be migrated between nodes for shared access. Avoid accessing the same page from different NUMA nodes.
  - granularity: typical 32k pages on ARM, 4k pages on x86. What about huge pages (2Mb)?
  - keep sharing processes confined to the same NUMA node
  - use per-thread memory
  - no jumping across memory
  - reported by VTune as front-end stall (instruction stall!)

- atomics are slower than older non-NUMA systems; in particular read-modify-write
- use NUMA-aware thread pools

- Linux kernel presure points:
  - TLB: shootdowns (inter-processor interrupts!) by:
    - NUMA page migrations
    - changes in memory mappings
    - memory access modes/protection
    - (explicit) changes to the in-kernel page table
    - use huge pages as relief ```/proc/sys/kernel/mm/transparent_hugepage/enabled``` always (by kernel as it sees fit; in practice: must request hp explicitly)
      - madvise(addr, size); addr is 4K aligned; size is x*2M
      - memory must be reserved/come from mmap()
  - disable NUMA migration:
    - use numactl
    - bind program/thread to a single NUMA node/set of
    - turn off NUMA balancing ```/proc/sys/kernel/numa_balancing``` 0
    - overall program likely to be slower now

- low CPU usage on NUMA at OS-level (idle time). Likely IO bound
  - try lowering the ```kernel.sched_migration_cost_ns```
  - move processes that use network NICs to node 0 (NIC connects to PCIE of node 0
  
- CPU stays high (100%):
  - maybe frequency drop when load is high: not enough wattage/power
  
- CPU simulators? Digital Ocean


@see https://www.youtube.com/watch?v=wGSSUSeaLgA

