Error Codes

# Error Codes

## Error Codes Related to Creation

| Error Code | Error Message                                                                             |
| :--------- | :---------------------------------------------------------------------------------------- |
| 0000       | Successfull Process (Link creation successfull)                                           |
| 0001       | Error in payinMerchantID not valid (the field is empty)                                   |
| 0002       | Error in payinAmount not valid (the field is empty)                                       |
| 0003       | Error in payinCurrency not valid (the field is empty)                                     |
| 0004       | Error in payinMerchantOperationNumber not valid (the field is empty)                      |
| 0005       | Error in payinVerification not valid (the field is empty)                                 |
| 0006       | Error in payinTransactionErrorURL not valid (the field is empty)                          |
| 0007       | Error in payinTransactionOKURL not valid (the field is empty)                             |
| 0008       | Error in payinProcessorCode not valid                                                     |
| 0009       | Error payinMerchantID not valid (it's wrong)                                              |
| 0010       | Error in payinVerification (it's wrong)                                                   |
| 0011       | Error in merchant not enabled                                                             |
| 0012       | Error in payinTransactionErrorURL not valid                                               |
| 0013       | Error in payinTransactionOKURL not valid                                                  |
| 0015       | Error in payinAmount format not valid                                                     |
| 0017       | Error in payinCurrency not valid                                                          |
| 0018       | Error in processor not valid                                                              |
| 0019       | Error in currency, not exist for merchant                                                 |
| 0022       | Error in transaction payinCustomerTypeDocument no exits                                   |
| 0023       | Error in transaction, payinCustomerDocument no exits                                      |
| 0024       | Error in transaction, payinCustomerDocument no exits                                      |
| 0025       | Customer Type Document invalid                                                            |
| 0026       | Customer Document invalid                                                                 |
| 0030       | Error due to non-compliance with pre-authorization rules (only for Argentina)             |
| 0031       | Error in processor, code value no registered                                              |
| 0032       | Error in processor, key no registered                                                     |
| 0040       | Error in transaction, cbu is required                                                     |
| 0041       | Error in transaction, cuit is requiredYUNO LOGIC:                                         |
| 0042       | Error on sendGateWay YUNO                                                                 |
| 0043       | CBU and CUIT don't match                                                                  |
| 0044       | The subscription is inactive                                                              |
| 0046       | CBU Exceeds maximum length                                                                |
| 0047       | CUIT Exceeds maximum length                                                               |
| 0048       | Error in collectorCode: invalid or unsupported value.                                     |
| 0049       | Error in CollectorCode: this value is not enabled for your account                        |
| 0050       | Error in collectorCode: the processor is temporarily unavailable. Please try again later. |
| 0051       | Error in collectorCode: collector not available or currency - min/max amount not allowed  |
| 0099       | Internal Error Payin                                                                      |

## Error Codes Related to Payment confirmation

| Error Code | Error Message                                                                      |
| ---------- | ---------------------------------------------------------------------------------- |
| 9001       | Refer to card issuer                                                               |
| 9002       | Refer to card issuer's special conditions                                          |
| 9003       | Invalid merchant                                                                   |
| 9007       | Pick-up card, special condition                                                    |
| 9012       | Invalid transaction                                                                |
| 9013       | Invalid amount                                                                     |
| 9014       | Card number does not exist                                                         |
| 9015       | Issuer or bin number does not exist                                                |
| 9017       | Customer cancellation                                                              |
| 9019       | Retry Transaction                                                                  |
| 9025       | Unable to locate record on file                                                    |
| 9030       | Format error                                                                       |
| 9039       | No credit account                                                                  |
| 9051       | Not sufficient funds                                                               |
| 9052       | Non-existing current account                                                       |
| 9054       | Expired card                                                                       |
| 9055       | Incorrect personal identification number                                           |
| 9057       | Transaction not permitted to cardholder                                            |
| 9061       | Effective advance is exceeded                                                      |
| 9062       | Restricted card                                                                    |
| 9065       | Merchant has exceeded the number of daily transactions                             |
| 9075       | Allowable number of PIN tries exceeded                                             |
| 9078       | Card not activated                                                                 |
| 9080       | Card expiration date invalid                                                       |
| 9089       | Customer info invalid                                                              |
| 9092       | Financial institution or intermediate network facility cannot be found for routing |
| 9093       | Transaction cannot be completed                                                    |
| 9094       | Duplicate transmission                                                             |
| 9096       | System malfunction                                                                 |
| 9097       | Operation Timeout                                                                  |
| 9099       | Error                                                                              |
| 9005       | Do not honor                                                                       |
| 9020       | Invalid response                                                                   |
| 9031       | Bank not supported by switch                                                       |
| 9034       | Suspected fraud                                                                    |
| 9040       | Requested function not supported                                                   |
| 9041       | Lost card                                                                          |
| 9043       | Stolen card, pick-up                                                               |
| 9056       | No card record                                                                     |
| 9058       | Transaction not permitted to terminal                                              |
| 9062       | Restricted card                                                                    |
| 9066       | Card acceptor call acquirer's security department                                  |
| 9068       | Response received too late                                                         |
| 9094       | Duplicate transmission                                                             |
| 9090       | Issuer or switch is inoperative                                                    |
| 9000       | Succeeded                                                                          |

<br />

## Error Codes for Virtual Accounts

### 🛑Header Errors

| Error Code                   | Type         | HTTP Status      | Message                                    |
| :--------------------------- | :----------- | :--------------- | :----------------------------------------- |
| `ERR_MISSING_HEADER`         | `auth_error` | 400 Bad Request  | Missing required header: X-Api-Key         |
| `ERR_UNAUTHORIZED`           | `auth_error` | 401 Unauthorized | Invalid API key                            |
| `ERR_MISSING_HEADER`         | `auth_error` | 400 Bad Request  | Missing required header: X-Timestamp       |
| `ERR_TIMESTAMP_OUT_OF_RANGE` | `auth_error` | 401 Unauthorized | Timestamp too old or too far in the future |
| `ERR_MISSING_HEADER`         | `auth_error` | 400 Bad Request  | Missing required header: X-Signature       |
| `ERR_INVALID_SIGNATURE`      | `auth_error` | 401 Unauthorized | Invalid signature                          |

### ⚠️Body Errors

| Error Code                      | Type               | HTTP Status               | Message                                                                |
| :------------------------------ | :----------------- | :------------------------ | :--------------------------------------------------------------------- |
| `ERR_MISSING_FIELD`             | `auth_error`       | 400 Bad Request           | Missing required field: owner.referenceId                              |
| `ERR_INVALID_LENGTH`            | `auth_error`       | 401 Unauthorized          | Invalid length: must be between 1 and 36 characters                    |
| `ERR_INVALID_FIELD`             | `auth_error`       | 400 Bad Request           | Missing required header: X-Timestamp                                   |
| `ERR_MISSING_FIELD`             | `auth_error`       | 401 Unauthorized          | Timestamp too old or too far in the future                             |
| `ERR_UNSUPPORTED_DOCUMENT_TYPE` | `auth_error`       | 400 Bad Request           | Missing required header: X-Signature                                   |
| `ERR_INVALID_DOCUMENT_TYPE`     | `auth_error`       | 401 Unauthorized          | Invalid signature                                                      |
| `ERR_MISSING_FIELD`             | `validation_error` | 400 Bad Request           | Missing required field: owner.referenceId                              |
| `ERR_INVALID_LENGTH`            | `validation_error` | 400 Bad Request           | Invalid length: must be between 1 and 36 characters                    |
| `ERR_INVALID_FIELD`             | `validation_error` | 400 Bad Request           | Invalid owner.type: must be PERSON or COMPANY                          |
| `ERR_MISSING_FIELD`             | `validation_error` | 400 Bad Request           | Missing required field: owner.document.type                            |
| `ERR_UNSUPPORTED_DOCUMENT_TYPE` | `validation_error` | 400 Bad Request           | Unsupported document type: must be one of \[CUIT, DNI, RUC]             |
| `ERR_INVALID_DOCUMENT_TYPE`     | `business_error`   | 422 Unprocessable Entity  | Invalid document type: CUIT not allowed for merchant country           |
| `ERR_MISSING_FIELD`             | `validation_error` | 400 Bad Request           | Missing required field: owner.document.number                          |
| `ERR_INVALID_FORMAT`            | `validation_error` | 400 Bad Request           | Document number must contain only digits                               |
| `ERR_INVALID_DOCUMENT_NUMBER`   | `business_error`   | 422 Unprocessable Entity  | Invalid document number for type CUIT                                  |
| `ERR_MISSING_FIELD`             | `validation_error` | 400 Bad Request           | Missing required field: owner.firstName (required when type=PERSON)    |
| `ERR_INVALID_LENGTH`            | `validation_error` | 400 Bad Request           | Length must be between 1 and 50                                        |
| `ERR_MISSING_FIELD`             | `validation_error` | 400 Bad Request           | Missing required field: owner.lastName (required when type=PERSON)     |
| `ERR_INVALID_LENGTH`            | `validation_error` | 400 Bad Request           | Length must be between 1 and 50                                        |
| `ERR_MISSING_FIELD`             | `validation_error` | 400 Bad Request           | Missing required field: owner.companyName (required when type=COMPANY) |
| `ERR_INVALID_LENGTH`            | `validation_error` | 400 Bad Request           | Length must be between 1 and 80                                        |
| `ERR_MISSING_FIELD`             | `validation_error` | 400 Bad Request           | Missing required field: owner.email                                    |
| `ERR_INVALID_EMAIL`             | `validation_error` | 400 Bad Request           | Invalid email format                                                   |
| `ERR_INVALID_LENGTH`            | `validation_error` | 400 Bad Request           | Length must be between 1 and 50                                        |
| `ERR_MISSING_FIELD`             | `validation_error` | 400 Bad Request           | Missing required field: owner.phone.countryCode                        |
| `ERR_INVALID_PHONE_COUNTRY`     | `validation_error` | 400 Bad Request           | Invalid country code: must be 1-4 digits                               |
| `ERR_MISSING_FIELD`             | `validation_error` | 400 Bad Request           | Missing required field: owner.phone.number                             |
| `ERR_INVALID_PHONE_NUMBER`      | `validation_error` | 400 Bad Request           | Invalid phone number: must be 4–14 digits                              |
| `ERR_MISSING_FIELD`             | `validation_error` | 400 Bad Request           | Missing required field: account.category                               |
| `ERR_INVALID_FIELD`             | `validation_error` | 400 Bad Request           | Invalid account.category: must be VIRTUAL                              |
| `ERR_MISSING_FIELD`             | `validation_error` | 400 Bad Request           | Missing required field: account.type                                   |
| `ERR_INVALID_FIELD`             | `validation_error` | 400 Bad Request           | Invalid account.type: must be CVU, CCI or CLABE                        |
| `ERR_MISSING_FIELD`             | `validation_error` | 400 Bad Request           | Missing required field: account.country                                |
| `ERR_INVALID_COUNTRY`           | `validation_error` | 400 Bad Request           | Country must be a 3-letter ISO 3166-1 code                             |
| `ERR_MISSING_FIELD`             | `validation_error` | 400 Bad Request           | Missing required field: account.currency                               |
| `ERR_INVALID_CURRENCY`          | `validation_error` | 400 Bad Request           | Currency must be 3-letter ISO 4217 code                                |
| `ERR_INVALID_DOCUMENT_TYPE`     | `business_error`   | 422 Unprocessable Entity  | Invalid currency code for country                                      |
| `ERR_INVALID_ACCOUNT_NAME`      | `validation_error` | 400 Bad Request           | Invalid account.name: must match ^\[a-zA-Z0-9.*-]{6,20}$                |
| `ERR_INVALID_ACCOUNT_NAME`      | `validation_error` | 400 Bad Request           | Invalid account.name: must match ^\[a-zA-Z0-9.*-]{3,15}                 |
| `ERR_INVALID_METADATA_KEY`      | `validation_error` | 400 Bad Request           | Metadata key 'segment@#' is invalid: must match ^\[a-zA-Z0-9\_-]{1,20}$  |
| `ERR_INVALID_METADATA_VALUE`    | `validation_error` | 400 Bad Request           | Metadata value too long or contains invalid characters                 |
| `ERR_TOO_MANY_METADATA_ENTRIES` | `validation_error` | 400 Bad Request           | Metadata supports up to 5 key-value pairs                              |
| `ERR_INTERNAL_ERROR`            | `system_error`     | 500 Internal Server Error | Unexpected server error                                                |