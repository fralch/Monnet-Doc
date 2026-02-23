# Virtual Accounts

## About this Guide

Virtual Accounts are a modern and efficient way for merchants to enable local deposit capabilities and streamline payment tracking and reconciliation across LATAM.

A Virtual Account is a unique, country-specific bank account identifier assigned to an individual user. It looks and functions like a regular bank account for receiving local transfers, but its core purpose is to precisely identify the origin of each incoming payment.

When a user sends money to their Virtual Account, the deposit is automatically captured by our platform and immediately notified to the merchant through a webhook. This allows the merchant to instantly fund the user’s balance, process a top-up, or activate a service—without manual review or reconciliation.

Virtual Accounts offer a scalable and automated way to manage incoming bank transfers, reduce operational overhead, and give users a simple, familiar deposit experience using their local banking rails.

### 🧩 How It Works

![How It Works](https://files.readme.io/3ea6baee04d06a9280ddd540e6a8b595eec54a7e0d38f3f272029f4912d271aa-image.png)

### ➡️ Flow

**1.-** User requests a Virtual Account from the Merchant.
**2.-** Merchant creates a Virtual Account for the user via Monnet API.
**3.-** Monnet returns a unique, country-specific Virtual Account to the Merchant.
**4.-** Merchant provides the Virtual Account details to the User.
**5.-** User sends a local bank transfer to their assigned Virtual Account
**6.-** Monnet detects the incoming deposit and attributes it to the correct user.
**7.-** Monnet sends a webhook notification to the Merchant with deposit details.
**8.-** Merchant credits the user’s balance or delivers the corresponding service.

### 📘 Before You Begin

Before integrating with the Virtual Accounts API, request your API credentials from Monnet’s Technical Support team. Once activated, you will gain access to the sandbox environment for testing, and after completing the certification process, production credentials will be enabled. Please review the details and recommendations for each method carefully before proceeding.

### 🔗 Related Links

This section provides direct access to all API endpoints associated with the Virtual Accounts product. Each endpoint is designed to help you create, manage, and control Virtual Accounts throughout their lifecycle. Use these links as a reference to explore available methods, understand their functionality, and integrate them efficiently into your platform.

* 🆕 [Create Virtual Account](https://payinmonnetpayments.readme.io/reference/create-virtual-account)
  Endpoint to create a new Virtual Account in Monnet.

* 🔍 [Get Virtual Account Details](https://payinmonnetpayments.readme.io/reference/get-virtual-account-details)
  Endpoint to retrieve the details of a previously created Virtual Account.

* ✏️ [Update Virtual Account Information](https://payinmonnetpayments.readme.io/reference/update-virtual-account-information)
  Endpoint to update non-sensitive information of a Virtual Account.

* 🔄 [Update Virtual Account Status](https://payinmonnetpayments.readme.io/reference/update-virtual-account-status)
  Endpoint to change the operational status of a Virtual Account.

* 📨 [Deposit Notification](https://payinmonnetpayments.readme.io/reference/deposit-notification)
  Deposit notification sent to the merchant with the final status of the transaction.

* 📚 [Additional Resources](https://payinmonnetpayments.readme.io/reference/va-additional-resources)
  Key references and guides for API integration.
