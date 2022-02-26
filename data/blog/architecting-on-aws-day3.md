---
title: 'Architecting on AWS - Day 3, Sydney'
date: 2014-03-26 02:00
draft: false
tags: ['AWS']
summary: 'Notes from the Architecting on AWS course held in Sydney'
---

These are my notes from day 3 of the Amazon 'Architecting on AWS' course which was run in Sydney in March 2014. The trainer/presenter was John Rotenstein from AWS.

# Simple Queue Service

- Regionally based
- Automatically replicated throughout AZs
- Concept of dead letter queues
- Eventually consistent
- FIFO, kind of...
- "Visibility Timeout" is the invisibility timeout
  - Can be overridden in code
- Messages can be retained for up to 14 days
- Messages are base64 encoded
- Important for loosely coupled services

# Simple Notification Service

1. Publish a message
2. Subscribers get the message

- Email
- HTTP/HTTPS
- SMS (USA only)
- SQS
- Push to iOS and Android

- SNS can be used to fan out messages. e.g. send same message to multiple SQS queues
- You wouldn't use SNS to email customers

- 1 million free mobile push

# Simple Workflow Service

- Orchestration tool

- Decider
  - Business logic
  - Decision logic
- Activity workers do work assigned by the Decider

# Simple Email Service

- Bulk and transactional email service
- No SPAM is allowed
- SES not available in Australia

# Cloud Search

- Search engine for your web app
- Runs as an instance
- Auto scales
- Managed Apache Solr

# Migrating Applications

- [Whitepaper](http://media.amazonwebservices.com/CloudMigration-main.pdf)
- Build a cloud strategy

  - New apps
    - Build as cloud ready
  - Existing apps
    - "No brainer"
    - Planned phase

- Run a PoC
- Engage AWS to help

- Migrate your data
  - Riverbed
  - Aspera

## Cloud benefits

- Zero upfront costs (OPEX vs CAPEX)
  - Simple calculator
  - [TCO calculator](http://tco.2ndwatch.com)
- On demand provisioning
- Instant scalability
- Business agility
- Developer productivity
- etc..
