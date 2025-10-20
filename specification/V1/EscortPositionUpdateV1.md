# EscortPositionUpdateV1

This message is sent by the Fleet Management System (FMS) to the Autonomous Haulage System (AHS) at a frequency of 1 Hz to report the current position and motion state of the active escorting vehicle (Escorter). The purpose is to enable all Autonomous Vehicles (AVs) to maintain an up‑to‑date dynamic protection zone around the escort.

| Sender | Triggered by | Triggers |
| --- | --- | --- |
| `FMS` | Escort activation (immediately after sending `ActivateEscortRequestV1`) and every 1 s while active | AHS to forward/update internal AV state used for protection zone maintenance |

## Message Attributes

The `EscortPositionUpdateV1` payload object contains the following properties (top-level message header fields such as `Protocol`, `Version`, `Timestamp`, `EquipmentIds`, and `EscorterId` are defined in `MessageHeader.md`).

| Key | Req. Level | Type | Unit / Format | Description |
| --- | --- | --- | --- | --- |
| `"EscorterId"` | shall | UUID | UUID | Identifier of the Escorter Vehicle |
| `"GPSWeek"` | shall | uint32 | GPS Week | GPS Week when the position sample was measured (NOT when message transmitted) |
| `"GPSMilliSecondInWeek"` | shall | uint32 | millisecond in GPS Week | millisecond in GPS Week when the position sample was measured (NOT when message transmitted) |
| `"TODO field name"` | should  | string | Id | Unique ID on redundant network channel e.g. V2X station id |
| `"Latitude"` | shall | double | degrees (WGS84) | 6+ decimal places (≈0.11 m). |
| `"Longitude"` | shall | double | degrees (WGS84) | 6+ decimal places. |
| `"Elevation"` | shall | double | meters | Elevation relative to WGS84 ellipsoid (2 decimals). |
| `"Heading"` | shall | double | degrees | 0–359; 0 = true north, clockwise increase. |
| `"Speed"` | shall | double | m/s | Instantaneous ground speed (consider conversion from native km/h). |
| `"LatitudeAccuracy"` | shall | double | meters (1σ) | 1‑sigma horizontal positional accuracy latitude component. |
| `"LongitudeAccuracy"` | shall | double | meters (1σ) | 1‑sigma horizontal positional accuracy longitude component. |
| `"ElevationAccuracy"` | shall | double | meters (1σ) | 1‑sigma vertical accuracy. |
| `"HeadingAccuracy"` | shall | double | degrees (1σ) | 1‑sigma heading accuracy. |
| `"SpeedAccuracy"` | shall | double | m/s (1σ) | 1‑sigma speed accuracy. |

> [!NOTE]
> - At minimum: `EscorterId`, `GPSWeek`, `GPSMilliSecondInWeek`, `Latitude`, `Longitude`, `Elevation`, `Heading`, `Speed` MUST be present.
> - Accuracy fields help AVs tune dynamic buffer growth; include when available.

> [!NOTE]
> Why V2X StationId?
> - V2X CAM messages can be received with smaller latency than the `EscortPositionUpdateV1` messages therefore StationId of the Escorter is a valuable information to the AV.

## Example
```JSON
{
  "Protocol": "Open-Autonomy",
  "Version": 1,
  "Timestamp": "2025-09-26T10:15:30.125Z",
  "EquipmentIds": [
    "f0c3d5ab-2d6e-4a12-b9d9-9eaf1efc0abc",
    "9b8b6d54-1234-4c81-a911-5555bbbb7777"
  ],
  "EscortPositionUpdateV1": {
    "EscorterId": "11111111-2222-3333-4444-555555555555",
    "GpsWeek": 2444,
    "GpsMilliSecondInWeek": 345678900,
    "V2XStationId": 23983958,
    "Latitude": 59.1546127,
    "Longitude": 17.6212361,
    "Elevation": 428.32,
    "Heading": 87.8,
    "Speed": 4.2,
    "LatitudeAccuracy": 0.8,
    "LongitudeAccuracy": 0.9,
    "ElevationAccuracy": 1.5,
    "HeadingAccuracy": 2.0,
    "SpeedAccuracy": 0.2,
  }
}
```

## Validation Rules
- SHALL be published at 1 Hz (±100 ms tolerance recommended).
- SHALL monotonically increase `Timestamp` (no duplicates or regressions within a session).
- SHALL include `Speed` even if zero.
- If accuracy metrics are unknown, omit the respective fields rather than sending zero.

## Failure / Degradation Handling
// TODO: Document how the system should respond to message loss, unavailable data, etc..
| Condition | Recommended Behavior |
| --- | --- |

## Versioning
Additional optional fields MAY be appended in future minor protocol versions; consumers SHALL ignore unknown fields.
