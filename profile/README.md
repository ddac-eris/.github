# Eris

**A real-time multiplayer coding game, built twice on AWS: once as a monolith, once as an event-driven serverless system.**

Group project for **CT071-3-3-DDAC: Designing and Developing Applications on the Cloud** (Asia Pacific University).

## The game

- Up to **10 players** compete in a **60-minute** match on a **5×5 board** of programming questions.
- Solve a question in **Python** to claim its block. The **earliest correct submission**, by server receive time, wins it.
- Every submission is judged against 2 sample and 100 hidden test cases: `AC` · `WA` · `CE` · `RE` · `TLE`.
- The player with the most blocks when time runs out wins.

## Two architectures, one game

The same game runs on two architectures. We compare them with CloudWatch metrics, X-Ray traces, load tests (k6, 500 concurrent players) and cost data.

| | Monolithic | Event-driven serverless |
|---|---|---|
| Compute | One Fastify app on an EC2 Auto Scaling group behind an ALB + CloudFront | Lambda services (Player, Match, Submission, Realtime, Judge) |
| Communication | In-process calls, BullMQ queue, Redis pub/sub | Amazon EventBridge bus, SQS, EventBridge Pipes |
| Data | RDS PostgreSQL + ElastiCache Redis | *To be decided:* DynamoDB or RDS |
| Real-time | WebSockets on the app servers | API Gateway WebSocket API |
| Judge | Worker runs the judge container with `docker run` | Judge Lambda (same container image) in an isolated subnet |

Shared by both: Amazon Cognito, S3 for test cases, ECR for the judge image, AWS CDK for infrastructure. Region `us-east-1`.

## Repositories

| Repository | What it is | Stack |
|---|---|---|
| [`eris-core`](https://github.com/ddac-eris/eris-core) | Game rules shared by both architectures: claim rule, ranking, board, event types | TypeScript |
| [`eris-judge`](https://github.com/ddac-eris/eris-judge) | Sandboxed Python judge; one container image for both architectures | Python 3.12, Docker |
| [`eris-monolithic`](https://github.com/ddac-eris/eris-monolithic) | The monolithic app | TypeScript, Fastify, Prisma, BullMQ, CDK |
| [`eris-serverless`](https://github.com/ddac-eris/eris-serverless) | The event-driven serverless app | TypeScript, Lambda, EventBridge, CDK |
| [`eris-shared-infra`](https://github.com/ddac-eris/eris-shared-infra) | AWS resources both apps use: Cognito, S3 test cases, ECR | TypeScript, CDK |

```
eris-shared-infra ──► Cognito · S3 test cases · ECR (IDs published to SSM)
        │
        ├──► eris-monolithic ──┐
        └──► eris-serverless ──┤── both depend on eris-core (game rules)
                               └── both run the eris-judge image
```

**Deploy order:** `eris-shared-infra` → push the `eris-judge` image → `eris-monolithic` / `eris-serverless`.

## Team

<!-- TODO: add team members -->
| Name | Role |
|---|---|
| Chan Yong Liang| Backend Developer, Cloud Engineer|
| Kok Jia Yin| Backend Developer, Cloud Engineer|
| Chong Han Hui Isaac| Full Stack Developer|
| Tan Po Hui| Full Stack Developer|
| Low Tian Wei| Full Stack Developer|

---

<sub>University coursework, built for learning and comparison. Not a production service.</sub>
