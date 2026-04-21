# SRE Interview Preparation Guide
> **Role:** Site Reliability Engineer  
> **Background:** 5 years DevOps | AWS | Kubernetes | Terraform | CI/CD  
> **Platform:** Cloud-based Payment Management System (Multi-tenant B2B SaaS)

---

## Table of Contents

1. [Tell me about yourself](#1-tell-me-about-yourself)
2. [What is your application architecture?](#2-what-is-your-application-architecture)
3. [What business does your application fulfill?](#3-what-business-does-your-application-fulfill)
4. [What companies are you serving from your application?](#4-what-companies-are-you-serving)
5. [What is the request flow?](#5-what-is-the-request-flow)
6. [What happens if the Payment Service goes down mid-transaction?](#6-payment-service-failure-mid-transaction)
7. [How do you perform a Kubernetes version upgrade?](#7-kubernetes-version-upgrade)
8. [What types of autoscaling exist — HPA vs VPA?](#8-autoscaling-hpa-vs-vpa)
9. [Tell me about GitHub Actions](#9-github-actions)
10. [New Relic Features](#10-new-relic-features)
11. [One Production Issue and RCA](#11-production-issue-and-rca)
12. [How have you implemented CI/CD?](#12-cicd-implementation)
13. [How does NGINX Ingress handle traffic during a deployment?](#13-nginx-ingress-during-deployment)
14. [How do you ensure no message is lost in SQS?](#14-no-message-loss-in-sqs)
15. [What is your RTO and RPO for database failure?](#15-rto-and-rpo-for-database-failure)
16. [How do you handle noisy neighbour (tenant load affecting others)?](#16-noisy-neighbour-problem)
17. [How do you scale your EKS cluster during peak traffic?](#17-eks-cluster-scaling-during-peak)
18. [How do you manage different SLAs for different clients?](#18-managing-different-slas)
19. [What is your deployment strategy — do you deploy to all tenants at once?](#19-deployment-strategy)
20. [How do you ensure data consistency across microservices?](#20-data-consistency-across-microservices)
21. [What are your SLOs and how did you define them?](#21-slos-and-how-defined)
22. [How do you handle a database failover in production?](#22-database-failover-in-production)
23. [How does traffic flow from the user to your microservices?](#23-traffic-flow)
24. [How do you handle a service going down — failover strategy?](#24-service-failover-strategy)
25. [How do you manage secrets and configs across environments?](#25-secrets-and-config-management)
26. [What are your SLIs and SLOs for this system?](#26-slis-and-slos)
27. [Why do you want to move to SRE from DevOps?](#27-why-move-to-sre-from-devops)

---

## 1. Tell me about yourself

**Answer:**

"I'm Sravan, a DevOps Engineer with around 5 years of experience, currently working at TCS in the Cloud & Platform Operations team. My team is a cross-functional group of about 8 engineers — including 3 DevOps engineers, 2 cloud architects, a DBA, and a couple of application developers — all working together to ensure platform reliability and delivery velocity.

In terms of my personal contribution, I primarily **own the Kubernetes infrastructure** — we run production workloads on Amazon EKS, and I'm responsible for cluster health, auto-scaling policies, ingress configuration, and high availability. I also **lead our CI/CD pipeline setup** using Jenkins and GitHub Actions, which has significantly reduced our deployment cycle time.

Beyond that, I handle **Terraform-based infrastructure provisioning** for our AWS environments and I've written several Python automation scripts that eliminated a lot of manual toil for the team. On the observability side, I manage our **New Relic dashboards and alerting**, and we've been consistently maintaining 99.9% uptime in production.

I also participate actively in **incident management and RCA** — which is something I've come to enjoy because it sharpens your ability to think under pressure and build more resilient systems. That's actually one of the reasons I'm now targeting an SRE role — I want to be more deeply involved in reliability engineering, SLO/SLI definition, and building systems that are self-healing by design."

**Key Points:**
- Team size: 8 engineers (realistic for TCS mid-sized ops team)
- Always end with SRE motivation
- Say "I own" not "I worked on" — sounds senior
- Mention uptime (99.9%) — quantified impact

---

## 2. What is your application architecture?

**Answer:**

"Our application follows a **microservices architecture** hosted entirely on AWS. Let me walk you through the layers:

At the **frontend**, we have a React-based web application served via **CloudFront** as the CDN, backed by an **S3 bucket** for static assets. All traffic comes in through **Route 53** for DNS resolution.

The **API layer** consists of multiple independent microservices built in **Java**. These services are containerized using **Docker** and deployed on **Amazon EKS**. Each microservice has its own deployment, and we use an **NGINX Ingress Controller** to route traffic to the appropriate service based on path or hostname rules.

For **inter-service communication**, we use REST APIs for synchronous calls and **SNS/SQS** for asynchronous event-driven communication — for example, payment event triggers go through SQS to decouple the services.

On the **database layer**, we use **PostgreSQL on Amazon RDS** with Multi-AZ enabled for high availability. Some services that need fast read access use **ElastiCache (Redis)** as a caching layer.

For **authentication**, we use **Amazon Cognito** — it handles user pools, token generation, and OAuth flows, so our services don't manage auth logic themselves.

All infrastructure is **provisioned via Terraform** — we follow a modular structure with separate state files per environment — dev, staging, and production.

On the **observability side**, we use **New Relic** for APM, infrastructure metrics, and alerting. We have dashboards per microservice tracking error rates, latency, and throughput — which directly map to our SLIs."

**Architecture Summary:**
```
User
 │
 ▼
Route 53 → CloudFront → S3 (React Frontend)
 │
 ▼
ALB (Application Load Balancer)
 │
 ▼
EKS Cluster
 ├── Payment Service   ──┐
 ├── Settlement Service──┼──► RDS PostgreSQL (Multi-AZ)
 └── Notification Svc  ──┘         │
          │                    ElastiCache (Redis)
          ▼
       SNS / SQS (Async messaging)
          │
    Amazon Cognito (Authentication)

Monitoring: New Relic (APM + Infra)
IaC: Terraform
CI/CD: Jenkins + GitHub Actions
```

---

## 3. What business does your application fulfill?

**Answer:**

"Our application is a **cloud-based payment management system** — it's essentially a **B2B fintech platform** that helps enterprises manage their payment workflows end to end.

At a high level, the platform serves three core business needs:

**First, Payment Processing** — businesses can initiate, track, and reconcile payments through the platform. The microservices architecture ensures that each payment function — authorization, settlement, and reconciliation — is handled independently, so a failure in one doesn't cascade to others.

**Second, User & Access Management** — different users have different roles. A finance manager sees different capabilities than a regular employee. We handle this through **Amazon Cognito** with role-based access control.

**Third, Audit & Compliance** — in the fintech space, every transaction needs to be traceable. Our system maintains detailed logs of every payment event, who triggered it, and what the outcome was. This is critical for regulatory compliance and internal audits.

From an SRE perspective, this is a **high-stakes, zero-tolerance application** — any downtime or latency spike directly impacts business revenue and customer trust. That's why we maintain **99.9% uptime**, have defined SLOs around payment success rate and transaction latency, and have runbooks for every major failure scenario."

> 💡 **Power Statement:** *"In a payment system, reliability is not a technical goal — it's a business obligation. That mindset is what drove me toward SRE."*

---

## 4. What companies are you serving?

**Answer:**

"Our platform is a **multi-tenant B2B SaaS product**, so we serve multiple enterprise clients from the same infrastructure — but with strict data isolation between tenants.

We primarily serve clients across **three industry verticals:**

**Retail & E-commerce** — largest clients by transaction volume. They use our platform to manage vendor payments, supplier settlements, and customer refund workflows. During peak seasons, transaction volumes spike **3x to 5x** — so auto-scaling on EKS is critical.

**Logistics & Supply Chain** — for freight payment reconciliation and multi-party settlement. These clients care a lot about **audit trails and reporting accuracy**.

**Mid-size Financial Services** — for internal treasury operations and inter-department payment flows. These are the most **compliance-sensitive** clients.

In terms of scale, we handle payments for around **15–20 enterprise clients**, with combined transaction volume of **50,000–80,000 transactions per day** in normal load, significantly higher during peak.

From an SRE standpoint, the multi-tenant nature means we have to be very careful about the **blast radius** of any incident. That's why we have **namespace-level isolation on EKS**, resource quotas per tenant, and separate alerting thresholds per client SLA."

> 💡 **Power Statement:** *"Because we're multi-tenant, a single incident can have a wide blast radius — so our reliability practices are designed not just for uptime, but for controlled fault isolation."*

---

## 5. What is the request flow?

**Answer:**

"Let me walk you through the request flow end to end:

1. **DNS** — User's browser hits **Route 53** for DNS resolution
2. **Static assets** — HTML/CSS/JS served from **CloudFront + S3** (never touches backend)
3. **Authentication** — API calls go to **Amazon Cognito** which validates credentials and issues a JWT token
4. **Load Balancing** — Authenticated request hits **ALB** with NGINX Ingress Controller on EKS behind it
5. **Routing** — Ingress routes by URL path: `/api/payments` → Payment Service, `/api/settlements` → Settlement Service
6. **Sync communication** — Services talk via REST for real-time responses
7. **Async communication** — Events pushed to **SQS/SNS** for decoupled flows (e.g., post-payment triggers settlement and notification independently)
8. **Data layer** — Persistent data goes to **RDS PostgreSQL Multi-AZ**; frequent reads served from **ElastiCache Redis**
9. **Observability** — Every service emits metrics/traces to **New Relic** and logs to **CloudWatch**"

---

## 6. Payment Service Failure Mid-Transaction

**Answer:**

"This is one of our most critical failure scenarios. We handle it at multiple levels:

**Prevention — Multiple replicas:** Payment Service runs as **3+ pods in EKS**. If one pod crashes, Kubernetes routes traffic to healthy pods within seconds via liveness probes.

**In-flight requests — Connection draining:** ALB has connection draining enabled with a 30-second timeout. In-flight requests get a chance to complete or gracefully fail before pod termination.

**Data integrity — ACID transactions:** A payment record is only committed once all steps — authorization, ledger debit, and event publish — succeed atomically in **RDS PostgreSQL**. If the service crashes mid-way, the DB transaction rolls back automatically. No phantom debits.

**Message reliability — Outbox Pattern:** Before publishing a payment event to SQS, we write it to an outbox table in the **same DB transaction**. A separate process polls this table and publishes to SQS. Even if the service crashes right after DB commit, the event will still be published — no message is lost.

**User experience — Pending status:** We return a `pending` status instead of a hard failure. The client polls a status endpoint. Once the service recovers and processes the outbox, the transaction completes and the user is notified.

**Alerting:** New Relic fires within 60 seconds if Payment Service error rate exceeds 1% or pod count drops below minimum. On-call engineer is paged, checks the runbook, and can scale up pods or roll back deployment in under 5 minutes."

| Failure Point | Protection Mechanism |
|---|---|
| Single pod crashes | Kubernetes liveness probe + multiple replicas |
| In-flight request lost | ALB connection draining (30s timeout) |
| DB write partial | ACID transactions on RDS PostgreSQL |
| SQS event not published | Outbox pattern — retried on recovery |
| Full service outage | Async status polling + Notification Service |
| No one notices | New Relic SLO breach alert → PagerDuty |

> 💡 **Power Statement:** *"We design for failure, not against it — our assumption is that any service can go down at any time, so we make sure a crash is recoverable, not catastrophic."*

---

## 7. Kubernetes Version Upgrade

**Answer:**

"We follow a structured, phased approach to EKS version upgrades to ensure zero downtime.

**Phase 1 — Planning:**
- Check AWS EKS release notes and Kubernetes changelog for deprecated APIs, breaking changes
- Validate all our Helm charts and manifests against the new API versions using `kubectl convert` or `Pluto` tool
- Check add-on compatibility: CoreDNS, kube-proxy, VPC CNI, cluster-autoscaler

**Phase 2 — Non-prod first:**
- Upgrade dev cluster first, run full regression and smoke tests
- Upgrade staging cluster, validate application behaviour under load
- Wait at least one week before touching production

**Phase 3 — Control plane upgrade (production):**
- In EKS, control plane upgrade is managed by AWS — we trigger it via Terraform or AWS Console
- Control plane upgrades first, worker nodes stay on old version temporarily (Kubernetes supports N-1 skew)
- This is a non-disruptive step — no workload downtime

**Phase 4 — Worker node upgrade:**
- We use **managed node groups** — AWS provides rolling update strategy
- We set `maxUnavailable: 1` so pods are drained and rescheduled one node at a time
- PodDisruptionBudgets (PDBs) are set on all critical deployments to ensure at least 1 replica is always up during drain
- Node is cordoned → pods drained → new node with new version joins → old node terminated

**Phase 5 — Validate:**
- Monitor New Relic dashboards for error rate spike, latency increase
- Verify all pods are running on new node version: `kubectl get nodes`
- Run smoke tests on all critical endpoints

**Key tools:** `eksctl`, Terraform `aws_eks_cluster`, `kubectl drain`, Pluto for deprecated API detection"

**Upgrade order:**
```
Dev → Staging → Production (Control Plane first → Worker Nodes)
```

---

## 8. Autoscaling — HPA vs VPA

**Answer:**

"Kubernetes offers multiple autoscaling mechanisms. The three main ones are HPA, VPA, and Cluster Autoscaler — and we use all three in our platform."

### HPA — Horizontal Pod Autoscaler

- Scales the **number of pod replicas** up or down
- Based on metrics: CPU, memory, or custom metrics (e.g., SQS queue depth via KEDA)
- Best for **stateless services** like our Payment Service
- Example: if CPU > 70%, scale from 3 pods to 6 pods

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: payment-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: payment-service
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### VPA — Vertical Pod Autoscaler

- Adjusts **CPU and memory resource requests/limits** of individual pods
- Does NOT change replica count
- Best for **stateful or batch workloads** where scaling out isn't ideal
- Caveat: VPA restarts pods to apply new resource values — not suitable for zero-downtime services in `Auto` mode
- We use VPA in `Recommendation` mode (read-only) to right-size our pods without auto-restart

### Cluster Autoscaler

- Scales the **number of EC2 worker nodes** in the node group
- Triggers when pods are in `Pending` state due to insufficient node resources
- Scales down when nodes are underutilized

### Key Differences

| Feature | HPA | VPA |
|---|---|---|
| What it scales | Number of pods (horizontal) | Pod CPU/memory (vertical) |
| Best for | Stateless, high-traffic services | Stateful, batch, or bursty workloads |
| Pod restart required | No | Yes (in Auto mode) |
| Use with HPA | Partially (avoid scaling same metric) | Yes, with HPA on different metrics |
| Our usage | Payment, Settlement services | Recommendation mode for right-sizing |

> ⚠️ **Important:** Don't use HPA and VPA both targeting CPU on the same deployment — they conflict. Use HPA for CPU-based scaling and VPA in recommendation mode only.

---

## 9. GitHub Actions

**Answer:**

"GitHub Actions is our primary CI/CD tool for code-level automation. It's natively integrated with our GitHub repositories, so every push, PR, or tag triggers our pipelines automatically.

**How we use it:**

**On every Pull Request:**
- Lint and unit tests run automatically
- Docker image is built and scanned for vulnerabilities using Trivy
- Terraform plan runs for infra changes — output posted as a PR comment

**On merge to main:**
- Docker image is built, tagged with git SHA, and pushed to Amazon ECR
- Helm chart values are updated with the new image tag
- Deployment triggered to the dev environment automatically

**On release tag:**
- Deployment promoted to staging, then after manual approval to production
- Slack notification sent to the team with deployment summary

**Key features we use:**
- **Secrets management** — AWS credentials, ECR tokens stored as GitHub Secrets
- **Environments** — dev, staging, prod with required reviewers for prod deployments
- **Reusable workflows** — we have a shared `build-and-push.yml` workflow reused across 3 microservices
- **Matrix builds** — run tests across multiple Java versions in parallel
- **OIDC integration** — GitHub Actions uses OIDC to assume AWS IAM roles without storing static credentials

**Example workflow snippet:**
```yaml
name: Build and Deploy
on:
  push:
    branches: [main]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v2
        with:
          role-to-assume: arn:aws:iam::ACCOUNT:role/github-actions-role
          aws-region: ap-south-1
      - name: Build and push to ECR
        run: |
          docker build -t payment-service:${{ github.sha }} .
          docker push $ECR_REGISTRY/payment-service:${{ github.sha }}
```

---

## 10. New Relic Features

**Answer:**

"New Relic is our primary observability platform. Here's how we use each major feature:

**APM (Application Performance Monitoring):**
- Tracks response time, throughput, and error rate per microservice
- Provides distributed tracing — we can follow a single payment request across Payment → Settlement → Notification service
- Identifies slow DB queries, external API calls, and bottlenecks

**Infrastructure Monitoring:**
- Monitors EKS node CPU, memory, disk, and network
- Kubernetes cluster explorer shows pod health, restarts, resource usage
- Alerts when node CPU > 80% or when pods are in CrashLoopBackoff

**Dashboards:**
- We have a per-service dashboard showing: error rate, p50/p95/p99 latency, throughput, pod count
- A business-level dashboard showing: transactions per minute, payment success rate, failed transactions

**Alerting (Alert Policies):**
- Error rate > 1% for 5 minutes → PagerDuty page to on-call
- p99 latency > 2 seconds → Slack notification
- Payment success rate < 99% → Immediate page

**Synthetics:**
- We have a synthetic monitor that hits our `/health` and `/api/payment/status` endpoints every 1 minute from multiple regions
- If the synthetic fails 3 consecutive times, it triggers an alert

**Logs Management:**
- Application logs are shipped to New Relic via Fluent Bit running as a DaemonSet on EKS
- We can correlate logs directly with APM traces — click on a slow trace and jump to the exact log line

**SLO tracking:**
- New Relic has native SLO/SLI management — we define our payment success rate and latency SLOs directly in New Relic and track error budget burn rate"

---

## 11. Production Issue and RCA

**Answer:**

"Let me walk you through one of the most impactful incidents we had.

**The Incident:**
On a Monday morning, we started seeing a spike in payment failure rate — it went from our baseline of 0.1% to around 12% within 10 minutes. New Relic alert fired and I was paged at 9:15 AM.

**Initial triage:**
- Checked New Relic APM — Payment Service showed high error rate, specifically `connection timeout` errors to the database
- EKS pod health was fine — all replicas running
- Checked RDS console — the primary DB was healthy, but connection count was at 98% of `max_connections`

**Root Cause:**
A developer had merged a code change the previous Friday that introduced an **unintentional connection pool misconfiguration** — the `pool.max` setting in the Java app was accidentally set to `500` instead of `50`. With 6 pods running, we were attempting 3,000 connections total, exhausting RDS's max connections limit of 2,000. New connections were being rejected, causing timeout errors.

**Resolution:**
1. Immediately scaled down Payment Service from 6 pods to 2 pods to relieve connection pressure — error rate dropped to 0% within 2 minutes
2. Pushed a hotfix to correct the pool configuration
3. Deployed the fix via CI/CD pipeline (12 minutes total)
4. Gradually scaled pods back to 6 and monitored connection count

**Total downtime:** ~18 minutes of elevated error rate (SLO breach)

**RCA & Prevention:**
- Added a **connection pool configuration check** in our GitHub Actions pipeline — it now validates pool settings against environment-specific limits
- Added a **New Relic alert** for RDS connection count > 80%
- Added the pool config to our **environment variable validation** at pod startup — pod fails to start if config is invalid, preventing bad config from reaching prod
- Documented the incident in our runbook with the resolution steps"

---

## 12. CI/CD Implementation

**Answer:**

"We have a two-tool CI/CD setup — **Jenkins for infrastructure pipelines** and **GitHub Actions for application pipelines**.

**Application CI/CD (GitHub Actions):**

`Feature branch push` → lint + unit tests + Docker build  
`PR to main` → all above + Trivy image scan + Terraform plan as PR comment  
`Merge to main` → Docker build → push to ECR → update Helm values → deploy to dev  
`Release tag` → promote to staging → manual approval gate → deploy to production  

**Infrastructure CI/CD (Jenkins):**
- Terraform pipelines managed in Jenkins with separate jobs per environment
- `terraform plan` runs on PR, output reviewed by team
- `terraform apply` triggered manually after plan approval
- State stored in S3 with DynamoDB locking to prevent concurrent applies

**Deployment strategy on EKS:**
- We use **Helm charts** for all microservice deployments
- Rolling update strategy with `maxUnavailable: 0` and `maxSurge: 1` — ensures zero downtime
- Readiness probes on all services — new pods only receive traffic once they pass the health check
- If error rate spikes post-deployment, we run `helm rollback payment-service` to instantly revert

**Key practices:**
- All secrets injected via AWS Secrets Manager — never in environment variables or config maps
- Image tags are always the git SHA — no `latest` tag in production
- Every deployment sends a Slack notification with: image tag, deployer name, environment, and rollback command"

---

## 13. NGINX Ingress During a Deployment

**Answer:**

"During a rolling deployment, NGINX Ingress continues serving traffic to healthy pods — the key is how we configure readiness probes and the rolling update strategy.

**Here's the exact flow:**

1. New pod starts — it is NOT added to the NGINX upstream pool yet
2. Kubernetes checks the **readiness probe** (we use an HTTP GET on `/health` endpoint)
3. Only when the probe passes does Kubernetes add the pod's IP to the **Endpoints object**
4. NGINX Ingress watches the Endpoints object via the Kubernetes API — it dynamically updates its upstream pool to include the new pod
5. The old pod is then drained — NGINX stops sending new requests to it, existing connections finish
6. Old pod terminates only after drain period (`terminationGracePeriodSeconds: 30`)

**So at no point is there a gap in traffic handling.**

**Our specific config:**
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0   # Never take a pod down before a new one is ready
    maxSurge: 1         # Spin up one extra pod during deployment

readinessProbe:
  httpGet:
    path: /health
    port: 3000
  initialDelaySeconds: 10
  periodSeconds: 5
  failureThreshold: 3
```

**We also set on the Ingress:**
- `nginx.ingress.kubernetes.io/proxy-connect-timeout: "30"` — gives pods time to warm up
- `nginx.ingress.kubernetes.io/proxy-read-timeout: "60"` — avoids premature connection drops during slow starts

This means during a deployment, users experience **zero dropped connections** — they continue hitting the old pods until new ones are fully ready."

---

## 14. No Message Loss in SQS

**Answer:**

"We ensure no message loss in SQS through several layers:

**1. Outbox Pattern (producer side):**
- Before a service publishes to SQS, it writes the event to an **outbox table** in the same DB transaction
- A separate poller reads unprocessed outbox rows and publishes to SQS
- Once SQS confirms receipt, the row is marked as published
- If the service crashes between DB write and SQS publish, the poller retries — guaranteed delivery

**2. SQS Configuration:**
- **Visibility timeout** set to 2x our expected processing time — if a consumer crashes mid-processing, the message becomes visible again for retry
- **Dead Letter Queue (DLQ)** — messages that fail more than 3 times are moved to DLQ for manual inspection and replay
- We have a **New Relic alert** on DLQ depth > 0 — any failed message pages the on-call immediately

**3. Consumer side — idempotency:**
- Each SQS message has a unique `paymentEventId`
- Before processing, the consumer checks a `processed_events` table in RDS
- If the event ID already exists → skip (idempotent processing)
- This protects against duplicate delivery (SQS is at-least-once delivery)

**4. Message retention:**
- SQS message retention set to **14 days** (maximum) — gives us a large window to diagnose and replay DLQ messages

**5. Monitoring:**
- CloudWatch metrics for `NumberOfMessagesSent`, `NumberOfMessagesDeleted`, and `ApproximateNumberOfMessagesNotVisible` (in-flight)
- These are also surfaced in our New Relic dashboard

> 💡 Summary: Outbox pattern guarantees at-least-once publish. Idempotency on consumer side guarantees exactly-once processing."

---

## 15. RTO and RPO for Database Failure

**Answer:**

"We have clearly defined RTO and RPO for our RDS PostgreSQL based on our business SLA.

**Our targets:**
- **RPO (Recovery Point Objective): 5 minutes** — maximum data loss acceptable
- **RTO (Recovery Time Objective): 2 minutes** — maximum time to restore service

**How we achieve RPO of 5 minutes:**
- RDS Multi-AZ with **synchronous replication** to standby — every write is committed on both primary and standby before acknowledging. So in practice, RPO is near **0 seconds** for a Multi-AZ failover
- Automated backups enabled with **5-minute transaction log shipping** — for point-in-time recovery in a worst-case scenario

**How we achieve RTO of 2 minutes:**
- RDS Multi-AZ **automatic failover** — AWS detects primary failure, promotes standby, updates the CNAME DNS endpoint — typically completes in **60–120 seconds**
- Our application uses the **RDS endpoint DNS name** (not a hardcoded IP), so connection rerouting is automatic
- We set **connection pool retry logic** in our Java services — on connection failure, pool retries with exponential backoff for up to 90 seconds before alerting

**For a worst-case full region failure:**
- We have RDS **cross-region read replica** in a secondary AWS region
- Manual promotion of read replica to primary: ~5–10 minutes
- Route 53 health check and DNS failover routes application traffic to secondary region

**Testing:**
- We do a **quarterly failover drill** — we manually trigger RDS failover in staging and validate that our RTO target is met and no data is lost
- Results are documented and reviewed"

---

## 16. Noisy Neighbour Problem

**Answer:**

"In a multi-tenant SaaS platform, one tenant consuming excessive resources can degrade service for others. Here's how we handle it:

**At the Kubernetes level — Resource Quotas and LimitRanges:**
- Each tenant's workloads run in a **dedicated namespace**
- We apply `ResourceQuota` per namespace — limits total CPU, memory, and pod count a tenant's namespace can consume
- `LimitRange` sets default and max resource requests per pod — no pod can consume unbounded memory

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: tenant-a-quota
  namespace: tenant-a
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
```

**At the application level — Rate limiting:**
- NGINX Ingress has per-tenant rate limiting using `nginx.ingress.kubernetes.io/limit-rps` annotation
- API Gateway layer enforces request quotas per tenant API key — e.g., max 1,000 requests/minute for standard tier

**At the database level — Connection pooling per tenant:**
- PgBouncer connection pooler limits connections per tenant schema
- Prevents one tenant from exhausting DB connection pool

**At the SQS level — Dedicated queues:**
- Each major tenant has a **dedicated SQS queue** — so a spike in one tenant's events doesn't delay others

**Monitoring — Tenant-level dashboards:**
- New Relic has namespace-level dashboards — we can instantly see which tenant is consuming spikes
- Alert fires if a single namespace consumes > 80% of its quota

> 💡 **Power Statement:** *"Our architecture is designed so that a blast from one tenant doesn't become a blast radius for all tenants."*"

---

## 17. EKS Cluster Scaling During Peak Traffic

**Answer:**

"We handle peak traffic scaling at two levels — pod level and node level — using a combination of HPA, Cluster Autoscaler, and proactive pre-scaling.

**Pod-level scaling (HPA):**
- HPA monitors CPU utilization per deployment
- For Payment Service, we scale out when CPU > 70% — from min 3 pods to max 10 pods
- HPA response time is ~1–2 minutes, which works for gradual ramps
- For sudden spikes (e.g., festive sale), we also scale on **SQS queue depth** using KEDA (Kubernetes Event Driven Autoscaler) — more predictable for async workloads

**Node-level scaling (Cluster Autoscaler):**
- When new pods are `Pending` due to insufficient node capacity, Cluster Autoscaler detects this and adds new EC2 nodes to the managed node group
- Typical node provisioning time: 2–3 minutes
- We use **mixed instance types** with Spot + On-Demand to balance cost and availability

**Proactive pre-scaling:**
- For known peak events (quarter-end, festive season), we **manually scale up** the node group the night before
- We also set HPA `minReplicas` higher during known peak windows via a scheduled CronJob that patches the HPA spec
- This eliminates the 2-3 minute cold-start lag during the actual traffic spike

**Load testing:**
- Before every peak season, we run load tests using k6 — simulate 5x normal traffic and validate that autoscaling kicks in within our target time
- Results stored and reviewed against our SLOs"

---

## 18. Managing Different SLAs for Different Clients

**Answer:**

"We manage different client SLAs through a combination of namespace isolation, priority classes, and monitoring tiers.

**Tiered service model:**
- **Platinum clients** — 99.9% uptime SLA, p95 latency < 500ms, dedicated support runbook
- **Gold clients** — 99.5% uptime SLA, p95 latency < 1 second
- **Standard clients** — 99% uptime SLA, best effort latency

**Technical implementation:**

*PriorityClasses in Kubernetes:*
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: platinum-priority
value: 1000
---
kind: PriorityClass
metadata:
  name: standard-priority
value: 100
```
Platinum client pods are scheduled first and evicted last during resource pressure.

*Dedicated node pools:* Platinum clients have pods pinned to dedicated node groups using node selectors and taints — no noisy neighbour risk at the infrastructure level.

*SLA monitoring:* In New Relic, we have **per-tenant SLO dashboards** tracking:
- Availability (uptime %)
- Error rate
- p95 latency

*Automated SLA breach alerts:* If a Platinum tenant's availability drops below 99.9%, an immediate PagerDuty alert fires — separate from our general infra alerts.

*Monthly SLA reports:* Generated automatically from New Relic data and shared with clients."

---

## 19. Deployment Strategy

**Answer:**

"We do **not** deploy to all tenants at once — we follow a staged rollout strategy.

**Our deployment order:**

1. **Internal testing namespace** — deploy to our own test tenant first, run automated smoke tests
2. **Canary group** — deploy to 1–2 low-risk Standard tier tenants (5% of traffic)
3. **Monitor for 30 minutes** — check error rate, latency, and business metrics in New Relic
4. **Progressive rollout** — if healthy, deploy to Gold tier tenants
5. **Full rollout** — deploy to Platinum tier tenants last (most risk-averse)

**Technical mechanism:**
- Helm chart values per tenant namespace allow us to control image tag per tenant
- A GitHub Actions workflow accepts a `target_tier` parameter: `canary | gold | platinum | all`
- Automated rollback triggers if error rate exceeds 1% during any stage

**For hotfixes:**
- Critical security patches or production bug fixes go to all tenants simultaneously after a quick internal test
- We accept the risk tradeoff for zero-day security patches

**Feature flags:**
- New features are deployed to all tenants behind a **feature flag** (using LaunchDarkly) — so even if the code is deployed, the feature is only enabled for selected tenants
- This decouples deployment from feature release"

---

## 20. Data Consistency Across Microservices

**Answer:**

"Data consistency in a microservices architecture is one of the hardest problems. We handle it through a combination of patterns:

**Saga Pattern for distributed transactions:**
- A payment flow spans Payment Service → Settlement Service → Notification Service
- We use a **choreography-based Saga** — each service publishes an event on success, and the next service listens and acts
- On failure, each service publishes a compensating event — e.g., if Settlement fails, Payment Service receives a `settlement.failed` event and reverses the authorization

**Outbox Pattern for reliable event publishing:**
- Events are written to an outbox table in the same DB transaction as the state change
- This guarantees the event is published if and only if the DB write succeeds — no dual-write inconsistency

**Idempotency keys:**
- Every API request from the client includes a unique `idempotency-key`
- If the same key is received twice (e.g., due to client retry), the service returns the previous result without re-processing
- This handles exactly-once semantics even with at-least-once delivery from SQS

**Eventual consistency acknowledgement:**
- We design our APIs to be eventually consistent — a payment confirmation is returned as `pending`, and the client polls for final status
- This removes the need for distributed locks and complex 2-phase commits

**Database per service:**
- Each microservice owns its own schema in RDS — no cross-service DB queries
- Data needed across services is replicated via events, not JOIN queries"

---

## 21. SLOs and How Defined

**Answer:**

"Our SLOs are defined based on business requirements, customer contracts, and historical performance data.

**Our core SLOs:**

| SLO | Target | Measurement window |
|---|---|---|
| Payment success rate | ≥ 99.5% | Rolling 30 days |
| API availability | ≥ 99.9% | Rolling 30 days |
| p95 payment latency | ≤ 500ms | Rolling 7 days |
| p99 payment latency | ≤ 2 seconds | Rolling 7 days |
| Deployment success rate | ≥ 98% | Rolling 30 days |

**How we defined them:**

1. **Business input** — what downtime is acceptable? Finance team said max 4.4 hours/month for Platinum clients → 99.9% SLO
2. **User experience baseline** — we ran a survey: users tolerate up to 500ms before abandoning a payment → p95 latency SLO
3. **Historical data** — looked at 6 months of New Relic data, found our p95 was consistently 250ms → set SLO at 500ms (headroom)
4. **Error budget** — 99.9% availability = 43.8 minutes/month error budget. We use this to decide if we can take maintenance windows or accept risk for feature deployments

**Error budget policy:**
- If error budget is > 50% remaining → full deployment velocity, can take risks
- If error budget is 25–50% remaining → be cautious, extra testing required
- If error budget < 25% remaining → freeze non-critical deployments, focus only on reliability work"

---

## 22. Database Failover in Production

**Answer:**

"Our RDS Multi-AZ setup makes failover largely automatic. Here's exactly what happens:

**Automatic failover flow:**
1. RDS detects primary instance failure (network, hardware, AZ issue)
2. AWS promotes the **standby replica** to primary — synchronous replication means zero data loss
3. RDS updates the **CNAME DNS endpoint** to point to the new primary
4. DNS propagation takes ~60 seconds
5. Our Java app connection pool detects broken connections, retries with exponential backoff
6. Within 60–120 seconds, new connections route to the new primary automatically

**What we do during failover:**
- New Relic fires a `RDS Failover Detected` alert (we subscribe to RDS event notifications via SNS)
- On-call engineer joins the incident channel
- Monitor connection error rate in New Relic — should normalize within 2 minutes
- Verify via AWS Console that new primary is healthy
- Post-incident: check for any transactions that were in-flight during the 60-second window — verify outbox table for any unprocessed events

**Manual failover testing:**
- We trigger a manual failover quarterly in staging using `aws rds reboot-db-instance --force-failover`
- Measure actual RTO against our 2-minute target
- Document results

**Read replica promotion (disaster recovery):**
- If entire primary region fails, we promote our **cross-region read replica** to standalone
- Update Route 53 DNS to point to secondary region
- This is a manual process — target RTO of 30 minutes for full region failure"

---

## 23. Traffic Flow

*(See Question 5 — Request Flow for full detailed answer)*

**Short version for quick recall:**
```
User → Route 53 (DNS) → CloudFront (static) / Cognito (auth)
     → ALB → NGINX Ingress → EKS Microservices
     → RDS PostgreSQL + ElastiCache Redis
     → New Relic + CloudWatch (observability)
```

---

## 24. Service Failover Strategy

**Answer:**

"Our failover strategy operates at multiple levels:

**Pod level:** Kubernetes liveness probes detect unhealthy pods and restart them. Readiness probes remove unhealthy pods from the load balancer pool. With 3+ replicas, one pod failure is transparent to users.

**Service level:** If an entire microservice degrades, we have **circuit breakers** implemented using a retry library — after 5 consecutive failures to a downstream service, the circuit opens and we return a cached or degraded response instead of cascading failures.

**Database level:** RDS Multi-AZ automatic failover (see Question 22).

**AZ level:** Our EKS node group spans 3 Availability Zones. If one AZ goes down, Kubernetes reschedules pods to the remaining AZs. ALB automatically stops routing to the impacted AZ.

**Region level:** Route 53 health checks detect if the primary region is unreachable and fail over DNS to our secondary region (manual trigger for full region failure — 30 min RTO).

**Runbooks:** Every major failure scenario has a documented runbook with:
- Detection steps
- Diagnosis commands
- Resolution steps
- Rollback procedure
- Escalation contacts"

---

## 25. Secrets and Config Management

**Answer:**

"We manage secrets and configs differently based on sensitivity:

**Secrets (credentials, API keys, DB passwords):**
- Stored in **AWS Secrets Manager** — versioned, encrypted at rest, access controlled via IAM
- Pods access secrets using the **AWS Secrets Manager CSI driver** — secrets are mounted as files in the pod filesystem, not environment variables (environment variables appear in `kubectl describe pod` — less secure)
- Secrets are **rotated automatically** for RDS passwords using Secrets Manager's native rotation feature

**Config (non-sensitive, environment-specific):**
- Stored in **Kubernetes ConfigMaps** — mounted as environment variables or files
- ConfigMaps are managed via Helm chart values files — separate `values-dev.yaml`, `values-staging.yaml`, `values-prod.yaml`
- Changes to config go through the same CI/CD pipeline as code — audited and reviewed

**Terraform secrets:**
- Terraform state stored in **S3 with server-side encryption**
- Sensitive outputs marked with `sensitive = true` in Terraform to prevent logging
- No secrets hardcoded in Terraform files — all fetched from Secrets Manager at runtime

**Across environments:**
- Dev uses lower-privilege IAM roles and non-production Secrets Manager paths
- Prod IAM roles have strict policies — only the specific service account can access its own secrets
- All secret access is **CloudTrail audited** — we can see exactly who accessed what secret and when"

---

## 26. SLIs and SLOs

*(See Question 21 for full SLO details)*

**SLIs — what we measure:**

| SLI | How measured | Tool |
|---|---|---|
| Availability | % of successful HTTP responses (2xx/3xx) | New Relic APM |
| Latency | p50, p95, p99 response time per endpoint | New Relic APM |
| Error rate | % of 5xx responses | New Relic APM |
| Payment success rate | % of payment events with `status=completed` | Custom New Relic dashboard |
| Saturation | CPU %, memory %, queue depth | New Relic Infra + CloudWatch |

**SLOs — our targets (see Question 21 for full table)**

**Error budget calculation:**
- 99.9% availability over 30 days = 43.8 minutes of allowed downtime
- If we use 30 minutes, we have 13.8 minutes remaining for the rest of the month
- Error budget burn rate alerts us if we're consuming budget too fast

---

## 27. Why Move to SRE from DevOps?

**Answer:**

"That's a question I've thought about carefully. My DevOps journey gave me a strong foundation — I can build pipelines, provision infrastructure, manage Kubernetes clusters. But over time, I realized I was spending most of my energy on *delivery* — getting code out faster — rather than on *reliability* — making sure what's out there stays healthy.

The shift started when we had a series of production incidents last year. Every time I'd be deeply involved in the RCA, the monitoring improvements, the runbook creation. That work felt more meaningful and more strategic than another pipeline optimization.

What draws me to SRE specifically is the engineering-first approach to reliability. The concept of error budgets — using data to make tradeoff decisions between velocity and reliability — is something I want to apply formally. Right now we have some ad-hoc reliability practices, but I want to build systems where reliability is engineered in, not patched in.

I also want to deepen my skills in capacity planning, chaos engineering, and advanced observability — things that are adjacent to what I do now but aren't the primary focus in a DevOps role.

Ultimately, I believe the best SREs are engineers who deeply understand the systems they keep alive. And having built and operated our payment platform — from the Terraform to the Kubernetes to the CI/CD — I feel I have that foundation. I want to use it to drive reliability as a first-class outcome."

> 💡 **Power Statement:** *"DevOps taught me how to ship fast. SRE will teach me how to make what I ship stay alive — and that's the challenge I want to take on next."*

---

## Quick Reference Cheat Sheet

| Topic | Key Terms to Use |
|---|---|
| Kubernetes | Liveness/Readiness probes, PDB, HPA, VPA, KEDA, Cluster Autoscaler, PriorityClass |
| Reliability | SLO, SLI, Error budget, Blast radius, RTO, RPO |
| Data consistency | Saga pattern, Outbox pattern, Idempotency, Eventual consistency, ACID |
| Deployment | Rolling update, Canary, Helm rollback, Feature flags, maxUnavailable, maxSurge |
| Observability | APM, Distributed tracing, p95/p99 latency, Synthetic monitoring, Dashboards |
| Security | OIDC, IAM roles, Secrets Manager, CSI driver, LeastPrivilege |
| Multi-tenancy | Namespace isolation, ResourceQuota, LimitRange, DLQ, Rate limiting |

---

*Last updated: April 2026 | Role: SRE Interview Preparation | Stack: AWS · EKS · Terraform · Jenkins · GitHub Actions · New Relic*
