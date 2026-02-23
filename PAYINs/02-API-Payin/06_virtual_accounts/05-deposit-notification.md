# Deposit Notification

Once Monnet receives a notification indicating that a user has made a deposit into a previously created Virtual Account, Monnet will immediately trigger a webhook event to the merchant.
This event will be sent to the POST URL that the merchant registered during the onboarding process.

**To receive these notifications, the merchant must provide Monnet with a public and secure HTTPS endpoint capable of handling HTTP POST requests**. The implementation language is flexible; however, the endpoint must:

* Accept and process POST requests with the parameters defined in this section of the documentation.
* Return an immediate and properly formatted HTTP response to confirm receipt.

![Deposit Notification Flow](https://files.readme.io/07907425081ec506b195cff90f6840e58c2f86038ab5c08331b9ab18b7ecbb09-Deposit_Notification_Flow.png)

***

## 📦 Webhook Payload

Monnet will send the following information

| Field | Description |
| :--- | :--- |
| operationId | **String (Required)** <br> Unique identifier of the payin operation inside Monnet. Important for reconciliation and for locating this deposit on our merchant portal. |
| status.code | **String (Required)** <br> Payin status code for the operation. <br> For Virtual Accounts, Monnet will notify merchants exclusively of the following operation statuses: **`5`/ `Autorizado`**or **`6`/ `Denegado`** |
| status.description | **String (Required)** <br> Description of the status. <br> For Virtual Accounts, Monnet will notify merchants exclusively of the following operation statuses: **`5`/ `Autorizado`**or **`6`/ `Denegado`** |
| merchantId | **String (Required)** <br> Identifier of the merchant in Monnet’s platform. |
| amount | **String (Required)** <br> Deposit amount in decimal format with two fraction digits. |
| currency | **String (Required)** <br> ISO 4217 currency code. |
| payinMethod | **String (Required)** <br> Payment method used for the operation. <br> For Virtual Accounts, this value will be **`VA`**. [See more details](https://payinmonnetpayments.readme.io/reference/payin-method) |
| depositDetails.account.id | **String (Required)** <br> Internal identifier of the Virtual Account within Monnet. This value matches the `id` returned to the merchant when the Virtual Account is created. |
| depositDetails.account.category | **String (Required)** <br> Account category. This field returns the same value that the merchant provided when creating the Virtual Account in Monnet. |
| depositDetails.account.type | **String (Required)** <br> Type of bank account or identifier. This field returns the same value that the merchant provided when creating the Virtual Account in Monnet. |
| depositDetails.account.number | **String (Required)** <br> Full account number assigned to the Virtual Account. This value matches the `account.number` returned to the merchant when the Virtual Account is created. |
| depositDetails.account.name | **String (Required)** <br> Descriptive name or label assigned to the Virtual Account. This field returns the same value provided by the merchant when creating or updating the Virtual Account in Monnet. |
| depositDetails.owner.fullName | **String (Required)** <br> Full name of the owner of the Virtual Account. This field returns the same value that the merchant provided when creating the Virtual Account in Monnet. |
| depositDetails.owner.documentType | **String (Required)** <br> Type of identification document of the owner. This field returns the same value that the merchant provided when creating the Virtual Account in Monnet. |
| depositDetails.owner.documentNumber | **String (Required)** <br> Identification document number of the owner. This field returns the same value that the merchant provided when creating the Virtual Account in Monnet. |
| depositDetails.owner.referenceId | **String (Required)** <br> Optional reference identifier sent by the merchant when creating the virtual account. This field returns the same value that the merchant provided when creating the Virtual Account in Monnet. |
| depositDetails.depositor.fullName | **String (Required)** <br> Full name of the person who made the deposit. |
| depositDetails.depositor.documentType | **String (Optional)** <br> Type of identification document of the depositor. |
| depositDetails.depositor.documentNumber | **String (Optional)** <br> Identification document number of the depositor. |

#### 🔔 Body example

The merchant must send a JSON payload with the following structure:

```json
{
 "version":"1.0",
 "operationId": "7019283",
  "status": {
    "code": "5",
    "description": "Aprobado"
  },
  "merchantId": "722",
  "amount": "200.00",
  "currency": "ARS",
  "payinMethod": "VA",
  "depositDetails": {
    "account": {
      "id": "acc_8af98b8c8a4",
      "category": "VIRTUAL",
      "type": "CVU",
      "number": "200300310012",
      "name": "john.doe"
    },
    "owner": {
      "fullName": "Joe Doe",
      "documentType": "CUIT",
      "documentNumber": "20123456783",
      "referenceId": "COMPANY-USER-1234"
    },
    "depositor": {
      "fullName": "Miles Morales",
      "documentType": "CUIT",
      "documentNumber": "32216549878"
    }
  },
  "errors": [],
  "receivedAt": "2025-02-21T14:22:05Z"
}
```

## ✅ Successful Response – 200

Monnet expects an HTTP 200 response to confirm successful notification delivery.

> **📘 Best Practices**
>
> To ensure the notification flow remains consistent, reliable, and high-performance, we recommend reviewing the following notes.
>
> ✔️ The response to Monnet must always be immediate.\
> Any processing or business logic that the merchant executes behind this endpoint must be handled asynchronously.
>
> ✔️ For merchants that already have an existing integration with Monnet, it is recommended to create and provide a separate webhook URL specifically for Virtual Account deposit notifications.\
> This separation ensures that the standard Payin Operation callbacks and the new Virtual Account Deposit webhook events remain independent, preventing processing conflicts or unintended behavior in the merchant’s systems.

## Error Codes

Review the list of error codes at the [following link](https://payinmonnetpayments.readme.io/reference/error-codes-hidden).
