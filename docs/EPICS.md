# Epic Decomposition

Generated: 2026-04-04
Last Updated: 2026-04-05
Based on: docs/PROJECT_CONTEXT.md, docs/GAP_ANALYSIS.md
Mode: Full (updated with EPIC-23, EPIC-24; EPIC-3 renamed and re-linked)

---

## Epic Overview

| # | Epic Name | Domain | Priority | Est. Stories | Dependencies | Status |
|---|-----------|--------|----------|--------------|--------------|--------|
| 1 | Architecture Governance & Decisions | Governance | Must Have | 6 | None | Active |
| 2 | Kubernetes Cluster & GitOps Foundation | Infrastructure | Must Have | 5 | EPIC-1 | Active |
| 3 | Observability Stack Deployment | Infrastructure | Must Have | 6 | EPIC-2, EPIC-23 | Active |
| 4 | Security & Network Foundation | Security | Must Have | 7 | EPIC-2 | Active |
| 5 | CI/CD & Operator Development Framework | Platform Engineering | Must Have | 5 | EPIC-2 | Active |
| 6 | Core CRD Operators — Phase 1 | Operator/API | Must Have | 8 | EPIC-3, EPIC-4, EPIC-5 | Active |
| 7 | Multi-Tenant Isolation & Grafana RBAC | Multi-Tenancy | Must Have | 6 | EPIC-3, EPIC-4, EPIC-6 | Active |
| 8 | Telemetry Collection Pipelines | Data Pipeline | Must Have | 7 | EPIC-3, EPIC-6, EPIC-7 | Active |
| 9 | Self-Monitoring & Operational Readiness | Operations | Must Have | 6 | EPIC-3, EPIC-4 | Active |
| 10 | Platform SLOs & Graceful Degradation | Reliability | Must Have | 5 | EPIC-9 | Active |
| 11 | Profiling Pipeline | Data Pipeline | Should Have | 5 | EPIC-3, EPIC-6, EPIC-7 | Active |
| 12 | CRD Operators — Phase 2 (Alerting & Remediation) | Operator/API | Should Have | 8 | EPIC-6, EPIC-7, EPIC-8 | Active |
| 13 | AI Alert & Remediation Agents | AI/ML | Should Have | 6 | EPIC-12 | Active |
| 14 | Metering, Billing & FinOps | Metering | Should Have | 6 | EPIC-6, EPIC-7 | Active |
| 15 | Dashboard-as-Code & Visualization Templates | Visualization | Should Have | 5 | EPIC-7, EPIC-8 | Active |
| 16 | OTel Adoption, Data Governance & Training | Developer Experience | Should Have | 6 | EPIC-8 | Active |
| 17 | Compliance, Legal & Regulatory | Compliance | Should Have | 4 | EPIC-1 | Active |
| 18 | Backup, Restore & Disaster Recovery | Reliability | Should Have | 5 | EPIC-3, EPIC-16 | Active |
| 19 | Capacity Planning Process | Operations | Should Have | 4 | EPIC-9, EPIC-3 | Active |
| 20 | Migration Adapters | Migration | Could Have | 6 | EPIC-8, EPIC-12 | Active |
| 21 | External OBS-as-a-Service | Commercial | Could Have | 7 | EPIC-7, EPIC-14, EPIC-17, EPIC-18, EPIC-22 | Active |
| 22 | Service Catalogue & Business Model Definition | Commercial / Product Management | Should Have | 5 | EPIC-1, EPIC-17 | Active |
| 23 | Technology Stack Evaluation & PoC | Architecture / Infrastructure | Must Have | 6 | EPIC-2 | Active |
| 24 | Platform Release & Upgrade Management | Operations / Process | Should Have | 5 | EPIC-3 (or EPIC-23 outcome), EPIC-9 | Active |

---

## Dependency Graph

```
EPIC-1: Architecture Governance & Decisions (Must Have)
  ├── EPIC-2: Kubernetes Cluster & GitOps Foundation (Must Have)
  │     ├── EPIC-23: Technology Stack Evaluation & PoC (Must Have) ★ NEW
  │     │     └── EPIC-3: Observability Stack Deployment (Must Have) [was "LGTMP Stack Deployment"]
  │     │           ├── EPIC-9: Self-Monitoring & Operational Readiness (Must Have)
  │     │           │     └── EPIC-10: Platform SLOs & Graceful Degradation (Must Have)
  │     │           │           └── EPIC-19: Capacity Planning Process (Should Have)
  │     │           ├── EPIC-24: Platform Release & Upgrade Management (Should Have) ★ NEW
  │     │           │     └── also depends on EPIC-9 (self-monitoring must exist to validate upgrades)
  │     │           ├── EPIC-18: Backup, Restore & Disaster Recovery (Should Have)
  │     │           └── EPIC-4: Security & Network Foundation (Must Have) [parallel with EPIC-3]
  │     │                 ├── EPIC-6: Core CRD Operators — Phase 1 (Must Have)
  │     │                 │     ├── EPIC-7: Multi-Tenant Isolation & Grafana RBAC (Must Have)
  │     │                 │     │     └── EPIC-8: Telemetry Collection Pipelines (Must Have)
  │     │                 │     │           ├── EPIC-11: Profiling Pipeline (Should Have)
  │     │                 │     │           ├── EPIC-15: Dashboard-as-Code & Visualization Templates (Should Have)
  │     │                 │     │           ├── EPIC-16: OTel Adoption, Data Governance & Training (Should Have)
  │     │                 │     │           ├── EPIC-12: CRD Operators — Phase 2 (Should Have)
  │     │                 │     │           │     ├── EPIC-13: AI Alert & Remediation Agents (Should Have)
  │     │                 │     │           │     └── EPIC-20: Migration Adapters (Could Have)
  │     │                 │     │           └── EPIC-14: Metering, Billing & FinOps (Should Have)
  │     │                 │     └── EPIC-5: CI/CD & Operator Dev Framework (Must Have) [parallel with EPIC-6]
  │     │                 └── (feeds into EPIC-6)
  │     └── EPIC-5: CI/CD & Operator Dev Framework (Must Have) [parallel with EPIC-4]
  └── EPIC-17: Compliance, Legal & Regulatory (Should Have) [parallel track — no infrastructure needed]
        ├── EPIC-22: Service Catalogue & Business Model Definition (Should Have)
        │     └── also depends on EPIC-1 (DG-4, DG-8, RG-8 decisions)
        └── EPIC-21: External OBS-as-a-Service (Could Have)
              ├── also depends on EPIC-7, EPIC-14, EPIC-18, EPIC-22
```

> **Note:** All dependencies are acyclic. Epics within the same phase tier that share a common parent can run in parallel.

---

## Recommended Execution Order

### Phase 1 — Bootstrap (Target: end Q2 2026)

**Wave 1 — Parallel (Start immediately, no blockers):**
- **EPIC-1**: Architecture Governance & Decisions
- **EPIC-17**: Compliance, Legal & Regulatory (legal review can begin in parallel; no infrastructure dependency)

**Wave 2 — Parallel (after EPIC-1 is unblocked):**
- **EPIC-2**: Kubernetes Cluster & GitOps Foundation (depends on EPIC-1 decisions)
- Continue **EPIC-17** if not yet complete

**Wave 3 — Parallel (after EPIC-2):**
- **EPIC-23**: Technology Stack Evaluation & PoC ★ NEW — must complete before EPIC-3 can begin
- **EPIC-4**: Security & Network Foundation (can begin in parallel — not dependent on tech choice)
- **EPIC-5**: CI/CD & Operator Development Framework (can begin in parallel)

**Wave 4 — Parallel (after EPIC-23 + EPIC-4 + EPIC-5):**
- **EPIC-3**: Observability Stack Deployment (deploys whichever stack EPIC-23 selected)

**Wave 5 — Parallel (after EPIC-3 + EPIC-4 + EPIC-5):**
- **EPIC-6**: Core CRD Operators — Phase 1
- **EPIC-9**: Self-Monitoring & Operational Readiness

**Wave 6 — Parallel (after EPIC-6 + EPIC-9):**
- **EPIC-7**: Multi-Tenant Isolation & Grafana RBAC
- **EPIC-10**: Platform SLOs & Graceful Degradation
- **EPIC-18**: Backup, Restore & Disaster Recovery *(may begin in parallel with EPIC-6)*

**Wave 7 — After EPIC-7:**
- **EPIC-8**: Telemetry Collection Pipelines *(completes Phase 1)*

### Phase 2 — Multi-Tenant + AI (Target: end October 2026)

**Wave 8 — Parallel (after EPIC-8):**
- **EPIC-11**: Profiling Pipeline
- **EPIC-12**: CRD Operators — Phase 2 (Alerting & Remediation)
- **EPIC-14**: Metering, Billing & FinOps
- **EPIC-15**: Dashboard-as-Code & Visualization Templates
- **EPIC-16**: OTel Adoption, Data Governance & Training
- **EPIC-19**: Capacity Planning Process *(after EPIC-10)*
- **EPIC-24**: Platform Release & Upgrade Management ★ NEW *(after EPIC-3 + EPIC-9)*

**Wave 9 — After EPIC-12:**
- **EPIC-13**: AI Alert & Remediation Agents

### Phase 3 — Production-Ready (Target: Q1 2027)

**Wave 10 — Parallel (after EPIC-13 + EPIC-17 + EPIC-14 + EPIC-18 ready):**
- **EPIC-20**: Migration Adapters *(after EPIC-8 + EPIC-12)*
- **EPIC-21**: External OBS-as-a-Service *(after EPIC-7 + EPIC-14 + EPIC-17 + EPIC-18)*

---

## Gap Coverage Map

45 of 48 open gaps from GAP_ANALYSIS.md are covered (3 resolved gaps excluded: RSG-1, RSG-2, RSG-4). 3 open gaps not yet mapped: DG-9 (service catalogue tier content), RG-9 (service catalogue definition), RSG-5 (Grafana frontend expertise, deferred to 2027+):

| Gap ID | Covered By |
|--------|------------|
| RG-1 | EPIC-6 (Phase 1 CRDs), EPIC-12 (Phase 2 CRDs) |
| RG-2 | EPIC-16 (SOC integration coordination), EPIC-8 (log routing implementation) |
| RG-3 | EPIC-17 (regulatory requirements research) |
| RG-4 | EPIC-21 (external self-service onboarding) |
| RG-5 | EPIC-6 (CRD versioning strategy) |
| RG-6 | EPIC-18 (backup/restore procedures) |
| RG-7 | EPIC-4 (network policies), EPIC-7 (tenant isolation enforcement) |
| RG-8 | EPIC-10 (platform SLOs/SLAs definition) |
| KG-1 | EPIC-1 (AI approach decision Spike), EPIC-13 (implementation) |
| KG-2 | EPIC-19 (capacity planning process and measurement) |
| KG-3 | EPIC-11 (eBPF kernel compatibility Spike) |
| KG-4 | EPIC-16 (NTP coordination with PU-OPS-COMMON-TOOLS) |
| KG-5 | EPIC-17 (Grafana Alloy license verification) |
| KG-6 | EPIC-8 (cross-PU trace propagation design) |
| KG-7 | EPIC-2 (PU-STORAGE S3 compatibility Spike) |
| DG-1 | EPIC-17 (AGPL legal review) |
| DG-2 | EPIC-1 (DA validation gate) |
| DG-3 | EPIC-1 (RTO/RPO decision), EPIC-18 (implementation) |
| DG-4 | EPIC-21 (pricing model decision) |
| DG-5 | EPIC-14 (FinOps/PU-BILLING migration plan) |
| DG-6 | EPIC-1 (GitOps tool selection decision) |
| DG-7 | EPIC-2 (Helm chart structure decision) |
| DG-8 | EPIC-1 (multi-tenancy boundary decision), EPIC-21 (implementation) |
| TG-1 | EPIC-2 |
| TG-2 | EPIC-3 |
| TG-3 | EPIC-8 |
| TG-4 | EPIC-7 |
| TG-5 | EPIC-4 |
| TG-6 | EPIC-4 |
| TG-7 | EPIC-9 |
| TG-8 | EPIC-2 (S3 compatibility validation) |
| TG-9 | EPIC-5 |
| TG-10 | EPIC-15 |
| TG-11 | EPIC-20 |
| TG-12 | EPIC-14 |
| TG-13 | EPIC-10 |
| PG-1 | EPIC-16 |
| PG-2 | EPIC-9 |
| PG-3 | EPIC-19 |
| PG-4 | EPIC-6 (CRD change management process) |
| RSG-3 | EPIC-1 (AI decision unblocks hiring), EPIC-13 (AI agents) |
| RSG-6 | EPIC-9 (on-call rotation definition) |
| DG-10 | EPIC-23 (Technology Stack Evaluation & PoC) |
| PG-5 | EPIC-24 (Platform Release & Upgrade Management) |
| PG-6 | EPIC-24 (non-production environment strategy as part of upgrade lifecycle) |

