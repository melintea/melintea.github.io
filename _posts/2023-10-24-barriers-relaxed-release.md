---
layout: post
title: Barriers usage
---


- relaxed
  - for counters 
  - when decrementing: use acquire-release if used for refcounting
- release
  - for an index. Has dependent data that must be visible when updated.
