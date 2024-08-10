---
layout: post
title: Barriers usage
---


    Barrier    Usage
    ---------  ------  
    relaxed    Counters (when incrementing; when decrementing: use acquire-release if used for refcounting).
    release    Index. Has dependent data that must be visible when updated.
