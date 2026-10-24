# Eris

**Eris** is a multiplayer competitive-programming game, built for the APU module *Designing and Developing Applications on the Cloud* (CT071-3-3-DDAC).

Up to 10 players share a 5×5 board of Python questions in a 60-minute match. The first player whose code the server records as Accepted claims that block, and claimed blocks stay taken. The player with the most blocks wins.

We build the same game twice on AWS and compare performance, scalability, availability and cost:

| | Modular monolith | Event-driven serverless |
|---|---|---|
| Compute | One Fastify (TypeScript) image on EC2, run as three Auto Scaling groups: `api`, `ws` and `worker` | AWS Lambda per service around an Amazon EventBridge bus |
| Data | Amazon RDS for PostgreSQL (Multi-AZ), Amazon ElastiCache for Redis | Amazon DynamoDB (one table per service) |
| Messaging | Amazon SQS (judge jobs, results, drafts) | EventBridge + SQS, outbox via DynamoDB Streams and EventBridge Pipes |
| Realtime | WebSocket tier + Redis pub/sub | API Gateway WebSocket API |
| Judge | `docker run` of the judge image per job | Judge Lambda (container image) |

Both use Amazon Cognito, Amazon CloudFront and Amazon Route 53, and run the same judge image from Amazon ECR.

## Repositories

| Repo | What it is |
|---|---|
| [`eris-backend`](../../eris-backend) | npm workspaces monorepo: shared game rules (`packages/core`), the monolith (`apps/monolithic`), the serverless app (`apps/serverless`) and shared AWS CDK infrastructure (`infra/shared`) |
| [`eris-judge`](../../eris-judge) | Python 3.12 judge: runs code against test cases in a sandbox and returns a verdict (AC/WA/CE/RE/TLE); used by both architectures |
| [`eris-frontend`](../../eris-frontend) | Next.js frontend with the Monaco editor; the same build runs against both backends |

## Tech

TypeScript · Node.js 24 · Fastify · Prisma · Python 3.12 · AWS CDK · GitHub Actions

## Team

<!-- TODO: add team members -->
| Name | Role |
|---|---|
| Chan Yong Liang| Joker|
| Kok Jia Yin| Joker|
| Chong Han Hui Isaac| Joker|
| Tan Po Hui| Joker|
| Low Tian Wei| Joker|

---

<sub>University coursework, built for learning and comparison. Not a production service.</sub>
