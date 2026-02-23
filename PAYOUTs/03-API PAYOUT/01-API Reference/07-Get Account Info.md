# Get Account Info

Validates account ownership and existence using a CBU, CVU, or Alias.

## Description

The **Get Account Info** endpoint allows merchants in Argentina to validate a bank account’s ownership using a **CBU, CVU, or Alias**, without executing a payout. This service helps ensure account authenticity, improve user conversion, and meet regulatory requirements—particularly relevant for industries such as **gambling**, where it’s essential to verify that deposits are made to the player’s own account.

## Considerations

* **Geographic scope:** Currently available only for Argentina *(country = ARG)*.
* **Payment keys:** Only valid for **CBU**, **CVU**, or **Alias**.
* **Functional scope:** This service does **not** trigger any payout flow.
* **Authentication:** Based on [OAuth 2.0 bearer tokens](https://docs.monnet.io/OoF1XuWTEpPjI0i2ERRO/quick-start/authentication/authorization-protocol-oauth-2.0-bearer-tokensen).
* **Rate limit:** Limited to **60 requests per minute (TPM)**; additional requests will be rejected.

### Input Parameters

**Endpoint Production:** [**https://accountinfo.payout.monnnet.io/ms-payouts-account-validations/api/v1/account-validations/account-info**](https://accountinfo.payout.monnnet.io/ms-payouts-account-validations/api/v1/account-validations/account-info)

**Endpoint Certification:** [**https://cert.accountinfo.payout.monnet.io/ms-payouts-account-validations/api/v1/account-validations/account-info**](https://cert.accountinfo.payout.monnet.io/ms-payouts-account-validations/api/v1/account-validations/account-info)

### Request

#### Headers

| Header        | Type   | Mandatory | Description                                                                                             | Format/Example                       |
| ------------- | ------ | --------- | ------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| Content-Type  | string | Yes       | Indicates the media type of the request payload. Must be JSON format.                                   | application/json                     |
| MID           | string | Yes       | Merchant identifier assigned by Monnet. Used to identify the requesting merchant.                       | 1234                                 |
| Authorization | string | Yes       | JWT access token obtained from the authentication endpoint. Must be sent **without** the bearer prefix. | eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9 |

#### Fields

The request must include the following fields:

<table><thead><tr><th width="174.66656494140625">Field</th><th width="107.77777099609375">Type</th><th width="121.66650390625">Mandatory</th><th>Description</th><th>Format / Example</th></tr></thead><tbody><tr><td><code>first_name</code></td><td>string</td><td>Yes</td><td>Account holder’s first name</td><td>Maximum length: 255 characters (general rule) / Rodrigo</td></tr><tr><td><code>last_name</code></td><td>string</td><td>Yes</td><td>Account holder’s last name</td><td>Maximum length: 255 characters (general rule) / Suarez</td></tr><tr><td><code>document_type</code></td><td>integer</td><td>Yes</td><td>Type of identification document</td><td>Possible values:<br>1, 2 or 3</td></tr><tr><td><code>document_number</code></td><td>string</td><td>Yes</td><td>Identification document number</td><td>For Argentina: 7–13 characters allowed, depending on <code>document type</code>:<br>• CUIL: 11–13<br>• CUIT: 11–13<br>• DNI: 7–8</td></tr><tr><td><code>account_identifier</code></td><td>string</td><td>Yes</td><td>Account to be validated</td><td>For Argentina: 6–22 characters allowed, depending on <code>account type</code>:<br>• Alias: 6–20<br>• CBU: 22<br>• CVU: 22</td></tr><tr><td><code>country</code></td><td>string</td><td>Yes</td><td>Country (ISO) . For this scope, only ARG (Argentina) is accepted.</td><td>Length: 3 characters, ISO format (3166-1-alpha-3)</td></tr></tbody></table>

#### Example Request <a href="#example-request" id="example-request"></a>

```
{
"first_name": "Rodrigo",
"last_name": "Suarez",
"document_type": "DNI",
"document_number": "12345678",
"account_identifier": "1230001312000001234567",
"country": "ARG"
}
```

### Response

<table><thead><tr><th>Field</th><th width="112.111083984375">Format</th><th>Description</th></tr></thead><tbody><tr><td><code>code</code></td><td>string</td><td>Indicates the status of the validation process.<br>If the validation is successful, the returned value is <strong>“200”</strong>.</td></tr><tr><td><code>title</code></td><td>string</td><td>Description associated with the result code</td></tr><tr><td><code>validated_</code>
<br><code>holders</code></td><td>Array</td><td>Array of objects representing the validated account holders</td></tr><tr><td><code>validated_</code>
<br><code>name</code></td><td>string</td><td><p></p><p>Validated account holder’s name</p></td></tr><tr><td><code>validated_</code>
<br><code>documentt</code>
<br><code>ype</code></td><td>string</td><td>Validated account holder’s document type (For Argentina: <strong>CUIT/CUIL</strong>)</td></tr><tr><td><code>validated_</code>
<br><code>document</code></td><td>string</td><td>Validated account holder’s identification document number</td></tr><tr><td><code>account_identifier</code></td><td>string</td><td>Validated account number (For Argentina: <strong>CBU/CVU or ALIAS</strong>)</td></tr><tr><td><code>bank_name</code>
<br></td><td>string</td><td>Name of the financial institution or PSP to which the account belongs</td></tr></tbody></table>

#### Examples Response <a href="#examples-response" id="examples-response"></a>

a. Example response when the account exists and matches the account holder:

```
{
  "code": 200,
  "title": "Identity verification successful.",
  "validated_holders": [
    {
      "validated_name": "Juan Paredes",
      "validated_documenttype": "CUIT",
      "validated_document": "20123456789"
    }
  ],
  "account_identifier": "1230001312000001234567",
  "bank_name": "BANCO HSBC"
}
```

b. Example response when the account exists and corresponds to more than one account holder:

```
{
  "code": 200,
  "title": "Identity verification successful.",
  "validated_holders": [
    {
      "validated_name": "Juan Paredes",
      "validated_documenttype": "CUIT",
      "validated_document": "20123456789"
    },
    {
      "validated_name": "Mario Gonzales",
      "validated_documenttype": "CUIT",
      "validated_document": "27234567890"
    }
  ],
  "account_identifier": "1230001312000001234567",
  "bank_name": "BANCO HSBC"
}
```

c. If the beneficiary’s data does not match the validated account holder or co-holder:

```
{
  "code": "GAI0007",

  "title": "Not found",

  "status": "404",

  “detail”: “Beneficiary is not the Account Owner. The provided document_number does not match the account owner or any co-owners.”
}
```

d. If the submitted account is inactive or does not exist, and its data cannot be validated.

```
{
  "code": "GAI0006",

  "title": "Inactive account",

  "status": "422",

  “detail”: “The account is inactive. Validation
  cannot proceed.”
  }
```

### Error Codes

Below are the error codes that the Get Account Info service may return:

| Error Codes                                    | Title                            | Detail                                                                                             |
| ---------------------------------------------- | -------------------------------- | -------------------------------------------------------------------------------------------------- |
| GAI0001                                        | Bad Request                      | <p>One or more fields in the                                                                       |
| <br>request do not meet the required           |                                  |                                                                                                    |
| <br>validations. Please review the submitted   |                                  |                                                                                                    |
| <br>parameters.</p>                            |                                  |                                                                                                    |
| GAI0002                                        | Bad Request                      | <p>Country not available. Please                                                                   |
| <br>review the submitted parameter.</p>        |                                  |                                                                                                    |
| GAI0003                                        | Too Many Requests                | <p>The request has been rejected                                                                   |
| <br>because the maximum number of              |                                  |                                                                                                    |
| <br>allowed requests for this endpoint has     |                                  |                                                                                                    |
| <br>been exceeded. Please try again later.</p> |                                  |                                                                                                    |
| GAI0004                                        | Service Unavailable              | <p>Network services are currently                                                                  |
| <br>unavailable. All configured providers      |                                  |                                                                                                    |
| <br>failed to respond after retry attempts.    |                                  |                                                                                                    |
| <br>Please try again later</p>                 |                                  |                                                                                                    |
| GAI0005                                        | No providers available           | <p>The selected country is                                                                         |
| <br>supported but has no enabled providers     |                                  |                                                                                                    |
| <br>at this time. Please try again later or    |                                  |                                                                                                    |
| <br>contact support.</p>                       |                                  |                                                                                                    |
| GAI0006                                        | Inactive account                 | <p>The account is inactive.                                                                        |
| <br>Validation                                 |                                  |                                                                                                    |
| <br>cannot proceed.</p>                        |                                  |                                                                                                    |
| GAI0007                                        | Not found                        | <p>Beneficiary is not the Account                                                                  |
| <br>Owner. The provided document\_number       |                                  |                                                                                                    |
| <br>does not match the account owner or        |                                  |                                                                                                    |
| <br>any co-owners.</p>                         |                                  |                                                                                                    |
| GAI0008                                        | <p>Internal Server               |                                                                                                    |
| <br>Error</p>                                  | <p>An unexpected error occurred  |                                                                                                    |
| <br>while processing the request. Please       |                                  |                                                                                                    |
| <br>contact support if the issue persists.</p> |                                  |                                                                                                    |
| GAI0009                                        | <p>Service                       |                                                                                                    |
| <br>Unavailable</p>                            | <p>Account Validating service is |                                                                                                    |
| <br>temporarily unavailable. Please try again  |                                  |                                                                                                    |
| <br>later</p>                                  |                                  |                                                                                                    |
| GAI0010                                        | Not found                        | <p>Account identifier invalid or                                                                   |
| <br>does not exist.</p>                        |                                  |                                                                                                    |
| GAI0011                                        | Not found                        | <p>Alias not found or not                                                                          |
| <br>registered.</p>                            |                                  |                                                                                                    |
| GAI0012                                        | Authentication failed            | Authentication with the service failed. Please verify the provided credentials or contact support. |
| GAI0013                                        | General error                    | An error occurred while processing the account information request.                                |
