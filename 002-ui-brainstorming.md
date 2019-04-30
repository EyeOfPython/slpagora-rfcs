# Motivation

An interactive console application might have its retro charme, but for most users, and especially professional daytraders, a clear interface with all necessary data on-screen would be preferable.

# Constraints/Requirements

1. The UI should be serverless, if possible. The only thing the server does is hosting the necessary JS/HTML/CSS files (which might even be hosted on IPFS), but if possible, should not do any kind of processing. This way we avoid falling under any kind of KYC laws. It should also be easy to setup a mirror website doing the same thing, given the source.
2. Therefore, it makes most sense to have a single-page application that does all the requests to certain API endpoints (such as BitDB or SLP REST API).
3. The onboarding should be as smooth as possible, no signup required. The user should be prompted with a QR code as early as possible, maybe even right after visiting the page for the first time, or at least when selecting to accept an offer without having set up anything yet. 
4. The wallet should (probably) be stored encrypted in localStorage and once the user loads the site again after having created the wallet, he should be prompted for the password to decrypt it for this session. This would be a bit cumbersome, so I'm not sure about this one.
5. Funding and withdrawing from the wallet, creating and accepting offers should be simple, only a few clicks, but it should be difficult to accidentally accept an offer by misclicking etc.

# Features

1. List of all offers, ordered by timestamp (most recent first)
2. List of all tokens for which at least one offer exists, ordered by volume (most traded first), with current/last price
3. List of all offers for the selected token, ordered by price (lowest first)
4. Current BCH balance, current balance and price of the selected token or of the most owned tokens
5. Funding the wallet using a QR code or bitcoin URI. Both BCH and SLP tokens should be fundable this way.
6. Withdrawing from the wallet by specifying a "withdraw to" address and confirming.
7. Creating a new offer by selecting token ID of the wallet's tokens, the sell amount in <token> and the ask amount in BCH.
8. Accepting an offer by selecting the offer and confirming it.
