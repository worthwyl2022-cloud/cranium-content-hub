# Cranium Core Implementation Guide — Enterprise Deployment Playbook

**Version**: 1.0  
**Last Updated**: August 25, 2026  
**Audience**: Enterprise architects, compliance officers, technical leads

---

## 1. Pre-Implementation: Discovery & Invariant Scoping (Weeks 1-4)

### 1.1 Stakeholder Alignment Workshop

**Participants**: Executive sponsor, domain SMEs, compliance officer, IT leadership, end users

**Agenda** (8 hours total):
1. **Cranium Core Architecture Overview** (1 hour)
   - Dual-Lane governance model
   - Lane A vs. Lane B responsibilities
   - Quarantine workflow and approval gates

2. **Risk & Threat Identification** (2 hours)
   - What LLM failures are most costly in your domain?
   - Which facts/parameters cannot be hallucinated?
   - Regulatory/compliance gaps?

3. **Invariant Brainstorm** (3 hours)
   - Domain-specific constitutional rules
   - Protected data categories
   - Critical dependencies and constraints

4. **Success Metrics Definition** (2 hours)
   - Baseline productivity/compliance metrics
   - Post-deployment KPIs
   - User adoption targets

### 1.2 Domain Invariant Inventory

Create a structured invariant catalog:

```yaml
Invariant Categories:
  - Data Integrity (PII, PHI, financial records)
  - Regulatory Compliance (SOX, HIPAA, GDPR, etc.)
  - Business Rules (pricing, approval authorities)
  - Historical Facts (timeline events, precedents)
  - Character/Entity Rules (in narrative systems)
  
Format per Invariant:
  id: INV-001
  category: Data Integrity
  description: "Patient SSN cannot be modified after initial entry"
  mass_weight: 15.0  # Higher = stronger enforcement
  verification_method: "Cryptographic hash vs. original"
  exception_policy: "Requires CEO + compliance officer approval"
  audit_trail: "Full logging with timestamp and approver ID"
```

### 1.3 Regulatory & Compliance Mapping

**Checklist**:
- [ ] Identify applicable regulations (HIPAA, GDPR, SOX, SEC, etc.)
- [ ] Map regulations to specific invariants
- [ ] Document audit trail requirements
- [ ] Verify multi-tenancy isolation needs
- [ ] Plan for incident response & breach notifications

**Deliverable**: Compliance impact assessment (1-2 pages)

---

## 2. Implementation: Cranium Core Deployment (Weeks 5-10)

### 2.1 Infrastructure Setup

**Requirements**:
- Existing LLM integration (Gemini, Claude, GPT, Llama endpoint)
- SQLite instance (for invariant storage) or managed database
- API gateway for prompt/response interception
- Audit logging infrastructure

**Deployment Options**:

**Option A: Cloud-Native (Recommended)**
```
User App
   ↓
API Gateway (Cranium Core wrapper)
   ├─ Lane A: Immutable Canon Lookup
   ├─ Lane B: Valence Field + Quarantine
   └─ Audit Logger → CloudSQL
   ↓
LLM Endpoint (Gemini/Claude/GPT)
```

**Option B: On-Premises**
```
User App
   ↓
Cranium Core Docker Container
   ├─ Local SQLite (invariants)
   ├─ Lane A & B Logic
   └─ Audit File System
   ↓
LLM Endpoint (local or cloud)
```

### 2.2 Invariant Import & Initialization

**Step 1**: Load invariant catalog into Lane A store
```bash
cranium import-invariants --source=invariant-catalog.yaml \
                          --environment=production \
                          --encryption=AES-256
```

**Step 2**: Configure quarantine approval workflows
```yaml
Quarantine Rules:
  high_sensitivity:
    fields: [patient_identifier, dosage, financial_amount]
    default_action: QUARANTINE
    approval_required: 2  # Require 2 approvers
    timeout_hours: 24

  medium_sensitivity:
    fields: [clinical_notes, contract_summary]
    default_action: QUARANTINE
    approval_required: 1
    timeout_hours: 8
```

**Step 3**: Test Lane A enforcement
```bash
cranium test-invariants --scenario=adversarial-probes \
                        --temperature-range=[0.1,1.2] \
                        --report=html
```

### 2.3 Pilot Deployment: 10-20% Traffic

**Phase 1A**: Non-production testing
- Route 10% of staging traffic through Cranium Core
- Monitor latency, quarantine rates, false positives
- Collect user feedback on quarantine UI

