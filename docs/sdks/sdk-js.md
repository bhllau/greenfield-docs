---
title: Javascript SDK
---

# Quickstart

The BNB Greenfield JavaScript SDK is designed for front-end environments and provides an API for interacting with BNB Greenfield decentralized storage. It offers a range of operations, including retrieving permission details, gas fees, etc. The SDK also includes a crypto component for signing transactions and sending them to BNB Greenfield.

However, it should be noted that this SDK does not include methods for interacting with BNB Smart Chain (BSC). For a comprehensive understanding of available operations, refer to the [API Reference](https://github.com/bnb-chain/greenfield-js-sdk).

## Install

```bash
# QA
npm install @bnb-chain/greenfield-chain-sdk@alpha

# TESTNET
npm install @bnb-chain/greenfield-chain-sdk@beta
```

## Usage

To utilize the SDK functionality, users need to instantiate a client object from the SDK. This client object serves as the interface to interact with BNB Greenfield and perform the desired operations.

### Create client
```js
import {Client} from '@bnb-chain/greenfield-chain-sdk'
const client = Client.create(GRPC_URL, GREEN_CHAIN_ID);
```

The SDK offers two types of operations - sending transactions to BNB Greenfield, allowing users to modify the state of the blockchain; the second type enables users to send queries and retrieve metadata information about objects stored on the blockchain. 

### Transactions

#### 1. Transaction construction

The SDK offers functionality for transferring tokens between accounts, providing a straightforward and convenient way to perform token transfers. With the SDK, users can easily initiate and execute token transfers within the desired accounts, streamlining the process of managing and exchanging tokens.

The SDK includes functionality for simulating and broadcasting transactions, allowing users to retrieve essential information related to gas fees, and sending the transaction over network.

```js
const { simulate, broadcast } = await client.account.transfer({
  fromAddress: address,
  toAddress: transferInfo.to,
  amount: [
    {
      denom: 'BNB',
      amount: ethers.utils.parseEther(transferInfo.amount).toString(),
    },
  ],
});
```

#### 2. Simulate Transactions

```js
// simulate tx
const simulateInfo = await simulate({
    denom: 'BNB',
});

// This includes details such as gas limit, gas price, and overall gas fee.
```

#### 3. Broadcast Transactions
```js
// broadcast tx
const broadcastRes = await broadcast({
  denom: 'BNB',
  gasLimit: Number(simulateInfo.gasLimit),
  gasPrice: simulateInfo.gasPrice,
  payer: address,
  granter: '',
});
```
`broadcast` use `window.ethereum` as signature provider by default.

If you want to use others, you can set `signTypedDataCallback`:

```js
// TrustWallet
const broadcastRes = await broadcast({
  //...
  signTypedDataCallback: async (addr: string, message: string) => {
    return await window.trustwallet.request({
      method: 'eth_signTypedData_v4',
      params: [addr, message],
    });
  }
});
```
For the Nodejs environment, you can set `privateKey` with the private key:
```js
const broadcastRes = await broadcast({
  //...
  privateKey: '0x.......'
});
```

#### 4. Multi-Transactions

The SDK also provides support for bundling multiple operations into a single transaction, thereby reducing gas fees. This feature allows users to optimize their transactions by combining several operations together, minimizing the overall gas cost associated with executing them individually. By leveraging this functionality, users can effectively manage their gas fees and enhance the efficiency of their transactions within the blockchain network using the SDK.

```js
const createGroupTx = await client.group.createGroup(params);
const mirrorGroupTx = await client.crosschain.mirrorGroup({
    groupName,
    id,
    operator,
  });

const principal = {
          type: PermissionTypes.PrincipalType.PRINCIPAL_TYPE_GNFD_GROUP,
          value: GRNToString(newGroupGRN(address as string, groupName)),
        };

const statement: PermissionTypes.Statement = {
          effect: PermissionTypes.Effect.EFFECT_ALLOW,
          actions: [PermissionTypes.ActionType.ACTION_GET_OBJECT],
          resources: [
            GRNToString(
              type === 'Data'
                ? newObjectGRN(bucketName, name)
                : newObjectGRN(bucketName, '*'),
            ),
          ],
        };

const policyTx = await client.object.putObjectPolicy(bucketName, name, {
            operator: address,
            statements: [statement],
            principal,
          });

const { simulate, broadcast } = await multiTx([
          createGroupTx,
          mirrorGroupTx,
          policyTx,
        ]);
```

### Querying Metadata

```js
// get account info
await client.account.getAccount(address);
```

### Storage Providers

In addition, the SDK provides support for querying the list of storage providers available and offers generic search capabilities for exploring metadata attributes.

#### 1. List Storage Providers

```js
export const getSps = async () => {
  const sps = await client.sp.getStorageProviders();
  const finalSps = (sps ?? []).filter(
    (v: any) => v?.description?.moniker !== 'QATest',
  );

  return finalSps;
};
```

#### 2. Search for objects

It's important to note that even if an object is set to private, its metadata remains publicly accessible. This metadata includes information such as file size, file type, and file name. 

```js
export const searchKey = async (key: string) => {
  try {
    return await client.sp.listGroup(key, `${DAPP_NAME}_`, {
      sourceType: 'SOURCE_TYPE_ORIGIN',
      limit: 1000,
      offset: 0,
    });
  } catch (e) {
    return [];
}
```

## Code Repository SDK
- [Official JS implementation SDK](https://github.com/bnb-chain/greenfield-js-sdk)