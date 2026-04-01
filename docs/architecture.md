# Architecture Notes

## Overview

The stack deploys a VPC (10.36.0.0/16, multi-AZ) fronting an ECS Fargate cluster behind an ALB, with Aurora MySQL handling transactional data (profiles, bids, milestones) and DynamoDB holding real-time messages behind a two-GSI access pattern (conversationId PK + timestamp SK, plus userId and receiverId GSIs for inbox queries) [from-code]. Two separate Cognito User Pools split freelancer and client identities with custom attributes per role [from-code]. A Node.js Lambda handles payment webhooks with a DLQ and VPC attachment [from-code], while Step Functions orchestrates the full project lifecycle from BiddingOpen through PaymentProcessed, integrating Lambda, SES, and SNS at state transitions [from-code]. CloudFront sits in front of S3 portfolio storage via OAI [from-code], and the entire stack is environment-parameterized through CDK context with consistent resource prefix tagging [from-code].

## Key Decisions

- Two Cognito User Pools give clean role separation and independent custom attribute schemas, but every cross-pool operation (a freelancer acting as a client) requires token exchange logic or a separate identity linking layer — adds ~2 weeks of auth middleware work [inferred]
- Aurora MySQL in multi-AZ with read replicas costs roughly $400-600/month in dev if you forget to size down instance class; a single Aurora Serverless v2 cluster with auto-pause would cut dev costs by ~70% but adds cold-start latency on first query after idle [inferred]
- DynamoDB on-demand billing absorbs traffic spikes cleanly but at 8,000 active users with frequent messaging, provisioned capacity with auto-scaling typically runs 30-40% cheaper once traffic patterns stabilize [editorial]
- NAT Gateway count is correctly env-gated (1 for dev, 2 for prod), but a single NAT Gateway in dev means all private subnet egress flows through one AZ — Lambda VPC cold starts will occasionally time out if that AZ has issues [from-code]
- Step Functions Standard workflow logs every state transition and charges per state transition (~$0.025/1000); at scale with frequent milestone approvals across thousands of concurrent projects, Express Workflows would reduce orchestration cost by 10x [editorial]
- CloudFront OAI is used instead of Origin Access Control (OAC) — OAI is the legacy approach and AWS recommends OAC for new distributions; OAI will eventually be deprecated, creating a future migration burden [inferred]