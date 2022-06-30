# Quick Start

{% hint style="info" %}
**Good to know:** A quick start guide can be good to help folks get up and running with your API in a few steps. Some people prefer diving in with the basics rather than meticulously reading every page of documentation!
{% endhint %}

## Get your API keys

Oh wait⚡

You don't need any API Keys or permission from us to use WagPay SDK or for bridging tokens

Truly Permissionless

## Install the `@wagpay/sdk`

The best way to interact with our API is to use one of our official libraries:

{% tabs %}
{% tab title="Node" %}
```
# Install via NPM
npm install --save @wagpay/sdk @wagpay/types

# Install via yarn
yarn add @wagpay/sdk @wagpay/types
```
{% endtab %}
{% endtabs %}

## Fetching Available Routes

Before doing actual swapping, you need to get available routes for tokens between chains you want to swap

In this example, we will swap `USDC on Polygon` to `ETH on Ethereum`

Routes in WagPay are combination of DEX(s) and Bridge(s) paired in a way so that user needs to pay as less fees as possible and receive higher returns as possible within lowest time

#### Route Interface

```typescript
interface Routes {
	name: string // Name of the Bridge
	bridgeTime: string // Avg time the bridge will take
	contractAddress: string // contract address of WagPay Provider for that bridge
	amountToGet: string // avg amount you will receive on toChain
	transferFee: string // bridge fees + dex fees + gas fees
	uniswapData: UniswapData
	route: RouteResponse
}

interface RouteResponse {
	fromChain: string // chain id of from chain
	toChain: string // chain id of to chain 
	fromToken: Token
	toToken: Token
	amount: string // amount that user is sending (in wei)
}

interface DexData {
	dex: string // dex address
	fees: number // fees of dex
	chainId: number // chain id on which we have to use dex
	fromToken: Token 
	toToken: Token
	amountToGet: number // amount user will receive after swapping
}

interface Token {
	chainAgnositcId: CoinKey // CoinKey is an enum of all the coins (tokens) supported by WagPay
	symbol: string // Symbol of Token
	name: string // Name of Token on respective chain
	address: string // address of the token
	decimals: number // decimals of the token
	chainId: ChainId // chain id out of which token is based
}
```

#### Let's get some routes for `USDC on Polygon to ETH on Ethereum`&#x20;

{% tabs %}
{% tab title="Node" %}
```javascript
import WagPay, { ChainId, CoinKey } from "@wagpay/sdk"
const wagpay = new WagPay()

let query: RouteData = {
	fromChain: ChainId.ETH,
 	toChain: ChainId.POL,
 	fromToken: CoinKey.ETH,
 	toToken: CoinKey.MATIC, 
 	amount: '1000000000000000000' // in wei
}

wagpay.getRoutes(query)
  .then(routes => {
     console.log(routes)
  })
```
{% endtab %}
{% endtabs %}

* We are importing `WagPay` and `PaymentInterface` from `wagpay`
* We are initiating `WagPay` class with the `API_KEY`&#x20;
* Next, we are creating an Payment Intent which is of Interface `PaymentInterface`
* We create that payment intent into our database
* User can pay using the unique link provided
* If User has confirmed & paid the transaction, `checkPayment` will resolve to `true` else `false`

``
