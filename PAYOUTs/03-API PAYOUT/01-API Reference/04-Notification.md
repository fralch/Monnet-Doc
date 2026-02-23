# Notification

## **Monnet Notifications for SUCCESS and REJECTED Stages**

Monnet is pleased to offer an efficient notification system to keep our customers informed about the status of transactions. This feature is essential to ensure smooth and timely communication between Monnet and the Merchant.

We have implemented automatic notifications for the "SUCCESS" and "REJECTED" stages in payouts. These statuses reflect the final outcome of business operations and are vital for transaction tracking and management.

### **Notification URL Configuration**

To enable these automatic notifications, we request your team to provide a valid and secure URL where Monnet can send the notifications. This URL should be prepared to receive and process notification data appropriately. Ensure your technical team is involved in configuring this URL to ensure a successful integration.

To configure the notification URL, please follow these steps:

1. Contact with our Integration team.
2. Share the URL designated for the notification.
3. Our team will configurate the URL internally and let you know.

### **Notification Composition**

Before delving into the specifics of a payout notification, it's essential to understand the composition of the notification itself. Each notification comprises a header and a body. The header includes a field named **"verification"**, which serves as a security signature to check the integrity and authenticity.

**Verification Process:** The **merchantId** is required to perform the following steps with the "verification" field:

1. Get the value of the "verification" field on the header.
2. Perform the signature verification provided by RSA algorithm using the following inputs:
   1. Content to verify: **concatenation of the merchantId and the body** of the notification
   2. Signature: verification signature obtained in step 1 using Base64 decoding
   3. Public key extracted from the provided certificate

### **Notification:** Certificate Setup

To ensure notification security, attach the provided notifier certificate in your commerce application's notification settings.

> [!NOTE]
> Public key should be extracted from the certificate, this can be done by code or using the openssl tool:

```bash
openssl.exe x509 -pubkey -noout -in notifier.internal.monnet.io.crt > pubkey.pem
```

The result should looking as

```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAlwnoASyLECcZwPXBkW2O
IT/L/rzB8xesLC14lt0oH1EnBEufChr0s1c3e/CzeFNwWBXMjsmIlBqYprLJMYg5
v/qijO7FeVvgGqUai7bdi/lZtiTKo6zyLGbK6/K7fw6JiWjcyRxn+oYwMXu7x0HJ
6YUwvU+p9/TEtXIuEpyoUbrk7G2h4N2GgreQh6cHZrmxxjZ3tyRWCDNfxKQtRJtn
UfVvlzgIamHf+XzD4x2SNexYI/E9SZMiCoNoyvOrkujea6ategOUmjGRKAXVExZz
9tomb+4VyFPc/zDPOf7hr5L62r9W201OfVYNrt8AgSyWcn8sexgHf/VlA6ISULYy
LQIDAQAB
-----END PUBLIC KEY-----
```

### **Successful Stage Notification**

When a transaction is recorded as "SUCCESS," Monnet will also send an automatic notification to the URL address provided by your team. This notification will confirm the positive outcome of the transaction and provide necessary details for your internal record-keeping.

#### **RESPONSE SUCCESS by Bank**

This is an Example.

```
headers:[
    "verification": "SecurityValue"
    ],
body:[
    "payout":{
        "merchantId": "1000",
        "id": "1111111111",
        "orderId": "12345678",
        "amount": "10000",
        "currency": "MXN", 
        "bankCode":"072",
        "accountNumber":"123456789112345678",
        "paymentCode":"123456",
        "name":"Test User",
        "customerEmail":"test@test.com"
    }
    "output":{
		"stage": "SUCCESS",
		"status": "PROCESSED",
		"statusChangeDateTime": "20220110223001",
		"authorizationCode": "A87786843867",
		"authorizationDate": "20220110175959"
	}
    
]
```

### **Implementations examples**

**Java**

