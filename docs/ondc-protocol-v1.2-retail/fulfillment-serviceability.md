# Fulfillment Serviceability construct for Provider

## Overview
This document aims to introduce a serviceability framework for retail providers (merchants) within the ONDC network. This framework enables a provider to establish and define the following aspects:

1. **Location constraint**: The provider can specify the geographic limits for order fulfillment. In hyperlocal contexts, this involves using latitude/longitude coordinates to create shapes like circles or polygons, utilizing systems such as h3 or s2 cells, or other globally recognized GPS coordinate systems. For inter-city deliveries, this is determined using pincodes. It's mandatory to clearly define the location constraints. Temporarily, if the location constraint is not specified, it will default to a radius of 3 kilometers.

2. **Timing constraint**: Operational hours and holidays for the store, during which the provider accepts orders. It is mandatory to clearly define these timing constraints. During a transitional phase, if the timing constraints are not specified, they will default to predetermined hours (to be determined).

