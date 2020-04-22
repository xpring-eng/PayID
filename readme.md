# PayID: A Protocol for Human-Readable, Travel-Rule-Compliant Addresses

With PayID, you can use human-readable addresses to transmit value in the currency of your choice. The PayID network allows participants to reach one another through a standardized address. For example, `alice$wallet.com` maps to the corresponding URL `https://wallet.com/alice`.

When you make an HTTP GET request that follows the PayID protocol, it resolves to an address on the underlying payment network. PayID is a payment-network-agnostic protocol, capable of sending payments in BTC, ETH, XRP, or fiat networks like ACH.

Each request must have a valid `Accept` request HTTP header. For example: `Accept: application/xrpl-mainnet+json` indicates the XRP Ledger Mainnet payment network and the JSON format of the response payload.

PayID is fully peer-to-peer with no central counterparty. Anyone can set up their own PayID server or integrate with a trustless blockchain solution so they have no counterparty risk.

The PayID protocol is designed to be simple, general, open, and universal. This makes it composable with any other existing namespace, including blockchain namespace projects like ENS and Unstoppable Domains or app-specific usernames.  Anyone with an existing username or address can get a PayID address that works across all platforms.  

Table of Contents
=================

* [Web standards](#web-standards)
* [Extensions and Travel Rule compliance](#extensions-and-travel-rule-compliance)
* [PayID integration and the Private PayID API](#payid-integration-and-the-private-payid-api)
* [Query users and manage payments with the PayID Public API](#query-users-and-manage-payments-with-the-payid-public-api)

## Web standards
PayID uses existing web standards and infrastructure, including HTTP and DNS. Each institution supports its users through its domain, creating a decentralized network where institutions maintain sovereignty over their own users. Each institution is empowered to participate in the network by deploying its own server with its own users. Additionally, a user who prefers to self-custody their cryptocurrency can deploy a personal PayID server, as described in [Set up a PayID server](#set-up-a-payid-server).

PayID does not rely on any consensus protocols, which simplifies infrastructure demands.

### Decentralized and trust-minimized
PayID is built on the most successful decentralized network in history: the web. PayID is fully peer-to-peer with no central counterparty. Anyone can set up their own personal PayID server so they have no counterparty risk.

### Protocol abstraction
PayID fully abstracts away underlying payment rail details from end users. A user only needs to keep track of a single PayID rather than a complex address for each network. Service operators can manage the underlying addresses behind the scenes according to their own security and privacy best practices without requiring the end user to take any action.

## Extensions and Travel Rule compliance

PayID is an extensible and flexible open standard. PayID can be extended to provide a simple solution for [Travel Rule](https://www.fatf-gafi.org/media/fatf/documents/recommendations/RBA-VA-VASPs.pdf) compliance, a relevant capability for many crypto services. Financial institutions can communicate regulatory requirements and the corresponding necessary data by using the PayID Public API, which is a standard REST API.

PayID improves the user experience of sending cryptocurrency payments, and it also provides a platform for present-day features like Travel Rule compliance and future features like subscriptions and pull payments. To achieve mainstream cryptocurrency adoption, we must build on top of PayID and other systems that integrate well with traditional financial infrastructure.

PayID provides an abstraction layer so that users can send payments to human-readable addresses while virtual asset service providers (VASPs) can meet the necessary compliance requirements in the background.

**Coming soon:** When a payment is sent to a PayID address, then if both entities are VASPs and must comply with the Travel Rule, they will exchange the user data in a secure handshake.

![Travel Rule](img/travel_rule.png)

## Future features and integrations
Features and integrations on the roadmap include:

* Invoices and receipts
* Additional payment rail standards
* Payment request ([W3C Standard](https://www.w3.org/TR/payment-request/)) integration
* Advanced payment type support: pull, push, streaming, subscription
* PayID forwarding and redirection

## PayID integration and the PayID Private API

You can deploy your own PayID server and then create PayIDs for your users using the PayID Private API. You can also query and modify this list of users. This API should be exposed internally only, so that only your company's systems can update PayID mappings.

Once you have set up a PayID server, anyone can use the PayID Public API to query address information. This API is publicly accessible so that anyone can send payments to your users' PayID addresses.

### Set up a PayID server
Participating institutions can use Xpring’s open source reference implementation of the PayID server. You might be able to ease the deployment process by connecting your PayID server to your existing user database so that it can respond to incoming GET requests.

To set up your own demo server, first ensure you have Docker installed, and then run these commands. You will create a local docker image.

```
git clone git@github.com:xpring-eng/payid.git
cd payid
./demo/run_payid_demo.sh
```

You can also set up your PayID server using [AWS and NGINX][aws-deploy.md].

You can then use the Private PayID API to:

* Create a user
* Get user information
* Modify a user
* Delete a user

![Open Source](img/open_source.png)

#### Demo

For demonstration purposes, once you have set up your PayID server, you can access the Private PayID API using these cURL commands.

**Get user information for an existing user**

```
curl --location --request GET 'bob$127.0.0.1:8080' \
--header 'Accept: application/xrpl-testnet+json'
```

**Create a user**

First, run this command to create a user.

```
curl --location --request POST 'http://127.0.0.1:8081/v1/users' \
--header 'Content-Type: application/json' \
--data-raw '{
    "pay_id": "alice$127.0.0.1",
    "addresses": [
        {
            "payment_network": "XRPL",
            "environment": "TESTNET",
            "details": {
                "address": "T7WSFgh6owANWoD2V3WRg6aeBveBzExkpDowirvnLDGL2YW"
            }
        }
    ]
}'
```

Then, run this command to get information about the newly-created user.

```
curl --location --request GET 'http://127.0.0.1:8080/alice' --header 'Accept: application/xrpl-testnet+json'
```

### Create and manage users with the Private PayID API

When you have your open source server set up, you can create and manage users with the Private PayID API.

#### API endpoint
Substitute your own domain for `{{http(s)}}{{host}}`. If running locally, you can use  `http://127.0.0.1:8081/`.

#### API version

Use `/v1`.

#### Create a user

This operation creates a single user.

```
POST {{http(s)}}{{host}}/v1/users
```

Payload: [Single user schema](#example-single-user-schema)

##### Response

```
201 Created
```

No response body.

##### Error responses

See [Example error schema](#example-error-schema).

<table>
<tr>
<th>Code</th>
<th>Description</th>
</tr>
<tr>
<td>400</td>
<td>Bad Request</td>
</tr>
<tr>
<td>409</td>
<td>Conflict</td>
</tr>
<tr>
<td>503</td>
<td>Service Unavailable</td>
</tr>
</table>

##### cURL example

```
curl --location --request POST '127.0.0.1/v1/users' \
--header 'Content-Type: application/json' \
--data-raw '{
    "pay_id": "alice$127.0.0.1",
    "addresses": [
        {
            "payment_network": "XRPL",
            "environment": "TESTNET",
            "details": {
                "address": "T7WSFgh6owANWoD2V3WRg6aeBveBzExkpDowirvnLDGL2YW"
            }
        }
    ]
}'
```

#### Retrieve PayID information for an existing user

You can query information about an existing user with the following request.

```
GET {{http(s)}}{{host}}/v1/users/{:$pay_id}
```

Example: `GET http://127.0.0.1:8081/v1/users/bob$xpring.money`

##### Response

```
200 OK
```

The user that matches the specified PayID address is returned. See [Example single user schema](#example-single-user-schema).

##### Error responses

See [Example error schema](#example-error-schema).

<table>
<tr>
<th>Code</th>
<th>Description</th>
</tr>
<tr>
<td>400</td>
<td>Bad Request</td>
</tr>
<tr>
<td>404</td>
<td>Not Found</td>
</tr>
<tr>
<td>503</td>
<td>Service Unavailable</td>
</tr>
</table>

##### cURL example

```
curl --location --request GET '{{payid_private_host}}/v1/users/alice${{payid_private_host_no_transport}}' \
--header 'Content-Type: application/json'
```

#### Modify a user

You can modify the user information associated with a particular PayID address.

```
PUT {{http(s)}}{{host}}/v1/users/{≈}
```

The request payload is the modified user schema for the specified PayID address.

##### Response

```
200 OK
201 Created
```

The response body is the updated [user schema](#example-single-user-schema).

##### Error responses

<table>
<tr>
<th>Code</th>
<th>Description</th>
</tr>
<tr>
<td>400</td>
<td>Bad Request</td>
</tr>
<tr>
<td>404</td>
<td>Not Found</td>
</tr>
<tr>
<td>409</td>
<td>Conflict</td>
</tr>
<tr>
<td>503</td>
<td>Service Unavailable</td>
</tr>
</table>

##### cURL example

```
curl --location --request PUT '127.0.0.1/v1/users/alice${{payid_private_host_no_transport}}' \
--header 'Content-Type: application/json' \
--data-raw '{
    "pay_id": "alice$127.0.0.1",
    "addresses": [
        {
            "payment_network": "XRPL",
            "environment": "TESTNET",
            "details": {
                "address": "T7WSFgh6owANWoD2V3WRg6aeBveBzExkpDowirvnLDGL2YW"
            }
        }
    ]
}'
```

#### Delete a user

```
{{http(s)://}{:pay_id}${{host}}{{private_port}}/v1/users/
```

##### Response

```
200 OK
```

##### Error responses

<table>
<tr>
<th>Code</th>
<th>Description</th>
</tr>
<tr>
<td>400</td>
<td>Bad Request</td>
</tr>
<tr>
<td>404</td>
<td>Not Found</td>
</tr>
<tr>
<td>503</td>
<td>Service Unavailable</td>
</tr>
</table>


##### cURL example

```
curl --location --request DELETE 'https://***REMOVED***/v1/users/alice$***REMOVED***/'
```

### Schemas

These schemas are used for request or response payloads for various requests.

#### Example single user schema

A single user can have multiple destinations, because the same user can have addresses on different networks for different currencies. This schema is used as a payload with a request to create a user, or as a response to a request to get information about a user.

```json
{
  "pay_id": "johndoe$xpring.money",
  "addresses": [
    {
      "payment_network": "XRPL",
      "environment": "TESTNET",
      "details": {
        "address": "TVQWr6BhgBLW2jbFyqqufgq8T9eN7KresB684ZSHKQ3oDth"
      }
    }, {
      "payment_network": "XRPL",
      "environment": "MAINNET",
      "details": {
        "address": "X7zmKiqEhMznSXgj9cirEnD5sWo3iZSbeFRexSFN1xZ8Ktn"
      }
    }
  ]
}
```

|Field | Description |
|------|-------------|
|`pay_id`| PayID user address |
|`addresses` | Object that includes payment address information for one or more payment networks.|
|`addresses.payment_network`| A payment network, like the bitcoin network, the XRPL, or ACH. |
|`addresses.environment`| "Environment" of the payment network for this payment address. For example, the XPRL has MAINNET, TESTNET, and DEVNET.|
|`address.details`| Actual payment information for this address. Must be in the form `CryptoAddressDetails` or `AchAddressDetails`. See [Interfaces](#interfaces).|

#### Example error schema

This example shows the format of an error payload.

```
{
    "statusCode": 422,
    "error": "Unprocessable Entity",
    "message": "Your data is bad"
}
```

### Request headers

PayID is a fundamentally neutral protocol. When you make a request, the HTTP `Accept` header of the request specifies the payment network and environment, and PayID is therefore capable of returning a user's address information for any network in which that user participates.

An example request has this form.

```
GET /user HTTP/1.1
HOST: wallet.com
```

The different header options are shown here, with example values.

| Currency | Header | Address payload |
|:--------|:-------|:---------|
| BTC     | application/btc+json   | { <br>address: '1BvBMSEYstWetAu4m4GFg7xJaNVN2'<br> }      |
| XRP     | application/xrpl-mainnet+json   | { <br>address: 'XV5sbjUmgPpvXv4ixFWZ5ptAYZ6PD28Sq49uo34VyjnmK5H'<br> } |
| ACH     | application/ach+json   | { <br> account: '363023456079',<br>routing: '011302838'<br>}  |
| ILP     | application/spsp4+json | { <br> destination_account: 'g.institution.alice',<br> shared_secret: 'Anofne0Slxjv0Aj4malghem'<br>  } |

### Code examples

#### Payment

This example demonstrates a payment to `alice$institution.com`.

```javascript
const axios = require('axios')
const btc = require('bitcoinjs-lib')

async function pay(payId, amount) {
  const user = payId.split('$')[0]
  const url = payId.split('$')[1]

  const response = await axios.get({
    method: 'GET',
    baseURL: 'https://' + url + '/' + user
    headers: { accept: 'application/btc+json' }
  }
  const tx = await new btc.Transaction(address, amount)
}
pay('alice$institution.com', 0.63)
```

#### Generate a seed-based wallet

In this example, you generate a seed-based wallet using the Xpring SDK. See [Xpring SDK docs](https://xpring.io/docs).

```javascript
// Xpring SDK
const { Wallet } = require("xpring-js");

const wallet = Wallet.generateWalletFromSeed("snRiAJGe3zRwiYDBm1M");
const tx = await wallet.send(0.63, 'alice$institution.com');
```

## Query users and manage payments with the PayID Public API

The PayID Public API does not require authentication, as it is open to any user. The PayID Private API is meant for administrators who are building a payment network.

### API endpoint

The PayID Public API treats a PayID address as the base endpoint for all of the following requests. Therefore, if you are requesting from `alice$wallet.com` the following paths would be:

```
GET alice$wallet.com
```

Substitute the appropriate URL for your PayID address.

### Get payment information

You can get payment information for a PayID address.

#### Request

```
Header: `Accept: <request header>`
```

```
GET <pay_ID-address>/
```

#### Response

```
200 OK
```

A `PaymentInformation` object is returned.

#### Error response

<table>
<tr>
<th>Code</th>
<th>Description</th>
</tr>
<tr>
<td>404</td>
<td>Not Found<br>
Occurs if PayID address does not exist.</td>
</tr>
<tr>
<td>503</td>
<td>Service Unavailable</td>
</tr>
</table>


### Interfaces

#### PaymentInformation type

```
Interface PaymentInformation {
  addressDetailType: AddressDetailType
  addressDetails: CryptoAddressDetails | AchAddressDetails
  proofOfControlSignature?: string
  payId?: string                       
}
```

#### CryptoAddressDetails

```
Interface CryptoAddressDetails {
	address: string
      tag?: string
}
```

#### AchAddressDetails

```
interface AchAddressDetails {
 accountNumber: string
 routingNumber: string
}
```

## Use Xpring SDK with PayID

[Xpring SDK](https://www.xpring.io/docs) can be used to simplify the process of developing with PayID. Currently only the Node.js version is available, and Java and Swift will soon be available.

### Demo

This demo uses PayID to resolve an address on the specified payment network, which is XRP Ledger in this case.

To execute this demo:

1. Clone the [Xpring SDK repo](https://github.com/xpring-eng/Xpring-SDK-Demo/).
2. Change to the `Xpring-SDK-Demo/node` directory.
3. Run `npm i` to install the dependencies.
4. Run `node src/index-payid.js` to execute the demo.

View [index-payid.js](https://github.com/xpring-eng/Xpring-SDK-Demo/blob/master/node/src/index-payid.js).

#### AchAddressDetails

```
Interface AchAddressDetails {
	accountNumber: string
	routingNumber: string
}
```

####

```
Interface Error {
      statusCode: integer
  	error: string
      message: string
}
```

### Headers for GET requests for PayID Public API

This launch of PayID includes those headers specific to the Xpring ecosystem. Each payment network is free to establish its own standard headers. These headers should be submitted with every GET request, but not POST.

#### Headers for XRP

<table>
<tr>
<th>Accept header</th>
<th>Description</th>
</tr>
<tr>
<td>application/xrpl-mainnet+json</td>
<td>Returns mainnet xAddress</td>
</tr>
<tr>
<td>
application/xrpl-testnet+json</td>
<td>Returns testnet xAddress</td>
</tr>
<tr>
<td>application/xrpl-devnet+json</td>
<td>Returns devnet xAddress</td>
</tr>
</table>

#### Headers for ACH

<table>
<tr>
<th>Accept header</th>
<th>Description</th>
</tr>
<tr>
<td>
application/ach+json</td>
<td>Returns account and routing number</td>
</tr>
</table>

#### Headers for ILP

<table>
<tr>
<th>Accept header</th>
<th>Description</th>
</tr>
<tr>
<td>
application/spsp4+json </td>
<td>Returns destination address and shared secret</td>
</tr>
</table>


#### Headers for BTC

<table>
<tr>
<th>Accept header</th>
<th>Description</th>
</tr>
<tr>
<td>application/btc-mainnet+json</td>
<td>Returns mainnet address</td>
</tr>
<tr>
<td>
application/btc-testnet+json</td>
<td>Returns testnet address</td>
</tr>
</table>


#### Headers for ETH

<table>
<tr>
<th>Accept header</th>
<th>Description</th>
</tr>
<tr>
<td>application/eth-mainnet+json</td>
<td>Returns mainnet address</td>
</tr>
<tr>
<td>application/eth-testnet-name+json</td>
<td>Returns testnet-name address. Ethernet has a number of different testnets. Refer to <a href="https://github.com/ethereum/EIPs/blob/master/EIPS/eip-155.md">EIP 155</a>.</td>
</tr>
</table>
