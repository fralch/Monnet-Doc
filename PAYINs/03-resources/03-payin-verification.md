Payin Verification

# Payin Verification

## ✍️Standard API Verification

```java Java
try {
    //keyMonnet = Provided for Monnet Payments
    String sha512 = payinMerchantID + payinMerchantOperationNumber + payinAmount + payinCurrency + keyMonnet;
    String shaCreado = getStringSHA(sha512.trim());
    } catch (Exception e) {
        // TODO: handle exception
        e.printStackTrace();
    }

public static String getStringSHA(String str) {

    try {
      MessageDigest md5 = MessageDigest.getInstance("SHA-512");
      md5.reset();
      md5.update(str.getBytes());
      return new String(Hex.encodeHex(md5.digest()));
    } catch (NoSuchAlgorithmException e) {
      e.printStackTrace();
      return null;
    }
  }
```
```php PHP
<?php
  //keyMonnet = Provided for Monnet Payments
  $purchaseVerication = openssl_digest($_POST['payinMerchantID'] .
  $_POST['payinMerchantOperationNumber'] . $_POST['payinAmount'] .
  $_POST['payinCurrency']. $keyMonnet, 'sha512');
?>
```

## ✍️Virtual Account API Signature

```java Java
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;

public class HmacUtil {

    private static final String HMAC_SHA512 = "HmacSHA512";

    public static String hmacSha512(String secret, String message) {
        try {
            Mac mac = Mac.getInstance(HMAC_SHA512);
            SecretKeySpec secretKeySpec = new SecretKeySpec(secret.getBytes(StandardCharsets.UTF_8), HMAC_SHA512);
            mac.init(secretKeySpec);

            byte[] rawHmac = mac.doFinal(message.getBytes(StandardCharsets.UTF_8));

            // convert to hex
            StringBuilder hex = new StringBuilder(2 * rawHmac.length);
            for (byte b : rawHmac) {
                hex.append(String.format("%02x", b));
            }

            return hex.toString();
        } catch (Exception e) {
            throw new RuntimeException("Error generating HMAC-SHA512", e);
        }
    }
}

/* Envio de petición */
import org.springframework.web.reactive.function.client.WebClient;

public class ApiClient {

    private final String apiKey;      // ejemplo: "mk_live_abc123"
    private final String secret;      // secreto privado del comercio
    private final WebClient webClient;

    public ApiClient(String apiKey, String secret, String baseUrl) {
        this.apiKey = apiKey;
        this.secret = secret;
        this.webClient = WebClient.builder().baseUrl(baseUrl).build();
    }

    public Mono<String> sendRequest(Object body, String referenceId) {

        long timestamp = System.currentTimeMillis() / 1000; // Unix en segundos

        // Construimos el mensaje según tu regla
        String message = timestamp + apiKey + referenceId;

        // Generamos la firma
        String signature = HmacUtil.hmacSha512(secret, message);

        return webClient.post()
                .uri("/payments")
                .header("X-Api-Key", apiKey)
                .header("X-Timestamp", String.valueOf(timestamp))
                .header("X-Signature", signature)
                .bodyValue(body)
                .retrieve()
                .bodyToMono(String.class);
    }
}
 
```
```php PHP
function hmac_sha512(string $secret, string $message): string {
    return hash_hmac('sha512', $message, $secret);
}

function sendSignedRequest(string $apiKey, string $secret, string $referenceId, array $body, string $url) {

    // 1. Timestamp Unix en segundos
    $timestamp = time(); 

    // 2. Construimos el mensaje
    $message = $timestamp . $apiKey . $referenceId;

    // 3. Firma HMAC-SHA512
    $signature = hmac_sha512($secret, $message);

    // 4. Convertimos el body a JSON sin modificar espacios
    $jsonBody = json_encode($body, JSON_UNESCAPED_UNICODE);

    // 5. Preparamos los headers
    $headers = [
        "Content-Type: application/json",
        "X-Api-Key: $apiKey",
        "X-Timestamp: $timestamp",
        "X-Signature: $signature"
    ];

    // 6. Ejecutamos cURL
    $ch = curl_init($url);

    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_POST, true);
    curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
    curl_setopt($ch, CURLOPT_POSTFIELDS, $jsonBody);

    $response = curl_exec($ch);

    if ($response === false) {
        throw new Exception("cURL Error: " . curl_error($ch));
    }

    curl_close($ch);

    return $response;
}
```