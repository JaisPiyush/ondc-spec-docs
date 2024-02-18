---
sidebar_position: 1
---
# Introduction

ONDC API contracts map business requirements for various use case scenarios to a set of attribute
keys in different APIs and ensures interoperability, between a buyer app & seller app, as follows:

- By defining the set of attribute keys that need to be exchanged to establish a handshake
between the buyer app & seller app.
- Clearly defining the single source of truth for each attribute key and thereby establishing
mutability & immutability of these keys on behalf of the participants at each end of the
transaction.

#### The API contract document is structured as follows:
* JSON payload for each API request & response.
* Definition & enumerations of specific keys, as applicable.
* Examples of usage of specific attribute keys for a business requirement as applicable.

## Retail API includes the following APIs:
### a. Pre-order APIs
* **/search** : buyer app specifies the search intent.
* **/on_search** : seller app responds with the catalog based on the search intent.
* **/select** : buyer app specifies the items & quantity selected by a buyer;
* **/on_select** : seller app responds with the serviceability info, quote & O2D TAT for
selected items.
* **/init** & **/on_init** : buyer & seller app specify & agree to the terms & conditions prior to
placing the order.
* **/confirm** : buyer app places the order on behalf of the buyer.
* **/on_confirm** : seller app responds to the order placed either through
auto-acceptance or deferred acceptance or rejection of the order.

### b. Post-order APIs
* **/status** : buyer app requests for the current status of the order.
* **/on_status** : seller app provides the current status of the order.
* **/cancel** : buyer app places cancellation request for the order.
* **/on_cancel** : seller app responds to the buyer app cancellation request or cancels
the order directly.
* **/update** : buyer app initiates a part return or cancel request (for specific items) on
behalf of the buyer.
* **/on_update** : seller app responds to the buyer app part return / cancel request or
initiates a part cancel request (for specific items) directly.
* **/support** : buyer app requests seller contact details.
* **/on_support** : seller app responds with seller contact details.
* **/track** : buyer app requests for live tracking of order.
* **/on_track** : seller app responds with URL for live tracking of order.
* **/rating** : buyer app provides rating on behalf of buyer.
* **/on_rating** : seller app responds to rating request from buyer app and provides
additional info, such as feedback form, as required.

#### NOTE
- All attribute keys, defined in this API contract, are mandatory, unless explicitly mentioned
otherwise.
- Participants may validate the request & response payloads, based on mandatory
attribute keys, as defined in this contract and respond with `NACK` (along with appropriate error
code as defined [here](https://github.com/ONDC-Official/developer-docs/blob/main/protocol-network-extension/error-codes.md)) for invalid payloads.
- Any payload, without optional attribute keys, cannot
be considered as invalid.
- This API contract has been defined considering the current scope of the MVP and includes certain
assumptions such as *pre-paid* payment collection by the buyer app.
- All participants are required to ensure full compliance with this API contract on the live ONDC
network.