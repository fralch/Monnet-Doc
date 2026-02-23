Yape One Shot

# Yape One Shot

## About this Guide

This page explains how the Yape One Shot Payment model works. This payment method allows users to approve one-time transactions securely via push notification from their Yape app, without requiring stored consent for future charges.

Unlike subscription models such as OCP, One Shot does not persist consent information. Each payment requires explicit user authorization through the Yape app, ensuring real-time validation and traceability

### Supported Use Case

**ONE\_TIME:** Single authorization per transaction, initiated by the merchant and approved by the user through the Yape app.

The One Shot architecture supports real-time interactions, offering a secure and user-friendly experience for both mobile and desktop flows, integrated into Monnet's white-label environment.

### Supported Devices

* **MOBILE:** The experience is triggered from a mobile app or mobile browser. The user is redirected to the Yape app via deeplink to authorize.
* **WEB:** The experience starts from the browser, where the user is presented with a set of clear instructions (e.g., enter your phone number in the app) to complete the authorization process.

## How It Works

![](https://files.readme.io/64d991d6c1719bc687e765a7d5bdcdf397f4335ee388f5ecafba481d38df80fe-image.png)

### Flow

**1.-** Shopper selects Yape One Shot as the payment method on the Merchant website or app.\
**2.-** Merchant sends a creation ttransaction request to Monnet.\
**3.-** Monnet validates and forwards the request to Yape (YapeOS).

**3.1.-** Create consent/Transaction(DeepLink generated\
**4.-** Redirect or Go to Yape app (Deep Link).\
**5.-** Shopper approval or denied consent/transaction.\
**6.-** Notify to Consent/Transaction status.

\*\*7.-\*\*Monnet sends final notification to the Merchant

## 🔗 Related Links

The **Yape One Shot** payment flow requires a white label interface to trigger and handle the user's approval experience:

* 🧱 [White Label](https://payinmonnetpayments.readme.io/reference/white-label)\
  Required for rendering the experience inside your mobile or desktop environment using deeplinks or manual steps.