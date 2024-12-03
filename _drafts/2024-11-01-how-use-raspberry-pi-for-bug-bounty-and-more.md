---
layout: single
title:  "Raspberry Pi for bug bounties and more"
author_profile: false
share: false
toc: false
read_time: false
related: false 
classes: Archive
mermaid: true 
---

## Old Network

<div class="mermaid">
graph BT;
    DNS-->Internet;
    subgraph one[ ]
    Router-- "No Control" -->DNS;
    end
    Desktop-->Router;
    Laptop-->Router;
    eReader-->Router;
    air[Air Quality Monitor]-->Router;
    Router;

</div>

## The New Network

