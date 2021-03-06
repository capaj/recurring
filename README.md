A node client for [recurly](https://recurly.com)'s v2 api, with support for secure parameter signing for [recurly.js](https://docs.recurly.com/recurlyjs) embedded forms.

[![on npm](http://img.shields.io/npm/v/recurring.svg?style=flat)](https://www.npmjs.org/package/recurring)  [![Tests](http://img.shields.io/travis/ceejbot/recurring.svg?style=flat)](http://travis-ci.org/ceejbot/recurring)  [![Dependencies](http://img.shields.io/david/ceejbot/recurring.svg?style=flat)](https://david-dm.org/ceejbot/recurring)  ![io.js supported](https://img.shields.io/badge/io.js-supported-green.svg?style=flat)  

__This code is still in development.__ We do not have complete coverage of the API yet.

# Recurly API

An example of typical usage:

```javascript
var recurly = require('recurring')(); // note function invocation
recurly.setAPIKey('your-api-key');
recurly.setRateLimit(400);
recurly.setCache(false);

var account = new recurly.Account();
account.id = 'account-uuid';
account.fetch(function(err) {
  account.fetchSubscriptions(function(err, subscriptions {
    console.log(subscriptions[0].plan);
    subscriptions[0].cancel(function(err, updated) {
      console.log(updated.state); // will be 'canceled'
    });
  });
});

recurly.Account.all(function(accounts) {
  // accounts is an array containing all customer accounts
});

recurly.Plan.all(function(plans) {
  // plans is an array containing all plans set up for your account
});

```

## Configuration

**recurly.setAPIKey()**  
In order to access the Recurly API you must supply your API key which can be setterFunc by calling the `setAPIKey()`
method.

```javascript
var recurly = require('recurring')();
recurly.setAPIKey('your-api-key');
```

**recurly.setRateLimit()**  
The recurly API has a rate limit policy in place that prevents excessive calls being made to the API. By default
sandbox accounts have a limit of 400 requests per second and live accounts have a limit of 1000 requests per second.
In order to help ensure that you do not exceed these limits Recurring provides a configurable rate limiter.
The rate limiter can be configured by calling the `setRateLimit()` method.

```javascript
var recurly = require('recurring')();
recurly.setRateLimit(400);
```

<!-- **recurly.clearCache()**

By default data fetched from Recurly will be cached in memory so that subsequent requests to fetch the same data does
not result in additional API calls to Recurly. The cache can be flushed by calling the `clearCache()` method.

```javascript
var recurly = require('recurring')();
recurly.clearCache();
```

**recurly.setCache()**

The cache can be disabled by calling the `setCache()` method.

```javascript
var recurly = require('recurring')();
recurly.setCache(false);
``` -->

## All data types

Recurly is not consistent about how it names the ID fields for each data type. For some it's `uuid` and for others
`foo_code`. Recurring hides this away: every data type has an `id` property that sets the correct field name for Recurly.

**instance.create()**  
Create an object of the given type by POSTing to Recurly.

```javascript
DataType.create(optionsHash, function(err, object));
```

**instance.fetch()**  
Fetch an item of a given type from Recurly. The item must have an id.

```javascript
instance.fetch(function(err, instance))
```

**instance.destroy()**  
Destroy, delete, close, cancel, or otherwise remove the specified object. Invokes http `DELETE` on the item's href. The
item must have an id.

```javascript
instance.destroy(function(err));
```

**instance.update()**  
Most data types have an `update()` method that changes the stored data.

```javascript
instance.update(options, function(err, updated));
```

## Plan

**Plan.all()**  
Fetch a list of plans. Responds with an array of all plans.

```javascript
Plan.all(function(err, plans));
```

**Plan.iterator()**  
Fetch a list of plans. Responds with an async iterator that lazy loads data from recurly in batches of 200.

```javascript
var iterators = require('async-iterators');
var iterator = Plan.iterator();
iterators.forEachAsync(iterator, function (err, data, next) {
  ...
  next();
});
```

**plan.fetchAddOns()**  
Fetch plan addons. Responds with a array of plan addons.

```javascript
plan.fetchAddOns(function(err, addons));
```

## Account

**Account.all()**  
Fetch a list of accounts. Responds with an array of all accounts in the passed-in state. Defaults to 'active'.

```javascript
Account.all(state, function(err, accounts));
```

**Account.iterator()**  
Fetch a list of accounts. Responds with an async iterator that lazy loads data from recurly in batches of 200.

```javascript
var iterators = require('async-iterators');
var iterator = Account.iterator();
iterators.forEachAsync(iterator, function (err, data, next) {
  ...
  next();
});
```

**account.close()**  
Close an account. Alias for delete.

```javascript
account.close(function(err));
```

**account.reopen()**  
Reopen a closed account:

```javascript
account.reopen(function(err, updated));
```

**account.fetchBillingInfo()**  
Fetch billing information for an account. Responds with an BillingInfo object.

```javascript
account.fetchBillingInfo(function(err, info));
```

**account.fetchSubscriptions()**  
Fetch subscription information for an account. Responds with an array of subscriptions for this account.

```javascript
account.fetchSubscriptions(function(err, subscriptions));
```

### Billing Info

**billingInfo.update()**  
Add/update billing information for an account.

```javascript
binfo = new recurly.BillingInfo();
binfo.account_code = '1234';
var billing_data = {
  first_name: 'Dummy',
  last_name: 'User',
  number: '4111-1111-1111-1111',
  month: 1,
  year: 2020,
  verification_value: '123',
  address1: '760 Market Street',
  address2: 'Suite 500',
  city: 'San Francisco',
  state: 'CA',
  country: 'USA',
  zip: '94102'
};

binfo.update(billing_data, function(err, binfo) {
  demand(err).not.exist();
  binfo.last_four.must.equal('1111');
});
```

**Using a billing token**

You can also update billing information using a recurly.js billing token in place of the raw billing information
(recommended)

```javascript
binfo = new recurly.BillingInfo();
binfo.account_code = '1234';
var billing_data = {
  token_id: 'bunYTdIdjfJJY6Z87j5NtA' // <- recurly.js billing token.
};
binfo.update(billing_data);
```

**skipAuthorization**

When adding billing information to an account Recurly may ake an authorization attempt against the card which may incur
charges depending on your payment gateway. In order to prevent Recurly from making this authorization attempt, a
``skipAuthorization`` paramater can be supplied along with the billing information.

```javascript
binfo = new recurly.BillingInfo();
binfo.account_code = '1234';
var billing_data = {
  number: '4111-1111-1111-1111',
  month: 1,
  year: 2020,
  skipAuthorization: true // <- do not make authorization request.
};
binfo.update(billing_data);
```

## Subscription

**Subscription.all()**  
Fetch a list of subscriptions. Responds with an array of all subscriptions in the passed-in state. Defaults to 'active'.

```javascript
Subscription.all(state, function(err, subscriptions));
```

**Subscription.iterator()**  
Fetch a list of subscriptions. Responds with an async iterator that lazy loads data from recurly in batches of 200.

```javascript
var iterators = require('async-iterators');
var iterator = Subscription.iterator();
iterators.forEachAsync(iterator, function (err, data, next) {
  ...
  next();
});
```

**subscription.cancel()**  
Cancel a subscription.

```javascript
subscription.cancel(function(err, updated));
```

**subscription.reactivate()**  
Reactivate a cancelled subscription.

```javascript
subscription.reactivate(function(err, updated));
```

**subscription.postpone()**  
Postpone a subscription.

```javascript
subscription.postpone(nextRenewalDate, function(err, updated));
```

**subscription.terminate()**  
Terminate a subscription using the specific refund type. Valid refund types are 'partial', 'full', and 'none'.

```javascript
subscription.terminate(refundType, function(err, updated));
```

## Coupon

**Coupon.all()**  
Fetch a list of coupons. Responds with an array of all coupons.

```javascript
Coupon.all(function(err, coupons));
```

**Coupon.iterator()**  
Fetch a list of coupons. Responds with an async iterator that lazy loads data from recurly in batches of 200.

```javascript
var iterators = require('async-iterators');
var iterator = Coupon.iterator();
iterators.forEachAsync(iterator, function (err, data, next) {
  ...
  next();
});
```

**coupon.redeem()**  
Redeem a coupon.

```javascript
coupon.redeem(options, function(err, redemption));
```

## Redemption

[TODO]

## Transaction

**Transaction.all()**  
Fetch a list of transactions. Responds with an array of all transactions.

```javascript
Transaction.all(function(err, transactions));
```

**Transaction.iterator()**  
Fetch a list of transactions. Responds with an async iterator that lazy loads data from recurly in batches of 200.

```javascript
var iterators = require('async-iterators');
var iterator = Transaction.iterator();
iterators.forEachAsync(iterator, function (err, data, next) {
  ...
  next();
});
```

**transaction.refund(amountInCents, function(err))**  
If amountInCents is omitted, the transaction is refunded in full. Responds with any errors; the transaction object is
updated.

## Errors

All callbacks follow the node convention of reporting any error in the first parameter. If a transaction with Recurly
succeeds but is rejected by Recurly for some reason-- inconsistent data, perhaps, or some other reason-- that err
parameter is an instance of RecurlyError. The original [transaction errors](http://docs.recurly.com/api/transactions/error-codes) reported by Recurly are available as an array of structs in the `errors` parameter. For instance, here's the result of a
billing info update with an invalid, expired CC:

```javascript
{
	name: 'RecurlyError',
	message: '2 transaction errors',
	errors: [
		{
			field: 'billing_info.number',
			symbol: 'invalid',
			message: 'is not a valid credit card number'
		},
		{
			field: 'billing_info.number',
			symbol: 'expired',
			message: 'is expired or has an invalid expiration date'
		}
	]
}

```

## SignedQuery

This provides the back-end support for signing parameters for forms embedded using recurly.js. See Recurly's [signature documentation](https://docs.recurly.com/api/recurlyjs/signatures) for details on which parameters must be signed for
each form type.

```javascript
var recurly = require('recurring');

var signer = new recurly.SignedQuery('your-private-api-key');
signer.set('account', { account_code: 'account-id' });
var signedParameters = signer.toString();
```

The `nonce` & `timestamp` parameters are generated for you if you don't provide them. The nonce is created using [node-uuid](https://github.com/broofa/node-uuid).

## FormResponseToken

After Recurly handles a form submission, it posts to you a token pointing to the form
results. Use a FormResponseToken object to fetch the results object represented by the token.

```javascript
var recurly = require('recurring');

var recurlyResponse = new recurly.FormResponseToken(token, 'subscription');
recurlyResponse.process(function(err, subscription) {
	if (err)
		return handleError(err);

	// subscription contains the new subscription data;
});
```

Having to hint about the type of the response is clunky; TODO fix.

## Contributing

Unit tests for whatever you fix/implement/improve would be awesome. Recurring's are written with [mocha](http://visionmedia.github.com/mocha/) and [chai](http://chaijs.com).

## License

MIT. See accompanying LICENSE file.
