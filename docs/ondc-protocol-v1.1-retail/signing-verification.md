---
sidebar_position: 2
---

# Signing & Verification of requests & responses

### The protocol relies on cryptography for signing and verification.
- Signing uses `ed25519` cryptography algorithm.
- Encryption uses `x25519` algorithm.
- Public keys should be `base64` encoded and uploaded on the ONDC registry.
- [reference implementation](https://github.com/ONDC-Official/reference-implementations/tree/main/utilities/signing_and_verification/python) for cryptographic steps.


### Authorization header signing,
- Convert json payload to `utf-8` byte array.
- Generate `Blake2b-512` has from utf-8 byte array.
- Generate `base64` encoding of `Blake2b` hash.
- Sign the generated base64 encoding, using your private signing key, and add the signature to the request auth header.

Let the below be the request body in this example 
```json
{"context":{"domain":"nic2004:60212","country":"IND","city":"Kochi","action":"search","core_version":"0.9.1","bap_id":"bap.stayhalo.in","bap_uri":"https://8f9f-49-207-209-131.ngrok.io/protocol/","transaction_id":"e6d9f908-1d26-4ff3-a6d1-3af3d3721054","message_id":"a2fe6d52-9fe4-4d1a-9d0b-dccb8b48522d","timestamp":"2022-01-04T09:17:55.971Z","ttl":"P1M"},"message":{"intent":{"fulfillment":{"start":{"location":{"gps":"10.108768, 76.347517"}},"end":{"location":{"gps":"10.102997, 76.353480"}}}}}}
```
Let NP keys be (base64 encoded)
```python
signing_public_key="Q0NEQ0MzMUY4N0REQjdFMDNFQjQ3MEVBN0M0ODRERjE4NjNGRUFDMTkzMEFEQUJEMkU0MkY2MzgzNUVBODE3NA=="
signing_private_key="MkFDNDY3QjM1QkI0MjE1QzU0RkNEMkVENjYzNTQ2NDU0MEIwQTY5RkE3QTIwMDAxMjM0NTM5NkNGNDdCODg4RA=="
```
Steps to be followed to create the Authorization header:
1. Generate the digest of the request body using `Blake2b-512` hash function ([implementation](https://github.com/ONDC-Official/reference-implementations/blob/7419f7948340b717cd5e1124f7d122e5785b831b/utilities/signing_and_verification/python/cryptic_utils.py#L19))
   ```python
   # request_body -> utf8 byte array -> blake2b_512 -> base64 -> digest
   digest = "b6lf6lRgOweajukcvcLsagQ2T60+85kRh/Rd2bdS+TG/5ALebOEgDJfyCrre/1+BMu5nA94o4DT3pTFXuUg7sw=="
   ```
2. Generate the `created` field, `expires` field. The `created` field expresses when the signature was created, the `expires` field expresses when the signature ceases to be valid. Both the value must be UNIX timestamp integer value. Any signature with `created` timestamp value is the future or `expires` timestamp value in the past **MUST NOT** be processed.
    ```python
    created = 1641287875
    expires = 1641291475
    ```
3. Concatenate `created`, `expires` and `digest` in the format shown below. The below string is the signing string which NP is going ti uses to sign the request ([implementation](https://github.com/ONDC-Official/reference-implementations/blob/7419f7948340b717cd5e1124f7d122e5785b831b/utilities/signing_and_verification/python/cryptic_utils.py#L26))
    ```python
    '''
    (created): 1641287875
    (expires): 1641291475
    digest: BLAKE-512=b6lf6lRgOweajukcvcLsagQ2T60+85kRh/Rd2bdS+TG/5ALebOEgDJfyCrre/1+BMu5nA94o4DT3pTFXuUg7sw==
    '''
    ```
4. NP will sign the above string using `ed25519` private key and generate base64 encoded string of the signature([implementation](https://github.com/ONDC-Official/reference-implementations/blob/7419f7948340b717cd5e1124f7d122e5785b831b/utilities/signing_and_verification/python/cryptic_utils.py#L37)).
     ```python
    signature = "MkQwNDdFNDNFMzNFMTMyRjI3NzNGMzEzM0MwQzc0OUU2MThERUMzQkVFMkIzNDM2NjJFQjZGQzY1MEIyNkY4RTc0QjY4Q0Q5NDMzQTMzMDc1NjgyODQ3QTA4QTk3QTNFOTVFOTMzNDUwNzQ1OUUyOTQzN0Q3N0ZEMjAyOTMyMDc="
    ```
5. Finally the authorization header will look like this ([implementation](https://github.com/ONDC-Official/reference-implementations/blob/7419f7948340b717cd5e1124f7d122e5785b831b/utilities/signing_and_verification/python/cryptic_utils.py#L66))
    ```python
    # Let
    subscriber_id='example-np.com'
    unique_key_id='np12345'
    header_str = f'"Signature keyId="{subscriber_id}|{unique_key_id}|ed25519",algorithm="ed25519",'\
                 f'created="{created}",expires="{expires}",header="(created) (expires) digest", signature="{signature}""'
    print(header_str)
    # Signature keyId="example-np.com|np12345|ed25519",algorithm="ed25519",created="1641287875",expires="1641291475",headers="(created) (expires) digest",signature="MkQwNDdFNDNFMzNFMTMyRjI3NzNGMzEzM0MwQzc0OUU2MThERUMzQkVFMkIzNDM2NjJFQjZGQzY1MEIyNkY4RTc0QjY4Q0Q5NDMzQTMzMDc1NjgyODQ3QTA4QTk3QTNFOTVFOTMzNDUwNzQ1OUUyOTQzN0Q3N0ZEMjAyOTMyMDc="

    ```
6. Include the auth header string in the header
    * If receiving NP is BAP or BPP:
        ```http
        HTTP POST
        Authorization: Signature keyId="example-np.com|np12345|ed25519",algorithm="ed25519",created="1641287875",expires="1641291475",headers="(created) (expires) digest",signature="MkQwNDdFNDNFMzNFMTMyRjI3NzNGMzEzM0MwQzc0OUU2MThERUMzQkVFMkIzNDM2NjJFQjZGQzY1MEIyNkY4RTc0QjY4Q0Q5NDMzQTMzMDc1NjgyODQ3QTA4QTk3QTNFOTVFOTMzNDUwNzQ1OUUyOTQzN0Q3N0ZEMjAyOTMyMDc=
        ```
    * If receiving NP is BG (Gateway), then `X-Gateway-Authorization` will contain the auth header.

#### NOTE
* The difference between `created` and `expires` field should be equal to TTL of the request context.
* The `expires` field value must not be greater than the life expectancy of the signing key.



### Authorization Header verification

The NP preforms the following steps for the authentication of the incoming request (all the values are taken from above examples).

Let this request receiving NP have
```python
subscriber_id_2="recv-example-np.com"
unique_key_id_2="rec_np123456798"
```

1. Extract `keyId` from the **Authorization** or **X-Gateway-Authorization** header to further extract `subscriber_id`, `unique_key_id` and `signature algorithm`.
    ```python
    key_id = "example-np.com|np12345|ed25519"
    subscriber_id='example-np.com'
    unique_key_id='np12345'
    signature_algorithm='ed25519'
    ```
2. If the `signature_algorithm` value doesn't match with `algorithm` value from the auth header, then NP should return `401` unauthorized error.

3. The NP will retrieve public key of the requesting NP via it's own cached copy of keys or, calling `lookup` API on registry with `subscriber_id` and `unique_key_id`. 

    #### NOTE:
    * A subscriber can have multiple keys and/or the same domain can be used for implementing multiple types of subscribers.
    * Sending both `subscriber_id` and `unique_key_id` is important
    

4. If not valid key is found, the NP must return `NACK` response with `401`.
5. Retrieved public key will verify the signature ([implementation](https://github.com/ONDC-Official/reference-implementations/blob/7419f7948340b717cd5e1124f7d122e5785b831b/utilities/signing_and_verification/python/cryptic_utils.py#L80)).
6. If verified the request is considered authenticated and forwarded to request handler.
7. If not, NP should response `NACK` with `401` and `WWW-Authenticate` header if requesting NP is not BG, else `Proxy-Authenticate` header will be used.

    1. Requesting NP is not BG.
        ```http
        HTTP/1.1 401 Unauthorized
        WWW-Authenticate: Signature realm="recv-example-np.com", header="(created) (expires) digest"
        ```
    2. Requesting NP is BG
        ```http
        HTTP/1.1 401 Unauthorized
        Proxy-Authenticate: Signature realm="recv-example-np.com", header="(created) (expires) digest"
        ```
    Request body
    ```js
    {
        "message": {
            "ack": {
                "status": "NACK"
            }
        }
    }
    ```
| Request sender NP | Request receiving NP | Authorization header from sender | Authentication error response header from receiver |
|-------------------|----------------------|----------------------------------|----------------------------------------------------|
| BAP               | BG                   | Authorization                    | WWW-Authenticate                                   |
| BAP               | BPP                  | Authorization                    | WWW-Authenticate                                   |
| BG                | BAP                  | X-Gateway-Authorization          | Proxy-Authenticate                                 |
| BG                | BPP                  | X-Gateway-Authorization          | Proxy-Authenticate                                 |
| BPP               |  BAP                 | Authorization                    | WWW-Authenticate                                   |    