**Metrics to Track**:
- Lane A violation rate (expected: <0.5%)
- Quarantine acceptance rate (expected: 75-90%)
- False positive rate (expected: 2-5%)
- P50/P95/P99 latency (expected: <5 ms total overhead)
- User approval time per item (target: <2 minutes)

**Duration**: 2-3 weeks minimum

**Go/No-Go Criteria**:
- ✅ Latency impact <5 ms (P99)
- ✅ Zero false positives on critical invariants
- ✅ Quarantine UI usability >4.0/5.0 in surveys
- ✅ Regulatory compliance sign-off received

### 2.4 Team Training & Workflow Enablement

**Training Tracks** (4 hours total per role):

**Executive Sponsor** (30 min):
- Architecture overview
- ROI and success metrics
- Risk mitigation strategy

**Domain SMEs & End Users** (2 hours):
- Quarantine review workflow
- Common rejection scenarios
- Appeal/exception process

**Compliance & IT Leadership** (2.5 hours):
- Audit trail and compliance reporting
- Incident response procedures
- Scaling and maintenance

**Deliverable**: Role-based training videos + job aids

---

## 3. Rollout: Staged Full Production Deployment (Weeks 11-14)

### 3.1 Canary Deployment (Week 11)

**Traffic Split**:
- 10% Cranium Core
- 90% Legacy (bypass Cranium Core)

**Monitoring Dashboard**:
```
Lane A Triggers:          5.2% of requests
Quarantine Holds:         18.3% of responses
False Positive Rate:      0.8%
User Approval Rate:       87.2% (avg 2.1 min)
Total Latency Impact:     +2.6 ms (P95)
Compliance Violations:    0
```

**Success Criteria**:
- No significant latency regression
- <1% false positive rate on critical invariants
- User satisfaction >4.2/5.0
- Zero production incidents linked to Cranium Core

### 3.2 Gradual Ramp (Weeks 12-13)

**Week 12**: 25% → 50% traffic
- Continue monitoring all metrics
- Weekly stakeholder sync
- Iteratively tune invariant sensitivity

**Week 13**: 50% → 100% traffic
- Final compliance validation
- Org-wide user communication
- Establish ongoing support model

### 3.3 Steady-State Operations (Week 14+)

**Ongoing Monitoring**:
```
Daily Metrics:
├─ Quarantine rejection rate (target: <5%)
├─ User approval time (target: <3 min)
├─ System latency (target: <3 ms overhead)
├─ Compliance incidents (target: 0)
└─ Invariant violation attempts (log & alert)

Weekly Review:
├─ Quarantine patterns (false positive analysis)
├─ User feedback collection
├─ Audit trail review (compliance check)
└─ Performance optimization

Monthly Governance:
├─ Invariant effectiveness assessment
├─ User training refresher
├─ Regulatory update check
└─ ROI verification against baseline
```

---

## 4. Invariant Tuning & Continuous Improvement

### 4.1 False Positive Reduction

**Common Issue**: Quarantine rejection rate too high (>10%)

**Diagnosis**:
1. Review rejected items in audit log
2. Identify patterns (certain user types, content categories, etc.)
3. Assess whether invariant too strict

**Remedy**:
```yaml
Invariant Before (Strict):
  rule: "Patient name cannot be modified"
  mass: 15.0
  result: 23% false positive rate

Invariant After (Tuned):
  rule: "Patient name cannot be modified EXCEPT correction entries 
         marked with [CORRECTION] flag"
  mass: 14.5
  exception_criteria: "correction_flag = true AND approver_role = nurse"
  result: 2% false positive rate
```

### 4.2 Invariant Expansion

As you identify new risks, add invariants:

```yaml
New Invariant (Month 3):
  id: INV-042
  category: Regulatory Compliance
  description: "Interest rate spreads cannot exceed 250 basis points"
  trigger_condition: "numerical_assertion matches pattern 'spread > 250 bps'"
  enforcement: "Lane A hard rejection"
  audit_requirement: "SEC Rule 10b5 compliance logging"
```

### 4.3 User Feedback Loop

**Quarterly Surveys**:
- Quarantine workflow efficiency (target: >4.2/5.0)
- Training adequacy (target: >4.0/5.0)
- System reliability (target: >4.5/5.0)

**Monthly Focus Groups** (optional):
- 6-8 power users per domain
- Discuss pain points and improvement ideas
- Vote on new invariant proposals

---

## 5. Compliance & Audit

### 5.1 Audit Trail Structure

