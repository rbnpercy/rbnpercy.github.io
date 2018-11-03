---
layout: post
permalink: /tutorials/building-nodejs-app-with-bancor-apis
title: "Building your First Node.js App with the Bancor APIs"
description: ""
tags:
- bancor-network
- ethereum-applications
- node-js-ethereum
- nodejs-crypto
- bancor-api
- cryptocurrency
- crypto-app-build
---

**TL;DR**  Building your first Crypto application can be a very daunting experience.  Most likely, due to the fact that building the financial and Blockchain underpinning *is* indeed rather difficult.  Happily for us developers, however, the heavy lifting can be shifted onto someone else, and we can simply piggyback on top of their technology.


Building your First Node.js Application with the Bancor APIs
====================================

Building your first Crypto application can be a very daunting experience.  Most likely, due to the fact that building the financial and Blockchain underpinning *is* indeed rather difficult.  Happily for us developers, however, the heavy lifting can be shifted onto someone else, and we can simply piggyback on top of their technology.

Bancor offers a decentralized liquidity network that provides users with a simple, low-cost way to buy and sell tokens. Bancor’s open-source protocol empowers tokens with built-in convertibility directly through their smart contracts, allowing integrated tokens to be instantly converted for one another, without needing to match buyers and sellers in an exchange. 

The Bancor Wallet enables automated token conversions directly from within the wallet, at prices that are more predictable than exchanges and resistant to manipulation.

With the launch of the Bancor public API, we can now take advantage of the whole functionality of the Bancor Network, but from within our own applications!

In this tutorial, we will build a Web Application in Node.js, that wraps the Bancor Network functionality, allowing our app users to convert tokens, without having to visit the Bancor web app itself.  So without further delay, let's jump in!

**Note** - Once the public Bancor Node.js SDK is launched, this tutorial will be updated to use the API library, instead of wrapping raw HTTP requests.

