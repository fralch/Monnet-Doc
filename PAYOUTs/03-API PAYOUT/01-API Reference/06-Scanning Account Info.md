# Scanning Account Info

### Description

This endpoint enables merchants to retrieve the financial entities (banks and wallets) currently associated with a mobile phone number in **Peru**. The service provides the list of eligible entities available to receive deposits, helping merchants improve onboarding conversion and streamline payment setup.

{% hint style="danger" %}
**Note:** At this stage, the service is only available in Peru using the mobile phone number as the input parameter. In future releases, it may be expanded to other countries and support different identifiers (e.g., account information) instead of the mobile number.
{% endhint %}

### Considerations

* **Geographic scope:** Currently available only for **Peru** (`country = PER`).
* **Account type:** Only valid for **Cellphone Number** (`account_type = 4`).
* **Functional scope:** This service does **not** trigger any disbursement flow or funds validation; it is strictly informational.
* **Authentication:**  Based on [OAuth 2.0 bearer tokens](https://docs.monnet.io/OoF1XuWTEpPjI0i2ERRO/quick-start/authentication/authorization-protocol-oauth-2.0-bearer-tokensen).
* **Rate Limit:** There is a limit of 60 requests per minute (TPM); otherwise, requests will be rejected.

### Input Parameters

**Endpoint Production:** [**https://scanning.payout.monnet.io/ms-payouts-scanning-entities/api/v1/scanning/account-info**](https://scanning.payout.monnet.io/ms-payouts-scanning-entities/api/v1/scanning/account-info)

**Endpoint Certification:** [**https://cert.scanning.payout.monnet.io/ms-payouts-scanning-entities/api/v1/scanning/account-info**](https://cert.scanning.payout.monnet.io/ms-payouts-scanning-entities/api/v1/scanning/account-info)

### Request

#### Headers

| Header        | Type   | Mandatory | Description                                                                                             | Format/Example                       |
| ------------- | ------ | --------- | ------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| Content-Type  | string | Yes       | Indicates the media type of the request payload. Must be JSON format.                                   | application/json                     |
| MID           | string | Yes       | Merchant identifier assigned by Monnet. Used to identify the requesting merchant.                       | 1234                                 |
| Authorization | string | Yes       | JWT access token obtained from the authentication endpoint. Must be sent **without** the bearer prefix. | eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9 |

#### Fields

The request must include the following fields:

| Field                | Type            | Mandatory | Description                                                                | Format / Example                         |
| -------------------- | --------------- | --------- | -------------------------------------------------------------------------- | ---------------------------------------- |
| `request_id`         | `string` (UUID) | Yes       | Unique identifier per request/merchant, used for traceability.             | `"fa4f11ce-a997-46fb-b7c6-351975f0a6d4"` |
| `account_type`       | `string`        | Yes       | Type of identifier. For this scope only: **`4= Cellphone Number`**.        | `"4"`                                    |
| `account_identifier` | `string`        | Yes       | 9-digit mobile phone number starting with **9**. Regex validation applied. | `"948474375"`                            |
| `country`            | `string` (ISO3) | Yes       | Country code. For this scope, only **PER** (Peru) is accepted.             | `"PER"`                                  |

### Example Request

```
{
  "request_id": "fa4f11ce-a997-46fb-b7c6-351975f0a6d4",
  "account_type": "4",
  "account_identifier": "948474375",
  "country": "PER"
}
```

### Response

| Field                | Description                                                                                      |
| -------------------- | ------------------------------------------------------------------------------------------------ |
| `traceId`            | Unique identifier per response, used for traceability.                                           |
| `request_id`         | Unique identifier per request/merchant, used for traceability.                                   |
| `account_identifier` | 9-digit mobile phone number starting with **9**. Regex validation applied.                       |
| `entityBrand`        | List of entities associated with the queried account                                             |
| `entitystatus`       | status of each entity at this moment (possible values: **"available"** and **"not\_available"**) |
| `date`               | Response date                                                                                    |

### Examples Response

a. Success with all entities available to receive payouts.

```
{
  "traceId": "3caa82a1-c23e-48b8-ab8b-125432863db3",
  "data": {
    "requestId": "fa4f11ce-a997-46fb-b7c6-351975f0a6d4",
    "accountIdentifier": "948474375",
    "entities": [
      { "entityBrand": "Yape", "status": "AVAILABLE" },
      { "entityBrand": "Plin", "status": "AVAILABLE" }
    ],
    "date": "2024-03-21T19:37:29Z"
  }
}
```

b. Example of an entity associated with the account but not available in Monnet for processing payouts:

```
{
  "traceId": "3caa82a1-c23e-48b8-ab8b-125432863db3",
  "data": {
    "requestId": "fa4f11ce-a997-46fb-b7c6-351975f0a6d4",
    "accountIdentifier": "948474375",
    "entities": [
      { "entityBrand": "Yape", "status": "AVAILABLE" },
      { "entityBrand": "BancoX", "status": "NOT_AVAILABLE" }
    ],
    "date": "2024-03-21T19:37:29Z"
  }
}
```

c. Example of an account with no registered entities

```
{
  "errors": [
    {
      "code": "PV002",
      "message": "The accountIdentifier has no associated entities"
    }
  ]
}
```

d. Example if a query is made for a country not supported by this API

```
{
  "errors": [
    {
      "code": "VAL004",
      "message": "Country not available"
    }
  ]
}
```

### Error Codes

Below are the error codes that the **Scanning Account Info** API service may return:

| Error Codes | Error Message                                                        |
| ----------- | -------------------------------------------------------------------- |
| 500         | Scanning service is temporarily unavailable. Please try again later. |
| AUTH001     | Authentication token is required                                     |
| AUTH002     | Invalid authentication token                                         |
| VAL001      | requestId duplicated                                                 |
| VAL002      | accountType not supported for country PER                            |
| VAL003      | accountIdentifier invalid format for country PER                     |
| VAL004      | Country not available                                                |
| VAL005      | Missing required field:                                              |
| VAL006      | request\_id invalid UUID format (must have at least 36 characters)   |
| RL001       | Too Many Requests                                                    |
| RU001       | No enabled providers available for country: \[COUNTRY\_CODE]         |