```java
import java.net.URLDecoder;
import java.net.URLEncoder;
import java.nio.charset.StandardCharsets;
import java.security.KeyFactory;
import java.security.PrivateKey;
import java.security.PublicKey;
import java.security.Signature;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.Base64;

public class MonnetSignatureVerification {
    private static String certificate = "MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAlwnoASyLECcZwPXBkW2OIT/L/rzB8xesLC14lt0oH1EnBEufChr0s1c3e/CzeFNwWBXMjsmIlBqYprLJMYg5v/qijO7FeVvgGqUai7bdi/lZtiTKo6zyLGbK6/K7fw6JiWjcyRxn+oYwMXu7x0HJ6YUwvU+p9/TEtXIuEpyoUbrk7G2h4N2GgreQh6cHZrmxxjZ3tyRWCDNfxKQtRJtnUfVvlzgIamHf+XzD4x2SNexYI/E9SZMiCoNoyvOrkujea6ategOUmjGRKAXVExZz9tomb+4VyFPc/zDPOf7hr5L62r9W201OfVYNrt8AgSyWcn8sexgHf/VlA6ISULYyLQIDAQAB";
    private static String signature = "icVMmXEaDmbUP7hRahME1mDrRkEUwUkxMQbwgbvlJVbN31zKrdFb4rAskWvzDvn7NUgrinafNV0Y6thrnKE+qgeCHUKE5Yyzj1iJPtG3lod27J3zW9+8m+1+l0p28nZGKMd16TKU55RwOlEjRSG/+eEqDLmIuZofTaXt6POqlS7itAVSthHeczVf5rNfumhe50gieAjS/iDCPCRvRWAQWcfC9UF8TFDA02za8RDsAk9nsWOWh9HYUF4cvnSEMz6hO1VAg+dmkzGbvXP3NS3R2WB7jPVP3ANbrLWeEev/kPNr1LIzSNmuwGnyyBrHJ/4ggPCtJrRQV/1nFudvzsL5jQ==";
    private static String content = "234{\"payout\":{\"id\":\"29\",\"country\":\"MEX\",\"amount\":\"1\",\"currency\":\"MXN\",\"orderId\":\"BJB_fa2d561b-0d72-4c87-ae0e-85abe7b00ad0\",\"merchantId\":\"234\",\"bankCode\":\"646\",\"name\":\"A name ÀÁÄÇÑ {{randomFirstName}} A lastname ÀÁÄÇÑ {{randomLastName}}\"},\"output\":{\"stage\":\"REJECTED\",\"status\":\"REJECTED_BANK\",\"statusChangeDateTime\":\"2024-05-29T13:16:57.267524Z\"},\"errors\":[{\"code\":\"4099\",\"message\":\"Unknown Error\"}]}";
  
    public static void main(String args[]) {
      try{
        boolean verificationResult = verifySignatureWithSHA256RSA(content, signature, certificate);
        if (verificationResult){
            System.out.println("The notification is authentic");
        }else{
            System.out.println("The notification is NOT authentic");
        }
      }catch(Exception e){
          System.out.println(e.getMessage());
      }
    }
    
    private static boolean verifySignatureWithSHA256RSA(String rspContent, String signature, String strPk) throws Exception {
        PublicKey publicKey = getPublicKeyFromBase64String(strPk);

        Signature publicSignature = Signature.getInstance("SHA256withRSA");
        publicSignature.initVerify(publicKey);
        publicSignature.update(rspContent.getBytes(StandardCharsets.UTF_8));

        byte[] signatureBytes = Base64.getDecoder().decode(signature);
        return publicSignature.verify(signatureBytes);
    }
    
    private static PublicKey getPublicKeyFromBase64String(String publicKeyString) throws Exception {
        byte[] b1 = Base64.getDecoder().decode(publicKeyString);
        X509EncodedKeySpec X509publicKey = new X509EncodedKeySpec(b1);
        KeyFactory kf = KeyFactory.getInstance("RSA");
        return kf.generatePublic(X509publicKey);
    }
}
```

