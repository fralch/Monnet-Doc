# Getting Started

With just one integration, your company can sell to consumers based in several countries of Latin America. Offering local payment options is an excellent opportunity to increase your sales and reach even more customers.

The Monnet Payment Payin API has a Test Environment (CERT) and a Production Environment (PROD). To integrate your system with Monnet Payments, use the test credentials that were given to you at the beginning of the integration.

The following URLs are the basis of the endpoints for the supported environments:

### URLs

* **CERT:** `https://cert.payin.api.monnetpayments.com/`
* **PROD:** `https://payin.api.monnetpayments.com/`

A complete endpoint consists of the base URI of the environment, the identifier of the Merchant, and the resource.

For example, if we want to create a transaction, the endpoint would be:

`https://cert.payin.api.monnetpayments.com/api-payin/v3/online-payments`

In order to create a complete request is necessary to send the correct HTTP POST request.

## Start Your Integration

The first thing you need to do in order to start using our platform is to retrieve your API Key and Signature Key by logging in into your account in our test environment. If you do not have an account yet, please contact your account representative.

With this account, you can test the entire API functionality without performing any real charges or payments. Use this environment to integrate and test your application.

All the examples of this documentation are based on the CERT environment.

### Production Account

Once the account has been approved to move to Production, new credentials will be generated to access this environment.

> **Note:** Once you have access to the production environment, be careful of which environment you are doing your requests.

The location of the production environment is:

* **Back Office:** `https://payin.monnetpayments.com/`
* **API:** `https://payin.api.monnetpayments.com/`

Once you are done with your Production development, a Certification will be scheduled, where an integrator will test your integration with real payments.

## Test Your Integration

All the services have to be tested before going live. Connections, protocols, and file formats have to be in accordance with the conventions identified throughout this documentation.

The following areas will be tested during this certification:

* Check-out
* Payment Flow
* Transaction creation
* Notification

Once all tests have been completed, the account will be able to receive payments.
