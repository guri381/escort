# ActivateEscortRequestV1

This message is sent by the Fleet Management System (FMS) to the Autonomous Haulage System (AHS) to announce an escort instance that Autonomous Vehicles (AVs) SHALL acknowledge and enforce. Each AV SHALL respond with an `ActivateEscortResponseV1` indicating one of: Pending, Active, or Rejected (see `ActivateEscortResponseV1`).

| Sender | Triggered By | Expects |
| --- | --- | --- |
| FMS | Successful escort creation (immutable parameters established) | An `ActivateEscortResponseV1` from each addressed AV |

## Structure
The `ActivateEscortRequestV1` object conveys immutable escort parameters and a current Escorter position snapshot.

## Attributes

| Key | Type | Required | Description |
| --- | --- | :---: | --- |
| `EscortId` | UUID | Yes | Unique identifier of the escort instance. Stable for its lifecycle. |
| `EscorterId` | UUID | Yes | Identifier of the Escorter (staffed instrumented vehicle). |
| `Length` | Number (meters) | Yes | Protection Zone trailing distance. All Escortees SHALL remain within this longitudinal limit. |
| `Width` | Number (meters) | Yes | Lateral Protection Zone extent used in open areas. Lane boundaries supersede Width on roads. |
| `EscortPositionUpdate` | `EscortPositionUpdateV1` | Yes | Position snapshot used to seed AV prediction and Avoidance Zone calculation. |

> [!NOTE]
> Additional dynamic fields (e.g., speed, heading, accuracies) are encapsulated within `EscortPositionUpdateV1` and are not duplicated here.

## Validation Rules
An `ActivateEscortRequestV1` SHALL be rejected by an AV if any of the following conditions hold:
- Missing mandatory attribute.
- `Length` <= 0 or `Width` <= 0.
- Position snapshot timestamp older than a configured staleness threshold (implementation recommendation: > 5 s).
- Coordinates outside site bounds or failing basic latitude/longitude range checks.
- GPS week / millisecond combination inconsistent (e.g., millisecond >= 604800000).
- EscorterId mismatch between top-level and nested `EscortPositionUpdate` (if present there).

Receivers SHOULD log the rejection reason with a standardized error code.

## Idempotency
Duplicate requests (same `EscortId` and identical immutable parameters) SHALL NOT create a new escort context; receivers MUST treat them as retransmissions.

## Example
```json
{
  "Protocol": "Open-Autonomy",
  "Version": 1,
  "Timestamp": "2025-10-20T09:30:10.435Z",
  "EquipmentIds": [
    "f0c3d5ab-2d6e-4a12-b9d9-9eaf1efc0abc",
    "9b8b6d54-1234-4c81-a911-5555bbbb7777"
  ],
  "ActivateEscortRequestV1": {
    "EscorterId": "11111111-2222-3333-4444-555555555555",
    "EscortId": "00000000-0000-0000-0000-000000000001",
    "Length": 200.0,
    "Width": 6.0,
    "EscortPositionUpdate": {
      "EscorterId": "11111111-2222-3333-4444-555555555555",
      "GpsWeek": 2444,
      "GpsMilliSecondInWeek": 345678900,
      "Latitude": 59.1546127,
      "Longitude": 17.6212361,
      "Elevation": 428.32,
      "Heading": 87.8,
      "Speed": 4.2,
      "LatitudeAccuracy": 0.8,
      "LongitudeAccuracy": 0.9,
      "ElevationAccuracy": 1.5,
      "HeadingAccuracy": 2.0,
      "SpeedAccuracy": 0.2
    }
  }
}
```

## Notes
- `V2XStationId` omitted here; include it only if defined as part of `EscortPositionUpdateV1` schema.
- All numeric accuracy fields represent 1σ (one standard deviation) estimates in meters or degrees as contextually appropriate.
- Consumers SHOULD verify uniqueness of `EscortId` within active escort registry prior to accepting.