Every Cranium Core decision logged with:
```json
{
  "timestamp": "2026-08-25T14:32:17Z",
  "request_id": "req-7f8e2d3c",
  "user_id": "u-42156",
  "workspace_id": "ws-medical-clinic",
  "input_prompt": "[truncated for privacy]",
  "lane_a_triggers": ["INV-012: Patient SSN protection", "INV-018: DOB immutability"],
  "lane_b_quarantine": true,
  "quarantine_reason": "Clinical assertion: 'morphine dose 500mg' exceeds safety threshold",
  "approval_status": "PENDING",
  "approver_id": "pharm-0847",
  "approval_time": "2026-08-25T14:35:04Z",
  "approval_action": "REJECT - dose unsafe; recommend 10mg",
  "final_response": "[LLM response with rejected content removed]",
  "latency_ms": 2.4,
  "regulatory_flags": ["HIPAA_AUDIT_REQUIRED"]
}
```

### 5.2 Compliance Reporting

**Monthly Reports**:
- Invariant violation attempts (count + categories)
- Quarantine hold rate and approval timeline
- User appeal/override requests
- Regulatory incidents (0 target)

**Annual SOC 2 Audit**:
- Audit trail completeness review
- Access control verification
- Encryption status check
- Incident response readiness

---

## 6. Troubleshooting & Support

### Common Issues & Resolutions

| Issue | Cause | Resolution |
|-------|-------|-----------|
| **High latency (>10 ms)** | Lane B NLI classifier overloaded | Scale up container; optimize invariant count |
| **High quarantine rate (>15%)** | Invariants too strict | Tune mass weights; add exceptions |
| **User approval bottleneck** | Quarantine queue backing up | Add more approvers; streamline UI |
| **Regulatory compliance gap** | Audit trail not logging correctly | Verify database connection; check firewall |
| **False positives on Lane A** | Invariant matching too broad | Narrow string matching; use regex refinement |

### Support Channels

- **Email**: wyl.mathes@cranium.ambi.cc
- **Slack**: Cranium Core workspace (for enterprise clients)
- **Documentation**: Full API reference at content-hub repo
- **Bug Reports**: GitHub issues (private repo for clients)

---

## 7. Success Metrics & ROI Tracking

### Pre-Deployment Baseline

Document these metrics **before** Cranium Core deployment:

```yaml
Baseline Metrics:
  productivity:
    - Documents completed per FTE per week
    - Compliance review turnaround time
    - LLM-assisted vs. manual ratio
  
  compliance:
    - Regulatory incidents per quarter
    - Audit findings per review
    - Hallucination incidents (if tracked)
  
  cost:
    - Review/approval labor hours
    - Regulatory remediation costs
    - System infrastructure costs
```

### 90-Day Post-Deployment Assessment

Compare to baseline:

```yaml
Post-Deployment Metrics (90 days):
  productivity:
    - ✅ Documents/FTE/week: 12 → 30 (+150%)
    - ✅ Review turnaround: 168 hrs → 24 hrs (-86%)
    - ✅ LLM assist utilization: 5% → 65% (+1200%)
  
  compliance:
    - ✅ Regulatory incidents: 2 → 0 (100% reduction)
    - ✅ Audit findings related to LLM: 3 → 0
    - ✅ Hallucination incidents: 12/mo → 0 (100% elimination)
  
  cost:
    - ✅ Annual labor savings: $4.2M
    - ✅ Deployment cost: $180K
    - ✅ ROI payback period: 10 weeks
```

### Ongoing Quarterly Reviews

**Q1 Review Agenda**:
- Compare vs. baseline & targets
- Identify process improvements
- Gather user feedback
- Adjust invariants based on patterns
- Plan next-quarter roadmap

---

## 8. Conclusion & Next Steps

**You are now ready to deploy Cranium Core in your enterprise.**

### Implementation Timeline Summary

| Phase | Duration | Key Milestone |
|-------|----------|---------------|
| Discovery & Scoping | 4 weeks | Invariant catalog + compliance mapping |
| Infrastructure & Pilot | 6 weeks | 10-20% traffic with <5ms latency |
| Staged Rollout | 4 weeks | 100% production traffic by week 14 |
| Steady State | Ongoing | Monitoring, tuning, compliance verification |

### Contact for Enterprise Deployment

**Cranium Core Deployment Team**

📧 **wyl.mathes@cranium.ambi.cc**  
🌐 **worth-wyl-media-d9ead881.base44.app**  
📞 **702-602-7543**

**Ready to start your enterprise transformation?** Schedule your Architecture Review workshop today.

---

*© 2026 Cranium Core Research & Governance Architecture Team. Confidential — Enterprise Use Only*
