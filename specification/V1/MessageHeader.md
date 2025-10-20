# Message Header

This document describes the top-level data structure that serves as a message header for all other messages defined within this specification.

## Message Attributes
| Key | Value | Format | Required | Description |
| --- |:---:|:---:|:---:| --- |
| `"Protocol"` | `"Open-Autonomy"` | string | True | The protocol of the messages. |
| `"Version"` | 1 | integer | True | The version of the protocol. |
| `"Timestamp"` | DateTime | ISO 8601 (UTC)  | True | The date-time of when the message is sent in ISO 8601 format. |
| `"EquipmentIds"` | Array<EquipmentId> | UUID | True | The UUID identifying the equipment defined in the ISO 23725 - FleetDefinitionV2, indicating the equipment(s) that the message is intended for or from. In every message from the AHS to the FMS, this list should include only one UUID which corresponds to the original sender of the message. |

> [!IMPORTANT]
> `EquipmentIds` is an array (rather than the single `EquipmentId`) because some messages are broadcastet at 1 Hz to all active AVs. By listing all intended recipients in one payload the FMS avoids transmitting an identical position update separately to each AV, reducing bandwidth and processing overhead. This is a deliberate, deviation from the standard header pattern to support high‑frequency distribution efficiency.


### Message Header Examples
An example `ActivateEscortRequestV1` enclosed in the message header data structure (note multiple EquipmentIds)

```JSON
{
  "Protocol":"Open-Autonomy",
  "Version": 1,
  "Timestamp": "2018-10-31T09:30:10.435Z",
  "EquipmentIds": [
    "f0c3d5ab-2d6e-4a12-b9d9-9eaf1efc0abc",
    "9b8b6d54-1234-4c81-a911-5555bbbb7777"
  ],
  "ActivateEscortRequestV1": {...}
}
```

An example `ActivateEscortResponseV1` enclosed in the message header data structure (note single id in EquipmentIds)

```JSON
{
  "Protocol":"Open-Autonomy",
  "Version": 1,
  "Timestamp": "2018-10-31T09:30:10.435Z",
  "EquipmentIds": [
    "f0c3d5ab-2d6e-4a12-b9d9-9eaf1efc0abc",
  ],
  "ActivateEscortResponseV1": {...}
}
```