[*The complete sample code for this tutorial can be found on GitHub, here.*](http://github.com/rbnpercy/tokey)


<br/>
## Setting up our App

For this tutorial, we'll create a simple Node.js application using the Express web framework.  The best way to create a new project with Express is by utilising the `express-generator` package.  Firstly, if you have not already got the generator installed, do so by running:

``` bash
$ npm install express-generator -g
```

We will be using the *EJS* engine and *Sass* for CSS preprocessing.  Go ahead and run the following command to generate the project skeleton:

```bash
$ express --view=ejs --css=sass --git tokey
```

Then, you can run the following to install dependencies, and start the application:

```bash
Change directory:
   $ cd tokey

Install dependencies:
   $ npm install

Run the app:
   $ DEBUG=tokey:* npm start
```

***Great***!  We have our Node application up and ready to start interacting with the Bancor Network API's.  Our application will work as a traditional Token conversion marketplace, taking our user from selecting the token they wish to purchase, to viewing the pricing data of their chosen token against their currency token, to submitting the transaction to the relevant Blockchain.


<br/>
## Fetching Available Token Pairs

The first function of our application will be to allow our users to view the Tokens available for them to trade.  The Bancor API takes HTTP requests and returns standard, unformatted JSON.  For example - if we want to view a list of the available Token pairs, we can send the following `curl` command:

```bash
curl "https://api.bancor.network/0.1/currencies/convertiblePairs"
```

And we will have returned:

```bash
{"data":{"BNT":"ETH","STORMBNT":"BNT","RVTBNT":"BNT","DBETBNT":"BNT","POEBNT":"BNT","BMCBNT":"BNT","CLNBNT":"BNT","POA20BNT":"BNT","KINBNT":"BNT","BAXBNT":"BNT","STX":"BNT","TKNBNT":"BNT","OMGBNT":"BNT","BATBNT":"BNT","ELFBNT":"BNT","GTOBNT":"BNT","MADBNT":"BNT","ENJBNT":"BNT","DAIBNT":"BNT","J8TBNT":"BNT","MYBBNT":"BNT","VIBBNT":"BNT","GNOBNT":"BNT","POWRBNT":"BNT","SNTBNT":"BNT","KNCBNT":"BNT","MANABNT":"BNT","REQBNT":"BNT","SHPBNT":"BNT","AIONBNT":"BNT","MDTBNT":"BNT","BOXXBNT":"BNT","FTRBNT":"BNT","RCNBNT":"BNT","LDCBNT":"BNT","SIGBNT":"BNT","FKXBNT":"BNT","SENSEBNT":"BNT","UPBNT":"BNT","XDCEBNT":"BNT"...}}
```

To utilise this in our app, we need to create a function that is going to wrap this HTTP request, receive the response, format it and display it on a web page.

The first thing we need to do, is to create a route on our application to display handle this data.  Open up `app.js`, on ***line 8*** beneath the default generated line of `var indexRouter = require('./routes/index');` 

Add another line as follows: `var tokenRouter = require('./routes/tokens');`, then in the same file, ensure our app knows to *use* this.

Add the following: `app.use('/tokens', tokenRouter);`.

Now, in the `tokey/routes/` folder, add a file `tokens.js`, and have it reflect the following:

```javascript
var express = require('express');
var router = express.Router();
var request = require('request');


/* GET token pages. */
router.get('/available', function(req, res, next) {
  res.render('tokens/available', { title: 'Tokens' });
});

router.get('/prices', function(req, res, next) {
  res.render('tokens/prices', { title: 'Token Pricing' });
});

router.get('/convert', function(req, res, next) {
  res.render('tokens/convert', { title: 'Token Conversion' });
});

module.exports = router;

```

Now that we have our routes setup, add a `tokens` folder into the default `views` folder, and add the file: `available.ejs`.  Have it reflect:

```html
<!DOCTYPE html>
<html>
  <head>
    <title><%= title %></title>
    <link rel='stylesheet' href='/stylesheets/style.css' />
  </head>
  <body>
    <main id="main">
      <div class="container">
        <h1><%= title %></h1>
        <p>Here we have the <%= title %> we can utilise.</p>
      </div>
    </main>

    <footer id="footer">
      <div class="container">
        <p>&copy; Copyright 2018 - @rbin</p>
      </div>
    </footer>
  </body>
</html>
```

You can now visit the `/tokens/available` page in your browser, and see that we're now ready to include the data from the Bancor API.  The Bancor *available pairs API* returns tradable tokens in a long JSON array.  For our users, we want to format this and display as individual items.

To do this, firstly, we are going to expand the router function that applies to our *"Available pairs"* resource.  In this function, we are going to use the Node `Request` library to call the Bancor API and return the results whenever a user requests our resource.  We are then going to parse the results and display in our application.

Open up the `routes/tokens.js` file, and update it to reflect the following:

```javascript
var request = require('request');


/* GET token pages. */
router.get('/available', function(req, res, next) {
  request('https://api.bancor.network/0.1/currencies/convertiblePairs', (error, response, body) => {

    tokens = JSON.parse(response.body);
    res.render('tokens/available', {title: 'Available Tokens', tokens: tokens.data});
  })
});
```

As you can see in the above snippet, when our users load the `/available` route, we dispatch a request to the Bancor API to fetch the Tokens we can offer.  We then parse the JSON results received and pass them as a variable into our EJS template using the `res.render` functionality.

**Note:** for the sake of brevity in this tutorial, I have not included error handling on fetching the Bancor results, but this should be done for any live applications you are to build.

Now that we have fetched and parsed the available token pairs from Bancor, we need to display them in our EJS template.  Open up `views/tokens/available.ejs` and include the following in the *main* section:

```ejs
<main>
  <div class="container">
    <h1><%= title %></h1>
    <p>Here we have the <%= title %> which we can utilise:</p>
    
    <%  Object.keys(tokens).forEach(function (key) { %>
      <li><%= key %></li>
    <% }) %>
    
  </div>
</main>
```

All we do in our EJS template is iterate over the parsed JSON results, and list each of the `keys` as a bullet point.

Loading this page, you will now get the following results:

![Bancor Available Token Pairs](/assets/img/available-pairs-page.png) 



<br/>
## Fetch Token Prices

The next piece of functionality will allow our users to gather price data on their desired token, against the currency token they are looking to pay with.  For the sake of this tutorial, we are *only* going to display information assuming our user wishes to purchase ***Bancor*** tokens, with ***Ethereum*** tokens.

In a real world situation, we would extend this to clickable links on the available tokens, a submittable form for specifying tokens, or utilise URL query params so that our users can specify which token they wish to purchase and with which they wish to do so.

In this example, I have hardcoded the token type values into the Bancor API URL to reflect the above, but for a live application, this would always be dynamic.

Open up `routes/tokens.js`, and update your `/prices` route to reflect the following:

```js
router.get('/prices', function(req, res, next) {
  request('https://api.bancor.network/0.1/currencies/BNT/ticker?fromCurrencyCode=ETH', (error, response, body) => {

    prices = JSON.parse(response.body);
    res.render('tokens/prices', {title: 'Token Prices', prices: prices.data});
  })
});
```

Now, open up `views/tokens/prices.ejs`, and have it reflect this:

```html
<main id="pricesPage">
  <div class="container">
    <h1><%= title %></h1>
    <table id="tokenprices">
      <tr class="heads">
        <%  Object.keys(prices).forEach(function (key) { %>
          <td><%= key %></td>
        <% }) %>
      </tr>
      <tr>
        <%  Object.values(prices).forEach(function (val) { %>
          <td><%= val %></td>
        <% }) %>
      </tr>
    </table>

  <hr />

  <p>You are requesting to buy <b><%= prices.name %></b> tokens, using the currency <b><%= prices.displayCurrency %></b>.</p>

  <p>For <b>1</b> <b><%= prices.displayCurrency %></b> token(s), you will receive <b><%= prices.price %></b> <b><%= prices.name %></b> tokens.</p>

  </div>
</main>
```

As you can see in the above code, we display a table with the parsed JSON data we have gathered from the Bancor API, containing the price data on the tokens requested by our user.  Below that table, we also create a short paragraph describing to the user exactly the pricing data they have requested.  Saving and running the application (with added CSS styling), will output the following:

![Token Pricing Data](/assets/img/token-price-data.png) 


<br/>
## Allow our Users to Trade / Convert Tokens

Now that our users can see exactly which Tokens asre available to be traded, and can find out the prices associated with doing so, it's time for the final piece of this tutorial's sample functionality; allowing our users to *build* a token convert transaction, before submitting it to the relevant Blockchain.

In our application, once again for the sake of a simple example, we are going to hardcode the values that would ordinarily be dynamically specified by our user.  Just like our last function, we're going to have our user trade ***Ethereum*** for ***Bancor*** tokens.

In our app, the user submits the details of the tokens they wish to convert, and our application responds with the transaction format and data string that the user required to submit to their chosen Blockchain *(currently only Ethereum supported).*

The first action we need to take is to update our `/convert` route, and add a further route that submits the details our user requests to receive back the actual formatted data string that is then signed and submitted to the *Ethereum Blockchain.*

Open up `routes/tokens.js` and update the following:

```js
router.get('/convert', function(req, res, next) {
  res.render('tokens/convert', { title: 'Token Conversion Data', output: {}, toblock: {} })
});

router.post('/convert', (req, res) => {
  res.render('tokens/convert', {
    title: 'Token Conversion Data:',
    output: JSON.stringify(req.body),
    toblock: {}
  })
});

router.post('/convert/submit', (req, res) => {
  res.render('tokens/convert', {
    title: 'Token Conversion Data:',
    output: JSON.stringify(req.body),

    request.post({url:'https://api.bancor.network/0.1/currencies/convert', form: output}, function optionalCallback(err, httpResponse, body) {
      toblock: JSON.parse(body);
    });

  })
});
```

Now that we have our routing functionality added for the Conversion process, we need to create our `convert` EJS template that will allow the user to enter conversion data in a web form, that then returns the JSON request our application sends to the Bancor API quickConvert function, which in turn passes back the data string that our user must *sign &amp; execute* on their chosen Blockchain.

If not already done, go ahead and create the file `views/tokens/convert.ejs` and update the `main` section to the following:

```html
<main>
  <div class="container">
    <h1><%= title %></h1>

    <form method="post" action="/tokens/convert" novalidate>
      <div class="form-field">
        <label for="blockchainType">Blockchain Type</label>
        <input class="input" id="blockchainType" name="blockchainType" />
      </div>
      <div class="form-field">
        <label for="from_currency_id">fromCurrencyId</label>
        <input class="input" id="fromCurrencyId" name="fromCurrencyId" value="" />
      </div>
      <div class="form-field">
        <label for="toCurrencyId">toCurrencyId</label>
        <input class="input" id="toCurrencyId" name="toCurrencyId" value="" />
      </div>
      <div class="form-field">
        <label for="amount">amount</label>
        <input class="input" id="amount" name="amount" value="" />
      </div>
      <div class="form-field">
        <label for="minimum_return">minimumReturn</label>
        <input class="input" id="minimumReturn" name="minimumReturn" value="" />
      </div>
      <div class="form-field">
        <label for="owner_address">ownerAddress</label>
        <input class="input" id="ownerAddress" name="ownerAddress" value="" />
      </div>
      <div class="form-actions">
        <button class="btn" type="submit">Go!</button>
      </div>
    </form>

    <br/>
    <h3>Conversion Params:</h3>
    <p> <%- output %> </p>

    <form method="post" action="/tokens/convert/submit" novalidate>
      <div class="form-actions">
        <button class="btn" type="submit">Get Blockchain Transaction</button>
      </div>
    </form>


    <hr/>
    <h3>To Blockchain:</h3>
    <p class="toBlock"><%- toblock %></p>

  </div>
</main>
```

<br/>

Saving up and launching the application, you will notice that our `/convert` page offers our users a web form to fill in the details of the tokens they wish to convert:

![Token Conversion Data](/assets/img/token_conversion_data.png) 

When the user submits this form, our application parses the form details into JSON format, ready to submit to the Bancor Convert API.

![Token Conversion Params](/assets/img/conversion_params.png) 

Once our user clicks on the *"Get blockchain transaction"* button, our application posts the form data (parsed into JSON), to the Bancor Convert API ready to receive back the actual data string that the user must *sign &amp; exec* on the Blockchain.

![Formatted Blockchain Data String](/assets/img/formatted_to_blockchain.png) 

[The format for submitting to the Ethereum Blockchain can be found here.](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_sendrawtransaction)


And with that, this tutorial is drawing to a close.  Although we have covered a fairly simple example in this usecase, the advanced functionality of the Bancor Network offers us developers the chance to create really rather excellent applications.

Simply by wrapping the Bancor HTTP APIs, we have created our own Node.js powered token exchange (albeit a simple one).

When the official Bancor API libraries are launched in the near future, yet more opportunities will be afforded to us and we will be launching many more technical tutorials to aid development and exploration.

If you have any queries regarding this article, I encourage you to reach out to me at [robin@percy.pw](mailto:robin@percy.pw), or alternatively on [Twitter](http://twitter.com/rbin).


Thanks for reading!

 [- **@rbin**](http://robin.percy.pw)
 
 <br/>
 [*The complete sample code for this tutorial can be found on GitHub, here.*](http://github.com/rbnpercy/tokey)
