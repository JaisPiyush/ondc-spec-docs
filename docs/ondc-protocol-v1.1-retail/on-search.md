---
sidebar_position: 6
---
TODO: Complete on_search after verification of schema

# /on_search
The /on_search endpoint in the ONDC API is designed for seller apps to respond with their catalog based on the search intent specified by the buyer app. 

Context sent by the seller app will contain `bpp_id` and `bpp_uri` along with `bap_id` and `bap_uri`.

```ts
context = {
    "key": "MPowBQYDK2VuAyEAtuo9bSWQtUPOdcrKB/ei56qPSfQlyXrERPL61izaYPO=",
    "domain": "ONDC:RET11",
    "action":"search",
    "country":"IND",
    "city":"std:011",
    "core_version":"1.1.0",
    "bap_id":"buyerapp.com",
    "bap_uri":"https://buyerapp.com/ondc",
    "bpp_id": "sellerapp.com",
    "bpp_uri": "https://sellerapp.com"
    "transaction_id":"3df395a9-c196-4678-a4d1-5eaf4f7df8dc",
    "message_id":"1655281254955",
    "timestamp":"2023-02-03T08:00:20.000Z",
    "ttl":"PT30S"
    }
```

1. OpenAPI schema for on_search API can be found [here](https://github.com/beckn/protocol-specifications/blob/master/api/transaction/components/io/OnSearch.yaml).

2. The on_search API delivers a Catalog within the message object, as detailed in the [*Catalog*](https://github.com/beckn/protocol-specifications/blob/master/schema/Catalog.yaml).

## Response
```ts
{
    "context": context,
    "message": {
        "catalog": {
            "descriptor": seller_descriptor,
            "fulfillments": fulfillments,
            "payments": payments,   
            "providers": providers,
            "items": items,
            "ttl": "PT40S",
            "exp": "2023-02-03T08:01:00.000Z"
            // ... The examples provided do not include fields such as 'offers,' but these fields operate in accordance with their respective schemas
        }
    }
}
```


## Catalog

### Seller descriptor
These are description of BPP itself. BAP can render this info while showing products. It’s defined in the [Descriptor.yaml](https://github.com/beckn/protocol-specifications/blob/master/schema/Descriptor.yaml) schema

```ts
seller_descriptor = {
    "name": "ABC seller store",
    "short_desc": "Online eCommerce store",
    "long_desc": "Online eCommerce store",
    "images": [
        "https://abc.com/images/shop-img" 
    ]
}
```



### Fulfillment
These are the various modes of fulfillment offered by the BPP, especially when the BPP itself handles fulfillment on behalf of its on-boarded providers. The specifics are outlined in the [Fulfillment.yaml](https://github.com/beckn/protocol-specifications/blob/master/schema/Fulfillment.yaml).

In the response, BPP will send `id` and `type` fields for fulfillments along with optional value `tags`. The enum values for `type` field can be found [here](/docs/annexure/enums#fulfillment-type-enum).

```ts
fulfillments = [
    {
        "id": 1,
        "type": "Delivery"
    },
    {
        "id": 2,
        "type": "Self-Pickup"
    },
    {
        "id": 3,
        "type": "Delivery and Self-Pickup"
    }
]
```

### Providers
Information about the various providers that are part of the BPP's network, as specified in [Provider.yaml](https://github.com/beckn/protocol-specifications/blob/master/schema/Provider.yaml).

```ts
providers = [
    {
        "id": 'P1',
        "time": {
            "label": 'disable',
            "timestamp": "2023-02-03T08:00:30.000Z"
            // Other time fields can be provided here
        },
        "descriptor": {
            "name": "ABC Store 1",
            "short_desc": "ABC Store 1",
            "long_desc": "ABC Store 1",
            "images": []
        }
        "@ondc/org/fssai_license_no": "12345678901234",
        "ttl":"P1D",
        "locations": [
            {
                "id":"location-id-1",
                "gps":"12.967555,77.749666",
                "address": {
                    "street":"Jayanagar 4th Block",
                    "city":"Bengaluru",
                    "area_code":"560076",
                    "state":"KA"
                },
                "circle": {
                    "gps":"12.967555,77.749666",
                    "radius": {
                        "unit":"km",
                        "value":"3"
                    }
                },
                "time": {
                    "days":"1,2,3,4,5,6,7",
                    "schedule": {
                        "holidays": [
                            "2022-08-15",
                            "2022-08-19"
                        ]
                    },
                    "frequency": "PT4H",
                    "times": [
                        "1100",
                        "1900"
                    ],
                    "range": {
                        "start": "1100",
                        "end": "2100"
                    }
                }
            }
        ],
        "items": ProviderItems,
        "fulfillments": [{
            "contact": {
                "phone":"9886098860",
                "email":"abc@xyz.com"
            }
        },]
        "tags": ProviderTags
    }
]
```
| field Name | Value Type | Description |
|---------------|------------|-------------|
| id | string(uuid) | Unique id for provider, must be same as bpp_id for ISN seller apps |
| time.label | [enum](/docs/annexure/enums#time-label) | Add/Remove provider from BAP's cache |
| @ondc/org/fssai_license_no | string | Valid 14 digit FSSAI # required for F&B provider |
| ttl | string(ISO8601 duration) | Validity of provider catalog |
| locations | Array of [Location](https://github.com/beckn/protocol-specifications/blob/master/schema/Location.yaml) | Locations for provider with unique id for each location |
|Location.circle.radius | [Scalar](https://github.com/beckn/protocol-specifications/blob/master/schema/Scalar.yaml) | Serviceability radius for provider & location, cap of 3km. |
|Location.time | [Time](https://github.com/beckn/protocol-specifications/blob/master/schema/Time.yaml) | Store Location timings - includes holidays and either fixed ("range") or split timing ("frequency", "times") | 
| Location.time.days | string(number separated by ',') | Days of week, 1- Monday till 7 - Sunday|
| Location.time.schedule.frequency & Location.time.schedule.times | [Frequency](https://github.com/beckn/protocol-specifications/blob/master/schema/Duration.yaml) & Times is an array of string(date-time) | The 'frequency' and 'times' parameters determine the store's operational schedule. Specifically, 'frequency' indicates the duration of each open period as defined in 'times.' For instance, in the given example, the store operates in 4-hour intervals, such as from 1100 to 1500 and from 1900 to 2300. |
| Location.range | [Time range](https://github.com/beckn/protocol-specifications/blob/372a9f25548a46748fcd1d26fcd36ca41ba05cea/schema/Time.yaml#L11) | Fixed timings are outlined under 'days.' In this context, it signifies that the store operates from 1100 to 2100 every day, Monday through Sunday. It is essential that the end time is later than the start time, and the total operating hours within a day should not exceed 24 hours. For instance, a store open 24 hours should set the start range as '0000' and the end range as '2359'. |


#### Provider Items

OpenAPI schema can be found [here](https://github.com/beckn/protocol-specifications/blob/core-v1.1.0/schema/Item.yaml).

```ts
ProviderItems = [
    {
        "id":"I1",
        "descriptor": {
            "name":"Atta",
            "code":"1:XXXXXXXXXXXXX",
            "symbol":"https://abc.com/images/07.png",
            "short_desc":"Ashirwad Atta 5kg",
            "long_desc":"Ashirwad Atta 5kg",
            "images":
            [
            "https://abc.com/images/07.png"
            ]
        },
        "quantity":{
            "available": {
                "count":"1"
            },
            "maximum":{
                "count":"2"
            }
        },
        "price":{
            "currency":"INR",
            "value":"170.0",
            "maximum_value":"180.0"
        },
        "category_id":"Foodgrains",
        "fulfillment_id":"1",
        "location_id":"location-id-1",
        "@ondc/org/returnable":true,
        "@ondc/org/cancellable":true,
        "@ondc/org/return_window":"P7D",
        "@ondc/org/seller_pickup_return":false,
        "@ondc/org/time_to_ship":"PT45M",
        "@ondc/org/available_on_cod":false,
        "@ondc/org/contact_details_consumer_care":"Ramesh,ramesh@abc.com,1800425444444",
        "@ondc/org/statutory_reqs_packaged_commodities": {
            "manufacturer_or_packer_name":"ITC",
            "manufacturer_or_packer_address":"ITC Quality Care Cell,P.O Box No.592,Bangalore-560005",
            "common_or_generic_name_of_commodity":"Ashirwad Atta",
            "net_quantity_or_measure_of_commodity_in_pkg":"5kg",
            "month_year_of_manufacture_packing_import":"01/2023"
        },
        "@ondc/org/statutory_reqs_prepackaged_food": {
            "nutritional_info":"Energy(KCal)-(per 100kg) 420,(per serving 50g)250;Protein(g)-(per100kg) 12,(per serving 50g) 6",
            "additives_info":"Preservatives,Artificial Colours",
            "brand_owner_FSSAI_license_no":"12345678901234",
            "other_FSSAI_license_no":"12345678901234",
            "importer_FSSAI_license_no":"12345678901234"
        },
        "@ondc/org/mandatory_reqs_veggies_fruits": {
            "net_quantity":"100g"
        },
        "tags": [
            {
                "code": "veg_nonveg",
                "list": [
                    {
                        "code": "veg",
                        "value" :"yes"
                    }
                ]
            }
        ]
    }
],
    
```
TODO: Complete the schema info after verification

#### Provider Tags
```ts
ProviderTags = [
    {
        "code":"serviceability",
        "list": [
            {
                "code":"location",
                "value":"location-id-1"
            },
            {
                "code":"category",
                "value":"Eggs, Meat & Fish"
            },
            {
                "code":"type",
                "value":"10"
            },
            {
                "code":"val", 
                "value":"3"
            },
            {
                "code":"unit",
                "value":"km"
            }
        ]
    },
    {
        "code":"serviceability",
        "list": [
            {
                "code":"location",
                "value":"location-id-1"
            },
            {
                "code":"category",
                "value":"Kitchen Accessories"
            },
            {
                "code":"type",
                "value":"12"
            },
            {
                "code":"val",
                "value":"IND"
            },
            {
                "code":"unit",
                "value":"country"
            }
        ]
    }
]

```

### Exp
This is a timestamp indicating when the catalog will expire.

### TTL
This denotes the duration in seconds after which the catalog will expire. It indicates how long a BAP can cache the catalog and use the cached catalog before it needs to refresh the data.