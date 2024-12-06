# Hyperledger Fabric Quant Sandbox Tutorial - A Loyalty Program Use Case

## Introduction
This short tutorial will guide us on how to create a simple contract interaction flow for a loyalty program use case.

In the Hyperledger Fabric Quant sandbox, we have deployed two extended QRC Tier 1 smart contracts.

The first contract we have available is a QRC-20 type contract with an additional token "request" functionality. Quant Fungible Tokens (QRC-20) are used to represent multiple individual but replaceable units, such as currency, shares, or fractionalised assets. In this tutorial, QRC-20 tokens represent Loyalty Points acquired through an airline loyalty program.

Here is a list of the most useful QRC-20 functions to us:

| Function Type  |  Function Name | What Does it Do?  | Extension (Y/N)  | Notes   |
|---|---|---|---|---|
| Write  | transfer  |  Transfers the given amount from the Payer to the Payee (Recipient) account | N  |   |
| Write  |  approve | Approves the given amount to be taken from the Payer’s (Spender) account by the Payee (Recipient) account  |  N |   |
| Write  | transferFrom  |  Initiated by the Payee, the function takes the given amount to be from the Payer’s (Sender) account and transfers it to the Payee (Recipient) account | N  |  This function requires pre-approval for the amount to be taken – see the Approve function |
| Write  | requestTokens  |  Sends the default amount of tokens to the transaction sender's account, if the new balance of the account does not exceed the maximum token amount. | Y  | Function added to support the Loyalty program use case  |
| Read  | balanceOf  |Takes an address and returns the current balance for that address of the QRC-20 token as a uint256|  N |   |
| Read  | decimals  |  Returns the number of decimals for the QRC-20 token as a uint8 | N  |   |
| Read  | name  |  Returns the name of the QRC-20 token as a string | N  |   |
| Read  | symbol  |  Returns the symbol of the QRC-20 token as a string |  N |   |


The second contract we have available is a QRC-721 contract with an additional token "redemption" functionality. Quant Non Fungible Tokens (QRC-721) are used to represent a unique asset like a piece of art, digital content, or media and is understood as an irrevocable digital certificate of ownership and authenticity for a given asset, whether digital or physical. In this tutorial, a QRC-721 token represents a Flight Reward acquired by exchanging Loyalty Points.

Here is a list of the most useful QRC-721 functions to us

| Function Type  |  Function Name | What Does it Do?  | Extension (Y/N)  | Notes   |
|---|---|---|---|---|
| Write  | transferFrom  |Transfers the given token ID to the given ‘to’ identity from the given ‘from’ identity  | N  |  If the caller of the TransferFrom is NOT the same as the ‘from’ address, or has not been approved to take the token using the approve function then the caller has to be recorded as an approved current operator of the ‘from’ address in the smart contract |
| Write  |  approve | Approves the given token ID to be transferred to the given ‘to’ identity   |  N |   |
| Write  | RedeemToken | Exchange QRC-20 tokens for a QRC-721 token | Y  | Function added to support the Loyalty program use case  |
| Read  | ownerOf  |Returns the owner of the token |  N |   |
| Read  | owner  |Returns the account with the owner role of this token contract, i.e. the account that can mint new tokens |  N |   |
| Read  | balanceOf  |Returns the number of tokens in an owner’s account |  N |   |
| Read  | name  |  Returns the name of the QRC-721 token as a string | N  |   |
| Read  | symbol  |  Returns the symbol of the QRC-721 token as a string |  N |   |

## Loyalty Program - Step by Step Tutorial

Applications interact with the contracts in the following manner:

- 100 Loyalty Point tokens are requested by calling the "requestTokens" function in the QRC-20 contract
- 1 Flight Reward token is redeemed in exchange for 100 Loyalty Points by calling the "redeemToken" function in  the QRC-721 contract

In this step by step tutorial, we will go through a simple flow:

1. Create an Identity
2. Request Loyalty Point tokens
3. Check account balances for the amount of Loyalty Points we own
4. Redeem Flight Rewards for Loyalty Points

### 1. Create an Identity

