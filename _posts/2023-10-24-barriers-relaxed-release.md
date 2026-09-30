---
layout: post
title: Barriers usage
---


- relaxed
  - for counters 
  - when decrementing: use acquire-release if used for refcounting
- release
  - for an index. Has dependent data that must be visible when updated.
- consume
  - semantics of "not a barrier but a data dependency" (TBD)
  - use acquire instead (that is what the compiler should normally do)

