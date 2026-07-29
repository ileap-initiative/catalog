# iLEAP Approval Catalog

This repository is the official catalog of iLEAP approvals. Approvals are issued by the [SINE Foundation](https://sine.foundation) with the support of [Smart Freight Centre (SFC)](https://www.smartfreightcentre.org), under the [iLEAP Approval process](https://sinefoundation.notion.site/iLEAP-Certification-Pilot-2fe407933aab80e78c67ee2495f6e1b2).

Each approval is stored as an immutable JSON file. The git history is a log of when records were added, not the source of truth for approval history — that is provided by the records themselves.

---

## Repository structure

```
approvals/         One JSON file per granted approval
revocations/       One JSON file per revoked approval
tools/             One JSON file per tool and tool provider
schema/            JSON Schema definitions for validation
examples/          Annotated example files (one per record type)
  approvals/
  revocations/
  tools/
```

---

## Tools

Before an approval can be added to this catalog, the tool being approved must have a record in `tools/`. Files are named `<uuid>.json`, where the UUID is assigned when the tool is first registered.

A tool record contains:
- **`uuid`** — unique identifier for this tool record; used as the filename and referenced by approval records
- **`provider`** — the company developing and maintaining the tool, including optional SFC Certificate Number and renewal date
- **`software`** — the software being registered: name, optional description, and optional URL

See `examples/tools/example.json` for a fully annotated example.

---

## Approvals

Each file in `approvals/` represents an approval that was granted to a tool. Files are named `<uuid>.json`, where the UUID is the approval's own unique identifier (not the tool UUID).

An approval record contains:
- **`uuid`** — unique identifier for this approval record; used as the filename
- **`tool_id`** — UUID of the tool being approved; must match the `uuid` of a record in `tools/`
- **`approval`** — [iLEAP Technical Specifications](https://specs.ileap.global) standard, version, date, validity, and Data Transaction coverage
- **`test_evidence`** — references to the ACT and PACT conformance test runs that evidence the approval, including any exceptions (skipped or non-blocking test cases) with rationale

See `examples/approvals/example.json` for a fully annotated example.

### Data Transactions

| DT | Description | Requirement |
|----|-------------|-------------|
| DT1 | ShipmentFootprint | Mandatory |
| DT2 | TOC / HOC | Mandatory |
| DT3 | TransportActivityData | Optional |
| DT4 | — | Out of scope |

DT1 and DT2 must be `tested` for an approval to be valid. The schema enforces this.

### Validity

An approval is valid for 1 year from its `approval.date`, provided the provider's SFC Certification is renewed before `provider.sfc_certification_renewal_date`. If the SFC certification lapses, the iLEAP approval is no longer valid regardless of `valid_until`.

---

## Revocations

Approvals can become invalid before their `valid_until` date — for example if a provider's SFC Certification lapses, if their implementation is found to no longer conform, or if they voluntarily withdraw. When this happens, a revocation record is added to `revocations/` rather than modifying or deleting the original approval. This preserves a complete, auditable history of all approvals ever granted and the reasons they were ended.

Files in `revocations/` are named `<uuid>.json`, where the UUID is the revocation's own unique identifier. The original approval file is never modified.

A revocation record contains its own `uuid`, references the approval being revoked via `approval_id`, and includes the revocation date, reason, and optional notes.

Possible revocation reasons: `sfc_cert_lapsed` · `conformance_lost` · `voluntary_withdrawal` · `administrative`

See `examples/revocations/example.json` for a fully annotated example.

---

## Checking whether an approval is currently valid

An approval in `approvals/` is currently valid if **all** of the following hold:

1. No file exists in `revocations/` with a matching `approval_id`
2. Today's date is before `approval.valid_until`
3. The provider's SFC Certification has been renewed (check `provider.sfc_certification_renewal_date` in the tool record)

---

## Schema validation

All schemas are in `schema/` and follow [JSON Schema draft 2020-12](https://json-schema.org/draft/2020-12/schema).

To validate a file locally using [ajv](https://github.com/ajv-validator/ajv):

```sh
npm install ajv ajv-formats
ajv validate -s schema/tool.json -d tools/<uuid>.json --spec=draft2020 -c ajv-formats
ajv validate -s schema/approval.json -d approvals/<uuid>.json --spec=draft2020 -c ajv-formats
ajv validate -s schema/revocation.json -d revocations/<uuid>.json --spec=draft2020 -c ajv-formats
```
