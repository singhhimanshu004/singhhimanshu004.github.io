---
title: "Kafka Canary Health Checks and Cluster Health Index"
description: "A monitoring framework that produced a real-time Kafka cluster health index used by 12,000+ applications for DR failover decisions."
problem: "Application teams needed an operationally useful view of Kafka cluster health to support disaster-recovery failover decisions."
role: "I created a Kafka canary health-check monitoring framework that produced a real-time cluster health index for application failover use cases."
tech:
  - "Kafka"
  - "Kafka Streams"
  - "Python"
  - "Java"
  - "Observability"
  - "Linux"
impact: "The real-time cluster health index was used by 12,000+ applications for failover to a DR cluster."
featured: true
order: 3
---

## Case study

Kafka health is not only about whether a process is running. For application teams, the more important question is whether the cluster is healthy enough to depend on during normal operations or whether failover to a disaster-recovery cluster is required.

I created a monitoring framework for canary health checks on Apache Kafka clusters. The framework produced a real-time cluster health index, giving consuming applications a practical signal for DR failover decisions.

The project required thinking about health from the perspective of application dependency, not only infrastructure status. A canary-based approach helped represent whether the streaming path was working in practice, while the health index translated that signal into something easier for downstream applications to use.

The impact was broad: 12,000+ applications used the cluster health index for failover to a DR cluster. That scale made clarity and reliability of the signal especially important, because the output was part of operational decision-making for many dependent systems.
