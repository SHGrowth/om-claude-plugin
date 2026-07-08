---
project: crosscheck-fixture
generated: 2026-07-09
phase: 2-verifying
total_epics: 3
total_stories: 5
---

# Gap Analysis — Crosscheck Fixture

## Epic 3: Invoicing compliance

### Story 3.1: Manual KSeF e-invoice export and number recording
- **Description**: As an operator, I want to export an approved invoice to an FA XML file and record the resulting KSeF number, so that the submission obligation is met.
- **Acceptance criteria**:
  - [ ] FA XML generator produces a file conforming to the MF schema
  - [ ] Operator can manually enter the KSeF number on an approved invoice
  - [ ] Uploaded FA XML is archived with a checksum
- **Source**: fixture
- **Priority**: P0
- **Dependencies**: none
- **Status**: done

#### Gap analysis
- **Verdict**: ❌ Missing
- **Evidence**:
  - `packages/core/src/modules/sales`: no KSeF workflow
- **Grounding query**: `WYSLANA_KSEF`
- **Grounding source**: checkout
- **Gaps**:
  - No FA XML generator
- **Effort**: 4
- **Suggested implementation path**:
  - Build the export flow
- **Upstream pipeline**: official-modules PR #29 (open)
- **Investigated**: 2026-07-09

### Story 3.2: Invoice preview by clerk
- **Description**: As a clerk, I want to preview an invoice and email it to the customer, so that they receive it quickly.
- **Acceptance criteria**:
  - [ ] Clerk can preview an invoice and email it to the customer
- **Source**: fixture
- **Priority**: P1
- **Dependencies**: none
- **Status**: done

#### Gap analysis
- **Verdict**: ✅ Implemented
- **Evidence**:
  - `packages/core/src/modules/sales`: preview exists
- **Grounding query**: `InvoicePreview`
- **Grounding source**: checkout
- **Gaps**:
  - none
- **Effort**: 0
- **Suggested implementation path**:
  - none
- **Upstream pipeline**: none
- **Investigated**: 2026-07-09

## Epic 5: Integrations

### Story 5.1: Tenant webhook notifications for order events
- **Description**: As an integrator, I want tenant webhook notifications, so that external systems stay in sync.
- **Acceptance criteria**:
  - [ ] Tenant webhook fires on order events
- **Source**: fixture
- **Priority**: P1
- **Dependencies**: none
- **Status**: done

#### Gap analysis
- **Verdict**: ❌ Missing
- **Evidence**:
  - `packages/core/src/modules/events`: no tenant webhook dispatch
- **Grounding query**: `tenant webhook`
- **Grounding source**: checkout
- **Gaps**:
  - No webhook outbox
- **Effort**: 3
- **Suggested implementation path**:
  - Follow the planned spec
- **Upstream pipeline**: spec: .ai/specs/SPEC-040-2026-03-01-tenant-webhooks-outbox.md (planned, unbuilt)
- **Investigated**: 2026-07-09

### Story 5.2: Quarterly compliance attestation reporting
- **Description**: As a compliance officer, I want a quarterly attestation report, so that audits go smoothly.
- **Acceptance criteria**:
  - [ ] Compliance officer can download a quarterly attestation report
- **Source**: fixture
- **Priority**: P2
- **Dependencies**: none
- **Status**: done

#### Gap analysis
- **Verdict**: ❌ Missing
- **Evidence**:
  - `packages/core/src/modules/reports`: no attestation report
- **Grounding query**: `attestation report`
- **Grounding source**: checkout
- **Gaps**:
  - No quarterly attestation report
- **Effort**: 2
- **Suggested implementation path**:
  - Build the report
- **Upstream pipeline**: PR #108 (open)
- **Investigated**: 2026-07-09

## Epic 6: Warehouse

### Story 6.1: Barcode scanning at goods receipt
- **Description**: As a warehouse worker, I want barcode scanning at goods receipt, so that intake is fast.
- **Acceptance criteria**:
  - [ ] Barcode scanner input creates a receipt line
- **Source**: fixture
- **Priority**: P2
- **Dependencies**: none
- **Status**: needs-review

#### Gap analysis
<!-- Filled by phase 2 via the gate. Do not edit by hand. -->
- **Verdict**: ⚪ not yet analyzed
- **Evidence**:
- **Grounding query**:
- **Grounding source**:
- **Gaps**:
- **Effort**:
- **Suggested implementation path**:
- **Upstream pipeline**:
- **Investigated**:
