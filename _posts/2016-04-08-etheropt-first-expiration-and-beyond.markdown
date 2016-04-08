---
---
The first <a href="http://etheropt.github.io">Etheropt</a> expiration was successful. The transaction, <a href="https://live.ether.camp/transaction/106aeaaea61732">seen here on EtherCamp's Ethereum blockchain explorer</a>, represents the first decentralized options expiration in the history of mankind.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">The very first decentralized options expiration just ran successfully. <a href="https://twitter.com/hashtag/etheropt?src=hash">#etheropt</a> <a href="https://twitter.com/hashtag/ethereum?src=hash">#ethereum</a> <a href="https://twitter.com/hashtag/blockchain?src=hash">#blockchain</a></p>&mdash; Etherboost (@etherboost) <a href="https://twitter.com/etherboost/status/718336072305020928">April 8, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Now that the first expiration is done, we're ready to release a new version of Etheropt with some new features.

## One smart contract per expiration

Etheropt now has one smart contract per expiration. The main benefit of this is that the gas cost for crossing a transaction will now be less. On the Etheropt page, each expiration now has its own tab, and you will have to add funds to any expiration you want to trade. Here's a screenshot from the testnet version:

<img src="/images/screenshot1.png" />

Along with this change, the expiration transaction will now automatically send your final profit/loss back to your Ethereum account, zeroing out your smart contract balance. The smart contract for the expiration that just expired (which did not have this feature) will still be listed in the last tab if you need to withdraw funds from it.

## Transaction log

Etheropt now has a realtime transaction log so you can actually see when someone adds or withdraws funds, and when someone crosses an option trade. Here's a screenshot from the testnet version:

<img src="/images/screenshot2.png" />

## Documentation

The main page now has some detailed guides to help new users understand how to get started, how options work, and how to trade on Etheropt. Here's a screenshot:

<img src="/images/screenshot3.png" />

## Other smaller features

Other smaller features:

 * When executing a trade, it is now possible to change the "Good for N blocks" number so you can make your order rest on the order book for longer than the default 10 blocks.
 * The gas price is fixed at 20 Gwei, and the default gas limit for sending an order is now 1M. So you'll only need 0.02 in your Ethereum account in order to match an order. In reality, the transaction will cost less than that and you'll be refunded the difference. For adding or withdrawing funds, the default gas limits will amount to at most 0.004 eth and 0.006 eth respectively.
 * It is possible to send funds with an order so you can effectively fund your account and execute a trade at the same time. This has been implemented in the smart contract but not in the GUI yet.

## New expirations

Perhaps most importantly, Etheropt now has an assortment of expirations you can trade: weekly, monthly, quarterly, and yearly.

We owe a special shoutout to [Reality Keys](https://www.realitykeys.com) for providing the cryptographically signed settlement prices and of course [Ethereum](https://ethereum.org) for making decentralized finance (among other things) possible.

Trade away! If you need help, come [chat on Gitter](https://gitter.im/etheropt/etheropt.github.io).
