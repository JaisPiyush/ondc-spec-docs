---
sidebar_position: 5
---

# /search

A `/search` request is initiated by the Buyer Application (BAP) and directed to either the Buyer Gateway (BG) or the Buyer Provider Platform (BPP). Within this request, the `context` object will exclude the fields `bpp_id` and `bpp_uri`.

For the examples of search requests let's assume a context
```ts
context = {
    "key": "MCowBQYDK2VuAyEAtuo9bSWQtUPOdcrKB/ei56qPSfQlyXrERPL61izaYRU=",
    "domain": "ONDC:RET11",
    "action":"search",
    "country":"IND",
    "city":"std:011",
    "core_version":"1.1.0",
    "bap_id":"buyerapp.com",
    "bap_uri":"https://buyerapp.com/ondc",
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

## 1. Search by city
The "Search by City" request in the ONDC API is used to query catalogs from sellers located within a specified city.
```ts
{
    "context": {
    "key": "MCowBQYDK2VuAyEAtuo9bSWQtUPOdcrKB/ei56qPSfQlyXrERPL61izaYRU=",
    "domain": "ONDC:RET11",
    "action":"search",
    "country":"IND",
    "city":"std:011",
    "core_version":"1.1.0",
    "bap_id":"buyerapp.com",
    "bap_uri":"https://buyerapp.com/ondc",
    "transaction_id":"3df395a9-c196-4678-a4d1-5eaf4f7df8dc",
    "message_id":"1655281254860",
    "timestamp":"2023-02-03T08:00:00.000Z",
    "ttl":"PT30S"
    },
    "message": {
        "intent": {
            "fulfillment": {
                "type": "Delivery"
            },
            "payment": {
                "@ondc/org/buyer_app_finder_fee_type":"percent",
                "@ondc/org/buyer_app_finder_fee_amount":"3"
            }
        }
    }
}
```
In this request, we're employing an alternative context configuration that excludes location details, as we aim to conduct the search without specifying a precise location.

1. The `payment` field container buyer app finder fee, details can be found [here](/docs/annexure/payment-settlement).
2. Fulfillment type enum values can be found [here](/docs/annexure/enums#fulfillment-type-enum).


## 2. Search by item name
This request aims to query the catalog using `item.name` and will consider the fulfillment's end location as specified in `context.location`.
```ts
{
    "context": context,
    "message": {
        "intent": {
            "item": {
                "descriptor": {
                    "name": "coffee"
                }
            },
            "fulfillment": {
                "type": "Delivery"
            },
            "payment": {
                "@ondc/org/buyer_app_finder_fee_type":"percent",
                "@ondc/org/buyer_app_finder_fee_amount":"3"
            }
        }
    }
}
```

## 3. Search by category
```ts
{
    "context": context,
    "message": {
        "intent": {
            "category": {
                "id": "Grocery"
            },
            "fulfillment": {
                "type": "Delivery"
            },
            "payment": {
                "@ondc/org/buyer_app_finder_fee_type":"percent",
                "@ondc/org/buyer_app_finder_fee_amount":"3"
            }
        }
    }
}
```

Id and sub-categories of each parent category can be found [here](https://docs.google.com/spreadsheets/d/1ayRbp-WmXwwbzp7z1MgRO0NuKZM1AQk4GGZ8SE4NTnw/edit#gid=0).

## 4. Search by fulfillment location
```ts
{
    "context": context,
    "message": {
        "intent": {
            "fulfillment": {
                "type": "Delivery"
            },
            "payment": {
                "@ondc/org/buyer_app_finder_fee_type":"percent",
                "@ondc/org/buyer_app_finder_fee_amount":"3"
            }
        }
    }
}
```