# EscortPositionUpdateV1

This message is sent by the Fleet Management System (FMS) to the Autonomous Haulage System (AHS) at a nominal frequency of 1 Hz to report current position and motion state of the active Escorter. It enables Autonomous Vehicles (AVs) to maintain and update their Avoidance Zone relative to the Protection Zone.

| Sender | Triggered By | Effect |
| --- | --- | --- |
| FMS | Escort Pending or Active (after `ActivateEscortRequestV1`) and periodic (every ~1 s) | Provides latest Escorter pose for AV prediction & Avoidance Zone constraint |

## Structure
`EscortPositionUpdateV1` conveys an instantaneous pose sample (timestamped in GPS time) plus optional accuracy metrics and auxiliary identifiers.

## Attributes
| Key | Type | Unit | Required | Description |
| --- | --- | --- | :---: | --- |
| `EscorterId` | UUID | — | Yes | Identifier of the Escorter; MUST match escort definition. |
| `GpsWeek` | Integer | week count | Yes | GPS week number when sample measured (0–1023 rollover handling required). |
| `GpsMilliSecondInWeek` | Integer | ms | Yes | Milliseconds within GPS week (range 0–604799999). |
| `V2XStationId` | Integer | — | No | V2X station identifier enabling correlation with low‑latency V2X CAM data. |
| `Latitude` | Double | degrees | Yes | WGS84 latitude; precision ≥ 1e‑6 degrees. |
| `Longitude` | Double | degrees | Yes | WGS84 longitude; precision ≥ 1e‑6 degrees. |
| `Elevation` | Double | meters | Yes | Height above WGS84 ellipsoid; precision ≥ 0.01 m. |
| `Heading` | Double | degrees | Yes | Bearing clockwise from true north; range [0.0, 360.0). |
| `Speed` | Double | m/s | Yes | Ground speed (non‑negative); precision ≥ 0.1 m/s. |
| `LatitudeAccuracy` | Double | meters (1σ) | No | 1σ estimated latitude positional uncertainty. |
| `LongitudeAccuracy` | Double | meters (1σ) | No | 1σ estimated longitude positional uncertainty. |
| `ElevationAccuracy` | Double | meters (1σ) | No | 1σ estimated vertical uncertainty. |
| `HeadingAccuracy` | Double | degrees (1σ) | No | 1σ estimated heading uncertainty. |
| `SpeedAccuracy` | Double | m/s (1σ) | No | 1σ estimated speed uncertainty. |

## Semantics
- `GpsWeek` / `GpsMilliSecondInWeek` timestamp the measurement instant, not transmission time (which is provided by message `Timestamp` header).
- Negative speed values SHALL NOT be used; reversing movement is outside escort assumption scope.
- Accuracy fields, if present, MUST be strictly positive.
- Omitted accuracy fields indicate unknown values (MUST NOT be substituted with zero).
- Heading wraps at 360.0 degrees; a heading of 360.0 MUST NOT be sent (use 0.0 instead).

## Validation Rules
Reject sample if any mandatory attribute missing or invalid:
- `GpsMilliSecondInWeek` not in [0,604799999].
- `Latitude` outside [-90,90] or `Longitude` outside [-180,180].
- `Heading` < 0.0 or ≥ 360.0.
- `Speed` < 0.0.
- Any provided accuracy field ≤ 0.0.
- `EscorterId` mismatch with escort instance.

## Publication Requirements
- Nominal rate: 1 Hz. Implementations SHOULD keep interval jitter within ±100 ms.
- `Timestamp` (header) SHOULD monotonically increase; regressions MAY trigger prediction model expansion.
- `Speed` SHALL be included even if zero.

## Degradation Handling
| Condition | Recommended Behavior |
| --- | --- |
| Missed update (single interval) | AV expands Avoidance Zone using max feasible motion since last sample. |
| Consecutive missed updates (≥2) | AV progressively enlarges Avoidance Zone; may flag degraded escort tracking. |
| Stale GPS timestamp (> 5 s old) | Receiver MAY discard and request fresh sample; treat as missed update. |
| Invalid field detected | Reject sample; log error; retain last valid sample for prediction. |

## Examples
Full sample with optional fields:
```json
{
  "Protocol": "Open-Autonomy",
  "Version": 1,
  "Timestamp": "2025-10-20T10:15:30.125Z",
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
    "SpeedAccuracy": 0.2
  }
}
```

Minimal mandatory sample:
```json
{
  "Protocol": "Open-Autonomy",
  "Version": 1,
  "Timestamp": "2025-10-20T10:15:31.125Z",
  "EquipmentIds": ["f0c3d5ab-2d6e-4a12-b9d9-9eaf1efc0abc"],
  "EscortPositionUpdateV1": {
    "EscorterId": "11111111-2222-3333-4444-555555555555",
    "GpsWeek": 2444,
    "GpsMilliSecondInWeek": 345679000,
    "Latitude": 59.1546128,
    "Longitude": 17.6212362,
    "Elevation": 428.33,
    "Heading": 87.9,
    "Speed": 4.3
  }
}
```

## Versioning
Additional optional fields MAY be added in future versions. Receivers SHALL ignore unknown optional fields while preserving mandatory validation. Backward compatibility for removed optional fields SHOULD be maintained for at least one major protocol iteration.

## Notes
- V2X integration: `V2XStationId` allows associating faster V2X CAM data with the escort position timeline for refined prediction.
- Accuracy metrics enhance dynamic Avoidance Zone sizing; absence implies conservative expansion strategy.

