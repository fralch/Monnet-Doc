# Status

**STAGES:** This will determine the stage in which the payout flow is.&#x20;

* **STATUS:** The status of a payout refers to the current state of the payment being processed.&#x20;

The flow is demonstrated in the following flowchart:

It's important for users to understand that "STAGE" refers to the current state in which a payment is in the flow, while "STATUS" refers to the state of the payout itself.

Regarding the explanation of the different stages and payment statuses, you could do it as follows:

<figure><img src="https://1642515047-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FMvYrq8H0R42hedo5qReG%2Fuploads%2FE2MxzBoxUvmXYCxpj151%2Fimage.png?alt=media&#x26;token=3572d667-14f8-4159-9781-2c7e16b9404f" alt=""><figcaption><p>Payout Flow</p></figcaption></figure>

* **IN\_PROGRESS:** This is the stage in which the payment is while it's being processed. Within this stages, there are several possible "status" that indicate the progress of the payment.
  * CREATED: This status indicates that the payment has been created but has not been authorized yet.
  * AUTHORIZED: This status indicates that the payment has been authorized and is ready to be processed.
  * WAITING\_ROUTING: This status indicates that the payment is waiting to be routed through the payment network.
  * PENDING\_BANK: This status indicates that the payment has been sent to the receiving bank and is waiting for confirmation.
* **PENDING:** This is the stage in which the payment is while awaiting approval or confirmation.
  * PENDING\_BALANCE: This status indicates that the merchant have no enough balance to process the payout and must credit funds for the flow to continue. It is configurable for a merchant to have this flow or to be automatically rejected when there is insufficient balance.&#x20;
* **SUCCESS:** This status indicates that the payment has been successfully processed. Within this stage, there is only one possible "status": PROCESSED.&#x20;
* **REJECTED:** This status indicates that the payment has been rejected or has failed at some point in the process. Within this stage, there are several possible "status" that indicate the reason for the rejection or failure.
  * DATA\_ERROR: This status indicates that there has been an error in the payment data.
  * BUSSINESS\_ERROR: This status indicates that there has been an error related to the business.
  * DENIED: This status indicates that the payment has been denied by a fraud validation or AML rules.
  * NO\_BALANCE: This status indicates that the payment is on hold due to insufficient balance. IT IS NECESARY TO SEND THE PAYOUT AGAIN.
  * REJECTED\_BANK: This status indicates that the receiving bank has rejected the payment.
  * REVERSED: This status indicates that the payout has been reversed by the provider or destination bank.
  * ERROR: This status indicates that, for some control-related reason, the payout was manually rejected before being sent to the payment network.\
    Example: Due to network maintenance, the payout was rejected without being sent to the payment network.

{% hint style="info" %}
"It is important to note that the payout status can change throughout the payout flow and should not be considered as a definitive field for determining the payment status. Instead, the **'STAGE'** field should be taken into account to know the current stage of the payment and define the corresponding actions.&#x20;

The 'status' field is purely informative and may provide additional details about the payment but should not be used as the final field for integration with the payment system.
{% endhint %}
