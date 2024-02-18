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
* **/on_confirm** : seller app responds to the order placed either through.
auto-acceptance or deferred acceptance or rejection of the order.