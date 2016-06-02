---
---
We are proud to announce a new [Etheropt](http://etheropt.github.io) feature that will let anyone market make from the browser. Previously, the only way to market make was by running a small server on your computer (read more about that [here](http://etherboost.github.io/blog/etheropt-1.0-features.html)). If you'd like to run that small server, you'll be contributing by helping to maintain the decentralized off-chain order book. But if that's too much for you to deal with, you can get involved by market making from the browser. We're eager to get more people involved with providing liquidity to the market. Set your own prices and capture profits!

## How to get started

First, pick an expiration you want to start market making. Then click the new "Market make" button:

<img src="/images/screenshot4.png" />

A dialog will pop up that looks like this:

<img src="/images/screenshot5.png" />

Click on the "Generate PDF" link.

## Generating a PDF

What is a PDF? If you remember from probability and statistics in high school or college, a PDF is a probability density function. Basically, the point is for you to draw a graphical representation of the probability distribution of where the ETH/USD price will be at expiration. Move your mouse **slowly** from left to right across the canvas. If you mess up, just press "Clear" and start over.

Here's an example of a complete PDF:

<img src="/images/screenshot6.png" />

Beneath the PDF canvas, you'll see a textbox labelled "PDF" and a list of option prices that have been calculated using this PDF. If you're already intimately familiar with option pricing, you might be screaming "what about volatility and Black Scholes!?" We designed the interface so that it would be intuitive for beginners to use. If you want to think in terms of volatilities and Black Scholes, you'll have to use the [market maker server](https://github.com/etheropt/etheropt.github.io/blob/master/market_maker.js).

If you're happy with your PDF and the prices it generates, copy the text in the textbox labelled "PDF" and go back to the main Etheropt page.

## Start market making

Paste the PDF into the dialog box:

<img src="/images/screenshot7.png" />

You'll need to specify a size and a width. For example, if your width is 0.1 and your size is 10, and the price for an option calculated by the PDF is 1.0, then your market will be 0.95 (10 eth) bid, 1.05 (10 eth) offer. Press "Start" to start market making. Once you hit start, you can click on the "My orders" tab and you should see your orders start flowing through.

## Advanced order types

We also added some advanced order types that you can find under "Advanced options" in the order dialog. "Good if ETH/USD above/below" will only work the order if ETH/USD is at or above/below the levels you specify. "Delta" and "Tie" let you modify the price as ETH/USD moves. The formula is this: price_to_work = price + delta * (eth_usd - tie).

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
