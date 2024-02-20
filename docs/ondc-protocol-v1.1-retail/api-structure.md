---
sidebar_position: 4
---

# API structure & transaction trail & Context

1. All APIs follow the same structure which includes
    * **Context** : Object includes `transaction_id`, `message_id` and other attribute keys that identify the source & destination of the message being sent through the API request or response.
    * **Message** : identifies the content of the message e.g the search intent, catalog details, order details, etc.

2. All APIs are asynchronous,
    * When a sender transmits a request to a recipient, the recipient will reply with either `ACK` (Acknowledgment) for a valid request or `NACK` (Negative Acknowledgment) if the request is invalid.
    * The recipient issues a response via a callback corresponding to the request API. For instance, the sender's */on_search* callback is triggered in response to a */search* request.


3. ONDC standard error codes can be found [here](/docs/annexure/error-codes).

## Transaction trail

Transaction trail for a buyer is maintained through a unique `transaction_id` for the pre-order
stage, i.e. `/search`, `/select`, `/init`, `/confirm`. However, considering that most buyer apps cache
catalogs that are reused by multiple buyer apps, for providing a good buyer experience, an
unique `transaction_id` is maintained for `/select`, `/init` & `/confirm`.

The reason for maintaining a unique `transaction_id` for these stages is due to the nature of how buyer apps function. Most buyer apps cache catalogs, and these catalogs are often reused by multiple buyer apps. To ensure a smooth and traceable buyer experience, it's essential to have a unique identifier for each transaction that persists through the selection, initiation, and confirmation stages.

For example, imagine a buyer using an app to purchase a product. During the `/search` stage, they look for items they are interested in. Once they select an item (the `/select` stage), a unique `transaction_id` is generated and associated with their action. This `transaction_id` continues to be associated with their transaction as they move to the `/init` stage, where they may specify details like payment methods or delivery addresses, and finally to the `/confirm` stage, where the order is finalized. This process ensures that the entire transaction can be tracked and managed effectively, providing a better experience for the buyer and easier management for the seller.

This approach is important in systems where multiple buyer apps interact with the same catalogs, as it ensures consistency and reliability in tracking and managing transactions across different platforms and interfaces​​​​.

1. Each request and response is uniquely identified by combining a `transaction_id` with a `message_id`. This allows the sender of a request to match the response to its corresponding request using these identifiers.
    * **Request**: The buyer initiates a /select API request to choose a product. This request includes a unique transaction_id (say T1) and a message_id (say M1).
    * **Response**: The seller (Seller NP) processes this request and sends back a response. This response will have the same transaction_id (T1) to indicate it's part of the same transaction. However, the message_id will be different (say M2) to signify it's a response and not the original request.
    * **Correlation by Buyer NP**: Upon receiving the response, the Buyer NP can easily correlate it to the original request by matching the transaction_id (T1). The difference in message_id (M1 for request, M2 for response) helps the Buyer NP distinguish between the request sent and the response received.

    This method ensures clarity and traceability, allowing participants in a transaction to accurately track and manage requests and responses throughout their interaction. It's especially useful in an asynchronous communication environment, where multiple requests and responses might be exchanged in a non-linear order​​.
2. In cases where an outdated (stale) request or response is received, characterized by a timestamp earlier than a similar request or response already processed (as identified by the `transaction_id` + `message_id`), the recipient has the option to issue a `NACK` response. This response should include the error code `20002` for the buyer app and `30022` for the seller app.

3. Error block can be sent in sync (in response to request) or async (along with callback) mode
and should have the following attribute keys: `error.type`, `error.code`.

4. Buyer applications that feature real-time search capabilities can utilize the same `transaction_id` throughout the entire pre-order process.

5. The unique identifier for an order within a network is determined by a combination of the `transaction_id` and the `order_id`.

Swagger API docs can be found [here](https://app.swaggerhub.com/apis/ONDC/ONDC-Protocol-Retail/1.0.29#/) and tag structure can be found [here](https://app.swaggerhub.com/apis/ONDC/ONDC-base-protocol/1.0.0#/TagGroup).


## Context

OpenAPI schema can be found [here](https://github.com/beckn/protocol-specifications/blob/master/schema/Context.yaml).

| Field name | Value type | Description | Example value |
|------------|------------|-------------|---------------|
| domain | [enum](/docs/annexure/domains) | specific classification system used for categorizing businesses and activities | ONDC:RET11 |
|action | string | indicates the type of action being requested, in this case, 'search'. It defines the purpose of the API call | search |
| country | string | specifies the country in which the search or transaction is taking place | IND|
| city | string([city code](https://docs.google.com/spreadsheets/d/1BtABwe4CXgN_sHIvDH51dJeYzRWdAhedYtzx10apOTs/edit#gid=1856583449)) | This field contains the code or identifier for the city where the search is being conducted or where the service or product is located (std code is used) | std:011 |
| version | string(major.minor.patch format) | Indicates the version of the ONDC protocol or API being used for the transaction. This helps in maintaining compatibility and understanding the structure of the request and response | 1.1.0 |
|bap_id | string(domain name) | Stands for Buyer Application ID. It uniquely identifies the buyer application making the request | buyerapp.com |
|bap_uri | string(url) | The URI of the Buyer Application. It provides a reference to the application's address or endpoint | https://buyerapp.com | 
|bpp_id | string(domain name) | Stands for Seller Application ID. It uniquely identifies the seller application making the request | sellerapp.com |
|bpp_uri | string(url) | The URI of the Seller Application. It provides a reference to the application's address or endpoint | https://sellerapp.com |
|location| [Location](https://github.com/beckn/protocol-specifications/blob/master/schema/Location.yaml)| The location where the transaction is intended to be fulfilled | `{"gps":"12.974002,77.613458"}`|
|transaction_id | string(uuid) | A unique identifier for the transaction. It is used to track and correlate different stages of the transaction process | 3df395a9-c196-4678-a4d1-5eaf4f7df8dc |
|message_id | string(uuid) | uniquely identifies each message or request sent in the context of a transaction | 1655281254860 |
|timestamp | string(RFC3339 format) | A timestamp marking when the request was made | 2023-02-03T08:00:00.000Z |
|ttl | string(ISO 8601 duration format) |This field indicates the validity period of the message or request. It specifies how long the message remains relevant or actionable | PT30S | 
| key| string | The encryption public key of the sender |  |

```ts
{
    "key": "MCowBQYDK2VuAyEAtuo9bSWQtUPOdcrKB/ei56qPSfQlyXrERPL61izaYRU=",
    "domain": "ONDC:RET11",
    "action":"search",
    "country":"IND",
    "city":"std:011",
    "core_version":"1.1.0",
    "bap_id":"buyerapp.com",
    "bap_uri":"https://buyerapp.com/ondc",
    "bpp_id": "sellerapp.com",
    "bpp_uri": "https://sellerapp.com",
    "transaction_id":"3df395a9-c196-4678-a4d1-5eaf4f7df8dc",
    "message_id":"1655281254860",
    "timestamp":"2023-02-03T08:00:00.000Z",
    "ttl":"PT30S",
    "location": {
        "gps": "12.974002,77.613458",
        "city": {
            "name": "Delhi",
            "code": "std:011"
        }
    }
}
```