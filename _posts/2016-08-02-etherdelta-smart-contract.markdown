---
---
We announced [EtherDelta](https://etherdelta.github.io) on Reddit a few weeks ago, and we saw some DAO trading volume leading up to the hardfork. In case you missed it, EtherDelta is a decentralized exchange for Ether and Ethereum tokens. It's the second major release for Etherboost, following in the footsteps of [Etheropt](https://etheropt.github.io), the decentralized options exchange. We have a pretty exciting announcement coming just around the corner, but that's for another post. In this post, we want to go over some of the technical details of EtherDelta.

## High level overview

At a high level, EtherDelta functions just like a normal exchange. But because its entire existence is defined by a smart contract, there are some nuances you should understand. That's what this post will cover. If you're looking for how-to guides about trading through the GUI, check out the Guides tab on [EtherDelta](https://etherdelta.github.io), and feel free to ask for help in the chat.

The EtherDelta smart contract allows you to deposit or withdraw Ether or any [ERC-20](https://github.com/ethereum/EIPs/issues/20) Ethereum token. Once you have deposited into the EtherDelta smart contract, you can start trading.

Like any other exchange, EtherDelta has an order book of resting orders. The resting orders, however, are not stored in the smart contract. When you submit a resting order to EtherDelta, it gets broadcast to a Gitter channel, which the GUI reads and parses to construct the order book. If you're an Etheropt user, this will be familiar to you, since Etheropt works in the exact same way.

So what exactly does a resting order look like? A resting order is a signed intent to trade. When you submit a resting order, you name your price and volume, and then sign the order with your Ethereum account. When someone else wants to trade with your resting order, he submits a transaction to the smart contract with your signed intent to trade and the volume he wishes to trade. The smart contract checks the signature and makes sure everyone has enough funds to cover the trade, and then executes the trade by moving funds between the two users' accounts.

The primary benefit of storing resting orders off-chain is that you don't have to create an Ethereum transaction and pay gas to submit a resting order. Resting an order is free, like it should be. A fee for placing a resting order would discourage market makers. We don't want to hurt market liquidity, and that's the main reason why the order book is off-chain.

## Smart contract

Now that we've gone through the high-level overview, let's look at the smart contract source code, which you can find [on GitHub](https://github.com/etherdelta/etherdelta.github.io/blob/master/etherdelta.sol).

The first hundred or so lines implement an ERC-20 token that is used by the test framework. Skip to the EtherDelta contract:

<pre><code>
contract EtherDelta {
	mapping (address => mapping (address => uint)) tokens;
	//mapping of token addresses to mapping of account balances
	//ether balances are held in the token=0 account
	mapping (bytes32 => uint) orderFills;
	address public feeAccount;
	uint public feeMake; //percentage times (1 ether)
	uint public feeTake; //percentage times (1 ether)
</code></pre>

The first section of code defines the variables the contract will keep in storage. The `tokens` variable is where user balances are stored. For example, if your address is `0x123...` and the DAO token address is `0xbb9...`, then your DAO balance will be in `tokens[0xbb9][0x123].` By special case, your Ether balance will be in `tokens[0][0x123]`. Note that all Ether amounts are in Wei, and all token amounts are in the base unit of the token (which is usually Wei, but depends on the token). The `orderFills` variable is used to keep track of orders that have been partially or completely filled. For example, if you create a resting order to buy 10 tokens with a hash of `0x234...`, and someone submits a transaction to sell you 5 tokens (taking out half of your order), then `orderFills[0x234]` will be changed to 5. The `feeAccount` variable holds the account to which EtherDelta trading fees are paid. The `feeMake` and `feeTake` variables hold the fee percentages for making and taking liquidity, times 1 ether. For example, since 1 ether = 10^18, 10^17 would represents 10%. In the deployed version of EtherDelta, the making fee is 0 and the taking fee is 0.3%.

<pre><code>
	event Order(address tokenGet, uint amountGet, address tokenGive, uint amountGive, uint expires, uint nonce, address user, uint8 v, bytes32 r, bytes32 s);
	event Trade(address tokenGet, uint amountGet, address tokenGive, uint amountGive, address get, address give);
	event Deposit(address token, address user, uint amount, uint balance);
	event Withdraw(address token, address user, uint amount, uint balance);
</code></pre>

The next section of code defines events. These are emitted by different transactions and stored in the blockchain. The GUI uses them to display a list of trades, deposits, and withdrawals.

<pre><code>
function EtherDelta(address feeAccount_, uint feeMake_, uint feeTake_) {
	feeAccount = feeAccount_;
	feeMake = feeMake_;
	feeTake = feeTake_;
}

function() {
	throw;
}
</code></pre>

Next, we have the constructor and the default function. The constructor simply initializes the fee account and fee percentages. The default function simply throws an error. In other words, any Ether sent to EtherDelta without a function call will be returned to sender.

<pre><code>
function deposit() {
	tokens[0][msg.sender] += msg.value;
	Deposit(0, msg.sender, msg.value, tokens[0][msg.sender]);
}

function withdraw(uint amount) {
	if (msg.value>0) throw;
	if (tokens[0][msg.sender] < amount) throw;
	tokens[0][msg.sender] -= amount;
	if (!msg.sender.call.value(amount)()) throw;
	Withdraw(0, msg.sender, amount, tokens[0][msg.sender]);
}
</code></pre>

The vanilla `deposit` and `withdraw` functions are to be used for depositing and withdrawing Ether only. Note that the `withdraw` function does all state changes before sending Ether to the account owner, to avoid potential recursive or reentrant call bugs.

<pre><code>
function depositToken(address token, uint amount) {
	//remember to call Token(address).approve(this, amount) or this contract will not be able to do the transfer on your behalf.
	if (msg.value>0 || token==0) throw;
	if (!Token(token).transferFrom(msg.sender, this, amount)) throw;
	tokens[token][msg.sender] += amount;
	Deposit(token, msg.sender, amount, tokens[token][msg.sender]);
}

function withdrawToken(address token, uint amount) {
	if (msg.value>0 || token==0) throw;
	if (tokens[token][msg.sender] < amount) throw;
	tokens[token][msg.sender] -= amount;
	if (!Token(token).transfer(msg.sender, amount)) throw;
	Withdraw(token, msg.sender, amount, tokens[token][msg.sender]);
}
</code></pre>

The `depositToken` and `withdrawToken` functions are specifically for handling Ethereum tokens.

<pre><code>
function balanceOf(address token, address user) constant returns (uint) {
	return tokens[token][user];
}
</code></pre>

The `balanceOf` function is a helper function to get a user's balance for a particular token.

<pre><code>
function order(address tokenGet, uint amountGet, address tokenGive, uint amountGive, uint expires, uint nonce, uint8 v, bytes32 r, bytes32 s) {
	if (msg.value>0) throw;
	Order(tokenGet, amountGet, tokenGive, amountGive, expires, nonce, msg.sender, v, r, s);
}
</code></pre>

As we mentioned, resting orders are stored off-chain. In the event that the off-chain broadcasting mechanism fails, users can always store resting orders on chain in the event log by calling the `order` function.

Let's take this opportunity to go over the parameters that comprise an order:

 * `tokenGet` is the token you want to get and `tokenGive` is the token you want to give. For example, if you want to buy DAO with ETH, then `tokenGet` is the DAO address, and `tokenGive` is the ETH token address (0, since ETH is a special case token address).
 * `amountGet` and `amountGive` represent the size and price you want to trade. For example, if you want to buy 100 DAO with 1 ETH, then `amountGet` would be 100 DAO, and `amountGive` would be 1 ETH, which implies a price of 0.01 DAO/ETH or 100 ETH/DAO.
 * `expires` is the block number the order expires in. After this block number, the order can no longer trade.
 * `nonce` is a number you can include with your order to make it relatively unique. This way, if you want to place two otherwise identical orders, they won't have the same hash. This is useful since `orderFills` keeps track of order fills by order hash.
 * `v`, `r`, and `s` hold the signature for `sha256(tokenGet, amountGet, tokenGive, amountGive, expires, nonce)` as signed by `msg.sender`.

<pre><code>
function trade(address tokenGet, uint amountGet, address tokenGive, uint amountGive, uint expires, uint nonce, address user, uint8 v, bytes32 r, bytes32 s, uint amount) {
	//amount is in amountGet terms
	if (msg.value>0) throw;
	bytes32 hash = sha256(tokenGet, amountGet, tokenGive, amountGive, expires, nonce);
	if (!(
		ecrecover(hash,v,r,s) == user &&
		block.number <= expires &&
		orderFills[hash] + amount <= amountGet &&
		tokens[tokenGet][msg.sender] >= amount &&
		tokens[tokenGive][user] >= amountGive * amount / amountGet
	)) throw;
	tokens[tokenGet][msg.sender] -= amount;
	tokens[tokenGet][user] += amount * ((1 ether) - feeMake) / (1 ether);
	tokens[tokenGet][feeAccount] += amount * feeMake / (1 ether);
	tokens[tokenGive][user] -= amountGive * amount / amountGet;
	tokens[tokenGive][msg.sender] += ((1 ether) - feeTake) * amountGive * amount / amountGet / (1 ether);
	tokens[tokenGive][feeAccount] += feeTake * amountGive * amount / amountGet / (1 ether);
	orderFills[hash] += amount;
	Trade(tokenGet, amount, tokenGive, amountGive * amount / amountGet, user, msg.sender);
}
</code></pre>

The `trade` function is the biggest chunk of logic. This is the function you call when you see a resting order you like and you want to trade it. The parameters are the same as the order parameters we just covered, plus an `amount`, which is the amount of the order you want to trade (in `amountGet` terms). For example, if you see the order to buy 100 DAO with 1 ETH, and you want to sell 50 DAO for 0.5 ETH, you would use an `amount` of 50 DAO.

The first thing the `trade` function does is construct an order hash. Then it checks to make sure the signature provided matches the order hash, the order hasn't expired, this trade won't overfill the remaining volume associated with the order, and both users have the funds required to complete the order. If all these things are true, the function moves funds from one account to the other, moves some funds to the fee account, and updates the `orderFills` variable with the amount that has been filled.

<pre><code>
function availableVolume(address tokenGet, uint amountGet, address tokenGive, uint amountGive, uint expires, uint nonce, address user, uint8 v, bytes32 r, bytes32 s) constant returns(uint) {
	bytes32 hash = sha256(tokenGet, amountGet, tokenGive, amountGive, expires, nonce);
	if (!(
		ecrecover(hash,v,r,s) == user &&
		block.number <= expires
	)) return 0;
	uint available1 = amountGet - orderFills[hash];
	uint available2 = tokens[tokenGive][user] * amountGet / amountGive;
	if (available1<available2) return available1;
	return available2;
}
</code></pre>

The last function is `availableVolume`, which is a helper function for checking how much volume is available on an order, taking into account the amount that has been filled so far and the funds available in the user's account.

## Conclusion

Hopefully this high-level and low-level look at the EtherDelta smart contract was helpful to you. We have some exciting announcements around the corner, so stay tuned!

## Mailing list

If you want to receive updates from Etherboost, please sign up for our mailing list:

<!-- Begin MailChimp Signup Form -->
<link href="//cdn-images.mailchimp.com/embedcode/horizontal-slim-10_7.css" rel="stylesheet" type="text/css">
<style type="text/css">
	#mc_embed_signup{background:#fff; clear:left; font:14px Helvetica,Arial,sans-serif; width:100%;}
	/* Add your own MailChimp form style overrides in your site stylesheet or in this style block.
	   We recommend moving this block and the preceding CSS link to the HEAD of your HTML file. */
</style>
<div id="mc_embed_signup">
<form action="//etherboost.us13.list-manage.com/subscribe/post?u=6ad46e524f84c79c4d8475e91&amp;id=b7675dc8dd" method="post" id="mc-embedded-subscribe-form" name="mc-embedded-subscribe-form" class="validate" target="_blank" novalidate>
    <div id="mc_embed_signup_scroll">
	<label for="mce-EMAIL">Subscribe to our mailing list</label>
	<input type="email" value="" name="EMAIL" class="email" id="mce-EMAIL" placeholder="email address" required>
    <!-- real people should not fill this in and expect good things - do not remove this or risk form bot signups-->
    <div style="position: absolute; left: -5000px;" aria-hidden="true"><input type="text" name="b_6ad46e524f84c79c4d8475e91_b7675dc8dd" tabindex="-1" value=""></div>
    <div class="clear"><input type="submit" value="Subscribe" name="subscribe" id="mc-embedded-subscribe" class="button"></div>
    </div>
</form>
</div>

<!--End mc_embed_signup-->
