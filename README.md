Smart-Market
============

Decentralized marketplace that automatically evaluates your level of trust for merchants based on a web of trust.

Running Smart Market
====================

Documentation and downloads will be here.

Motivation
==========

Bitcoins greatest accomplishment was removing trust from third parties and allowing people to hold and use money without a central authority. The goal of Smart Market is to remove the need for a central authority (eg. an auctioning website, or payment processor) along with having no down time and allow you to make "smart"-trades.

Features
========

Once the following features are complete, Smart Market will be on V0.1:
* Distributed web of trust
* Web of trust search to find high rated merchant
* Web of trust search to find mutually agreeable mediators
* Propogation of listing metadata
* Decentralized hosting of actual listings
* Private messaging

Use Cases
=========

* Marketplace with agreed upon mediators
* Distributed exchange

### Potential Future Use Cases
* Static hosting services
* Stock trading using some color coin or Freicoin
* Coinjoining network
* Anonymity network

Design
======

As this project is built, I may find it is safer to implement certain features in different ways, therefore, the following design ideas are subject to change.

### Distributed Web of Trust (WoT)
The distributed WoT will have both private and public ratings. Private ratings only effect how your web of trust acts locally, while public ratings are broadcasted and used by others to determine trust.

An example of a centralized WoT rating system can be found here: http://bitcoin-otc.com/viewratings.php

The trust ratings size will be minimal, following the format:
```
Rated user key (32 bytes), Rating user key (32 bytes), Rating (1 byte), Signature (72 bytes), Nonce (1-8+ bytes)
```
Each rating will be a little over 100bytes. At a rate of 7 ratings per kb, there can be "only" 7 million ratings per GB. This doesn't scale to the trade volume of major auctioning and sales websites. Completely distributed ratings (everyone having a full copy) are fine for V0.1, but eventually there will have to be a more scalable model that doesn't require every running client have a full copy.

There will be a default heuristic for getting a users trust based on your trust ratings, but it will be changeable. For example, some users might place a lot of trust in the ratings of those they trust, while others may think trust is less transitive. Users having different heuristics brings up issues with finding mutually agreeable mediators (you don't know who they trust), but there are many solutions. Assuming everyone has the same heuristic is sufficient for V0.1 though.

There are DoS problems with this (eg. a user rating everyone in the network or a bunch of faux users), but they will be covered in the Broadcasting section.

### Web of Trust Search
Based on listings in a category you are interested in, your downloaded WoT will determine the trust-rating of each merchant in the list you are viewing. The time needed to rate will be determined by the heuristics algorithm.

### Mutually Agreed Upon Mediators
Find an agreed upon mediator, or set of mediators using the WoT. Bitcoin scripts allow for you to choose N mediators and have the transaction spent to an address determined by either you and the merchant together (both parties are happy) or M of those N mediators along with either a buyer or merchant (whoever the M mediators side with).

For example, Bob could decide to sell me a TV. We might find 5 mediators we trust and require a 3/5ths vote. If Bob and I agree (he got paid and I got my TV) we both sign a transaction paying him. If we don't agree and he has a stronger case according to 3/5ths of the mediators, Bob and 3/5ths of the mediators will sign a transaction paying him. If I have a stronger case, 3/5th of the mediators and I will sign a transaction paying to myself and I will receive a refund.

### Listing Propogation
Unlike ratings, listings are not permenant. There probably more ratings than listings stored at a given time because of this. For V0.1 everyone will have every listing, but for scaling purposes, the same scheme used for ratings will probably be used.

Listings themselves will not be propogated, only the title and information used to get the rest of the listing will be. This is to prevent DoS.

The listing metadata will be formatted in this way:
```
Sellers ECDSA Key (32 bytes), Title (up to 64 bytes), Timestamp (8 bytes), Category (4 bytes), Nonce (1-8+ bytes)
```
Based on the listings data, a listing can be requested from one of the hosts. Listings themselves can contain text and/or an image. To prevent DoS attacks against the hosts, they will request either a proof of work (hashcash) or proof of stake (proof of ownership of Bitcoins).

### Hosting
A host is someone who decides to host listings for the network (and after V0.2 possibly do static web hosting). They are paid for their services by merchants, meaning the merchants are customers for hostings and the hosts are merchants who sell hosting.

A "proof of data transfer" is impossible without risking a sybil attack. This means it is up to the merchant to audit them and notify the mediators if they are cheating or not providing sufficient service.

For V0.1, broadcasters will send directly to those that request data. In a future version, the data will be routed through a circuit formed in the network.

### Private Messaging
In V0.1, a Bitmessage-like network (or Bitmessage itself) will be used for private messaging.

A future version may include the option to let users have PM accounts with hosters. The problems this brings up are difficulty auditing, problems with centralization and reduced anonymity. For these reasons Bitmessage may be the better solution.

### Broadcasting
To prevent DoS and malicious users flooding each users harddrive, there will have to be something throttling those spammers. Whenever a rating is made, is will have to have a proof of work associated with it. Another method of throttling spam ratings is not relaying ratings made by those you don't trust, however only the proof of work will be part of V0.1.

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

Joining: https://en.bitcoin.it/wiki/Satoshi_Client_Node_Discovery

Downloading/Communication: https://en.bitcoin.it/wiki/Network#Standard_relaying

### Working as a Buyer
Buyers are able to search for items by title and category.

```
list listItems(4bytes category, string search, file listings)
  list items = listing in listings where categoryByte is category and listing.title contains search
  for each item in items
    item.rating = findRating(item.sellerKey)
  return items
```

Categories are predefined by the user and the program. The program will come pre-configured with categories associated with codes (0x00000000=hosting, 0x00000001=mediating, 0x00000003=some-primary-category). The community will be able to define their own category codes and if they're used enough, the networking effect will bring a soft-consensus for what each category means. The soft consensus will include a template allowing compressed listings (eg. hosting template is upspeed byte, uptime byte, required proof of work, etc. This is most space effecient than a listing saying "My uptime is 95% and my upspeed is 1mb/s").

When requesting a listing, the host may require a (cheap) proof of work.

### Working as a Merchant
The merchants actions will be creating and modifying listings along with replying to messages.

Creating a listing will involve creating the listing structure with your public key, your title, the timestamp and the category byte. Then you complete a proof of work by finding a hash(listing structure, nonce) that is less than the network target. The hash is then stored for later revokation.

Modifying a listing involves revoking your previous listing with a revokation message:
```
previous listing hash (64 bytes), signature (72 bytes)
```
and adding a new listing to replace it.

#### Working as a host (who is a merchant for hostings)

A host will need to define parameters for the quality of their service. Each of these qualities must be auditable by mediators.

Hosting will be integrated with the programm meaning any legitemate listing requests will be fufilled automatically.

### Working as a Mediator

Mediators will have both manual and automatic tasks.

Automated tasks include running a script (eg. verifying a host isn't cheating) or another arbitrary script (after V0.1) when an event happens (it is a certain date, or a host auditing request is sent).

Manual tasks include acting as an oracle and answering questions like "did this person probably receive their product" or "did this event happen".

Questions and Stuff to Add
==========================

Is soft-distributing all these listings and ratings a good idea? Should they be contained in a blockchain instead? It would allow the network target to be better defined.

Must expand on ECDSA malleability issue for listing revokation and the solution.

Must expand on dealing with connecting to hosts.

Mention that each message (listing, rating, listing revokation) will have their type in the inv_vector as it is with the Bitcoin node communication.

What is a better name for this?
