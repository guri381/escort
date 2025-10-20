# ActivateEscortResponseV1

This message is sent by the Autonomous Haulage System (AHS) in response to the `ActivateEscortRequestV1` messages from the Fleet Management System (FMS), indicating whether the Autonomous Vehicle (AV) is in state pending, activated or rejected with respect to the escort request.

| Sender | Triggered by | Triggers |
| --- | --- | --- |
| `AHS`  | `ActivateEscortRequestV1` |  |


## Message Attributes

The `ActivateEscortResponseV1` message consists of the following properties.

| Key | Value | Format | Required | Description |
| --- | :---: | :---: | :---: | --- |
| `"EscortId"` | EscortId | UUID | True | The escort id in which the truck is responding to |
| `"Status"` | [`Pending`, `Activated`, `Rejected`] | String | True | Determine whether the AV is in state pending, activated or rejected with respect to the escort request.<br/>- `Pending`: The AV has processed the escort and has scheduled it for activation.<br/>- `Activated`: The AV has activated the escort and it is now adhering to the specified protection zone.<br/>- `Rejected`: The AV is unable to activate the escort due to an error explained in `"Reason"` |
| `"Reason"` | String Enum | String | False | The reason for rejecting escort request |

> [!IMPORTANT]
> - AV shall only `Reject` an escort request if there is an error preventing it from processing the escort request.
> - An AV may choose to send an `Activate` response without sending a `Pending` response first. In this case, the AV is indicating that it has activated the escort and is adhering to the specified policy. An AV shall send a `Pending` response if it cannot immediately adhere to the escort.

## Examples
### Typical Message
```JSON
{
  "Protocol": "Open-Autonomy",
  "Version": 1,
  "Timestamp": "2021-09-01T12:00:00Z",
  "EquipmentIds": [
    "f0c3d5ab-2d6e-4a12-b9d9-9eaf1efc0abc",
  ],
  "ActivateEscortResponseV1": {
    "EscortId": "123e4567-e89b-12d3-a456-426614174000",
    "Status": "Activated"
  }
}
```

### Rejection Message
```JSON
{
  "Protocol": "Open-Autonomy",
  "Version": 1,
  "Timestamp": "2021-09-01T12:00:00Z",
  "EquipmentIds": [
    "f0c3d5ab-2d6e-4a12-b9d9-9eaf1efc0abc",
  ],
  "ActivateEscortResponseV1": {
    "EscortId": "123e4567-e89b-12d3-a456-426614174000",
    "Status": "Rejected",
    "Reason": "UnexpectedOffline"
  }
}
```
