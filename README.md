# MicroCoin API
MicroCoin Rider is an API server for the MicroCoin ecosystem.
It acts as the interface between MicroCoin network and applications that want to access the MicroCoin network.
It allows you to submit transactions to the network, check the status of accounts, subscribe to transactions, etc.
Rider provides a RESTful API to allow client applications to interact with the MicroCoin network.
You can communicate with Rider using cURL or just your web browser. However, if you’re building a client application, you’ll likely want to use a MicroCoin SDK in the language of your client";

# Before you begin
Before you start developing useful to download the MicroCoin wallet. You can download the latest version from
the official [MicroCoin website](https://microcoin.hu)

## Supported programming languages
MicroCoin Rider is a simple REST API. You can call using any language what you prefer.
At this time we are offering PHP and Javascript SDKs, but you can generate your own using the [swagger codegen](https://github.com/swagger-api/swagger-codegen) project.

## Networks
We have to networks.
The Mainnet, where the production coin runs and the Testnet where you can develop.
The primary Mainnet endpoint: https://rider.microcoin.hu.
The primary Testnet endpoint: https://testnet.rider.microcoin.hu.

## Accounts
In other cryptocoins you can generate an "address" to receive coins.
In MicroCoin that's not possible, address's are like accounts and accounts are generated by the blockchain.
So, the main difference, is that if you don't have a MicroCoin account (mined by yourself or received from another account's owner) you cannot have receive MicroCoins.

### How can I receive an account?
An account can only be operated by a private key. Account's owners can change the account's key to a new one.
You can generate a Private/Public key pair. You send the PUBLIC KEY (Note: private key must always be kept private and only for you!) to the owner of an account.
The account's owner changes the key of an account to your new public key.
After this, the owner of the account will be you, and the old owner will not have access to operate with this account because he doesn't know the private key.
**For testing and developing you can use the Testnet. On the Testnet you can easily mining accounts.**

---

# JS SDK quickstart guide

## Download the client SDK
First you need a MicroCoin client SDK.
You can download it from [here](https://github.com/MicroCoinHU/MicroCoin-Javacript-SDK/releases/), or clone from our [Github](https://github.com/MicroCoinHU/MicroCoin-Javacript-SDK) repository.
```bash
git clone https://github.com/MicroCoinHU/MicroCoin-Javacript-SDK.git
```
In the dist folder you will find the precompiled, production ready files.

## Keys and signatures
MicroCoin works with ECDSA signatures, so you need to work with ECDSA keys and signatures.
You can use your favorite **ECDSA** package, or use `elliptic.js`. We are using `elliptic.js` in our demos.
You can find a detailed documentation on the official github page https://github.com/indutny/elliptic

## HTML boilerplate
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>MicroCoin client minimum project</title>
    <script src="/dist/microcoin-promise.js"></script>
    <script src="/dist/elliptic.js"></script>
</head>
<body>
<script>
    var api = new MicroCoin.AccountApi();
    api.getAccount("0-10").then(account => console.log(account));
</script>
</body>
</html>
```

## Generate new ECDSA keyPair
If you have no keys, you must generate a new key, then store it in a secure place.
**Please note: if you lose your key, you lose your accounts and your coins**
```js
var ec = new elliptic.ec("secp256k1");
var myKey = ec.genKeyPair();
```
## Import ECDSA private key
If you have a key, you can import it from a hexadecimal string.
```js
var ec = new elliptic.ec("secp256k1");
var myKey = ec.keyPair({ "priv":"PRIVATE KEY IN HEX", "privEnc":"hex" });
```

### Where is your private key?

![Private key](/img/privkey.png)

## Export ECDSA key
Sometimes you need save your key, or you need to display it. You can export your key using this method
```js
var exportedKey = {
   private: keyPair.getPrivate("hex"),
   public: {
       X: keyPair.getPublic().x.toString(16),
       Y: keyPair.getPublic().y.toString(16)
    }
}
```

## List your accounts
If you have accounts you can list there. First time you have no accounts, so you need get one.
Every account belongs to a public key. One public key can be used for multiple accounts.
```js
var accountApi = new MicroCoin.AccountApi();
// Never send your private key!
accountApi.myAccounts({
    "curveType":"secp256k1",
    "x": myKey.getPublic().getX("hex"),
    "y": myKey.getPublic().getY("hex")
}).then(myAccounts => console.log(myAccounts));
```

## Get single account
You can request information from a single account. You can see the balance, name, etc..
```js
var accountApi = new MicroCoin.AccountApi();
accountApi.getAccount("0-10").then(account => console.log(account));
```

## List accounts for sale
You can purchase accounts, but you need to know which accounts are for sale.
```js
var accountApi = new MicroCoin.AccountApi();
accountApi.getOffers().then(offers => console.log(offers));
```
## Purchase account
You can purchase any account for sale, if you have enough coins.
```js
var purchaseRequest = new MicroCoin.PurchaseAccountRequest();
purchaseRequest.setAccountNumber("34689-25"); // The account to purchase
purchaseRequest.setFounderAccount("1-22");   // The founder account will pay for the account
purchaseRequest.setFee(0);  // Optional miner fee
// This is key of the new owner. You can use your own key, or any key what you want.
// After the purchase completed you can only manage this account with this keyPair
purchaseRequest.setNewKey({
    "CurveType":"secp256k1",
    "X": myKey.getPublic().getX("hex"),
    "Y": myKey.getPublic().getY("hex")
});
// First prepare the transaction
accountApi.startPurchaseAccount(purchaseRequest).then(function (transaction) {
    // Now we need to sign our transaction using the founder account private key
    var signature = myKey.sign(transaction.getHash());
    // Now fill the signature property
    transaction.signature = { "r": signature.r, "s": signature.s };
    // Last we need to commit our transaction and we are done
    accountApi.commitPurchaseAccount(transaction).then((response)=>console.log(response), e => console.error(e));
});
```

## Sending coins
If you have enough balance, you can send coins from your accounts to any valid account.
```js
var transactionApi = new MicroCoin.TransactionApi();
var sendCoinRequest = new MicroCoin.TransactionRequest();
sendCoinRequest.setSender('0-10'); // Source account
sendCoinRequest.setTarget('1-22'); // Target account
sendCoinRequest.setAmount(0.0001); // Amount to send
sendCoinRequest.setFee(0); // optional miner fee, one transaction / block (5 min) is free
sendCoinRequest.setPayload("Hello MicroCoin"); // optional payload
// First we are creating a transaction
transactionApi.startTransaction(sendCoinRequest).then(function (transaction) {
    // When the transaction created, we need to sign the transaction
    var signature = myKey.sign(transaction.getHash());
    // Now fill the signature property
    transaction.signature = { "r": signature.r, "s": signature.s };
    // Last we need to commit our transaction and we are done
    transactionApi.commitTransaction(transaction).then((response)=>console.log(response), e => console.error(e));
});
```

## Change account owner
If you want change your account owner, you can do it with change the assigned public key.
```js
var request = new MicroCoin.ChangeKeyRequest();
request.setAccountNumber("0-10"); // The account to change
// newKey: Public key of the new owner
request.setNewOwnerPublicKey({
    "curveType":"secp256k1",
    "x": newKey.getPublic().getX("hex"),
    "y": newKey.getPublic().getY("hex")
});
// First we are creating a transaction
accountApi.startChangeKey(request).then(function (transaction) {
    // When the transaction created, we need to sign the transaction
    // myKey: key of the current owner 
    var signature = myKey.sign(transaction.getHash());
    transaction.signature = { "r": signature.r, "s": signature.s };
    // Last we need to commit our transaction and we are done, the new owner can use his/her account
    accountApi.commitChangeKey(transaction).then((response)=>console.log(response), e => console.error(e));
});
```
