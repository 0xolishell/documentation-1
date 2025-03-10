---
layout: ../../../layouts/MainLayout.astro
section: chainlinkFunctions
date: Last Modified
title: "Chainlink Functions Architecture"
whatsnext: { "Billing": "/chainlink-functions/resources/billing/" }
setup: |
  import ClickToZoom from "@components/ClickToZoom.astro"
---

:::note[Prerequisites]
Read the Chainlink Functions [introduction](/chainlink-functions/) to understand all the concepts discussed on this page.
:::

## Request and Receive Data

<ClickToZoom src='/images/chainlink-functions/requestAndReceive.png' />

Requests to _Chainlink Functions_ follow the [Request & Receive Data](/chainlink-functions/resources/concepts/) cycle.

1. A EOA initiates the transaction by calling the consumer contract.
1. The consumer contract must inherit [FunctionsClient](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.8/dev/functions/FunctionsClient.sol) to send the request to the [FunctionsOracle](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.8/dev/functions/FunctionsOracle.sol) contract.
1. The _FunctionsOracle_ contract:
   1. Calls the [FunctionsBillingRegistry](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.8/dev/functions/FunctionsBillingRegistry.sol) 's `startBilling` function to estimate the fulfillment costs and block the amount in the _Reservation balance_ (To learn more, read [Cost simulation](/chainlink-functions/resources/billing#cost-simulation-reservation)).
   1. Emits an `OracleRequest` event containing information about the request.
1. On reception of the event, each _DON_'s oracle initiates the API call on a serverless environment.
1. Each serverless environment calls the API provider to fetch the API response.
1. The _DON_ runs the [Off-chain Reporting protocol(OCR)](/chainlink-functions/resources/concepts/) to aggregate all the API responses.
1. The aggregate API response is transmitted by a DON's oracle node to the _FunctionsOracle_ contract.
1. The _FunctionsOracle_ contract calls the _FunctionsBillingRegistry_'s `fulfillAndBill` function to calculate the fulfillment costs and finalize the billing (To learn more, read [Cost calculation](/chainlink-functions/resources/billing#cost-calculation-fulfillment)).
1. The _FunctionsBillingRegistry_ contract calls back the consumer contract.

**Note**: Chainlink Functions requests are not limited to API requests. The diagram depicts an example of API requests, but you can request the DON to run any computation.

## Subscription Management

Chainlink Functions requests receive funding from [subscription accounts](/chainlink-functions/resources/concepts/). As explained in [Concepts](/chainlink-functions/resources/concepts/), the _Subscription App_ is a User Interface that abstracts the communications with the _Subscriptions contract_ (aka _Functions Billing Registry_ contract). The _Functions Billing Registry_ lets you manage your subscription accounts.

:::note
The _Subscriptions App_ is not available now, we recommend using the [functions hardhat starter kit](https://github.com/smartcontractkit/functions-hardhat-starter-kit) to communicate with the [Functions Billing Registry contract](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.8/dev/functions/FunctionsBillingRegistry.sol).
:::

### Create Subscription

<ClickToZoom src='/images/chainlink-functions/subscription/createSubscription.png' />

EOAs (Externally Owned Accounts) create subscriptions using the _Subscriptions App_. The App communicates with the _Functions Billing Registry_, which assigns a unique identifier (aka _Subscription ID_).

### Fund Subscription

<ClickToZoom src='/images/chainlink-functions/subscription/fundSubscription.png' />

You must fund your subscription accounts with enough LINK tokens:

1. Connect your EOA to the _Subscription App_.
1. Fund your subscription account. The _Subscriptions App_ abstracts the following:
   1. Call `transferAndCall` on the LINK token contract, transferring LINK tokens along with the _Subscription ID_ in the payload.
   1. The _Functions Billing Registry_ contract implements `onTokenTransfer`: It parses the _Subscription ID_ from the payload and funds the subscription account with the transferred LINK amount.

### Add Consumer

<ClickToZoom src='/images/chainlink-functions/subscription/addConsumer.png' />

You must allowlist your consumers' contracts on your subscription account before they can make Chainlink Functions requests. To do so:

1. Connect your EOA to the _Subscription App_.
1. Add the address of your consumer contract to your subscription account.
1. The _Subscription App_ calls the _Functions Billing Registry_ contract to add the consumer contract address to your subscription account.

### Remove Consumer

<ClickToZoom src='/images/chainlink-functions/subscription/removeConsumer.png' />

To remove a consumer contract:

1. Connect your EOA to the _Subscription App_.
1. Remove the address of your consumer contract from the allowlist.
1. The _Subscription App_ calls the _Functions Billing Registry_ contract to remove the consumer contract address from your subscription account.

**Note**: You can still remove consumers from your subscription if there are in-flight requests. Your consumer contract will still be called back, and your _Subscription Account_ will be charged.

### Cancel Subscription

<ClickToZoom src='/images/chainlink-functions/subscription/cancelSubscription.png' />

To cancel a subscription:

1. Connect your EOA to the _Subscription App_.
1. Cancel your subscription, passing the _Subscription Balance_ receiver account address. The _Subscriptions App_ abstracts the following:
   1. Call the `cancelSubscription` function on the _Functions Billing Registry_ contract, deleting the _Subscription ID_ and removing any existing consumers.
   1. The outstanding _Subscription Balance_ is sent to the receiver account.

**Note**: You cannot cancel a subscription if there are in-flight requests.

### Transferring ownership of a Subscription

Transferring ownership works as follows:

1. Connect your EOA to the _Subscription App_.
1. Initiate the ownership transfer by specifying the new owner's address.
1. The new owner must connect their EOA to the _Subscription App_ and accept the ownership.
