---
sidebar_position: 3
---

# Seller-on-record signing

1. Each [participant type](/docs/ondc-protocol-registry/network-participant), will have an entry in the registry for their entity. Additionally, MSN (aggregator) seller apps will also have 1 entry for each of their seller-on-record (merchant).

2. All requests & responses have to be signed by the sender and verified by the receiver. If any request fails verification, it should rejected (`NACK`) with http `401` error code.

3. Signing by buyer app & ISN seller app will be using their private key, for which the corresponding public key will be in the registry for their `ukid`.

4. For MSN seller apps, signing will be as follows:
    * **/on_search** will be signed using the private key of the entity and the appropriate`unique_key_id` will be provided in the auth header.
    * all other responses will be singed using the private key of the seller-on-record (merchant) and the appropriate `unique_key_id` will be provided in the auth header.

5. Verification of requests is using registry [lookup API](/docs/ondc-protocol-registry/api#lookup).
    
    Request:
    ```json
    {
        "subscriber_id": "example-np.com",
        "domain":"nic2004:52110",
        "ukId":"22bc6a66-35b3-11ed-a261-0242ac120002"
    }
    ```
    Response:
    ```json
    [
        {
            "subscriber_id":"example-np.com",
            "status":"SUBSCRIBED",
            "ukId":"22bc6a66-35b3-11ed-a261-0242ac120002",
            "subscriber_url":"https://example-np.com/",
            "country":"IND",
            "domain":"nic2004:52110",
            "valid_from":"2022-09-23T06:32:38.021Z",
            "valid_until":"2027-09-23T06:32:38.021Z",
            "type":"BPP",
            "signing_public_key":
            "9V6WbbMwWy953zUIIICOOQq4nk8zHHJRhrZ19juApL4=",
            "encr_public_key":
            "MCowBQYDK2VuAyEATDpic13936lmOrDdzMpQox0KWXMwb9sqsdd6fcD1LHM=",
            "created":"2022-09-23T06:37:10.541Z",
            "updated":"2022-09-23T06:37:10.541Z",
            "br_id":"22bc6a66-35b3-11ed-a261-0242ac120002",
            "city":"std:080"
        }
    ]
    ```