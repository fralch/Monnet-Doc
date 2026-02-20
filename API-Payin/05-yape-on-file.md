Yape on File

# Yape on File

## About this Guide

This page explains how the subscription model works using the **One click Payment (OCP)** framework. This payment method allows users to provide a single, secure consent (tokenized and stored) to authorize future charges without needing to reapprove each transaction.

### Supported Subscription Types

* **ON\_DEMAND:** Enables merchants to initiate charges at any time after the customer has authorized the subscription.
* **RECURRENT:** Intended for automated periodic payments (e.g., monthly billing cycles).

The OCP architecture is designed to be modular, secure, and scalable, enabling merchants and processors to adopt a standardized subscription flow easily and efficiently.

### Supported Devices

* **MOBILE:** Flow initiated from a mobile device.
* **WEB:** Flow initiated from a desktop or web browser.

## Supported Processors

| Processor | ProcessorCode | Description                                                                                                                        |
| --------- | :------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| Yape OCP  | Yape\_on\_file  | First wallet integrated under the One Card File (OCP) model. Enables on-demand payments through a single consent, stored securely. |

## How It Works

![](https://files.readme.io/43172e89ca5cf9cdc8aea2e251fdf75a4044c7657c3de09b39e0cafa7599a0e9-image.png)

### Flow

**1.-** Confirms the subscription request on the Merchant website or App.\
**2.-** Merchant sends a subscription creation request to Monnet.\
**3.-** Monnet forwards the request to the configured processor.\
**4.-** Request customer authorization.\
**5.-** Processor authorizes or rejects the subscription.\
**6.-** Monnet sends final notification to the Merchant.

## 🔗 Related Links

The **Yape On File** integration requires interaction with the following methods and components:

* 🔧 [Create Subscription](https://payinmonnetpayments.readme.io/reference/create-suscription)\
  Endpoint used to register a recurring payment subscription for the end user.

* 🗑️ [Cancel Subscription](https://payinmonnetpayments.readme.io/reference/cancel-suscription)\
  Endpoint to cancel an existing subscription.

* 🔁 [Webhook](https://payinmonnetpayments.readme.io/reference/ocp-webhook)\
  Used to receive status updates for subscription events (success, cancellation, failure).

* ✨ [Experience Guide](https://payinmonnetpayments.readme.io/reference/yapeocp-experience-guide)\
  Guide provided by the Yape team detailing the UX/UI standards, interaction flows, and design recommendations to ensure a consistent and seamless Yape OCP integration within the Monnet environment.

* ✨ [Lototype Use](https://payinmonnetpayments.readme.io/reference/yapeocp-experience-guide-logotype-use)\
  Guide provided by the Yape team outlining the design, branding, and user experience guidelines to ensure a consistent and aligned integration of Yape OCP within the Monnet ecosystem.

* 🧱 [White Label](https://payinmonnetpayments.readme.io/reference/white-label)\
  Describes the required white label implementation to embed the Yape user experience in mobile and desktop channels.

***