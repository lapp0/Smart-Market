Smart-Market
============

Decentralized marketplace that automatically evaluates your level of trust for merchants based on a web of trust. Smart-Market has no central point of control, but it does have an agenda (facilitate trade), and protocol (defined below), therefore it can be considered a distributed autonomous corporation.

Motivation
==========

Bitcoins greatest accomplishment was removing trust from third parties and allowing people to hold and use money without a central authority. The goal of Smart Market is to remove the need for a central authority (eg. an auctioning website, or payment processor) when trading while having no down time and to allow users to make "smart"-trades.

Features
========

Once the following features are complete, Smart Market will be on V0.1:
* Distributed web of trust
* Web of trust search to find high rated merchant
* Web of trust search to find mutually agreeable mediators
* Propagation of listing metadata
* Decentralized hosting of actual listings
* Private Messaging (associated Bitmessage account, not integrated)

Use Cases
=========

* Marketplace with agreed upon mediators
* Distributed exchange

### Potential Future Use Cases
* Static hosting services
* Coinjoining network

Design
======

As this project is built, I may find it is safer to implement certain features in different ways, therefore, the following design ideas are subject to change.

### Distributed Web of Trust (WoT)
The distributed WoT will have both private and public ratings. Private ratings only effect how your web of trust acts locally, while public ratings are broadcasted and used by others to determine trust.

