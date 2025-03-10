---
layout: ../../../../../layouts/MainLayout.astro
section: vrf
date: Last Modified
title: "Get a Random Number"
permalink: "docs/vrf/v2/subscription/examples/get-a-random-number/"
whatsnext:
  {
    "Programmatic Subscription": "/vrf/v2/subscription/examples/programmatic-subscription/",
    "Subscription Manager UI": "/vrf/v2/subscription/ui/",
    "Security Considerations": "/vrf/v2/security/",
    "Best Practices": "/vrf/v2/best-practices/",
    "Migrating from VRF v1 to v2": "/vrf/v2/subscription/migration-from-v1/",
    "Supported Networks": "/vrf/v2/subscription/supported-networks/",
  }
metadata:
  description: "How to generate a random number inside a smart contract using Chainlink VRF."
setup: |
  import VrfCommon from "@features/vrf/v2/common/VrfCommon.astro"
  import CodeSample from "@components/CodeSample/CodeSample.astro"
---

<VrfCommon callout="subscription"/>

This guide explains how to get random values using a simple contract to request and receive random values from Chainlink VRF v2. For more advanced examples with programmatic subscription configuration, see the [Programmatic Subscription](/vrf/v2/subscription/examples/programmatic-subscription/) page. To explore more applications of VRF, refer to our [blog](https://blog.chain.link/).

<VrfCommon callout="ui"/>

## Requirements

This guide assumes that you know how to create and deploy smart contracts on Ethereum testnets using the following tools:

- [The Remix IDE](https://remix.ethereum.org/)
- [MetaMask](https://metamask.io/)
- [Sepolia testnet ETH](/resources/link-token-contracts/#sepolia-testnet)

If you are new to developing smart contracts on Ethereum, see the [Getting Started](/getting-started/conceptual-overview/) guide to learn the basics.

## Create and fund a subscription

For this example, create a new subscription on the Sepolia testnet.

1. Open MetaMask and set it to use the Sepolia testnet. The [Subscription Manager](/vrf/v2/subscription/ui/) detects your network based on the active network in MetaMask.

1. Check MetaMask to make sure you have testnet ETH and LINK on Sepolia. You can get testnet ETH and LINK at [faucets.chain.link](https://faucets.chain.link/sepolia/).

1. Open the Subscription Manager at [vrf.chain.link](https://vrf.chain.link).
   <!-- prettier-ignore -->
   <div class="remix-callout">
          <a href="https://vrf.chain.link">Open the Subscription Manager</a>
    </div>

1. Click **Create Subscription** and follow the instructions to create a new subscription account. MetaMask opens and asks you to confirm payment to create the account on-chain. After you approve the transaction, the network confirms the creation of your subscription account on-chain.

1. After the subscription is created, click **Add funds** and follow the instructions to fund your subscription. For this example, a balance of 2 LINK is sufficient. MetaMask opens to confirm the LINK transfer to your subscription. After you approve the transaction, the network confirms the transfer of your LINK token to your subscription account.

1. After you add funds, click **Add consumer**. A page opens with your account details and subscription ID.

1. Record your subscription ID, which you need for your consuming contract. You will add the consuming contract to your subscription later.

You can always find your subscription IDs, balances, and consumers at [vrf.chain.link](https://vrf.chain.link/).

Now that you have a funded subscription account and your subscription ID, [create and deploy a VRF v2 compatible contract](#create-and-deploy-a-vrf-v2-compatible-contract).

## Create and deploy a VRF v2 compatible contract

For this example, use the [VRFv2Consumer.sol](https://remix.ethereum.org/#url=https://docs.chain.link/samples/VRF/VRFv2Consumer.sol) sample contract. This contract imports the following dependencies:

- `VRFConsumerBaseV2.sol`[(link)](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.8/VRFConsumerBaseV2.sol)
- `VRFCoordinatorV2Interface.sol`[(link)](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.8/interfaces/VRFCoordinatorV2Interface.sol)
- `ConfirmedOwner.sol`[(link)](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.8/ConfirmedOwner.sol)

The contract also includes pre-configured values for the necessary request parameters such as `vrfCoordinator` address, gas lane `keyHash`, `callbackGasLimit`, `requestConfirmations` and number of random words `numWords`. You can change these parameters if you want to experiment on different testnets, but for this example you only need to specify `subscriptionId` when you deploy the contract.

Build and deploy the contract on Sepolia.

1. Open the [`VRFv2Consumer.sol` contract](https://remix.ethereum.org/#url=https://docs.chain.link/samples/VRF/VRFv2Consumer.sol) in Remix.

   <!-- prettier-ignore -->
   <CodeSample src="samples/VRF/VRFv2Consumer.sol" showButtonOnly/>

1. On the **Compile** tab in Remix, compile the `VRFv2Consumer.sol` contract.

1. Configure your deployment. On the **Deploy** tab in Remix, select the **Injected Provider** environment, select the `VRFv2Consumer` contract from the contract list, and specify your `subscriptionId` so the constructor can set it.

   ![Example showing the deploy button with the subscriptionID field filled in Remix](/images/vrf/deployWithSubscriptionId.png)

1. Click the **Deploy** button to deploy your contract on-chain. MetaMask opens and asks you to confirm the transaction.

1. After you deploy your contract, copy the address from the **Deployed Contracts** list in Remix. Before you can request randomness from VRF v2, you must add this address as an approved consuming contract on your subscription account.

   ![Example showing the contract address listed under the Contracts list in Remix](/images/vrf/getContractAddress.png)

1. Open the Subscription Manager at [vrf.chain.link](https://vrf.chain.link/) and click the ID of your new subscription under the **My Subscriptions** list. The subscription details page opens.

1. Under the **Consumers** section, click **Add consumer**.

1. Enter the address of your consuming contract that you just deployed and click **Add consumer**. MetaMask opens and asks you to confirm the transaction.

Your example contract is deployed and approved to use your subscription balance to pay for VRF v2 requests. Next, [request random values](#request-random-values) from Chainlink VRF.

## Request random values

The deployed contract requests random values from Chainlink VRF, receives those values, builds a struct `RequestStatus` containing them and stores the struct in a mapping `s_requests`. Run the `requestRandomWords()` function on your contract to start the request.

1. Return to Remix and view your deployed contract functions in the **Deployed Contracts** list.

1. Click the `requestRandomWords()` function to send the request for random values to Chainlink VRF. MetaMask opens and asks you to confirm the transaction. After you approve the transaction, Chainlink VRF processes your request. Chainlink VRF fulfills the request and returns the random values to your contract in a callback to the `fulfillRandomWords()` function. At this point, a new key `requestId` is added to the mapping `s_requests`.

   Depending on current testnet conditions, it might take a few minutes for the callback to return the requested random values to your contract. You can see a list of pending requests for your subscription ID at [vrf.chain.link](https://vrf.chain.link/).

1. To fetch the request ID of your request, call `lastRequestId()`.

1. After the oracle returns the random values to your contract, the mapping `s_requests` is updated: The received random values are stored in `s_requests[_requestId].randomWords`.

1. Call `getRequestStatus()` specifying the `requestId` to display the random words.

You deployed a simple contract that can request and receive random values from Chainlink VRF. To see more advanced examples where the contract can complete the entire process including subscription setup and management, see the [Programmatic Subscription](/vrf/v2/subscription/examples/programmatic-subscription/) page.

:::note[Note on Requesting Randomness]
Do not re-request randomness. For more information, see the [VRF Security Considerations](/vrf/v2/security/) page.
:::

## Analyzing the contract

In this example, your MetaMask wallet is the subscription owner and you created a consuming contract to use that subscription. The consuming contract uses static configuration parameters.

::solidity-remix[samples/VRF/VRFv2Consumer.sol]

The parameters define how your requests will be processed. You can find the values for your network in the [Configuration](/vrf/v2/subscription/supported-networks/) page.

- `uint64 s_subscriptionId`: The subscription ID that this contract uses for funding requests.

- `bytes32 keyHash`: The gas lane key hash value, which is the maximum gas price you are willing to pay for a request in wei. It functions as an ID of the off-chain VRF job that runs in response to requests.

- `uint32 callbackGasLimit`: The limit for how much gas to use for the callback request to your contract's `fulfillRandomWords()` function. It must be less than the `maxGasLimit` limit on the coordinator contract. Adjust this value for larger requests depending on how your `fulfillRandomWords()` function processes and stores the received random values. If your `callbackGasLimit` is not sufficient, the callback will fail and your subscription is still charged for the work done to generate your requested random values.

- `uint16 requestConfirmations`: How many confirmations the Chainlink node should wait before responding. The longer the node waits, the more secure the random value is. It must be greater than the `minimumRequestBlockConfirmations` limit on the coordinator contract.

- `uint32 numWords`: How many random values to request. If you can use several random values in a single callback, you can reduce the amount of gas that you spend per random value. The total cost of the callback request depends on how your `fulfillRandomWords()` function processes and stores the received random values, so adjust your `callbackGasLimit` accordingly.

The contract includes the following functions:

- `requestRandomWords()`: Takes your specified parameters and submits the request to the VRF coordinator contract.

- `fulfillRandomWords()`: Receives random values and stores them with your contract.

- `getRequestStatus()`: Retrive request details for a given `_requestId`.

:::note[Security Considerations]
Be sure to review your contracts to make sure they follow the best practices on the [security considerations](/vrf/v2/security/) page.
:::

## Clean up

After you are done with this contract and the subscription, you can retrieve the remaining testnet LINK to use with other examples.

1. Open the Subscription Manager at [vrf.chain.link](https://vrf.chain.link/) and click the ID of your new subscription under the **My Subscriptions** list. The subscription details page opens.

1. Under your subscription details, click **Cancel subscription**. A field opens asking which wallet address you want to send the remaining funds to.

1. Enter your wallet address and click **Cancel subscription**. MetaMask opens and asks you to confirm the transaction. After you approve the transaction, Chainlink VRF closes your subscription account and sends the remaining LINK to your wallet.

## Vyper example

You must import the `VRFCoordinatorV2` Vyper interface. You can find it [here](https://github.com/smartcontractkit/apeworx-starter-kit/blob/main/contracts/interfaces/VRFCoordinatorV2.vy).
You can find a `VRFConsumerV2` example [here](https://github.com/smartcontractkit/apeworx-starter-kit/blob/main/contracts/VRFConsumerV2.vy). Read the _**apeworx-starter-kit**_ [README](https://github.com/smartcontractkit/apeworx-starter-kit) to learn how to run the example.
