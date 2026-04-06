# Scale-Adaptive Depth: Scoring Rubric & Thresholds

## Scoring Dimensions

| Dimension | 1 | 2 | 3 | 4 | 5 |
|-----------|---|---|---|---|---|
| **Volume** | <2K words | 2-10K words | 10-25K words | 25-50K words | >50K words |
| **Complexity** | Monolith / 1-2 components | 3-5 components | 6-10 components | 11-20 components | >20 components |
| **Clarity** (inverted) | All decided (<5% TBD) | Few TBDs (5-15%) | Some TBDs (15-30%) | Many TBDs (30-50%) | Mostly TBD (>50%) |
| **Consistency** (inverted) | Fully aligned (0) | Few conflicts (1-2) | Some conflicts (3-4) | Several conflicts (5-10) | Many conflicts (>10) |
| **Domain breadth** | Single domain | 2 domains | 3 domains | 4 domains | 5+ domains |
| **Risk indicators** | None | Mentioned once | Several mentions | Dedicated sections | Heavily regulated |

**Note on Clarity and Consistency**: These are scored inversely — a project with many TBDs
and contradictions needs MORE pipeline depth, not less. The scoring already accounts for this:
higher scores = more work needed.

## Composite Score Formula

```
composite = volume + complexity + clarity + consistency + domain_breadth + risk_indicators
```

Range: 6 (minimum) to 30 (maximum).

## Mode Thresholds

| Score Range | Mode | Description |
|------------|------|-------------|
| **6-12** | lite | Simple project. Collapsed phases (2+3 in one call). Orientation-only checkpoints. INVEST validation only. |
| **13-20** | full | Standard project. All phases. Standard checkpoints. Adversarial review. Default for most projects. |
| **21-30** | thorough | Complex project. All phases + Party Mode + Readiness Gate + Adversarial Review + Edge Case Hunter + Progressive Checkpoints. Full ceremony. |

## Calibration Notes

These thresholds are **initial defaults** based on early project analysis. They will need
calibration over time based on your team's experience.

**How to calibrate:**
1. After each pipeline run, the orchestrator logs the score and mode in `.pipeline-state.json`
2. Review past runs: was the recommended mode appropriate?
3. If "full" mode consistently feels too heavy → raise the full/thorough boundary (e.g., 15-22 instead of 13-20)
4. If "lite" mode misses important issues → lower the lite/full boundary (e.g., 6-10 instead of 6-12)

**User override is always available**: `/run-pipeline mode:lite` or `/run-pipeline mode:thorough`
bypasses the scoring entirely.

## What Counts as a "Component"?

For the Complexity dimension, count distinct systems/services/components mentioned in docs:
- Microservices, APIs, databases, message queues, caches
- Third-party integrations (payment gateways, auth providers, monitoring tools)
- Infrastructure components (Kubernetes, load balancers, CDNs)
- Frontend applications (web app, mobile app, admin panel)

A monorepo with one backend + one frontend + one database = 3 components.
A microservices system with 12 services + 3 databases + Kafka + Redis = 17 components.

## Domain Classification

Domains to scan for in the Domain Breadth dimension:
- **Infrastructure**: hosting, deployment, CI/CD, networking
- **Security**: authentication, authorization, encryption, compliance
- **Data**: databases, ETL, analytics, data pipelines
- **UX/Frontend**: UI components, user flows, accessibility
- **API/Integration**: REST, GraphQL, webhooks, third-party APIs
- **Monitoring/Observability**: logging, metrics, alerting, tracing
- **Business logic**: domain-specific rules, workflows, calculations
- **DevOps**: builds, testing, environments, release management