[An example of a centralized WoT rating system](http://bitcoin-otc.com/viewratings.php)

The trust ratings size will be minimal, following the format:
```
Rated user key (32 bytes), Rating user key (32 bytes), Rating (4 bytes),
Timestamp (8 bytes), Signature (72 bytes), Nonce (1-8+ bytes)
```

Rating structure:

| Rating Score | Amount Risked | Comment |
|--------------|---------------|---------|
| Value from -7 to 8 | represents many numbers from 0 to 10^15 satoshi | comment code clarifying what happened|
| 4 bits | 12 bits | 2 bytes |
| 0000=-7, 0001=-6... 1111=8 | Undefined implementation | 0x0000 = "smooth trade", 0x0001 = "ran off with money"...|

Each rating will be a little over 100bytes. At a rate of 7 ratings per kb, there can be "only" 7 million ratings per GB. This doesn't scale to the trade volume of major auctioning and sales websites. Completely distributed ratings (everyone having a full copy) are fine for V0.1, but eventually there will have to be a more scalable model that doesn't require every running client have a full copy.

There will be a default heuristic for getting a users trust based on your trust ratings, but it will be changeable. For example, some users might place a lot of trust in the ratings of those they trust, while others may think trust is less transitive. Users having different heuristics brings up issues with finding mutually agreeable mediators (you don't know who they trust), but there are many solutions. Assuming everyone has the same heuristic is sufficient for V0.1 though.

The default heuristic for V0.1 will be borrowed from bitcoin-otc. From each key, users will be able to determine first level trust (what they rated them), second level trust (what those they have first level trust for rated them) and so on. You only can have second level trust in Alice to the extent that your trust Bob (the person who rated Alice). For example, if your trust-level for Bob is 4 and his trust level trust for Alice is 7, then only 4 second level trust points will be added to Alices second level trust rating.

The format for a trust rating is defined to enable easy indexing. The first two fields, "Rated user key" and "Rating user key" allow your client to search the WoT for "Rating user"s that you have trust for and find the next level of trust from them. The rating bytes allow you to give a rating from -7 to 8 (first 4 bits) and a reason associated with some number between 0 and 2^12 - 1 that is hard coded in the client (eg. late delivery, perfect, many perfect transactions) as the next 12 bits. The timestamp is to make sure it is the most recent rating (next paragraph) and the signature is associated with the "Rating user key" and verifies that the rating was based by the rating users. Finally, the rating is hashed and if the hash of it is less than the target hash, it can be relayed by the network, otherwise the nonce is incremented and it is hashed again.

If a new rating comes along and it's first 64 bytes are already used in your WoT AND the timestamp is from a later time than the old rating (and the signature is valid and its hash is below the target), then it replaces the old rating in your WoT and is propagated. 

There are DoS problems with this (eg. a user rating everyone in the network or a bunch of faux users), but they will be covered in the Broadcasting section.

### Web of Trust Search
Based on listings in a category you are interested in, your downloaded WoT will determine your level one, level two ... level N (N defined by the user) trust for the merchant and display the listings based their trust-rating.

### Mutually Agreed Upon Mediators
Find an agreed upon mediator, or set of mediators using the WoT. Bitcoin scripts allow for you to choose N mediators and have the transaction spent to an address determined by either you and the merchant together (both parties are happy) or M of those N mediators along with either a buyer or merchant (whoever the M mediators side with).

For example, Bob could decide to sell me a TV. We might find 5 mediators we trust and require a 3/5ths vote. If Bob and I agree (he got paid and I got my TV) we both sign a transaction paying him. If we don't agree and he has a stronger case according to 3/5ths of the mediators, Bob and 3/5ths of the mediators will sign a transaction paying him. If I have a stronger case, 3/5th of the mediators and I will sign a transaction paying to myself and I will receive a refund.

### Listing Propagation
Unlike ratings, listings are not permanent. There probably will be more ratings than listings stored at a given time because of this. For V0.1 everyone will have every listing, but for scaling purposes, the same scaling scheme used for ratings will probably be used.

Listings themselves will not be propagated, only the title and information used to get the rest of the listing will be. This is to prevent DoS.

The listing metadata will be formatted in this way:
```
Sellers ECDSA Key (32 bytes), Title (up to 64 bytes), Timestamp (8 bytes),
Category (4 bytes), Signature (72 bytes), Nonce (1-8+ bytes)
```

The Sellers key will be used to lookup their trust-rating. The Title will be used to determine whether to ask the host for the listing information. The timestamp will be used to determine whether a listing has expired (listings must be re-added using a new PoW each week, but this will be done automatically by the client). Listing expiration prevents old listings from being up too long. The category byte is explained in the "Working as a Buyer" subsection of the "Client Operation" section below. The signature is used to verify that the listing belongs to the Seller in question and the nonce is, once again, used in the proof of work.

Based on the listings data, a listing can be requested from one of the hosts. Listings themselves can contain text and/or an image. To prevent DoS attacks against the hosts, they will request either a proof of work (hashcash) (V0.1) or proof of stake (proof of ownership of Bitcoins) (V0.2).

### Hosting
A host is someone who decides to host listings for the network (and in V0.2 possibly do static web hosting). They are paid for their services by merchants, meaning the merchants are customers for hosting and the hosts are merchants who sell hosting.

A "proof of data transfer" is impossible without risking a sybil attack. This means it is up to the merchant to verify the hosts are hosting them and notify the mediators if they are cheating or not providing sufficient service.

For V0.1, broadcasters will send directly to those that request data. In a future version, the data will be routed through a circuit formed in the network. The hosts will broadcast a connection information message which will be IP based for V0.1:
```
Hosts ECDSA KEY (32 bytes), IPv6 Address:Port (18 bytes), Timestamp (8 bytes), Signature (72 bytes), Nonce (1-8+ bytes)
```

The hosts will also broadcast a message with the listings associated with their ECDSA key so users can know to connect to them when they have a listing they want to view:
```
Hosts ECDSA KEY (32 bytes), Listing ID (?? bytes), Timestamp (8 bytes), Signature (72 bytes), Nonce (1-8+ bytes)
```

Like in the other objects, the timestamps are used for revoking and updating.

### Private Messaging
For V0.1, the client will simply give you someones Bitmessage address based on a public key you enter.

### Broadcasting
To prevent DoS and malicious users flooding each users hard drive, there will have to be something throttling those spammers. Whenever a rating is made, is will have to have a proof of work associated with it. Another method of throttling spam ratings is not relaying ratings made by those you don't trust, however only the proof of work will be part of V0.1.

The proof of work will be the same as Bitmessages (SHA512) for V0.1.

Client Operation
================

Upon opening the client, it will connect to the network, download any ratings and listings it has missed.

After connecting, the client is able to
* As a buyer, view and open listings
* As a merchant, update a listing you own, or add a new listing
* As a mediator, run any audits or settle any disputes

### Connection and Downloading
Because of the nature of the data in this network (everyone gets a copy and it is distributed), the network joining, topology and downloading will mostly copied from Bitcoin.

[Bitcoin joining](https://en.bitcoin.it/wiki/Satoshi_Client_Node_Discovery)

[Bitcoin downloading/Communication](https://en.bitcoin.it/wiki/Network#Standard_relaying)

Once you have joined the network, you will be able to broadcast objects (ratings, listings, listing revocations, etc) and relay objects.

### Working as a Buyer
Buyers are able to search for items by title and category.

```
list listItems(size_4 category, string search, file listings)
  list items = listing in listings where categoryByte is category and listing.title contains search
  for each item in items
    item.rating = findRating(item.sellerKey)
  return items
```

Categories are predefined by the user and the program. The program will come pre-configured with categories associated with codes (0x00000000=hosting, 0x00000001=mediating, 0x00000003=some-primary-category). The community will be able to define their own category codes and if they're used enough, the networking effect will bring a soft-consensus for what each category means. The soft consensus will include a template allowing compressed listings (eg. hosting template is upspeed byte, uptime byte, required proof of work, etc. This is more space efficient than a listing saying "My uptime is 95% and my upspeed is 1mb/s").

When requesting a listing, the host may require a (cheap) proof of work. The user will lookup the hosts that are associated with that listing, then they will lookup the current IP:Port of that host and request that specific listing.

### Working as a Merchant
The merchants actions will be creating and modifying listings along with replying to messages.

Creating a listing will involve creating the listing structure with your public key, your title, the timestamp and the category byte. Then you complete a proof of work by finding a hash(listing structure, nonce) that is less than the network target. The hash is then stored for later revocation.

Modifying a listing involves revoking your previous listing with a revocation message:
```
previous listing (109-120+ bytes), signature (72 bytes)
```
and adding a new listing to replace it.

#### Working as a host (being a merchant for hosting)

A host will need to define parameters for the quality of their service. Each of these qualities must be audit-able by mediators.

Hosting will be integrated with the program meaning any legitimate listing requests will be fulfilled automatically.

### Working as a Mediator

Mediators will have both manual and automatic tasks.

Automated tasks include running a script (eg. verifying a host isn't cheating) or another arbitrary script (after V0.1) when an event happens (it is a certain date, or a host auditing request is sent).

Manual tasks include acting as an oracle and answering questions like "did this person probably receive their product" or "did this event happen".

Along with that, they act as merchants and view requests for their services including working with contracts.

Planned Features for V0.2
=========================

Some features aren't needed for a complete marketplace, but are desired. These features must be implemented for this project to be considered on V0.2

### Proof of Stake (PoS)

Proof of work is not ideal because those running specialized hardware (or even GPUs) may be able to spam the network.

Along with solving the spamming problem, PoS may make it easy to prove your network is being jammed. I am researching this right now.

PoS will be the most important feature in V0.2 because it could solve the two most critical (known) vulnerabilites to the system: spamming using specialized hardware for generating SHA512 hashes and possibly a sybil attack to jam the network (these are the same vulnerabilities found in Bitmessage).

The proof of stake implementations details will be expanded here.

Proof of Stake requires users run a Bitcoin client (an SPV client works though).

### Private Messaging
By V0.2, an integrated PoS-based Bitmessage-like network will be used for private messaging.

A future version may include the option to let users have PM accounts with hosts. The problems this brings up are difficulty auditing, problems with centralization and reduced anonymity.

### Static Web Hosting

It is possible to have the mediators verify web hosting as well. The contract involves the mediator downloading the file at predetermined times and verifying that the hash of the file is the same as it was originally.

This is both useful and easy to integrate.

### Account Throw-Away Value (If necessary)
The WoT can determine how trustworthy a person has been in previous transaction, but if the merchants orders are valued higher than their WoT and future earnings and they aren't using a mediataor, then it is more profitable to scam their customers than continue operating.

To determine throw away value, a buyer must estimate the value of their WoT (estimated based on their trust-value) and subtract that from their throwaway value, which should be calculated by the client as the sum of all current open orders.

Letting users know your throwaway value will by opt-in because using this scheme (perhaps there is a better one such as a modified version of [gmaxwells proof of reserves](https://iwilcox.me.uk/2014/proving-bitcoin-reserves)) will let everyone know how many orders you have and the value of each.

If the merchant isn't using a mediator and is requiring you to send funds directly to them, then it is a good idea to check their [counterparty risk](https://en.wikipedia.org/wiki/Counterparty_risk#Counterparty_risk). If they have a significant amount of unpaid debt (incomplete sales), then a user may want to avoid them. Publicizing their throw-away value is the merchants decision. If they aren't publicizing their their throw-away value, then they should broadcast a message stating they aren't publicizing their throw-away value. If a buyer comes across a merchant that isn't agreeing to add your sale to their throw-away value and they haven't broadcasted a message saying they're not publicizing, then they may be hiding the fact that they have a large counterparty risk and they probably should be avoided.

Throw-away value messages are deleted from each users database if there is either rating for the merchant from after the sale time, or if the sale has expired. The throw-away messages will be formatted like:
```
Merchant Key (32 bytes), Buyer Key (32 bytes), Current Timestamp (8 bytes),
Expiration Timestamp (8 bytes), Risked BTC (7 bytes), Merchant Signature (72 bytes), Nonce (1-8+ bytes)
```

Not broadcasting throw-away value message:
```
Merchant Key (32 bytes), "Not Broadcasting" Byte (1 byte), Merchant Signature (72 bytes), Nonce (1-8+ bytes)
```

### To be continued

Possible Future Features
========================

These features may not be necessary or possible and will be marked as such.

### Message Splitting Scaling Scheme

As mentioned above, to scale, this program will need to split the messages. This must be done carefully because there are incentives for some users to jam the network.

### To be continued

Questions and Stuff to Add
==========================

Is soft-distributing all these listings and ratings a good idea? Should they be contained in a blockchain instead? It would allow the network difficulty target to be better defined.

What is a better name for this?

Possible way to make revocations more space efficient

Determine the most space-efficient and safe way to have listing IDs referenced in the hosts "advertisement".
