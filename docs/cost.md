# Cost Model

## Overview

This is a reference cost model. Actual costs vary by usage, region, and configuration.

## Key Cost Drivers

- Aurora MySQL in multi-AZ with read replicas costs roughly $400-600/month in dev if you forget to size down instance class; a single Aurora Serverless v2 cluster with auto-pause would cut dev costs by ~70% but adds cold-start latency on first query after idle [inferred]
- Step Functions Standard workflow logs every state transition and charges per state transition (~$0.025/1000); at scale with frequent milestone approvals across thousands of concurrent projects, Express Workflows would reduce orchestration cost by 10x [editorial]

## Estimated Monthly Cost

| Component | Dev (₹) | Staging (₹) | Production (₹) |
|-----------|---------|-------------|-----------------|
| Compute   | ₹2,000–5,000 | ₹8,000–15,000 | ₹25,000–60,000 |
| Database  | ₹1,500–3,000 | ₹5,000–12,000 | ₹15,000–40,000 |
| Networking| ₹500–1,000   | ₹2,000–5,000  | ₹5,000–15,000  |
| Monitoring| ₹200–500     | ₹1,000–2,000  | ₹3,000–8,000   |
| **Total** | **₹4,200–9,500** | **₹16,000–34,000** | **₹48,000–1,23,000** |

> Estimates based on ap-south-1 (Mumbai) pricing. Actual costs depend on traffic, data volume, and reserved capacity.

## Cost Optimization Strategies

- Use Savings Plans or Reserved Instances for predictable workloads
- Enable auto-scaling with conservative scale-in policies
- Use DynamoDB on-demand for dev, provisioned for production
- Leverage S3 Intelligent-Tiering for infrequently accessed data
- Review Cost Explorer weekly for anomalies