---

## Epic Details

---

### EPIC-1: Architecture Governance & Decisions

**Domain**: Governance
**Goal**: Resolve all critical blocking decisions so downstream teams can begin infrastructure and development work without ambiguity.
**Priority**: Must Have
**Estimated Size**: 6 stories

#### Description

PU-OBS is architecturally designed but sits in DRAFT status with several critical decisions unresolved — DA validation pending, RTO/RPO undefined, GitOps tooling not selected, AI approach undecided, multi-tenancy boundary for external customers unclear, and the AGPL legal situation for OBS-as-a-Service not started. These are not implementation tasks; they are Spikes, workshops, and facilitated decisions that gate all subsequent Epics. This Epic produces a set of formal Architecture Decision Records (ADRs) and the DA-approved architecture document, enabling work to begin in parallel on infrastructure (EPIC-2) and legal review (EPIC-17).

#### In Scope

- Spike: DA validation — schedule review, resolve open questions, obtain formal sign-off on architecture document (resolves DG-2)
- Spike: RTO/RPO definition — workshop with DA to agree specific numeric targets per data tier (resolves DG-3)
- Decision: GitOps tool selection — FluxCD vs ArgoCD with PU-OPS-COMMON-TOOLS input; produce ADR (resolves DG-6)
- Decision: Helm chart deployment strategy — upstream community charts vs custom umbrella chart vs fully custom; produce ADR (resolves DG-7)
- Spike: AI approach for Alert/Remediation Agents — evaluate LLM-based (RAG/embeddings), classical ML (anomaly detection), rule-based, and hybrid; produce decision document to unblock AI/ML hiring (resolves KG-1, RSG-3)
- Decision: Multi-tenancy boundary for external customers — same model as internal PUs vs separate cluster/instance; produce ADR (resolves DG-8)
- All output stored in `docs/decisions/`

#### Out of Scope

- Implementation of any decided approach (handled in downstream Epics)
- AGPL legal review (handled in EPIC-17, which runs in parallel)
- PU-STORAGE S3 compatibility validation (handled in EPIC-2 after cluster decisions are made)
- Pricing model finalization (handled in EPIC-21)
- Staffing/hiring (out of scope for this pipeline)

#### Dependencies

- Blocked by: None — this Epic starts immediately
- Blocks: EPIC-2 (cluster architecture and GitOps tooling decided), EPIC-13 (AI approach unblocks architecture), EPIC-21 (DG-8 multi-tenancy boundary)

#### Success Criteria

- DA has formally signed off on the architecture document (no longer DRAFT)
- Agreed numeric RTO/RPO targets documented for: management cluster, recent telemetry, historical data
- ADR published selecting FluxCD or ArgoCD with rationale
- ADR published selecting Helm chart structure with rationale
- AI approach decision document published with chosen architecture direction and hiring profile implications
- ADR published defining internal vs external multi-tenancy boundary model

#### Key Risks

- DA availability may delay sign-off beyond Q2 2026 target — escalate to program management if not scheduled within 2 weeks
- AI approach decision requires AI/ML expertise that may not yet be on the team — consider involving external advisor or vendor briefing
- Conflicting stakeholder preferences on multi-tenancy boundary (same cluster vs separate) could deadlock — set a decision deadline
- Some decisions (e.g., RTO/RPO) depend on business risk appetite that may require executive input

---

### EPIC-2: Kubernetes Cluster & GitOps Foundation

**Domain**: Infrastructure
**Goal**: Provision the dedicated OBS Kubernetes cluster, validate S3 storage compatibility, and establish the GitOps deployment pipeline so all subsequent infrastructure and application deployments can proceed.
**Priority**: Must Have
**Estimated Size**: 5 stories

#### Description

Nothing can be deployed until the dedicated Kubernetes cluster exists and the foundational delivery toolchain is operational. This Epic covers coordination with PU-BM to provision the cluster (3 AZs, HA control plane, worker node sizing), selecting and configuring the GitOps tool (per EPIC-1 decision), establishing the Helm repository in Harbor, and — critically — validating that PU-STORAGE's S3 API implementation supports the operations required by Mimir, Loki, and Tempo before the LGTMP stack is deployed. A failed S3 compatibility validation at this stage (rather than after LGTMP is deployed) avoids costly rework.

#### In Scope

- Coordinate with PU-BM to provision the dedicated OBS K8s cluster (3 AZ HA, target node sizing per architecture) (resolves TG-1)
- Configure cluster networking, DNS (via PU-OPS-COMMON-TOOLS), and storage classes for SSD/NVMe hot tier
- Install and configure selected GitOps tool (FluxCD or ArgoCD) in the cluster; establish bootstrap repo structure (resolves DG-6 implementation)
- Implement Helm chart deployment strategy per EPIC-1 ADR; configure Harbor OCI registry as chart repository (resolves DG-7 implementation)
- Spike: PU-STORAGE S3 compatibility validation — test multipart upload, lifecycle policies, versioning, and Mimir/Loki/Tempo-specific API operations; document results and remediation plan if gaps found (resolves KG-7, TG-8)
- Establish cluster RBAC baseline (service accounts, namespace structure per architecture: obs-system, obs-config, tenant-{name}, pu-{name})

#### Out of Scope

- LGTMP stack installation (EPIC-3)
- mTLS, OIDC, network policies (EPIC-4)
- Application CI/CD for operator development (EPIC-5)
- Production HA tuning (addressed in EPIC-10)

#### Dependencies

- Blocked by: EPIC-1 (GitOps tool and Helm strategy decisions; cluster architecture formally approved)
- Blocks: EPIC-3, EPIC-4, EPIC-5

#### Success Criteria

- Dedicated OBS K8s cluster is operational with HA control plane across 3 AZs
- GitOps tool is installed and managing cluster state from Git
- Harbor OCI registry is configured as the chart repository
- Namespace structure (obs-system, obs-config) is created and RBAC-gated
- S3 compatibility validation report is published; all required Mimir/Loki/Tempo S3 operations confirmed working (or a remediation plan is in place with PU-STORAGE)
- A full cluster teardown and rebuild from Git is demonstrated (GitOps integrity test)

#### Key Risks

- PU-BM hardware provisioning lead time is unknown — if cluster provisioning is delayed, the entire Phase 1 timeline shifts; this is the single biggest scheduling risk
- S3 compatibility failures (KG-7) could require switching storage backends or negotiating API additions with PU-STORAGE — allow 2-sprint buffer if incompatibilities are found
- Node sizing targets may need adjustment once actual LGTMP resource requirements are measured (addressed in EPIC-19)

---

### EPIC-3: Observability Stack Deployment

**Domain**: Infrastructure
**Goal**: Deploy the selected observability stack (confirmed by EPIC-23 PoC — default: Grafana LGTMP, alternatives: VictoriaMetrics/VictoriaLogs/Jaeger/Parca) on the OBS cluster with S3-backed storage, ready to receive telemetry from Phase 1 consumers.
**Priority**: Must Have
**Estimated Size**: 6 stories

#### Description

The observability backend stack is the entire technical foundation of PU-OBS. With no stack deployed (TG-2), the platform does not exist. This Epic covers the deployment, configuration, and baseline testing of the backends selected by EPIC-23's technology evaluation PoC for each pillar: metrics (Mimir or VictoriaMetrics), logs (Loki or VictoriaLogs), traces (Tempo or Jaeger), profiling (Pyroscope or Parca), and visualization (Grafana). Each backend is deployed in HA mode with S3 object storage for long-term retention. Retention tiers are configured per architecture (hot/warm/cold). This Epic delivers a working — but not yet multi-tenant or secured — observability cluster. **Note:** The specific deployment tasks will be finalized after EPIC-23 produces its technology decision documents.

#### In Scope