NodeJS

```javascript
import { createVerify } from 'crypto';

function verifySignature(content, signature, strPk) {
  try {
    var isVerified = false;
    const decodedVerification = Buffer.from(signature, 'base64');
    const verifier = createVerify('sha256');
    verifier.update(content);

    const verificationResult = verifier.verify(strPk, decodedVerification);
    if (verificationResult) {
      console.log("The notification is authentic");
      isVerified = true;
    } else {
      console.log("The notification is NOT authentic");
    }

    return verificationResult;
  } catch (err) {
    console.log("decodeWithPublicKeyBase64 error:" + err);
    return;
  }
}
```

Note: In the above example, crypto-js library could perform the signature verification using the certificate directly without extract the public key

### **Rejected Stages Notification**

If the event a transaction is marked as "REJECTED," Monnet will send an automatic notification to the URL address provided by your team. This notification will contain relevant details about the transaction, including rejection reasons and any additional pertinent information. It is crucial for your team to properly configure the notification URL to receive this information promptly.

#### **RESPONSE REJECTED by Bank**

```
headers:[
    "verification": "SecurityValue"
    ],
body:[
    "payout":{
        "merchantId": "1000",
        "Id": "1111111111",
        "orderId": "12345678",
        "amount": "10000",
        "currency": "MXN", 
        "bankCode":"072",
        "accountNumber":"123456789112345678",
        "paymentCode":"123456",
        "name":"Test User",
        "customerEmail":"test@test.com"
    }
    "output":{
		"stage": "REJECTED",
		"status": "REJECTED_BANK",
		"statusChangeDateTime": "20220110223001"
	},
	"errors":[
		{
            "code" : "1000",
            "message" : "Invalid Beneficiary Document ID"
        }
]
```

**RESPONSE REJECTED by Business Error**

```
headers:[
    "verification": "SecurityValue"
    ],
body:[
    "payout":{
        "merchantId": "1000",
        "id": "1111111111",
        "orderId": "12345678",
        "amount": "10000",
        "currency": "MXN", 
        "bankCode":"072",
        "accountNumber":"123456789112345678",
        "paymentCode":"123456",
        "name":"Test User",
        "customerEmail":"test@test.com"
    }
    "output":{
		"stage": "REJECTED",
		"status": "BUSINESS_ERROR",
		"statusChangeDateTime": "20220110223001"
	},
	"errors":[
		{
            "code" : "2005",
            "message" : "[orderId] Duplicated"
        }
]
```

#### **RESPONSE REJECTED by no balance**

```
headers:[
    "verification": "SecurityValue"
    ],
body:[
    "payout":{
        "merchantId": "1000",
        "id": "1111111111",
        "orderId": "12345678",
        "amount": "10000",
        "currency": "MXN", 
        "bankCode":"072",
        "accountNumber":"123456789112345678",
        "paymentCode":"123456",
        "name":"Test User",
        "customerEmail":"test@test.com"
    }
    "output":{
		"stage": "REJECTED",
		"status": "NO_BALANCE",
		"statusChangeDateTime": "20220110223001"
	}
]
```

#### **RESPONSE REJECTED by reversed**

```
headers:[
    "verification": "SecurityValue"
    ],
body:[
    "payout":{
        "merchantId": "1000",
        "id": "1111111111",
        "orderId": "12345678",
        "amount": "10000",
        "currency": "MXN", 
        "bankCode":"072",
        "accountNumber":"123456789112345678",
        "name":"Test User",
        "customerEmail":"test@test.com"
    }
    "output":{
	"stage": "REJECTED",
	"status": "REVERSED",
	"statusChangeDateTime": "20220110223001"
	},
    "errors": [
        {
          "code": "4001",
          "message": "Bank Rejection - Beneficiary is not the Account Owner"
        }
]
```
