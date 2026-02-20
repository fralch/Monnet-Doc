# Create a Transaction

## About This Guide

This page explains how to process payments using Monnet Payment's PAYIN API. This Integration is basically the same for all Latin American countries and we will walk you through all the additional possibilities here.

Before starting your integration, please make sure that you have a Monnet Payment's CERT account. *You don't have one?* Please contact your Account Representative.

## How It Works

![Customer Flow](https://files.readme.io/f72f281-Customer.png)

### Flow

**1.-** The customer confirms the sale on your website, selecting Monnet as a payment method.
**2.-** From your server, a transaction creation is requested and a redirection to the payment gateway will be done. There, the customer will receive payment instructions.
**3.-** The customer makes the payment.
**4.-** Monnet receives and validates the payment.
**5.-** Monnet sends a notification to your server once the payment is complete.
