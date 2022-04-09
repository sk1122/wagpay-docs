# Quick Start

{% hint style="info" %}
**Good to know:** A quick start guide can be good to help folks get up and running with your API in a few steps. Some people prefer diving in with the basics rather than meticulously reading every page of documentation!
{% endhint %}

## Get your API keys

Your API requests are authenticated using API keys. Any request that doesn't include an API key will return an error.

You can generate an API key from your [Dashboard](https://wagpay.xyz/dashboard) at any time.

## Install the library

The best way to interact with our API is to use one of our official libraries:

{% tabs %}
{% tab title="Node" %}
```
# Install via NPM
npm install --save wagpay

# Install via yarn
yarn add wagpay
```
{% endtab %}
{% endtabs %}

## Start Accepting Crypto Payments

WagPay works on Payment Intent Model, where you have to create an Intent of Payment on behalf of user, you will receive a unique payment link, user will be redirected there, they will complete the payment and you will receive confirmation that payment is received

Let's the code

{% tabs %}
{% tab title="Node" %}
```javascript
// Import WagPay
import WagPay, { PaymentInterface } from "wagpay";

// Initiate the Class
const wagpay = new WagPay("API_KEY_HERE");

// Create a Payment Intent Object
// WagPay provides an Interface for creating Payment Intent Object
const intent: PaymentInterface = {
    value: 20,
    from_email: 'email',
    currency: ['solana', 'ethereum', 'usdcsol', 'usdceth'],
    receiving_store: 'store_slug'
};

// Create a Payment Intent for a user
const paymentIntent = await wagpay.createPaymentIntent(intent);

// To confirm payment
// This promise will resolve to true if payment is successful
// and will resolve to false, if payment is not made within 10mins
const confirmed = await wagpay.checkPayment(paymentIntent);
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