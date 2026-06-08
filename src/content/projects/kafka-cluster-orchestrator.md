---
title: "Kafka Cluster Orchestrator Microservice"
description: "A microservice for Kafka cluster build, patch, and maintenance workflows that reduced build and patch time from 2 days to 2 hours."
problem: "Kafka cluster build and patch workflows were taking too long and required too much manual coordination across maintenance activities."
role: "I created an orchestrator microservice for Kafka cluster build, patch, and maintenance operations, turning repeated infrastructure tasks into a controlled service workflow."
tech:
  - "Kafka"
  - "Java"
  - "Python"
  - "Shell"
  - "Linux"
  - "Docker"
impact: "Cut Kafka cluster build and patch time from 2 days to 2 hours, improving platform maintenance speed and operational consistency."
featured: true
order: 2
---

## Case study

Kafka clusters need regular build, patch, and maintenance work, but manual execution slows down infrastructure teams and increases operational risk. A task that depends on many small steps can easily become a multi-day process when coordination, checks, and execution are handled manually.

I created a Kafka cluster orchestrator microservice to make those workflows repeatable. The service centered maintenance activity around a controlled workflow rather than ad hoc execution. That made it easier to standardize build and patch operations and reduce the amount of manual effort needed from engineers.

My contribution was focused on the platform-service layer: designing and building the microservice that coordinated cluster lifecycle actions for build, patch, and maintenance. The work combined Kafka platform knowledge with automation across Linux-based infrastructure.

The result was a reduction in build and patch time from 2 days to 2 hours. For a streaming platform, that kind of reduction matters because maintenance speed affects reliability, capacity planning, and how quickly the platform can respond to required changes.
