# Message Header

This section defines the common top‑level envelope used by all messages in the Escort Protocol. Every application payload (e.g., `ActivateEscortRequestV1`, `ActivateEscortResponseV1`) SHALL be embedded as a single additional top‑level property alongside the header attributes defined here.

## Structure
The header conveys protocol identification, versioning, temporal context, and equipment addressing.

## Attributes
| Key | Type | Required | Description |
| --- | --- | :---: | --- |
| `Protocol` | String | Yes | Protocol identifier. SHALL equal `Open-Autonomy`. Receivers MUST reject messages with unknown protocol identifiers. |
| `Version` | Integer | Yes | Major protocol version. Backward compatibility for lower versions is OPTIONAL; unsupported versions SHOULD be rejected. |
| `Timestamp` | String (ISO 8601 UTC, ms precision) | Yes | Generation time of the message payload. SHALL be UTC. SHOULD include milliseconds. MUST NOT be more than a configured future skew (recommendation: 2 s) ahead of receiver clock. |
| `EquipmentIds` | Array<UUID> | Yes | Target or source equipment identifiers. FMS→AHS broadcast messages MAY list multiple recipients. AHS→FMS messages SHALL list exactly one UUID (the sender). Array MUST be non‑empty and contain unique UUIDs. |
| `CorrelationId` | UUID | No | Correlates request/response or multi‑step exchanges. Strongly RECOMMENDED for reliability diagnostics. |
| `MessageId` | UUID | No | Unique identifier for the specific message instance (supports idempotent deduplication). |
| `SiteId` | String | No | Identifier for deployment/site context where multiple sites share infrastructure. |

## Validation Rules
A header SHALL be rejected if any mandatory attribute is missing or invalid:
- `Protocol` ≠ `Open-Autonomy`.
- Unsupported `Version`.
- `Timestamp` unparsable, not UTC, or beyond allowed future skew.
- `EquipmentIds` empty, contains duplicates, or contains invalid UUID format.
- `CorrelationId` present but not a valid UUID.
- `MessageId` present but not a valid UUID.

Receivers SHOULD log the rejection reason with a standardized error code and MAY request retransmission.

## Semantics
- `EquipmentIds` ordering is not significant; uniqueness is required.
- When multiple `EquipmentIds` appear, the payload is interpreted as a simultaneous broadcast of identical content to each listed AV.
- `CorrelationId` SHOULD remain stable across all related responses.
- `MessageId` SHOULD be globally unique (UUID v4 recommended).

## Extensibility
Implementations MAY add new optional header fields provided they do not alter semantics of existing mandatory fields. Unknown optional fields SHALL be ignored by receivers but preserved if relayed.

## Examples
Example escort activation request broadcast (multiple recipients):
```json
{
  "Protocol": "Open-Autonomy",
  "Version": 1,
  "Timestamp": "2025-10-20T09:30:10.435Z",
  "EquipmentIds": [
    "f0c3d5ab-2d6e-4a12-b9d9-9eaf1efc0abc",
    "9b8b6d54-1234-4c81-a911-5555bbbb7777"
  ],
  "CorrelationId": "22222222-3333-4444-5555-666666666666",
  "MessageId": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
  "ActivateEscortRequestV1": { "EscortId": "00000000-0000-0000-0000-000000000001" }
}
```

Example escort activation response (single sender):
```json
{
  "Protocol": "Open-Autonomy",
  "Version": 1,
  "Timestamp": "2025-10-20T09:30:11.012Z",
  "EquipmentIds": [
    "f0c3d5ab-2d6e-4a12-b9d9-9eaf1efc0abc"
  ],
  "CorrelationId": "22222222-3333-4444-5555-666666666666",
  "MessageId": "bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
  "ActivateEscortResponseV1": { "EscortId": "00000000-0000-0000-0000-000000000001", "Status": "Active" }
}
```

## Notes
- Legacy headers without `CorrelationId` or `MessageId` MAY be accepted for interoperability; implementations SHOULD migrate to include them.
- Millisecond precision supports prediction latency monitoring; if unavailable, receivers MAY down‑grade to nearest second but SHOULD flag reduced precision.
- Future versions MAY introduce a signature or MAC field for integrity/authentication.