There are currently two Quant Sandbox networks for Hyperledger Fabric:
- Quant Sandbox
- Quant Oracle Sandbox Testnet

Before being able to use any of the networks, you must register a new Application in [Quant Connect](https://connect.quant.network).  

An identity in the 'Quant Sandbox' network is automatically enrolled when creating an Application. The identity name will equal the clientId associated with the Application.

To enroll an identity in the 'Quant Oracle Sandbox Testnet' you must manually call the [Create Account](https://developers.quant.network/reference/createaccountfororaclefabric) endpoint. An identity with the name of the clientId that calls the endpoint will be enrolled in the network. You can only call the endpoint once per Application/clientId.

### 2. Request Loyalty Points

As we use our clientId and access token for authentication and Hyperledger Fabric is a permissioned technology, we do not require client side management of private keys.

To claim loyalty Point tokens, we use Overledgers Smart Contract API. The `smartContractId` field is set to `loyaltypoint` which is the id for the chaincode representing the Loyalty Point tokens.

> See the [Quant Developer Hub](https://developers.quant.network/reference/how-to-access-overledger-apis) for additional API documentation.

> Note that the URLs for interacting with the 'Quant Sandbox' and 'Quant Oracle Sandbox Testnet' are different. While the 'Quant Sandbox' network is available through the Overledger APIs, the 'Quant Oracle Sandbox Testnet' is available through Overledger Beta APIs.

Preparation Request:

- Quant Sandbox: POST `https://api.sandbox.overledger.io/api/preparations/transactions/smart-contracts/write` - [docs](https://developers.quant.network/reference/preparesmartcontractwrite)
- Quant Oracle Sandbox Testnet: POST `https://hook.eu2.make.com/qbtbxaze4rxiforl3sw2gs0tdx2mtw80` - [docs](https://developers.quant.network/reference/preparesmartcontractwritefororaclefabric)
```
{
  "location": {
    "technology": "Hyperledger Fabric",
    "network": "Quant Sandbox"
  },
  "signingAccountId": "<your-clientId-here>",
  "smartContractId": "loyaltypoint",
  "functionName": "requestTokens",
  "inputParameters": []
}
```

Preparation Response:
```
{
  "requestId": "a58e0a1f-2251-407c-8e4b-2bf5efa092ec",
  "gatewayFee": {
    "amount": "0",
    "unit": "QNT"
  },
  "dltFee": {
    "amount": "0",
    "unit": "N/A"
  },
  "nativeData": {}
}
```

Now that we have prepared a transaction, we get a confirmation of the prepared request and its assigned requestId. We can execute the transaction by posting the requestId to the execution API.

Execution Request:
- Quant Sandbox: POST `https://api.sandbox.overledger.io/api/executions/transactions` - [docs](https://developers.quant.network/reference/executesignedrequest)
- Quant Oracle Sandbox Testnet: POST `https://hook.eu2.make.com/jd77jtwdg4lm3gt6f23cn1ds5qh9xgdf` - [docs](https://developers.quant.network/reference/executesignedwritetransactionfororaclefabric)
```
{
  "requestId": "a58e0a1f-2251-407c-8e4b-2bf5efa092ec"
}
```

Execution Response:
```
{
  "location": {
    "technology": "Hyperledger Fabric",
    "network": "Quant Sandbox"
  },
  "requestId": "a58e0a1f-2251-407c-8e4b-2bf5efa092ec",
  "transactionId": "8b93b3662a0a467c08537131e60cac0a8fef0f6335a44cc0568d49f350cd29fd",
  "status": {
    "description": "Transaction successful",
    "value": "SUCCESSFUL"
  },
  "timestamp": "1656937087"
}
```

Please note that this function can only be invoked as long as the account does not hold any Loyalty Point tokens.

### 3. Check Account Balances

Now that we have our Loyalty Point tokens, we can query our account balance to check how many tokens we have received. The requestTokens function has been configured to deliver 100 tokens, and we can verify by calling the Smart Contract Read API.

The only input parameter required is the clientId.

Smart Contract Read Request:
- Quant Sandbox: POST `https://api.sandbox.overledger.io/api/smart-contracts/read` - [docs](https://developers.quant.network/reference/readsmartcontract)
- Quant Oracle Sandbox Testnet: POST `https://hook.eu2.make.com/9af8outacmb6bakbxmgq6t9qkt3v49ky` - [docs](https://developers.quant.network/reference/smartcontractreadfororaclefabric)
```
{
  "location": {
    "technology": "Hyperledger Fabric",
    "network": "Quant Sandbox"
  },
  "smartContractId": "loyaltypoint",
  "functionName": "balanceOf",
  "inputParameters": [
    {
      "type": "identity",
      "value": "<your-clientId-here>"
    }
  ],
  "outputParameters": [
    {
      "type": "string"
    }
  ]
}
```

Smart Contract Read Response:
```
{
  "location": {
    "technology": "Hyperledger Fabric",
    "network": "Quant Sandbox"
  },
  "smartContractId": "loyaltypoint",
  "functionName": "balanceOf",
  "outputParameters": [
    {
      "type": "string",
      "value": "100"
    }
  ]
}
```

As we can see, we got a balance of 100 tokens in the output parameters of the smart contract read response.
We can now proceed to use these Loyalty point tokens to redeem a reward.

### 4. Redeem Flight Rewards

To redeem the rewards (which are represented by QRC-721 tokens), we interact with the QRC-721 contract deployed on the chain. To do this, we use the following parameters in the request:
- `flightreward` as the smartContractId.
- `redeemToken` as the functionName.

The inputParameters are:
1. A stringified random non-negative integer less than 2^256 - this will be the ID assigned to the reward.
2. Any stringified JSON metadata - we have provided examples below.

Preparation Request:
- Quant Sandbox: POST `https://api.sandbox.overledger.io/api/preparations/transactions/smart-contracts/write` - [docs](https://developers.quant.network/reference/preparesmartcontractwrite)
- Quant Oracle Sandbox Testnet: POST `https://hook.eu2.make.com/qbtbxaze4rxiforl3sw2gs0tdx2mtw80` - [docs](https://developers.quant.network/reference/preparesmartcontractwritefororaclefabric)
```
{
  "location": {
    "technology": "Hyperledger Fabric",
    "network": "Quant Sandbox"
  },
  "signingAccountId": "<your-clientId-here>",
  "smartContractId": "flightreward",
  "functionName": "redeemToken",
  "inputParameters": [
    {
        "type": "string",
        "value": "12345678"
    },
    {
        "type": "string",
        "value": "{\"from\":\"London\",\"to\":\"Athens\",\"date\":\"1656938960\"}"
    }
  ]
}
```

Preparation Response:
```
{
  "requestId": "a4f8ce0c-1be7-4f08-b924-685e48a29694",
  "gatewayFee": {
    "amount": "0",
    "unit": "QNT"
  },
  "dltFee": {
    "amount": "0",
    "unit": "N/A"
  },
  "nativeData": {}
}
```

Execution Request:
- Quant Sandbox: POST `https://api.sandbox.overledger.io/api/executions/transactions` - [docs](https://developers.quant.network/reference/executesignedrequest)
- Quant Oracle Sandbox Testnet: POST `https://hook.eu2.make.com/jd77jtwdg4lm3gt6f23cn1ds5qh9xgdf` - [docs](https://developers.quant.network/reference/executesignedwritetransactionfororaclefabric)
```
{
  "requestId": "a4f8ce0c-1be7-4f08-b924-685e48a29694"
}
```

Execution Response:
```
{
  "location": {
    "technology": "Hyperledger Fabric",
    "network": "Quant Sandbox"
  },
  "requestId": "a4f8ce0c-1be7-4f08-b924-685e48a29694",
  "transactionId": "46c7ff69e6eec7039dbd911e67a4cde9cdcf4c93ee1580ed7f49d59540a99918",
  "status": {
    "description": "Transaction successful",
    "value": "SUCCESSFUL"
  },
  "timestamp": "1656939052"
}
```

As the transaction is successful, we can now make the same balance query request we have used before to check the number of Flight Rewards we hold. All we need to do is change the `smartContractId` to the `flightreward` contract instead.

In addition, we can use any of the standard functions from the QRC contracts specification.
