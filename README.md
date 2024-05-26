# What is PumpPortal?

**A 3rd-Party (unofficial) API for Pump.fun**. We provide HTTPS endpoints that allow you to programmatically trade on, and gather data from, the Pump.fun bonding curve. 

PumpPortal is designed to be easy to use, so you can start building in minutes: there are no libraries to install and nothing to configure. If you can send an HTTP request or connect to Websockets you can use PumpPortal!

Our Data API is free (subject to rate limits). Read the next section to get started building right away!

Our Trading API is gated by an API key and charges a 0.5% fee per trade.

We're excited to see your trading bots, websites, or any other applications you integrate with Pump.fun! Connect with us in Telegram to ask questions or show off what you're building. If you build something cool we might feature your project on this page!

# Local Trading API Docs

To get a transaction for signing locally and sending with a custom RPC, send a POST request to 

`https://pumpportal.fun/api/trade-local`

Your request body must contain the following options:

- `publicKey`: Your wallet public key
- `action`: "buy" or "sell"
- `mint`: The contract address of the token you want to trade (this is the text after the '/' in the pump.fun url for the token.)
- `amount`: The amount of SOL or tokens to trade. If selling, amount can be a percentage of tokens in your wallet (ex. amount: "100%")
- `denominatedInSol`: "true" if amount is SOL, "false" if amount is tokens
- `slippage`: The percent slippage allowed
- `priorityFee`: Amount to use as priority fee
- `pool`: (optional) Currently 'pump' and 'raydium' are supported options. Default is 'pump'.

If your parameters are valid, you will receive a serialized transaction in response. See the example below for how to send this transaction with Web3.js.

### Examples


```js
import { VersionedTransaction, Connection, Keypair } from '@solana/web3.js';
import bs58 from "bs58";

const RPC_ENDPOINT = "Your RPC Endpoint";
const web3Connection = new Connection(
    RPC_ENDPOINT,
    'confirmed',
  );

async function sendPortalTransaction(){
      const response = await fetch(`https://pumpportal.fun/api/trade-local`, {
          method: "POST",
          headers: {
              "Content-Type": "application/json"
          },
          body: JSON.stringify({
              "publicKey": "your-public-key",  // Your wallet public key
              "action": "buy",                 // "buy" or "sell"
              "mint": "token-ca-here",         // contract address of the token you want to trade
              "denominatedInSol": "false",     // "true" if amount is amount of SOL, "false" if amount is number of tokens
              "amount": 1000,                  // amount of SOL or tokens
              "slippage": 1,                   // percent slippage allowed
              "priorityFee": 0.00001,          // priority fee
              "pool": "pump"                   // exchange to trade on. "pump" or "raydium"
          })
      });
      if(response.status === 200){ // successfully generated transaction
          const data = await response.arrayBuffer();
          const tx = VersionedTransaction.deserialize(new Uint8Array(data));
          const signerKeyPair = Keypair.fromSecretKey(bs58.decode("your-wallet-private-key"));
          tx.sign([signerKeyPair]);
          const signature = await web3Connection.sendTransaction(tx)
          console.log("Transaction: https://solscan.io/tx/" + signature);
      } else {
          console.log(response.statusText); // log error
      }
}

sendPortalTransaction();
```
# Real-time Updates

Stream real-time trading and token creation data by connecting to the PumpPortal Websocket at `wss://pumpportal.fun/api/data`. 

Once you connect, you can subscribe to different data streams. The following methods are available: 
 
 - `subscribeNewToken` For token creation events.

 - `subscribeTokenTrade` For all trades made on specific token(s).

 - `subscribeAccountTrade` For all trades made by specific account(s).

 ### Examples:

```js
import WebSocket from 'ws';

const ws = new WebSocket('wss://pumpportal.fun/api/data');

ws.on('open', function open() {

      // Subscribing to token creation events
      let payload = {
          method: "subscribeNewToken", 
        }
      ws.send(JSON.stringify(payload));

      // Subscribing to trades made by accounts
      payload = {
          method: "subscribeAccountTrade",
          keys: ["AArPXm8JatJiuyEffuC1un2Sc835SULa4uQqDcaGpAjV"] // array of accounts to watch
        }
      ws.send(JSON.stringify(payload));

      // Subscribing to trades on tokens
      payload = {
          method: "subscribeTokenTrade",
          keys: ["91WNez8D22NwBssQbkzjy4s2ipFrzpmn5hfvWVe2aY5p"] // array of token CAs to watch
        }
      ws.send(JSON.stringify(payload));
});

ws.on('message', function message(data) {
      console.log(JSON.parse(data));
});
```
# Fees

### Data API

There is no charge to use the Data API.

### Trading API

We take a 0.5% fee on each trade. (Note: the fee is calculated before any slippage, so the actual fee amount may not be exactly 0.5% of the final trade value.)

This does not include Solana network fees, or any fees charged by the Pump.fun bonding curve.
