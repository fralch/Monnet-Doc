# Webhook

The merchant must implement an **HTTP POST** endpoint (webhook) to receive asynchronous notifications about changes in the subscription status.

When the subscription status is updated by the processor, Monnet will send a POST request to the webhook URL developed by the merchant and configured during the integration process. This notification will be triggered for each change in the subscription lifecycle, enabling the merchant to track and update the process in real time.

The following section details the complete set of data and parameters that will be received in the request body to perform the operation.

## 📦 Request Payload

| Field | Description |
| :--- | :--- |
| subscriptionId | **Integer** <br> Unique identifier generated and assigned by Monnet to each merchant. Required for identifying the merchant across all transactions and API operations. |
| chargeType | **String** <br> Indicates the type of subscription configured for the customer, as communicated in the notification.(ON_DEMAND, RECURRENT) |
| customerId | **String** <br> Indicates the unique customer identifier associated with the subscription, as communicated in the notification. |
| originType | **String** <br> Subscription source channel (WEB, MOBILE, etc.) |
| status | **String (Optional)** <br> Status code of the processing result. See **StatusDescripton** table. |
| statusDescription | **String** <br> Status description of the processing result. See **StatusDescripton** table. |
| errorDetails | **String (Optional)** <br> Container for error information returned when the processing fails. |
| code | **String (Optional)** <br> Processing status code (e.g., 9099 = success, any other = error) <br> Ver Error Code |
| message | **String (optional)** <br> Error description providing additional context about the failure |
| metadata | **Array(optional)** <br> Contains up to 5 key–value pairs used to replicate the metadata originally provided during the subscription creation. This field must only be included if metadata was sent in the subscription creation request; otherwise, it must be omitted. |
| key | **String (required)** <br> Identifier of the additional data sent within the metadata. |
| value | **String (required)** <br> Value associated with the corresponding key within the metadata. |

Below are the descriptions of the different status values that may be included in the notification payload. The merchant’s endpoint must be able to interpret these values and execute the corresponding actions within their internal systems.

## StatusDescription

| Status | StatusDescription |
| :--------- | :------------------------------------------- |
| PENDING | En espera de procesamiento o confirmación |
| EXPIRED | La suscripción ha expirado |
| AUTHORIZED | La suscripción fue autorizada exitosamente |
| FAILED | Falló el procesamiento de la suscripción |
| CANCELLED | La suscripción fue cancelada |
| DENIED | La suscripción fue denegada por el processor |

## 📦 Request Payload (JSON)

### Success Example without metadata

```json
{
  "subscriptionId": 6,
  "chargeType": "ON_DEMAND",
  "customerId": "006123061",
  "originType": "MOBILE",
  "status": "PENDING",
  "statusDescription": "En espera de procesamiento o confirmación"
}
```

### Success Example with metadata

```json
{
  "subscriptionId": 6,
  "chargeType": "ON_DEMAND",
  "customerId": "006123061",
  "originType": "MOBILE",
  "status": "PENDING",
  "statusDescription": "En espera de procesamiento o confirmación",
  "metadata": [
    {
      "key": "MerchantReference",
      "value": "98212321"
    }
  ]
}
```

### Error Example without metadata

```json
{
  "subscriptionId": 6,
  "chargeType": "ON_DEMAND",
  "customerId": "006123061",
  "originType": "MOBILE",
  "statusDescription": "La suscripción fue denegada por el processor",
  "errorDetails": {
    "code": "9099",
    "message": "Error"
  }
}
```

### Error Example with metadata

```json
{
  "subscriptionId": 6,
  "chargeType": "ON_DEMAND",
  "customerId": "006123061",
  "originType": "MOBILE",
  "statusDescription": "La suscripción fue denegada por el processor",
  "errorDetails": {
    "code": "9099",
    "message": "Error"
  },
  "metadata": [
    {
      "key": "MerchantReference",
      "value": "98212321"
    }
  ]
}
```

## ✅ Expected Webhook Response

The webhook should return **HTTP 200 OK** to confirm successful receipt of the notification.

## 🔐 Recommended Security

It is recommended to implement IP whitelisting as the authentication mechanism to ensure the legitimacy of incoming notifications.

## Errors Code

| Error Code | Error Message |
| :--------- | :------------------------------ |
| 9025 | Unable to locate record on file |
| 9099 | Error |
