# PulZ Governance Kernel

**Deterministic AI governance engine. Evidence-backed decisions. Confidence-gated actions. Tamper-proof audit chains.**

47KB of TypeScript. Zero dependencies. Unlimited use cases.

> Every other AI tool asks you to trust AI. The PulZ Kernel asks AI to prove itself — then decides whether it's allowed to act.

## What This Is

The PulZ Governance Kernel is a **runtime policy engine** for AI agents. It doesn't execute actions — it decides whether execution is allowed. It sits between the agent and the action, enforcing evidence requirements, confidence thresholds, approval workflows, and tamper-proof audit logging.

**This is not a guardrail.** Guardrails validate individual LLM inputs/outputs. The PulZ Kernel governs the entire decision lifecycle:

```
Evidence gathered → Confidence scored → Action classified → Approval routed → Decision recorded → Audit chain verified
```

## The 5 Systems

### 1. Evidence System
Every AI decision must be backed by evidence items with source provenance, confidence weights (0-1), and verification status.

```typescript
const evidence = createEvidenceItem({
  id: 'ev-001',
  type: 'api_response',
  source: { kind: 'api', ref: 'https://api.example.com/data' },
  excerpt: 'Temperature reading: 23.5°C',
  confidence_weight: 0.85,
  verified: true,
});
```

Evidence is tiered:
- **Tier 1** — Single-source, uncorroborated. Suitable for logging only.
- **Tier 2** — Multi-source with partial verification. Suitable for single approval.
- **Tier 3** — Multi-source, verified, traceable provenance. Required for high-impact actions.

### 2. Confidence Rubric
Numerical scoring (0-100) maps to action classes:

| Score | Action Class | What Happens |
|-------|-------------|--------------|
| 0-49 | **Blocked** | AI cannot act. Period. |
| 50-69 | **Approval Required** | Human must approve. Reversible actions only. |
| 70-89 | **Approval Required** | Human must approve. Reversible actions only. |
| 90-100 | **Automation Eligible** | Can auto-execute IF explicitly enabled. |

**Critical design choice:** `canExecuteWithoutApproval()` always returns `false`. The kernel categorically refuses unsupervised execution. Even at 100% confidence, a human must opt in.

### 3. Decision Frame
A full state machine for every decision:

```
draft → pending_review → approved → (executed)
                       → rejected
         approved → revoked
```

Each frame carries: objective, recommendation, evidence report ID, confidence score, risk level, allowed/blocked actions, action class, approval state, approver ID, timestamps.

### 4. Append-Only Audit Log
Blockchain-style hash chain. Every event is hashed. Each event's `before_hash` must match the previous event's `after_hash`.

```typescript
const log = new AppendOnlyLog();
log.append({ event_type: 'decision_approved', actor, related, snapshot_before, snapshot_after });

// Later: verify the entire chain is untampered
const intact = log.verifyChain(); // true if no modifications detected
```

### 5. Learning System
Incident log with root cause analysis and prevention gates. Records what failed, how it was detected, and what was changed to prevent recurrence.

## Action Classification

The kernel classifies every action into three governance tiers:

| Class | Route | When |
|-------|-------|------|
| **A** | Log + learn only | Low confidence, tier 1 evidence, low impact |
| **B** | Single approval | Moderate impact, tier 2+ evidence |
| **C** | Multi-gate approval | High impact, irreversible, or tier 3 evidence |

## Quick Start

```bash
npm install @pulz/kernel
```

```typescript
import {
  createEvidenceItem,
  createEvidenceReport,
  createDecisionFrame,
  transitionToReview,
  approveDecisionFrame,
  runAllGovernanceChecks,
  globalAuditLog,
} from '@pulz/kernel';

// 1. Gather evidence
const item = createEvidenceItem({ /* ... */ });
const report = createEvidenceReport({ items: [item], confidence_score: 75, evidence_tier: 'tier_2' });

// 2. Create decision
const frame = createDecisionFrame({
  id: 'dec-001',
  objective: 'Send outreach email to new contact',
  recommendation: 'Proceed with personalized template',
  evidence_report_id: report.id,
  evidence_tier: report.evidence_tier,
  confidence_score: report.confidence_score,
});

// 3. Governance check
const check = runAllGovernanceChecks(frame);
if (!check.passed) {
  console.error('Governance violations:', check.violations);
}

// 4. Route for approval
const reviewed = transitionToReview(frame);
const approved = approveDecisionFrame(reviewed, 'operator-ken');

// 5. Audit trail is automatic
console.log(globalAuditLog.verifyChain()); // true
```

## Architecture

```
packages/kernel/src/
├── primitives/          # Core types and data structures
│   ├── EvidenceItem     # Evidence with source provenance
│   ├── EvidenceReport   # Aggregated evidence reports
│   ├── ConfidenceRubric # Confidence scoring and action classification
│   ├── DecisionFrame    # Decision state machine
│   ├── GovernanceTypes  # Action classes, approval states, evidence tiers
│   └── AuditEvent       # Audit event structure
├── policies/            # Governance enforcement
│   ├── actionClassPolicy    # Determine action class from context
│   ├── confidencePolicy     # Evaluate confidence thresholds
│   ├── evidencePolicy       # Evidence tier requirements
│   └── governancePolicy     # Run all governance checks
├── validators/          # Input validation
│   ├── evidenceValidator    # Validate evidence items/reports
│   └── decisionValidator    # Validate decision state transitions
├── audit/               # Tamper-proof logging
│   ├── appendOnlyLog    # Hash-chain audit log
│   └── hash             # Deterministic hashing
└── learning/            # Incident tracking
    ├── incidentLog      # Append-only incident records
    └── verificationChecklist # Verification checklists
```

## Use Cases

- **AI Agent Governance** — Gate every agent action through evidence and approval
- **Governed Web Crawling** — Audit trail for every page visited, every extraction made
- **Coding Environments** — Ensure AI-generated code passes governance before execution
- **Enterprise Compliance** — EU AI Act Article 14 (human oversight) and Article 12 (record-keeping)
- **WebView MCP** — Governed browser agent with domain scoping and approval policies
- **Multi-Agent Systems** — Each agent gets its own audit chain, verified independently

## What This Is NOT

- Not an LLM. Not an agent. Not a runtime.
- Has no network access, no filesystem access, no execution capability.
- It is a pure decision engine — it decides IF something should happen, not HOW.
- Pair it with ORA (runtime) for the full stack.

## Compliance

The kernel's architecture maps directly to:
- **EU AI Act** Article 14 (Human Oversight) — approval_required is always true
- **EU AI Act** Article 12 (Record-Keeping) — append-only audit log with hash chain
- **OECD AI Principles** — human agency and oversight by design
- **PIPEDA** — audit trail for all data access decisions

## License

AGPL-3.0. Commercial licensing available from [3D3D Atlantic Cooperative](https://3d3d.ca).

## Credits

Created by **3D3D Atlantic Cooperative**. Powered by **ORA**.

*47KB. Zero dependencies. The conscience your AI agent needs.*
