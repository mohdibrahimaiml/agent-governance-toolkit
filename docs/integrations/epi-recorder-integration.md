# EPI Recorder + AGT Integration

> **Status: community-contributed third-party integration.** Maintained upstream at <https://github.com/mohdibrahimaiml/epi-recorder>.

This document describes how the Agent Governance Toolkit (AGT) composes with EPI Recorder, a portable evidence packaging system for AI agent execution.
## Positioning

AGT provides runtime governance for agent actions. EPI Recorder packages that governance evidence into a portable, cryptographically sealed .epi artifact that can be independently verified outside the AGT runtime.

| Concern | AGT | EPI Recorder |
|---------|-----|--------------|
| Output | In-memory audit log, FileAuditSink JSONL | Portable .epi artifact with Ed25519 signature |
| Verification | Runtime trust (agt verify --evidence) | Independent offline verification (browser, CLI) |
| Portability | Runtime-dependent | Self-contained (verifier embedded in artifact) |
| Audience | Operator / developer | Regulator, auditor, compliance officer |

## Field Mapping

AGT audit.export() entries map to EPI steps as follows:

| AGT Field | EPI Target | Mapping |
|-----------|-----------|---------|
| event_type | kind | Translated: tool_invocation to tool.call, policy_evaluation to policy.eval |
| action | governance.action | Translated: allow to allowed, deny to denied |
| outcome | governance.outcome | Translated: success to completed, failure to failed |
| agent_did | governance.agent_did | Preserved verbatim |
| timestamp | timestamp | Exact copy |
| resource | content.resource | Exact copy |
| policy_decision | content.policy_decision | Exact copy |
| data | content.agt_data | Preserved raw under namespace |
| entry_hash | content.agt_entry_hash | Preserved for chain verification |
| entry_id | content.agt_entry_id | Exact copy |
| Unknown fields | content.agt_unknown_fields | Preserved raw |

## Usage

```python
from epi_recorder.integrations.agt_adapter import import_agt

epi_path, report = import_agt("audit_export.json", workflow_name="loan-application")
```

```python
from epi_recorder.integrations.agt_adapter import export_evidence_receipt, build_agt_log_data
receipt = export_evidence_receipt("loan.epi")
log_data = build_agt_log_data(receipt, "loan.epi")
audit.log(event_type="external_evidence", data=log_data, outcome="success")
```

## Coverage Boundaries

| Capability | AGT | EPI |
|-----------|-----|-----|
| Runtime policy enforcement | Native | Not applicable |
| Portable signed evidence | External .epi artifact | Native |
| Offline verification | Requires AGT runtime | Browser or CLI, zero deps |
| Regulator-facing output | CLI or portal | Browser viewer embedded in artifact |
| SCITT transparency | Not available | Optional SCITT receipt |
| Pre-execution commitment | Not available | Not addressed (post-execution only) |

## Operational Considerations

- Zero AGT runtime dependencies. Only consumes AGT-exported file formats.
- Raw AGT evidence is embedded verbatim in the .epi artifact.
- Unknown AGT fields preserved under content.agt_unknown_fields.
- Mapping report records every transformation for auditability.
- Pre-execution commitment not addressed. EPI seals post-execution, which answers the regulator question for compliance evidence.

## Related

- EPI Recorder repo: https://github.com/mohdibrahimaiml/epi-recorder
- AGT adapter code: https://github.com/mohdibrahimaiml/epi-recorder/tree/main/epi_recorder/integrations/agt_adapter
- Discussion thread: https://github.com/microsoft/agent-governance-toolkit/discussions/806
- External accountability profiles: docs/integrations/external-operation-accountability-profiles.md