- Deploy Grafana Mimir in HA microservices mode (distributor, ingester, querier, compactor, store-gateway) with S3 backend; configure hot/warm/cold retention tiers (resolves TG-2 for metrics)
- Deploy Grafana Loki in HA mode with S3 backend; configure operational/audit/security retention tiers (resolves TG-2 for logs)
- Deploy Grafana Tempo in HA mode with S3 backend; configure hot/warm trace retention (resolves TG-2 for traces)
- Deploy Grafana Pyroscope in HA mode with S3 backend; configure profiling retention (resolves TG-2 for profiling)
- Deploy Grafana with initial configuration (single-org placeholder); verify connectivity to all four backends (resolves TG-2 for visualization)
- Validate end-to-end data flow: synthetic test data ingested → stored → queried via PromQL / LogQL / TraceQL / profiling API
- All deployments managed via GitOps (Helm charts in Git, reconciled by EPIC-2's GitOps tool)

#### Out of Scope

- Multi-org RBAC and multi-tenant configuration (EPIC-7)
- mTLS and OIDC security hardening (EPIC-4)
- OTel Collector gateway deployment (EPIC-8)
- Self-monitoring stack (EPIC-9)
- Production tuning and SLO compliance (EPIC-10)
- Profiling eBPF DaemonSet (EPIC-11)

#### Dependencies

- Blocked by: EPIC-2 (cluster + S3 storage validated), EPIC-23 (technology stack decision — must know which backends to deploy)
- Blocks: EPIC-6, EPIC-7, EPIC-8, EPIC-9, EPIC-11, EPIC-18, EPIC-24

#### Success Criteria

- All five LGTMP components are deployed and running in HA mode across 3 AZs
- Synthetic metrics, logs, traces, and profiling data can be ingested and queried end-to-end
- S3 backends are receiving data and retention compaction is operating
- All deployment manifests are version-controlled in Git and reconciled by GitOps tool
- Grafana is accessible and shows data from all four backends in Explore view
- No single point of failure exists in the deployment (component failure tests pass for at least one replica loss)

#### Key Risks

- LGTMP in microservices HA mode has significant resource requirements — if cluster nodes are under-sized, ingester OOMKills will occur; ensure Phase 1 sizing is validated before full load
- S3 API incompatibilities not caught in EPIC-2 will surface here — EPIC-2's S3 validation is a hard prerequisite
- Grafana AGPL license: confirm legal review status (EPIC-17) before production use; Phase 1 internal use only should be acceptable under AGPL during review

---

### EPIC-4: Security & Network Foundation

**Domain**: Security
**Goal**: Establish mTLS communication between all collectors and backends, integrate OIDC authentication via PU-IAM-AUTH, define and enforce Kubernetes NetworkPolicies for tenant isolation, and confirm NTP synchronization baseline.
**Priority**: Must Have
**Estimated Size**: 7 stories

#### Description

PU-OBS handles sensitive telemetry data from all PUs and must be secure from the start. This Epic addresses the four core security gaps: no mTLS (TG-5), no OIDC integration (TG-6), no NetworkPolicies (RG-7), and unconfirmed NTP synchronization (KG-4). Additionally, the self-monitoring stack independence requirement (TG-7 partially) means the security perimeter must be designed carefully to avoid circular dependencies. mTLS certificates are obtained via PU-CYPHERING; OIDC is integrated via PU-IAM-AUTH. NetworkPolicies define the enforcement layer for multi-tenant isolation (complementing the software-layer enforcement in EPIC-7).

#### In Scope

- Design and implement mTLS mutual authentication for all collector-to-backend communication paths; obtain certificates via PU-CYPHERING; configure cert rotation (resolves TG-5)
- Integrate OIDC authentication for Grafana and all management APIs via PU-IAM-AUTH (Keycloak/Ory); configure RBAC group mapping from OIDC claims (resolves TG-6)
- Define and deploy Kubernetes NetworkPolicies for: obs-system (backend-to-backend), obs-config (read-only from operators), tenant-{name} (namespace egress/ingress isolation), pu-{name} (same); validate with deny-by-default baseline (resolves RG-7)
- Spike: NTP synchronization baseline — determine NTP infrastructure provided by PU-OPS-COMMON-TOOLS; validate maximum clock skew across AZs; document acceptable skew threshold for trace correlation; alert if skew exceeds threshold (resolves KG-4)
- Configure audit logging infrastructure: immutable, append-only audit log stream for all CRD configuration changes and data access events
- Integrate PU-CYPHERING for secret management (certificates, service account tokens, S3 credentials)

#### Out of Scope

- SOC log routing integration (EPIC-16 and EPIC-8)
- Tenant-level query isolation enforcement in LGTMP backends (EPIC-7)
- OIDC-to-Grafana organization mapping per tenant (EPIC-7)
- AGPL legal review (EPIC-17)

#### Dependencies

- Blocked by: EPIC-2 (cluster exists; PU-CYPHERING and PU-IAM-AUTH accessible)
- Blocks: EPIC-6 (operators need secure API access), EPIC-7 (tenant isolation builds on network policies)

#### Success Criteria

- All collector-to-backend traffic is encrypted via mTLS; certificate rotation is automated
- Grafana login requires OIDC authentication via PU-IAM-AUTH; unauthenticated access is blocked
- NetworkPolicies are deployed with deny-by-default; cross-tenant traffic is blocked at the network layer
- NTP skew baseline is documented; maximum observed skew is within acceptable threshold for trace correlation
- Audit logging captures all CRD write events; logs are append-only and cannot be deleted by tenants
- Secrets are managed via PU-CYPHERING; no plaintext secrets in Git

#### Key Risks

- PU-IAM-AUTH integration may require coordination with PU-IAM-AUTH team; API and OIDC configuration may take longer than estimated
- PU-CYPHERING certificate issuance API and automation tooling may not be ready — validate early and fall back to manual issuance if needed for Phase 1
- NetworkPolicy misconfiguration can silently block legitimate traffic; test rigorously in a non-production namespace before enforcing in obs-system
- NTP skew issues (KG-4) may reveal infrastructure gaps that PU-OPS-COMMON-TOOLS must fix — not fully within PU-OBS control

---

### EPIC-5: CI/CD & Operator Development Framework

**Domain**: Platform Engineering
**Goal**: Establish the complete CI/CD pipeline, Go development toolchain, and kubebuilder scaffolding so the Go/Operator engineers can develop, test, and release CRD controllers with high velocity.
**Priority**: Must Have
**Estimated Size**: 5 stories

#### Description

No CI/CD pipeline exists for operator development (TG-9). With Phase 1 requiring at least four CRD operators (ObservabilityTenant, MetricsPipeline, LogPipeline, TracePipeline), operator engineers need a complete development framework before coding begins. This Epic establishes: Go module structure, kubebuilder scaffolding, unit/integration test patterns, a CI pipeline (build, test, lint, container image publish to Harbor), and a GitOps-integrated CD pipeline for controller deployment to the OBS cluster. This is a developer enablement Epic — it produces no observable platform behavior, but gates EPIC-6.

#### In Scope

- Go module repository structure and kubebuilder project scaffolding for the CRD operator monorepo (resolves TG-9)
- CI pipeline: build, unit test, integration test (envtest), golangci-lint, SAST scan, container image build and push to Harbor OCI registry
- CD pipeline: GitOps-integrated controller deployment (Helm chart per operator, reconciled by EPIC-2's GitOps tool)
- CRD versioning strategy: define v1alpha1 → v1beta1 → v1 promotion criteria, conversion webhook scaffolding, deprecation policy (resolves RG-5)
- CRD change management process: define process for schema changes, deprecation notices, consuming-PU migration support, and breaking change gating (resolves PG-4)
- Developer environment documentation: local development with kind/minikube, controller testing patterns, PR workflow

#### Out of Scope

- Actual CRD schema and controller implementation (EPIC-6, EPIC-12)
- Platform-level CI/CD (GitOps for infrastructure, handled in EPIC-2)
- Training materials for consuming PU teams (EPIC-16)

#### Dependencies

- Blocked by: EPIC-2 (Harbor registry available; GitOps tool configured)
- Blocks: EPIC-6

#### Success Criteria

- A scaffold Go CRD controller can be built, tested, and published to Harbor in a single CI run
- CD pipeline deploys updated controller to OBS cluster via GitOps on merge to main
- CRD versioning strategy document is published; conversion webhook scaffolding is in place
- CRD change management process is documented and approved by Tech Lead
- At least one end-to-end demo: CRD schema change → PR → CI pass → CD deploy to cluster

#### Key Risks

- kubebuilder/controller-runtime version compatibility with the target Kubernetes version must be verified early
- Envtest integration testing is slow; test suite design must balance coverage with CI execution time
- If Go engineers are not yet hired, this Epic may need to be led by the Tech Lead or Platform Engineer

---

### EPIC-6: Core CRD Operators — Phase 1

**Domain**: Operator/API
**Goal**: Implement and deploy the four Phase 1 CRD operators (ObservabilityTenant, MetricsPipeline, LogPipeline, TracePipeline) with a defined versioning strategy, enabling automated tenant onboarding and pipeline configuration via Kubernetes-native API.
**Priority**: Must Have
**Estimated Size**: 8 stories

#### Description

All 14 CRDs are currently "To be developed" (RG-1). Phase 1 requires at minimum four operators: ObservabilityTenant (the top-level tenant entity), MetricsPipeline, LogPipeline, and TracePipeline. These are the core of the PU-OBS API surface; once published, the CRD schemas are versioned and breaking changes must follow the upgrade strategy defined in EPIC-5. The ObservabilityTenant operator is the most complex — it must trigger Grafana organization creation, data source provisioning, OTel Collector gateway reconfiguration, and RetentionPolicy default application on every tenant create/update/delete event. Each operator must handle reconciliation loops, status conditions, and error cases per Kubernetes operator best practices.

#### In Scope

- ObservabilityTenant CRD v1alpha1: schema, controller (reconcile loop: Grafana org creation, data source provisioning, tenant_id label assignment, default RetentionPolicy creation, OTel Collector ConfigMap update), status conditions, RBAC
- MetricsPipeline CRD v1alpha1: schema, controller (reconcile: OTel Collector scrape config, Mimir remote-write endpoint, relabeling rules, rate limits), status conditions
- LogPipeline CRD v1alpha1: schema, controller (reconcile: Fluent Bit/OTel Collector pipeline config, multi-destination routing rules, tenant_id injection), status conditions
- TracePipeline CRD v1alpha1: schema, controller (reconcile: OTel Collector trace pipeline, sampling strategy, Tempo export endpoint), status conditions
- RetentionPolicy CRD v1alpha1 (global defaults only in Phase 1): schema, controller (reconcile: Mimir/Loki/Tempo compactor retention config), status conditions
- CRD admission webhooks: validation webhooks for all Phase 1 CRDs (reject invalid configurations before they reach controllers)
- Integration tests: operator end-to-end tests against a live OBS cluster via CI pipeline

#### Out of Scope

- ProfilingPipeline CRD (EPIC-11)
- AlertRule, AlertPolicy, RemediationPolicy, RemediationAction, RemediationRunbook CRDs (EPIC-12)
- MeteringRule, MeteringReport CRDs (EPIC-14)
- DashboardTemplate CRD (EPIC-15)
- Tenant-level RetentionPolicy overrides (Phase 2)

#### Dependencies

- Blocked by: EPIC-3 (LGTMP backends available for operator reconciliation), EPIC-4 (mTLS/OIDC secured; NetworkPolicies active), EPIC-5 (CI/CD pipeline and kubebuilder scaffold ready)
- Blocks: EPIC-7 (multi-tenancy relies on ObservabilityTenant operator), EPIC-8 (pipeline operators are prerequisites for collection pipeline configuration)

#### Success Criteria

- All four Phase 1 CRD operators are deployed and passing reconciliation in the OBS cluster
- Creating an ObservabilityTenant CR in a namespace automatically provisions a Grafana org, data sources, and default RetentionPolicy — no manual steps required
- Creating MetricsPipeline, LogPipeline, TracePipeline CRs configures the corresponding OTel Collector pipeline segments without manual intervention
- Validation webhooks reject malformed CRs with descriptive error messages
- All CRDs are published at v1alpha1; versioning strategy is enforced (no direct v1 publishing)
- Operator integration tests pass in CI on every PR

#### Key Risks

- ObservabilityTenant controller triggers cascading operations (Grafana API, OTel Collector ConfigMap, RetentionPolicy) — reconciliation ordering and idempotency are complex; budget extra engineering time
- Grafana API for org/data source management may require Grafana service account tokens — coordinate with EPIC-4 secret management
- CRD schema design decisions made now are expensive to reverse (versioning strategy notwithstanding) — Tech Lead must review all schema PRs

---

### EPIC-7: Multi-Tenant Isolation & Grafana RBAC

**Domain**: Multi-Tenancy
**Goal**: Enforce complete tenant isolation across all four telemetry backends (Mimir, Loki, Tempo, Pyroscope) and Grafana, such that no tenant can access another tenant's data through any query path.
**Priority**: Must Have
**Estimated Size**: 6 stories

#### Description

Tenant isolation is a security-critical property of PU-OBS. The architecture specifies two enforcement layers: (1) label injection at ingestion (tenant_id injected by OTel Collector gateway, cannot be overridden), and (2) X-Scope-OrgID enforcement at the query layer in each backend. This Epic implements both layers and configures Grafana multi-org to map each tenant to its own Grafana organization with scoped data sources. DG-8 (multi-tenancy boundary for external customers) resolution from EPIC-1 determines whether external customers share this same model or require a separate approach; this Epic implements the internal model only.

#### In Scope

- Configure X-Scope-OrgID enforcement in Mimir, Loki, Tempo, and Pyroscope (per-backend tenant isolation at query layer) — no cross-tenant queries possible
- Implement tenant_id label injection in OTel Collector gateway (enforced at ingestion; cannot be overridden by sending PU)
- Configure Grafana multi-org: one Grafana organization per ObservabilityTenant, auto-provisioned by the ObservabilityTenant operator (EPIC-6)
- Configure Grafana data sources scoped to tenant organization (PromQL, LogQL, TraceQL, profiling — all tenant-scoped)
- RBAC: PU team members assigned to their Grafana org via PU-IAM-AUTH OIDC group mapping; no cross-org access
- Tenant isolation test suite: automated tests verifying that a tenant cannot query another tenant's data via any of the four backend query APIs

#### Out of Scope

- External customer isolation model (EPIC-21 — pending DG-8 decision)
- NetworkPolicy enforcement (EPIC-4)
- Alerting and remediation per-tenant scoping (EPIC-12)
- Metering per-tenant scoping (EPIC-14)

#### Dependencies

- Blocked by: EPIC-3 (LGTMP backends deployed), EPIC-4 (OIDC + NetworkPolicies active), EPIC-6 (ObservabilityTenant operator drives Grafana org auto-provisioning)
- Blocks: EPIC-8 (tenant-scoped collection pipelines build on this isolation foundation), EPIC-12, EPIC-14, EPIC-21

#### Success Criteria

- Querying Mimir, Loki, Tempo, or Pyroscope with Tenant A's credentials returns only Tenant A's data
- Tenant B's X-Scope-OrgID is rejected if Tenant A's token is used — verified by automated test
- Grafana org auto-provisioning fires within 30 seconds of ObservabilityTenant CR creation
- PU team members can log into Grafana with their OIDC credentials and land in the correct organization automatically
- Isolation test suite runs in CI and all tests pass

#### Key Risks

- X-Scope-OrgID bypass via direct backend API calls (if backends are not properly firewall-restricted) — NetworkPolicies from EPIC-4 must block direct tenant-to-backend access
- Grafana multi-org auto-provisioning complexity — Grafana API must be called idempotently by the operator; errors during provisioning must not leave tenants in a broken state

---

### EPIC-8: Telemetry Collection Pipelines

**Domain**: Data Pipeline
**Goal**: Deploy and configure the full telemetry collection tier — OTel Collector gateway (DaemonSet + gateway), Fluent Bit DaemonSet, cross-PU trace propagation — so all PUs can send metrics, logs, and traces to PU-OBS through a unified, tenant-isolated collection layer.
**Priority**: Must Have
**Estimated Size**: 7 stories

#### Description

No OTel Collector is deployed (TG-3). The collection tier is the primary data ingestion layer: OTel Collector DaemonSet (node-level metrics/traces), OTel Collector gateway (centralized processing, tenant_id injection, rate limiting, routing), and Fluent Bit DaemonSet (node-level log collection). Cross-PU trace propagation (KG-6) — how W3C TraceContext or B3 headers flow across PU namespace boundaries — must be designed and documented; this may require coordination with all consuming PUs. Log routing to SOC (RG-2) is a multi-destination routing concern addressed in this Epic alongside the PG-1 SOC integration coordination in EPIC-16.

#### In Scope

- Deploy OTel Collector DaemonSet: node-level metrics scraping (kubelet, node-exporter), OTLP trace/log/metric receive; configured via MetricsPipeline/LogPipeline/TracePipeline CRDs from EPIC-6 (resolves TG-3)
- Deploy OTel Collector gateway: centralized OTLP receive, tenant_id label injection (enforced, cannot be overridden), relabeling, rate limiting, tail-based sampling for traces, multi-destination routing
- Deploy Fluent Bit DaemonSet: node-level container stdout/stderr log collection, journal logs; forward to OTel Collector gateway
- Spike: Cross-PU trace propagation — document the W3C TraceContext / B3 propagation model across PU boundaries; define whether a service mesh or explicit SDK propagation is required; produce integration guide for consuming PUs (resolves KG-6)
- Log multi-destination routing: Loki (primary), SOC forwarding endpoint (format and protocol TBD — coordinate with security team to resolve RG-2), archival S3 path, legacy tool forwarding stubs
- Graceful collection — configure OTel Collector Write-Ahead Log (WAL) for metrics and logs; Fluent Bit persistence for node-level log buffering during gateway unavailability; circuit breaker configuration (resolves TG-13 for collection tier)

#### Out of Scope

- Profiling collection (Grafana Alloy eBPF DaemonSet) — EPIC-11
- Alert evaluation and AI alert agents — EPIC-12, EPIC-13
- Final SOC integration format/protocol (depends on RG-2 resolution — coordinate in EPIC-16)
- Dashboard and visualization templates — EPIC-15

#### Dependencies

- Blocked by: EPIC-3 (backends to route to), EPIC-6 (CRD operators configure collection pipeline), EPIC-7 (tenant isolation enforcement active)
- Blocks: EPIC-11, EPIC-12, EPIC-14, EPIC-15, EPIC-16, EPIC-20

#### Success Criteria

- OTel Collector DaemonSet and gateway are deployed and operating on all cluster nodes
- Fluent Bit DaemonSet collects all container logs and forwards to gateway
- A test PU namespace sends metrics/logs/traces via OTLP; data appears in Mimir/Loki/Tempo scoped to correct tenant
- tenant_id injection is verified — a sending PU cannot override its own tenant_id label
- OTel Collector WAL is active; simulated gateway restart results in zero metric data loss
- Cross-PU trace propagation guide is published and reviewed by at least one consuming PU team

#### Key Risks

- KG-6 (trace propagation) may reveal that consuming PUs have no OTel SDK integration and cannot propagate trace context — this is a consuming PU responsibility, but PU-OBS must document the requirement clearly
- SOC log routing (RG-2) protocol/format is still undefined at the start of this Epic — implement SOC routing as a configurable endpoint that can be updated once SOC requirements are resolved via EPIC-16
- High telemetry volume from Phase 1 PUs may exceed initial OTel Collector gateway sizing — monitor resource usage closely in the first weeks and scale horizontally

---

### EPIC-9: Self-Monitoring & Operational Readiness

**Domain**: Operations
**Goal**: Deploy the independent self-monitoring stack, create incident runbooks, and establish the on-call rotation, so PU-OBS can be operated 24/7 with full awareness of its own health.
**Priority**: Must Have
**Estimated Size**: 6 stories

#### Description

PU-OBS cannot monitor itself using its own backends — the self-monitoring stack must be completely independent (architecture constraint). No self-monitoring stack exists (TG-7), no incident runbooks exist (PG-2), and no on-call rotation is formally defined (RSG-6). This Epic deploys a lightweight independent Prometheus + Alertmanager monitoring the LGTMP backends, OTel Collectors, and Grafana; creates runbooks for the most common failure scenarios; and defines the on-call rotation (initially shared with other PU SREs per the architecture constraints). Operational readiness is a Phase 1 gate — the platform must not reach consuming PUs without a defined operational model.

#### In Scope

- Deploy independent Prometheus (scrapes all LGTMP components, OTel Collectors, Grafana, Fluent Bit, cluster infra) in obs-system namespace with dedicated storage — does NOT use main Mimir backend (resolves TG-7)
- Deploy independent Alertmanager (routes alerts to on-call channels); confirm independent alerting path (not via Grafana Alerting, which depends on main backends)
- Self-monitoring dashboards in a separate Grafana instance (or isolated org) for the PU-OBS team
- Incident runbooks: create runbooks for minimum viable operational coverage — ingester OOMKill, compactor stuck, OTel Collector gateway down, Grafana unavailable, S3 storage errors, mTLS certificate expiry (resolves PG-2)
- Define and document the 24/7 on-call rotation structure, escalation chain, and hand-off process; agree with other PU SREs on initial shared on-call (resolves RSG-6)
- Alerting: define alert rules for critical self-monitoring conditions (PU-OBS platform SLI violations, component health, certificate expiry, disk usage)

#### Out of Scope

- Platform SLO definition and graceful degradation mechanisms (EPIC-10)
- Capacity planning process (EPIC-19)
- Tenant-facing alerting (EPIC-12)
- Backup and restore procedures (EPIC-18)

#### Dependencies

- Blocked by: EPIC-3 (LGTMP deployed — something to monitor), EPIC-4 (mTLS — self-monitoring Prometheus also needs secure access to scrape targets)
- Blocks: EPIC-10

#### Success Criteria

- Independent Prometheus + Alertmanager is operational and has no dependency on main Mimir/Loki/Grafana backends
- Critical alerts fire within 5 minutes of a simulated LGTMP component failure
- Runbooks exist for the top 6 failure scenarios and are stored in the ops documentation repository
- On-call rotation is defined, agreed with participating SREs, and documented; PagerDuty (or equivalent) is configured
- Self-monitoring stack survives a simulated main backend outage (Mimir down) without losing observability of the platform itself

#### Key Risks

- Self-monitoring Prometheus storage is limited (not backed by S3 — intentionally) — retention must be short (7-14 days); alert on Prometheus storage nearing capacity
- On-call agreement with other PU SREs may reveal resource conflicts — escalate to program management if SRE sharing is not feasible for Phase 1 24/7 coverage

---

### EPIC-10: Platform SLOs & Graceful Degradation

**Domain**: Reliability
**Goal**: Define and publish the PU-OBS platform SLOs/SLAs for consuming PUs, and implement graceful degradation mechanisms ensuring telemetry collection continues and alerting remains operational during partial backend failures.
**Priority**: Must Have
**Estimated Size**: 5 stories

#### Description

Consuming PUs need formal availability and performance commitments from PU-OBS to design their own operational models (RG-8). Currently no SLOs or SLAs are defined. In parallel, the platform has no graceful degradation implementations (TG-13): if LGTMP backends are degraded, telemetry data could be lost. This Epic defines SLOs for ingestion latency, query latency, and availability per signal type; publishes them as a service level commitment; and implements the graceful degradation mechanisms (Write-Ahead Logs, local buffering, circuit breakers, per-signal query degradation) that allow the platform to honor its SLOs under partial failure conditions.

#### In Scope

- Define platform SLOs: ingestion availability, ingestion latency (P99), query latency (P99), Grafana availability — per signal type (metrics, logs, traces, profiles) (resolves RG-8)
- Define SLA commitments for internal PUs (availability targets, planned maintenance windows, notification SLAs)
- Publish SLO/SLA document to all consuming PUs; create Grafana SLO dashboards (Error Budget tracking)
- Implement graceful degradation for metrics: OTel Collector WAL (if not already in EPIC-8), Mimir ingester circuit breaker, retry-with-backoff on remote write from DaemonSet
- Implement graceful degradation for logs: Fluent Bit persistence (if not already in EPIC-8), Loki write path circuit breaker
- Implement graceful degradation for traces: OTel Collector WAL for trace export; Tempo write circuit breaker
- Per-signal query degradation: if one backend is down, query APIs return degraded-mode responses for that signal only; other signals continue normally (resolves TG-13)

#### Out of Scope

- External customer SLAs (EPIC-21)
- DR/RTO/RPO (EPIC-18)
- Capacity planning (EPIC-19)

#### Dependencies

- Blocked by: EPIC-9 (self-monitoring provides the measurement infrastructure for SLOs)
- Blocks: EPIC-19 (capacity planning process uses SLO breach data as input)

#### Success Criteria

- SLO document published and distributed to all consuming PU tech leads
- Error budget dashboards are live in self-monitoring Grafana
- Simulated Mimir ingester failure: metrics collection continues via WAL, no data loss for up to 15 minutes of backend unavailability
- Simulated Loki write failure: log collection continues via Fluent Bit persistence; no log loss for up to 15 minutes
- Per-signal query degradation verified: Tempo down → traces unavailable, metrics and logs continue to be queryable
- SLO breach alerts fire correctly when simulated degradation causes SLI to drop below target

#### Key Risks

- RTO/RPO values from EPIC-1 (DG-3) directly inform the SLO targets — if EPIC-1 is delayed, SLO definition may also be delayed
- WAL and buffering add complexity to the collection components; test failure modes carefully to avoid data corruption on recovery

---

### EPIC-11: Profiling Pipeline

**Domain**: Data Pipeline
**Goal**: Deploy the Grafana Alloy eBPF DaemonSet for zero-instrumentation CPU and memory profiling, implement the ProfilingPipeline CRD operator, and integrate profiling with trace correlation in Grafana.
**Priority**: Should Have
**Estimated Size**: 5 stories

#### Description

Continuous profiling (Pyroscope backend) is part of the LGTMP stack but its collection mechanism is distinct: Grafana Alloy runs as a privileged DaemonSet with hostPID access, using eBPF to profile all processes without code changes. The eBPF approach has a kernel compatibility constraint (KG-3) that must be validated before deployment. The ProfilingPipeline CRD enables per-tenant profiling configuration (language-specific profilers, sampling rate, target pod selectors). Trace-to-profile correlation — linking a Tempo trace span to a Pyroscope profile flame graph — is the key differentiating capability enabled by this Epic.

#### In Scope

- Spike: eBPF kernel compatibility validation — verify all OBS cluster node kernel versions support BPF CO-RE and BTF required by Grafana Alloy; document results and any remediation needed (resolves KG-3)
- Spike: Grafana Alloy license verification — confirm the license of Grafana Alloy (listed as "--" in tech stack); determine if it falls under AGPL or another license; coordinate with EPIC-17 if legal review is needed (resolves KG-5)
- Deploy Grafana Alloy DaemonSet (privileged + hostPID) with security context review and documented risk acceptance
- ProfilingPipeline CRD v1alpha1: schema, controller (reconcile: Alloy profiling target configuration, language-specific profiler enable/disable, sampling rate, per-namespace targeting, Pyroscope namespace scoping)
- Language-specific profiler configuration: Java JFR, Go pprof, Python py-spy, Rust, .NET, Node.js — configurable via ProfilingPipeline CRD
- Trace-to-profile correlation configuration in Grafana (link from Tempo trace span to corresponding Pyroscope flame graph)

#### Out of Scope

- Pyroscope backend deployment (already in EPIC-3)
- Tenant isolation for profiling at the backend query layer (already in EPIC-7)
- eBPF security hardening beyond the required privileged DaemonSet context (out of current scope)

#### Dependencies

- Blocked by: EPIC-3 (Pyroscope backend deployed), EPIC-6 (CRD operator framework), EPIC-7 (tenant isolation for profiling data), EPIC-8 (collection pipeline patterns established)
- Blocks: None critical; feeds into EPIC-15 (profiling dashboards)

#### Success Criteria

- eBPF kernel compatibility is confirmed on all OBS cluster nodes; Grafana Alloy license is documented
- Grafana Alloy DaemonSet is running on all worker nodes and profiles processes without OTel SDK changes
- ProfilingPipeline CRD creates and reconciles Alloy profiling configuration; flame graphs appear in Pyroscope per tenant
- A Tempo trace span links to a corresponding Pyroscope flame graph in Grafana Explore
- ProfilingPipeline CR for a test namespace enables Go pprof profiling and shows goroutine and heap profiles

#### Key Risks

- eBPF profiling requires privileged + hostPID DaemonSet — this is a significant security surface; PU-OBS must document the risk acceptance and review with security stakeholders
- Kernel compatibility (KG-3) may reveal some nodes cannot support eBPF CO-RE — in this case, language-specific SDK-based profilers (pprof endpoints) are the fallback but require PU instrumentation
- Grafana Alloy license (KG-5) could be AGPL — if EPIC-17 determines AGPL is problematic, Alloy would need to be replaced (Parca is the fallback profiling agent)

---

### EPIC-12: CRD Operators — Phase 2 (Alerting & Remediation)

**Domain**: Operator/API
**Goal**: Implement the Phase 2 CRD operators — AlertRule, AlertPolicy, RemediationPolicy, RemediationAction, RemediationRunbook — enabling per-tenant alerting configuration and the remediation audit trail as Kubernetes-native resources.
**Priority**: Should Have
**Estimated Size**: 8 stories

#### Description

Phase 2 adds AI-powered alerting and automated remediation. The CRD layer for these capabilities must be implemented before the AI agents (EPIC-13) can be wired up. The AlertRule CRD translates tenant-defined alert rules into Mimir ruler and Loki ruler configurations. AlertPolicy handles notification routing. RemediationPolicy defines per-tenant remediation modes and blast radius limits. RemediationAction is an immutable audit trail CRD (status-only, append-only). RemediationRunbook defines deterministic playbooks. All five CRDs must enforce the safety model (opt-in, blast radius limits, dry-run mode) as validation webhooks.

#### In Scope

- AlertRule CRD v1alpha1: schema, controller (reconcile: translate PromQL/LogQL alert expressions to Mimir ruler / Loki ruler groups), validation webhook (expression syntax validation), status conditions
- AlertPolicy CRD v1alpha1: schema, controller (reconcile: Alertmanager route configuration per tenant), notification receiver configuration (email, webhook, PagerDuty), status conditions
- RemediationPolicy CRD v1alpha1: schema, controller (reconcile: blast radius limits, auto-approved action list, dry-run mode config, escalation policy config), validation webhook (safety constraints), status conditions
- RemediationAction CRD (status/audit): schema, write-once controller (audit trail — no updates or deletes permitted), RBAC ensuring tenants can read but not write or delete their own audit trail
- RemediationRunbook CRD v1alpha1: schema, controller (reconcile: playbook step validation, rollback step validation, approval requirement), status conditions
- Multi-signal alert evaluation: configure Grafana unified alerting to evaluate AlertRules across Mimir (metrics) and Loki (logs) simultaneously

#### Out of Scope

- AI Alert Agent and AI Remediation Agent implementation (EPIC-13)
- MeteringRule, MeteringReport CRDs (EPIC-14)
- DashboardTemplate CRD (EPIC-15)
- RetentionPolicy per-tenant overrides — if not implemented in EPIC-6, handle in this Epic as a Phase 2 extension

#### Dependencies

- Blocked by: EPIC-6 (operator framework and ObservabilityTenant in place), EPIC-7 (tenant isolation active), EPIC-8 (telemetry flowing — needed to validate AlertRule expressions against real data)
- Blocks: EPIC-13 (AI agents plug into the CRD event stream), EPIC-20

#### Success Criteria

- Creating an AlertRule CR in a tenant namespace results in a live alert rule in Mimir ruler or Loki ruler
- AlertPolicy CR configures notification routing correctly — test alert fires and reaches configured receiver
- RemediationPolicy CR with dry-run mode enabled results in dry-run-only remediation actions (no actual changes)
- RemediationAction CRDs are immutable — delete and update operations are rejected by admission webhook
- All five CRD controllers pass integration tests in CI
- Blast radius limits in RemediationPolicy are enforced — controller rejects policy specs that exceed defined safety thresholds

#### Key Risks

- Mimir ruler and Loki ruler configuration formats differ — AlertRule controller must abstract this cleanly; errors in ruler config can silently prevent alert evaluation
- RemediationAction immutability requires careful RBAC design; a misconfigured ClusterRole could allow deletion of audit trail records
- Phase 2 timeline (end October 2026) is aggressive given the CRD count — prioritize AlertRule and AlertPolicy first; RemediationPolicy and RemediationRunbook can follow

---

### EPIC-13: AI Alert & Remediation Agents

**Domain**: AI/ML
**Goal**: Design and implement the AI Alert Agent (cross-signal correlation, intelligent anomaly detection, alert enrichment) and AI Remediation Agent (automated incident resolution with blast radius enforcement), using the approach decided in EPIC-1.
**Priority**: Should Have
**Estimated Size**: 6 stories

#### Description

The AI approach is currently undecided (KG-1) — LLM-based, classical ML, rule-based, or hybrid. EPIC-1 resolves this decision. EPIC-13 then implements the chosen approach. The AI Alert Agent consumes alert events from the CRD layer (EPIC-12), correlates across Mimir metrics, Loki logs, and Tempo traces, deduplicates, enriches with root-cause hypotheses, and routes to both notification channels and the Remediation Engine. The AI Remediation Agent evaluates remediation options against the tenant's RemediationPolicy, executes auto-approved actions, requests human confirmation for others, and writes all outcomes to RemediationAction CRDs. AI remediation is opt-in, disabled by default.

#### In Scope

- AI Alert Agent: design and implement cross-signal correlation engine (approach per EPIC-1 ADR); integrate with alert event stream from Grafana Alerting/Mimir ruler/Loki ruler; produce enriched alert events with root-cause hypothesis, correlated signals, deduplication
- AI Alert Agent: anomaly detection integration (approach per EPIC-1 ADR — LLM, ML model, or statistical rules)
- AI Remediation Agent: evaluate RemediationPolicy for each incoming alert; execute auto-approved RemediationRunbook steps; request human confirmation for human-confirm actions; write RemediationAction CRD for every outcome
- Blast radius enforcement in agent: agent checks RemediationPolicy limits before executing each action; aborts and escalates if limit would be breached
- Dry-run mode: full agent execution with all actions logged but not applied; used for testing remediation policies
- AI agent observability: agent self-reports decision confidence, correlation sources, and action outcomes to Grafana dashboards; AI agent errors visible in self-monitoring (EPIC-9)

#### Out of Scope

- CRD schema for alerting and remediation (EPIC-12)
- Training data collection and model training infrastructure (if LLM/ML approach chosen — this may require a separate Spike in Phase 2)
- FinOps/GreenOps AI analytics (out of scope for PU-OBS AI agents)
- AI Remediation for external customers (EPIC-21 extension)

#### Dependencies

- Blocked by: EPIC-12 (CRD layer for alerting and remediation must exist); EPIC-1 (AI approach decision must be made)
- Blocks: EPIC-20 (migration adapters may use AI for alert translation)

#### Success Criteria

- AI Alert Agent receives a multi-signal incident (correlated metric anomaly + log error spike + trace latency increase) and produces an enriched alert with correlated signals identified
- AI Remediation Agent executes a RemediationRunbook in dry-run mode and produces a complete action log without making any changes
- A RemediationPolicy with a blast radius of "max 2 pod restarts" is enforced — agent aborts remediation after 2 restarts and escalates
- All remediation actions are recorded in RemediationAction CRDs (immutable audit trail)
- Tenant with RemediationPolicy mode=disabled receives no automated remediation actions
- AI agent decision confidence and error rate are visible in Grafana dashboards

#### Key Risks

- KG-1 (AI approach undecided) is resolved in EPIC-1 — if the decision is delayed, EPIC-13 cannot begin; this is the primary scheduling risk for this Epic
- RSG-3 (AI/ML hiring blocked by undecided approach) means the team may not be staffed for this Epic — the EPIC-1 decision unblocks hiring, but there is a lead time
- LLM-based approaches require careful data governance (no telemetry data should leave the sovereign platform) — on-premise model deployment or air-gapped inference must be designed
- False-positive remediation actions can cause incidents; dry-run mode and human-confirm mode must be thoroughly tested before enabling auto-approve in any tenant

---

### EPIC-14: Metering, Billing & FinOps

**Domain**: Metering
**Goal**: Implement the metering pipeline (MeteringRule, MeteringReport CRDs), aggregate per-tenant and per-PU usage data, export to PU-PROV billing API, and deliver FinOps/GreenOps dashboards for internal cost visibility.
**Priority**: Should Have
**Estimated Size**: 6 stories

#### Description

PU-OBS is a commercial service requiring usage-based billing for internal transfer pricing and future external customers. No metering pipeline exists (TG-12). The architecture decision to use custom CRD-based metering with Prometheus recording rules (over OpenMeter) has been made. MeteringRule CRDs define what to meter (ingestion rate, storage GB, active series, API calls) per tenant. An aggregation service queries Mimir (where usage metrics are stored via recording rules) and generates MeteringReport CRDs and exports to the PU-PROV billing API. DG-5 (PU-BILLING establishment timeline) determines whether FinOps permanently lives here or migrates — this Epic implements FinOps in PU-OBS with a migration-ready design.

#### In Scope

- MeteringRule CRD v1alpha1: schema, controller (reconcile: create Prometheus recording rules in Mimir for defined usage dimensions — ingestion rate, storage, active series, API call count per tenant), validation webhook, status conditions
- MeteringReport CRD (status/read-only): schema, controller (aggregation service queries recording rules → generates MeteringReport per tenant per reporting interval), read-only RBAC
- Aggregation service: queries Mimir recording rules, computes usage summaries per tenant, generates MeteringReport CRDs, exports to PU-PROV billing API (defines export format and API contract with PU-PROV)
- FinOps dashboards: cost per PU (based on metered usage × internal transfer price), storage cost breakdown, ingestion cost breakdown, trend over time
- GreenOps dashboards: energy consumption estimation per PU (compute hours × energy factor), carbon footprint estimation — initial implementation in PU-OBS (resolves DG-5 by building migration-ready FinOps)
- Document PU-BILLING migration plan: define handoff contract so FinOps/GreenOps capabilities can migrate when PU-BILLING is established

#### Out of Scope

- External customer billing (EPIC-21 extension — pricing model must be confirmed first per DG-4)
- PU-BILLING implementation (out of scope for PU-OBS)
- Actual pricing model confirmation (DG-4 — resolved in EPIC-21 planning)

#### Dependencies

- Blocked by: EPIC-6 (ObservabilityTenant CRD in place — metering is per-tenant), EPIC-7 (tenant isolation ensures metering is correctly scoped), EPIC-8 (telemetry flowing — recording rules need data)
- Blocks: EPIC-21 (external billing depends on metering pipeline)

#### Success Criteria

- MeteringRule CRDs are deployed for all Phase 1 tenants; Prometheus recording rules are active in Mimir
- MeteringReport CRDs are generated on the configured reporting interval (e.g., hourly) for each tenant
- Metering data is exported to PU-PROV billing API; PU-PROV confirms receipt and billing record creation
- FinOps dashboard shows per-PU cost breakdown with 30-day trend
- GreenOps dashboard shows estimated energy and carbon per PU
- PU-BILLING migration plan document is published

#### Key Risks

- PU-PROV billing API contract must be agreed jointly with PU-PROV team — delays in API specification will block the export pipeline
- Prometheus recording rules for metering consume Mimir query capacity — ensure recording rule evaluation does not impact query SLOs

---

### EPIC-15: Dashboard-as-Code & Visualization Templates

**Domain**: Visualization
**Goal**: Create the DashboardTemplate CRD and a library of pre-built dashboard templates (one per PU type and one per telemetry pillar), auto-provisioned for every new tenant, enabling immediate time-to-value for consuming PUs.
**Priority**: Should Have
**Estimated Size**: 5 stories

#### Description

No dashboard-as-code templates exist (TG-10). PU-OBS promises pre-built dashboards per PU type and telemetry pillar as part of the onboarding value proposition. The DashboardTemplate CRD defines dashboard JSON (Grafana dashboard format) with tenant-scoped variable injection (so the same template renders correctly for any tenant). The ObservabilityTenant operator (EPIC-6) auto-provisions dashboards from DashboardTemplates when a new tenant is created. This Epic creates the DashboardTemplate CRD and an initial library of templates covering the core use cases.

#### In Scope

- DashboardTemplate CRD v1alpha1: schema (grafanaJson, variables with tenant-scoped defaults, category, autoProvision flag, target PU type), controller (reconcile: push dashboard to correct Grafana org on template create/update), validation webhook
- Template library: minimum set for Phase 2 — Kubernetes cluster overview (CPU/memory/network/storage), metrics overview (active series, ingestion rate, query rate), logs overview (ingestion rate, error rate, top sources), traces overview (request rate, latency P99, error rate), profiling overview (CPU flame graph, memory flame graph)
- Template auto-provisioning: ObservabilityTenant operator updated to instantiate autoProvision=true templates for each new tenant at creation time
- Dashboard-as-code workflow: templates stored as YAML in Git; GitOps reconciliation updates Grafana on template changes
- Self-service dashboard guidance: documentation for PU teams on how to create custom dashboards within their Grafana org; Grafana folder structure per tenant

#### Out of Scope

- Grafana plugin development (2027+ roadmap)
- FinOps/GreenOps dashboards (EPIC-14)
- AI-powered dashboard recommendations (future)
- External customer managed dashboards (EPIC-21 extension)

#### Dependencies

- Blocked by: EPIC-7 (Grafana multi-org active — templates provision into tenant orgs), EPIC-8 (telemetry flowing — dashboards need data to display)
- Blocks: None critical

#### Success Criteria

- DashboardTemplate CRD is deployed; at least 5 templates exist in the library covering all four telemetry pillars
- Creating a new ObservabilityTenant automatically provisions all autoProvision=true dashboards within 60 seconds
- Updating a DashboardTemplate in Git results in Grafana dashboard updates across all tenant orgs via GitOps
- A PU team can log into their Grafana org and immediately see pre-built dashboards with their real telemetry data
- Template YAML passes validation webhook before deployment; malformed dashboard JSON is rejected

#### Key Risks

- Grafana dashboard JSON format changes between Grafana versions — pin the Grafana version before finalizing template JSON; version upgrades may require template migration
- Auto-provisioning many dashboards at once (for large numbers of tenants) could hit Grafana API rate limits — implement batching and back-off in the ObservabilityTenant operator

---

### EPIC-16: OTel Adoption, Data Governance & Training

**Domain**: Developer Experience
**Goal**: Publish the OTel semantic conventions guide for UCP, define the SOC integration specification, resolve NTP synchronization requirements, and deliver training materials so all consuming PU teams can instrument correctly and use PU-OBS effectively.
**Priority**: Should Have
**Estimated Size**: 6 stories

#### Description

PU-OBS recommends but cannot mandate OTel conventions (PG-1 — process gap). Without active coordination, each PU will instrument independently, producing inconsistent telemetry labels that break cross-PU dashboards and trace correlation. This Epic creates the coordination process: a UCP OTel Working Group, semantic conventions guide, compliance dashboard (showing which PUs are convention-compliant), and training materials. It also resolves the SOC integration specification (RG-2) — coordinating with the security/SOC team to define the log routing format and protocol — and addresses the NTP synchronization coordination point (KG-4, partially in EPIC-4 for measurement, here for cross-PU coordination).

#### In Scope

- OTel semantic conventions guide: publish UCP-specific naming conventions for service names, resource attributes, metric names, log labels, trace span names; based on OpenTelemetry semantic conventions with UCP extensions (resolves PG-1)
- OTel Working Group: establish recurring working group with representatives from each consuming PU; define governance process for convention changes
- OTel compliance dashboard: DashboardTemplate showing per-PU convention compliance rate (% of spans with required attributes, % of metrics with required labels); visibility only — not enforcement
- SOC integration specification: coordinate with security/SOC team to define log routing format (CEF, JSON, syslog), transport (HTTPS, Kafka, syslog-ng), filtering rules, and required fields; produce specification document (resolves RG-2); implement SOC routing configuration in EPIC-8 log routing once defined
- Training materials: PU instrumentation guide (OTel SDK setup for Go, Java, Python, .NET, Node.js), Grafana self-service guide, AlertRule CRD authoring guide; lab environment using PU-OBS staging cluster
- NTP cross-PU coordination: document maximum acceptable clock skew for trace correlation; communicate requirement to all PU teams; create monitoring alert if cross-PU skew exceeds threshold (resolves KG-4 cross-PU coordination, complements EPIC-4 cluster-level measurement)

#### Out of Scope

- Mandatory OTel convention enforcement (architecture decision: recommended, not mandatory)
- Training for external customers (EPIC-21)
- SOC integration implementation beyond the specification (the actual routing is in EPIC-8; SOC team's SIEM implementation is out of scope)

#### Dependencies

- Blocked by: EPIC-8 (telemetry flowing — training and compliance dashboard need real data; SOC routing implementation needs the pipeline to exist)
- Blocks: None — this is a developer experience and process Epic

#### Success Criteria

- OTel semantic conventions guide is published and reviewed by at least 3 consuming PU tech leads
- OTel Working Group has held at least 2 sessions; charter and governance process are documented
- OTel compliance dashboard is live and shows per-PU compliance rates for at least 3 PUs
- SOC integration specification is published and approved by the security/SOC team
- Training materials cover all major languages; at least one PU team has completed the instrumentation lab
- NTP skew documentation and alert threshold communicated to all PU teams

#### Key Risks

- PU team buy-in for OTel convention adoption is not guaranteed — the working group must demonstrate value early (e.g., better cross-PU dashboards) rather than mandating compliance
- SOC team may have specific format requirements (e.g., CEF for legacy SIEM) that require log transformation in the OTel Collector gateway — coordinate early in EPIC-8 log routing design

---

### EPIC-17: Compliance, Legal & Regulatory

**Domain**: Compliance
**Goal**: Complete the AGPL legal review for OBS-as-a-Service, verify the Grafana Alloy license, identify applicable regulatory log retention requirements, and produce a compliance position document enabling commercial operation.
**Priority**: Should Have
**Estimated Size**: 4 stories

#### Description

Multiple compliance and legal items are unresolved: the AGPL legal review (DG-1) is on the critical path for external commercial OBS-as-a-Service; the Grafana Alloy license is unlisted (KG-5); and no regulatory log retention requirements have been identified (RG-3). These are not implementation tasks — they are legal research, regulatory research, and engagement with the legal team. The AGPL review result determines whether the LGTMP stack can be used commercially or requires a fallback to Apache 2.0 alternatives (noting there is no mature Grafana alternative). This Epic runs in parallel with all infrastructure work and should start immediately.

#### In Scope

- AGPL legal review: engage legal team to assess AGPL 3.0 implications for commercial OBS-as-a-Service offering; evaluate options — (a) AGPL acceptable, (b) Apache 2.0 alternative components, (c) commercial Grafana Enterprise, (d) restrict offering scope; produce legal opinion document (resolves DG-1)
- Grafana Alloy license verification: research and confirm the license of Grafana Alloy (listed as "--"); document finding and flag to legal review if AGPL (resolves KG-5)
- Regulatory log retention research: identify applicable regulations for the UCP platform (GDPR, SOX, PCI-DSS, sector-specific); determine minimum retention periods for audit logs and security logs; produce a log retention requirements document that feeds into RetentionPolicy CRD defaults (resolves RG-3)
- Compliance position document: summarize legal and regulatory findings; define what is and is not permitted for Phase 3 external OBS-as-a-Service launch

#### Out of Scope

- Implementation of retention policies (EPIC-6, EPIC-12)
- External OBS-as-a-Service commercial design (EPIC-21)
- SOC integration specification (EPIC-16)

#### Dependencies

- Blocked by: EPIC-1 (architecture formally approved — legal reviews the approved architecture, not the DRAFT)
- Blocks: EPIC-21 (commercial launch depends on legal clearance)

#### Success Criteria

- Legal opinion document published on AGPL 3.0 for commercial OBS-as-a-Service
- Grafana Alloy license is confirmed and documented
- If AGPL is problematic: fallback plan is defined (Apache 2.0 alternatives or Grafana Enterprise commercial license path)
- Regulatory log retention requirements document is published; minimum retention periods are specified
- Compliance position document is approved by legal and architecture teams
- RetentionPolicy CRD default values for audit and security logs are updated per regulatory findings

#### Key Risks

- AGPL legal review timelines depend on legal team availability — start immediately; legal reviews can take weeks to months
- If AGPL is rejected for commercial use and the fallback is Apache 2.0 alternatives, Grafana has no mature OSS replacement — this would require a significant re-architecture and must be escalated immediately as a program-level risk

---

### EPIC-18: Backup, Restore & Disaster Recovery

**Domain**: Reliability
**Goal**: Define and implement backup procedures for all OBS data (Mimir, Loki, Tempo, Pyroscope, CRD state, Grafana configuration), validate restore procedures, and document the DR runbook with agreed RTO/RPO targets.
**Priority**: Should Have
**Estimated Size**: 5 stories

#### Description

No backup or restore procedures exist (RG-6). The architecture references RTO/RPO "to be refined with DA" (DG-3). EPIC-1 resolves the RTO/RPO values. This Epic then implements the backup mechanisms for each data tier — S3 object storage (cross-region or cross-AZ replication), Kubernetes CRD state (etcd backup or GitOps-based rebuild), Grafana configuration (dashboard-as-code export), and LGTMP component configuration (all in Git via GitOps). The most critical insight is that LGTMP backends store time-series data in S3 — the S3 replication strategy is the primary data backup mechanism. CRD state can be rebuilt from Git (GitOps) + S3 data.

#### In Scope

- DR strategy document: define Recovery Time Objective (RTO) and Recovery Point Objective (RPO) per tier (management cluster, recent telemetry, historical data) based on EPIC-1 (DG-3) agreed values (resolves RG-6 and DG-3 implementation)
- S3 data backup: configure cross-AZ S3 bucket replication for all four LGTMP backends; define and test backup verification procedure
- Kubernetes CRD state backup: define backup strategy (etcd snapshot for CRD state, Git as source of truth for configuration); automate etcd snapshots with off-cluster storage
- Grafana configuration backup: dashboard-as-code ensures Grafana config is in Git; validate full Grafana rebuild from Git
- DR runbook: step-by-step procedures for full cluster recovery, backend data recovery, and partial component recovery; test runbook annually
- Restore drill: conduct a restore drill — simulate cluster loss, recover from backups, validate data integrity and system functionality; document results

#### Out of Scope

- S3 storage implementation (provided by PU-STORAGE)
- RTO/RPO decision (EPIC-1)
- Self-monitoring of backup status (extend EPIC-9 self-monitoring to cover backup alerts)

#### Dependencies

- Blocked by: EPIC-3 (LGTMP deployed — something to back up)
- Blocks: None critical, but must complete before platform goes to production for Phase 3

#### Success Criteria

- S3 cross-AZ replication is configured and verified for all four LGTMP backends
- Etcd snapshots are automated and stored off-cluster; latest snapshot is < 1 hour old at all times
- Full cluster rebuild from Git + S3 is demonstrated and documented; rebuild time is measured against RTO target
- DR runbook is published, reviewed by SRE team, and stored in operational documentation
- Restore drill is conducted; all success criteria (RTO/RPO) are met or gaps are documented with remediation plans

#### Key Risks

- Agreed RTO/RPO values (from EPIC-1/DG-3) may be more aggressive than the architecture can support — validate feasibility before committing; renegotiate with DA if needed
- S3 cross-AZ replication depends on PU-STORAGE support for replication — verify PU-STORAGE replication capability as part of EPIC-2's S3 compatibility Spike

---

### EPIC-19: Capacity Planning Process

**Domain**: Operations
**Goal**: Establish a repeatable capacity planning process using Phase 1 baseline telemetry data, resolve the storage capacity uncertainty (10-50 TB/month range), and produce a concrete capacity plan for Phase 2 growth.
**Priority**: Should Have
**Estimated Size**: 4 stories

#### Description

Phase 1 is explicitly described as a sizing PoC — the 10-50 TB/month storage estimate is too wide for cost planning (KG-2). No capacity planning process exists (PG-3). Once Phase 1 is operational, actual telemetry volumes, ingestion rates, and resource utilization can be measured. This Epic establishes the measurement plan, tooling, and process to collect Phase 1 baseline data, analyze it, and produce a data-driven capacity plan for Phase 2 — including storage sizing, compute scaling recommendations, and cost projections.

#### In Scope

- Capacity measurement plan: define which metrics to collect (ingestion rate per PU per signal type, storage growth rate per backend, query load, OTel Collector CPU/memory) and measurement intervals (resolves PG-3)
- Baseline data collection dashboard: Grafana dashboard in self-monitoring showing capacity metrics (storage growth rate, ingestion rate trend, resource utilization per component) with 30-day trend
- Phase 1 baseline analysis: after 4-6 weeks of Phase 1 operation, analyze actual vs estimated volumes; narrow the 10-50 TB/month range to an actual figure (resolves KG-2)
- Phase 2 capacity plan: produce a document with Phase 2 storage sizing, compute node count recommendations, and cost projections based on Phase 1 baseline and expected Phase 2 PU onboarding growth
- Storage cost alert: alert when projected storage growth exceeds planned Phase 2 budget (early warning system)

#### Out of Scope

- Phase 3 capacity planning (deferred until Phase 2 baseline is measured)
- Storage provisioning implementation (PU-STORAGE and PU-BM responsibility)

#### Dependencies

- Blocked by: EPIC-9 (self-monitoring provides measurement infrastructure), EPIC-10 (SLOs define the performance targets capacity planning must protect)
- Blocks: None — feeds into Phase 2 planning decisions

#### Success Criteria

- Capacity measurement dashboard is live in self-monitoring Grafana
- Phase 1 baseline report is published with actual ingestion rates and storage growth per PU per signal type
- 10-50 TB/month estimate is narrowed to a specific range based on actual Phase 1 data
- Phase 2 capacity plan document is published with storage sizing, node count, and cost projection
- Storage cost alert is active and configured to trigger at 80% of planned Phase 2 budget

#### Key Risks

- Phase 1 telemetry volumes may be unexpectedly high due to noisy PU configurations — capacity planning must include PU log verbosity review and guidance
- If Phase 1 reveals that storage costs will exceed budget, escalation to program management may require re-scoping Phase 2 PU onboarding

---

### EPIC-20: Migration Adapters

**Domain**: Migration
**Goal**: Implement migration adapters for Prometheus federation, Zabbix/Nagios/Centreon exporters, and ELK compatibility layer, enabling legacy monitoring platforms to be migrated to PU-OBS during Phase 3.
**Priority**: Could Have
**Estimated Size**: 6 stories

#### Description

Phase 3 includes migration from legacy monitoring tools (Zabbix, Nagios, Centreon, ELK) to PU-OBS. No migration adapters exist (TG-11). Migration adapters are not needed until the platform is production-ready and consuming PUs choose to migrate — this is a Phase 3 concern that should not be started until Phase 2 is complete. The Prometheus federation adapter is the highest value (many internal PUs may already use Prometheus) and is simplest to implement. Zabbix/Nagios/Centreon exporters require legacy domain expertise (RSG-4). ELK dual-write requires careful log schema mapping.

#### In Scope

- Prometheus federation adapter: configure Prometheus federation endpoint in OTel Collector to scrape remote Prometheus servers; handle label remapping to OBS conventions; provide migration guide for PUs moving from Prometheus to OTel
- Zabbix/Nagios/Centreon exporters: implement or configure existing community exporters to bridge legacy metrics into OTel Collector; document per-tool migration runbook
- ELK compatibility layer: implement dual-write log routing (logs written to both Loki and Elasticsearch/OpenSearch simultaneously during migration period); provide index pattern mapping documentation
- Migration runbooks: per-tool step-by-step migration guide for PU teams; includes pre-migration checklist, dual-write validation procedure, cutover steps, and rollback procedure
- Migration validation dashboards: Grafana dashboard comparing metrics/log counts between legacy source and PU-OBS destination during dual-write period

#### Out of Scope

- Actual migration execution (PU team responsibility with migration specialist support)
- Migration specialist hiring and staffing (RSG-4 resolved separately — migration specialists are Phase 3 hires)
- ELK source system management (out of scope for PU-OBS)

#### Dependencies

- Blocked by: EPIC-8 (telemetry collection pipeline — adapters feed into OTel Collector), EPIC-12 (alert rule migration may reference Mimir ruler configuration)
- Blocks: None

#### Success Criteria

- Prometheus federation adapter is configured and a test remote Prometheus instance is successfully federated into Mimir
- At least one Zabbix/Nagios exporter is configured and producing metrics in OTel Collector
- ELK dual-write is implemented; a test log stream appears in both Loki and Elasticsearch simultaneously
- Migration runbook exists for each supported legacy tool (Prometheus, Zabbix/Nagios, ELK minimum)
- Migration validation dashboard confirms metric/log count parity between source and destination during dual-write

#### Key Risks

- Legacy tool domain expertise (Zabbix/Nagios/Centreon) is not currently on the PU-OBS team (RSG-4) — migration adapter work must wait until migration specialists are hired in Phase 3
- ELK dual-write may increase storage costs significantly during the migration period — capacity planning must account for this

---

### EPIC-21: External OBS-as-a-Service

**Domain**: Commercial
**Goal**: Design and implement the external OBS-as-a-Service offering — self-service onboarding, tier-based or usage-based pricing, external tenant isolation model, and managed dashboard/alerting experience — enabling commercial launch for external customers in Phase 3.
**Priority**: Could Have
**Estimated Size**: 7 stories

#### Description

External customers are a Phase 3+ commercial offering (DG-4 pricing model unconfirmed; DG-8 multi-tenancy boundary for external customers unresolved; RG-4 self-service onboarding not designed; DG-1 AGPL legal review needed). This Epic is gated on multiple prerequisite decisions and Epics completing. The external offering uses the same OBS infrastructure but may require a different isolation model for external customers (separate namespace tier, quota enforcement, plan-based feature gating). The customer journey is designed: subscription via catalog → automated onboarding → managed dashboards → usage-based billing → graceful termination.

#### In Scope

- Design external tenant isolation model per EPIC-1 (DG-8) decision — implement the selected model (shared cluster with stronger namespace isolation, or separate external cluster)
- External self-service onboarding flow: customer subscribes via PU-PROV catalog → PU-PROV creates external tenant namespace → ObservabilityTenant CRD created with external customer quotas and plan features → customer receives OTel Collector configuration (resolves RG-4)
- Plan-based feature gating: implement quota enforcement per subscription tier (ingestion rate limits, storage limits, features enabled/disabled per plan); configure MeteringRule for external billing dimensions
- External customer pricing: implement the confirmed pricing model (DG-4 — tier-based, usage-based, or hybrid) in the billing export pipeline; extend MeteringReport CRDs for external billing
- Managed dashboard experience for external customers: read-only pre-built dashboards (no self-service creation); managed alert rules within plan quota; no remediation access for external customers (Phase 3)
- Termination flow: automated data retention during contractual retention period; automated data purge after retention; tenant configuration cleanup; generate final MeteringReport for billing
- External customer documentation and onboarding guide

#### Out of Scope

- External customer AI remediation access (future, post-Phase 3)
- Grafana plugin marketplace (2027+)
- Direct external customer support tooling (handled by commercial team)
- Self-hosted/on-premises OBS-as-a-Service deployment

#### Dependencies

- Blocked by: EPIC-7 (multi-tenancy foundation), EPIC-14 (metering pipeline for billing), EPIC-17 (AGPL legal clearance for commercial use), EPIC-18 (DR/backup for external customer SLAs), EPIC-1 (DG-8 multi-tenancy boundary decision, DG-4 pricing model decision — note DG-4 finalization may occur within this Epic's planning)
- Blocks: None

#### Success Criteria

- External customer self-service onboarding completes end-to-end: subscription → namespace creation → OTel Collector config received → data visible in Grafana → billing record created
- External tenant isolation is verified — external customer cannot access internal PU data, and vice versa
- Plan quota enforcement is tested — a customer at their ingestion rate limit receives backpressure, not data loss
- Termination flow is tested — data retention, purge, and billing finalization complete correctly
- AGPL legal clearance is confirmed (from EPIC-17) before any external customer is onboarded
- External customer documentation is published and reviewed by at least one external beta customer

#### Key Risks

- DG-8 (multi-tenancy boundary) decision may result in a separate external cluster requirement — this significantly increases infrastructure and operational cost; budget implications must be assessed before deciding
- DG-4 (pricing model) must be confirmed before billing can be implemented; if pricing changes after implementation, billing pipeline rework is required
- AGPL legal clearance (DG-1 from EPIC-17) is a hard gate — if AGPL is rejected and no acceptable alternative exists, the external offering may not be viable with the current tech stack

---

### EPIC-22: Service Catalogue & Business Model Definition

**Domain**: Commercial / Product Management
**Goal**: Produce a formal, complete service catalogue defining every service PU-OBS offers, to which customer segments, at which tiers, with which feature entitlements, quotas, and SLA commitments — for both internal PUs and external customers
**Priority**: Should Have
**Estimated Size**: 5

#### Description

The documentation mentions pricing tiers (Basic/Professional/Enterprise) and internal transfer pricing but never formally defines the service catalogue: what is observable about what we offer, to whom, at what cost model, and what is guaranteed at each tier. Without this, the platform cannot enforce tier boundaries at runtime (ObservabilityTenant CRD quota fields cannot be populated), the commercial team cannot sell the product, PUs cannot budget for observability, and downstream Epics (EPIC-21 external OBS-as-a-Service) lack the feature/quota model they need to implement. This Epic closes RG-9 and DG-9.

#### In Scope

- **Internal PU service catalogue**: Define the entitlement model for internal platform PUs — flat platform fee + usage-based overages, default quotas (ingestion rate, storage limits, number of alert rules, retention tiers), what is included by default vs requires explicit opt-in (profiling, AI alerting, AI remediation, SOC routing)
- **External customer service catalogue**: Define the three tiers formally:
  - Basic: features included (metrics+logs+traces collection, pre-built dashboards, standard alert rules, standard retention), features excluded (AI alerting, AI remediation, profiling, SOC routing, FinOps dashboards, extended retention), support level, ingestion rate limits, storage quotas
  - Professional: same structure, expanded entitlements (custom dashboards, AI alerting, log routing options, extended retention), pricing delta
  - Enterprise: same structure, full entitlements (AI remediation opt-in, dedicated resources, SLA commitment, SOC routing, FinOps dashboards, premium support), pricing delta
- **Feature-to-tier matrix**: A clear table mapping every PU-OBS capability to the tier(s) at which it is available
- **Quota model**: Define the default and maximum quota values per tier for: ingestion rate (samples/sec, log GB/day), active series, trace volume, profiling sampling rate, number of CRDs (AlertRule, RemediationRunbook, etc.), number of Grafana users, retention periods
- **SLA per tier**: Map RG-8 platform SLOs to per-tier SLA commitments — Basic gets best-effort, Professional gets 99.5%, Enterprise gets 99.9% (example values to be confirmed)
- **Service catalogue document** (customer-facing): A structured document listing every service, its description, tier availability, and how to enable it — usable by sales, customers, and PU teams
- **CRD integration**: Define how the service catalogue maps to ObservabilityTenant CRD fields (quotas, features enabled/disabled) and MeteringRule dimensions so the platform can enforce and bill per-tier at runtime

#### Out of Scope

- Actual pricing (EUR amounts) — that is a commercial decision outside this Epic's scope (DG-4 confirms the pricing model structure, not the specific prices)
- Implementation of quota enforcement (that is EPIC-21)
- Legal review of the commercial offering (that is EPIC-17)
- FinOps/GreenOps pricing details (deferred to PU-BILLING)

#### Dependencies

- Blocked by: EPIC-1 (DG-4 pricing model decision, DG-8 multi-tenancy boundary decision, RG-8 platform SLO definition), EPIC-17 (AGPL legal clearance to understand what can legally be offered commercially)
- Blocks: EPIC-21 (External OBS-as-a-Service cannot implement plan-based feature gating without the catalogue)

#### Success Criteria

- Service catalogue document exists and is reviewed/approved by product management, commercial team, and Tech Lead
- Feature-to-tier matrix is complete — every PU-OBS capability appears in exactly one or more tiers with clear rationale
- Quota values are defined for all tiers with backing from capacity planning (EPIC-19 may refine these)
- ObservabilityTenant CRD quota fields are mapped to catalogue entries — developers can implement enforcement from the catalogue without ambiguity
- Internal PU entitlement model is validated with at least 2 consuming PU leads
- External tier definitions are validated with at least 1 potential external customer or commercial stakeholder

#### Key Risks

- DG-4 (pricing model) and DG-9 (tier content) are both open decisions that this Epic must close — if stakeholders are unavailable or disagree, the Epic stalls
- The service catalogue may reveal that Phase 1 deliverables are insufficient for even a Basic tier — triggering scope adjustments in Phase 1 Epics
- Internal PU entitlement model affects every PU's budget planning; getting alignment across all PU teams may take significant time

---

### EPIC-23: Technology Stack Evaluation & PoC

**Domain**: Architecture / Infrastructure
**Goal**: Conduct a structured, hands-on Proof of Concept evaluating the Grafana LGTMP stack against Apache 2.0 alternatives for each observability pillar, producing formal decision documents that confirm or change the technology selection before deployment begins.
**Priority**: Must Have
**Estimated Size**: 6

#### Description

The current architecture document recommends Grafana LGTMP (Mimir, Loki, Tempo, Pyroscope, Grafana) based on feature comparison and unified multi-tenancy model (X-Scope-OrgID). However, this recommendation has never been validated through hands-on evaluation (DG-10). The architecture is in DRAFT status and the stack selection is not final. Apache 2.0 alternatives exist for each backend pillar — VictoriaMetrics (metrics), VictoriaLogs/OpenSearch (logs), Jaeger (traces), Parca (profiling) — and may offer advantages in performance, operational simplicity, resource consumption, or licensing (avoiding AGPL entirely for some pillars). This Epic runs a structured PoC on the provisioned cluster (EPIC-2) to evaluate each pillar independently. Each PoC produces a decision document with benchmarks, operational complexity assessment, and a formal recommendation. The outcome directly determines what EPIC-3 deploys. **Note:** Grafana as the visualization layer has no mature OSS alternative; the PoC focuses on backend storage/query engines. AGPL licensing implications are evaluated in EPIC-17 in parallel.

#### In Scope

- **Spike: Metrics backend PoC** — Deploy Grafana Mimir and VictoriaMetrics side-by-side on the PoC cluster; ingest identical synthetic metrics workload; compare: ingestion throughput, query latency (PromQL), resource consumption (CPU/memory/disk), HA behavior (simulate replica loss), S3 storage efficiency, operational complexity (configuration, upgrades, debugging), multi-tenancy model. Produce decision document. (Partially resolves DG-10)
- **Spike: Logs backend PoC** — Deploy Grafana Loki and VictoriaLogs side-by-side; ingest identical synthetic log workload; compare: ingestion throughput, query latency (LogQL vs VictoriaLogs query language), resource consumption, label cardinality handling, S3 storage efficiency, multi-tenancy model, log routing capabilities. Produce decision document. (Partially resolves DG-10)
- **Spike: Traces backend PoC** — Deploy Grafana Tempo and Jaeger side-by-side; ingest identical synthetic trace workload; compare: ingestion throughput, query capabilities (TraceQL vs Jaeger query), resource consumption, S3 storage efficiency, trace-to-log/metrics correlation support, sampling strategies. Produce decision document. (Partially resolves DG-10)
- **Spike: Profiling backend PoC** — Deploy Grafana Pyroscope and Parca side-by-side; ingest identical profiling data; compare: eBPF support, language coverage, flame graph quality, resource consumption, trace-to-profile correlation, multi-tenancy. Produce decision document. (Partially resolves DG-10)
- **Integration assessment** — Evaluate cross-pillar integration of the selected backends: if a mixed stack is chosen (e.g., VictoriaMetrics + Loki + Tempo), assess whether trace-to-metrics/logs correlation still works, whether Grafana data source support is complete, and whether the unified multi-tenancy model (X-Scope-OrgID) is preserved or requires per-backend tenant configuration
- **Final technology decision document** — Consolidate per-pillar decisions into a single ADR (Architecture Decision Record) in `docs/decisions/`; update PROJECT_CONTEXT.md and architecture document to reflect the confirmed stack (resolves DG-10)

#### Out of Scope

- Production deployment of the selected stack (EPIC-3)
- AGPL legal review (EPIC-17 — runs in parallel)
- Grafana visualization evaluation (no alternative exists)
- Performance tuning and SLO compliance (EPIC-10)

#### Dependencies

- Blocked by: EPIC-2 (Kubernetes cluster must be provisioned and S3 storage validated to run PoC workloads)
- Blocks: EPIC-3 (cannot deploy the production stack without knowing which backends to use)

#### Success Criteria

- Each of the 4 pillar PoCs produces a decision document with quantitative benchmarks (throughput, latency, resource usage) and qualitative assessment (operational complexity, community health, documentation quality)
- Decision documents are stored in `docs/decisions/` and follow ADR format
- Integration assessment confirms the selected combination works end-to-end (or documents specific integration gaps requiring mitigation)
- Final technology decision document is reviewed and approved by Tech Lead and DA
- EPIC-3 scope is updated to reflect the confirmed technology choices

#### Key Risks

- PoC scope creep — each pillar evaluation could expand indefinitely; strict time-boxing per spike is essential
- Mixed stack risk — choosing different vendors per pillar (e.g., VictoriaMetrics + Loki) may lose the unified multi-tenancy advantage of the full LGTMP stack; integration assessment must flag this
- AGPL result dependency — if EPIC-17 determines AGPL is unacceptable, LGTMP backends may be disqualified regardless of PoC results; the PoC should proceed in parallel and evaluate alternatives as viable fallbacks
- Grafana has no alternative — if AGPL is rejected, Grafana visualization is the critical gap regardless of backend choices

---

### EPIC-24: Platform Release & Upgrade Management

**Domain**: Operations / Process
**Goal**: Establish a comprehensive release management process and non-production environment strategy for the PU-OBS observability platform, enabling safe, tested, and predictable upgrades of all OSS components throughout the platform lifecycle.
**Priority**: Should Have
**Estimated Size**: 5

#### Description

The PU-OBS platform is built entirely on rapidly evolving open-source components. Grafana, Mimir, Loki, Tempo, Pyroscope, OTel Collector, and Fluent Bit all release frequently (monthly to quarterly) with new features, breaking changes, deprecations, and CVE patches. Currently there is no process for tracking upstream releases, testing upgrades, managing breaking changes, performing staged rollouts, or rolling back failed upgrades (PG-5). There is also no non-production environment where upgrades can be validated before production deployment (PG-6). Without this Epic, the platform will either fall dangerously behind on security patches and features, or suffer production incidents from untested upgrades. This Epic establishes the processes, tooling, and environments to manage the platform lifecycle safely.

#### In Scope

- **Spike: Upstream release tracking** — Evaluate approaches for monitoring upstream releases of all PU-OBS OSS components (GitHub watch, RSS feeds, Renovate/Dependabot for Helm charts, manual review cadence); select approach and configure automated notifications to the PU-OBS team (partially resolves PG-5)
- **Non-production environment strategy** — Design and provision dev/staging environments for the OBS platform: define environment topology (full replica vs lightweight single-replica), data seeding strategy (synthetic data generators), infrastructure requirements, GitOps branch strategy (dev → staging → prod promotion); implement using EPIC-2's GitOps tooling (resolves PG-6)
- **Upgrade testing process** — Define the standard operating procedure for testing a component upgrade: deploy new version in staging → run validation suite (ingestion, query, alerting, multi-tenancy) → canary in production → full rollout; document acceptance criteria for each component type (backend, collector, visualization)
- **Rollback procedures** — Define rollback procedures for each component class: Helm rollback for stateless components, data migration rollback for stateful backends (Mimir, Loki, Tempo schema changes), CRD version rollback strategy; test rollback procedures in staging
- **Breaking change management** — Define process for handling upstream breaking changes: impact assessment, migration planning, communication to consuming PUs (especially for CRD schema changes that flow through to tenants), documentation updates
- **Upgrade cadence policy** — Define target upgrade cadence per component class (security patches: within days; minor versions: within sprint; major versions: planned migration); document in operational runbook

#### Out of Scope

- Initial stack deployment (EPIC-3)
- CRD schema versioning and change management (EPIC-6, PG-4)
- Production SLO monitoring during upgrades (EPIC-10 — but upgrade validation uses EPIC-10's SLO definitions)
- Backup/restore during upgrades (EPIC-18 — but upgrade procedures reference backup as pre-upgrade step)

#### Dependencies

- Blocked by: EPIC-3 (must have a deployed stack to define upgrade procedures for), EPIC-9 (self-monitoring must exist to validate that upgrades don't degrade platform health)
- Blocks: None (but all future upgrade cycles depend on this Epic's outputs)

#### Success Criteria

- Non-production environment (at minimum: staging) is provisioned and receives automated GitOps deployments
- Upstream release tracking is configured and the team receives notifications for all OSS component releases
- Upgrade testing SOP is documented and has been executed at least once end-to-end (upgrade a component in staging → validate → promote to production)
- Rollback procedures are documented and tested for at least one stateless component (e.g., OTel Collector) and one stateful component (e.g., Mimir)
- Breaking change management process is documented with communication template for consuming PUs
- Upgrade cadence policy is published and reviewed by Tech Lead

#### Key Risks

- Non-production environment cost — running even a lightweight staging replica of the full observability stack requires significant infrastructure; budget approval may be needed
- Staging fidelity — a lightweight staging environment may not catch issues that only manifest at production scale (e.g., compaction failures, ingester ring rebalancing); define which validations require production-scale testing
- Upstream velocity — if multiple critical CVEs arrive simultaneously, the team may not have capacity to test and roll out all patches promptly; prioritization criteria needed

---

*End of EPICS.md*
