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

In this example, we will swap `ETH on Ethereum` to `USDT on Polygon`

Routes in WagPay are combination of DEX(s) and Bridge(s) paired in a way so that user needs to pay as less fees as possible and receive higher returns as possible within lowest time

#### Route Interface

```typescript
interface Routes {
	name: string // Name of the Bridge
	bridgeTime: string // Avg time the bridge will take (in secs)
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

#### Let's get some routes for `ETH on Ethereum to USDT on Polygon`&#x20;

{% tabs %}
{% tab title="index.ts" %}
```javascript
import WagPay from "@wagpay/sdk"
import { ChainId, CoinKey } from "@wagpay/types"
const wagpay = new WagPay()

let query: RouteData = {
     fromChain: ChainId.ETH,
     toChain: ChainId.POL,
     fromToken: CoinKey.ETH,
     toToken: CoinKey.USDT, 
     amount: '1000000000000000000' // in wei
}

wagpay.getRoutes(query)
     .then(routes => {
          console.log(routes)
     })
     
// Output
// [
//   {
//     "name": "Celer",
//     "bridgeTime": "1200",
//     "contractAddress": "",
//     "amountToGet": "10166.168694",
//     "transferFee": "0.641311",
//     "uniswapData": {
//       "dex": "0x7cBBc355A50e19A58C2D8C24Be46Eef03093EDf7",
//       "fees": 30.5922,
//       "chainId": 1,
//       "fromToken": {
//         "name": "Ethereum",
//         "symbol": "ETH",
//         "address": "0xeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee",
//         "chainAgnositcId": "ETH",
//         "decimals": 18,
//         "chainId": 1
//       },
//       "toToken": {
//         "name": "Tether USD",
//         "symbol": "USDT",
//         "address": "0xdac17f958d2ee523a2206206994597c13d831ec7",
//         "chainAgnositcId": "USDT",
//         "decimals": 6,
//         "chainId": 1
//       },
//       "amountToGet": 10166.8078
//     },
//     "route": {
//       "fromChain": "1",
//       "toChain": "137",
//       "fromToken": {
//         "name": "Ethereum",
//         "symbol": "ETH",
//         "address": "0xeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee",
//         "chainAgnositcId": "ETH",
//         "decimals": 18,
//         "chainId": 1
//       },
//       "toToken": {
//         "name": "Tether USD",
//         "symbol": "USDT",
//         "address": "0xc2132D05D31c914a87C6611C10748AEb04B58e8F",
//         "chainAgnositcId": "USDT",
//         "decimals": 6,
//         "chainId": 137
//       },
//       "amount": "10000000000000000000"
//     }
//   }
//   ...more bridge
// ]
```


{% endtab %}
{% endtabs %}

#### Fetching routes with some extra options

Sometimes, you just want to not use a bridge or dex, it happens, either you get mad angry at the founder because of some random twitter post or there are some personal reasons

Tension not, we've got you covered

You can explicitly specify which bridge or dex, you want to allow or deny or even prefer

```typescript
import WagPay from "@wagpay/sdk"
import { ChainId, CoinKey } from "@wagpay/types"
const wagpay = new WagPay()

let query: RouteData = {
	fromChain: ChainId.ETH,
 	toChain: ChainId.POL,
 	fromToken: CoinKey.ETH,
 	toToken: CoinKey.USDT, 
 	amount: '1000000000000000000', // in wei
	bridge: {
		prefer: ['hyphen'],
		deny: ['hop']
	},
	dex: {
		prefer: ['uniswap']
	}
}

wagpay.getRoutes(query) // can also use async / await
	.then(routes => {
		console.log(routes)
	 })
```

This will prefer `Hyphen` bridge more than any other bridge and will not show `Hop` bridge, same for DEXs

## Let's execute the best route

You chose the best possible route or any other route, we need to execute it now

In Execution, user will asked for ERC20 approve (if fromToken is an ERC20) and then it will call [`WagPayBridge` Contract ](smart-contracts.md)&#x20;

```
import WagPay from "@wagpay/sdk"
import { ChainId, CoinKey } from "@wagpay/types"
const wagpay = new WagPay()

const getAndExecuteRoute = () => {
	let query: RouteData = {
		fromChain: ChainId.ETH,
	 	toChain: ChainId.POL,
	 	fromToken: CoinKey.ETH,
	 	toToken: CoinKey.MATIC, 
	 	amount: '1000000000000000000', // in wei
	}
	
	const routes = await wagpay.getRoutes(query)
	
	const provider = new ethers.providers.Web3Provider(window.ethereum) // provider can be any valid ethereum provider
	const signer = await provider.getSigner()

	// this will get erc20 approval if needed, do dex transaction and bridge transaction in single transaction
	// if erc20 approval is needed -> user needs to sign 2 transactions otherwise only one!!
	const executed = await wagpay.executeRoute(routes[0], signer)

	return executed
}

```

