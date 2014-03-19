Smart-Market
============

Decentralized marketplace that automatically evaluates your level of trust for merchants and based on a web of trust.

Running Smart Market
====================

Documentation and Downloads will be here.

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

Design
======

As this project is built, I may find it is safer to implement certain features in different ways, therefore, the following design ideas are subject to change.

### Distributed Web of Trust

The distributed web of trust will have both private and public ratings. Private ratings only effect how your web of trust acts locally, while public ratings are broadcasted and used by others to determine trust.

The trust rating will be minimal, following the format:
{Rated user keyhash (20 bytes), Rating user key (32 bytes), Rating (1 byte), Signature(72 bytes)}

Each rating will be a little over 100bytes. At a rate of 8 ratings per kb, there can be "only" 8 million ratings per GB. This doesn't scale to the trade volume of major auctioning and sales websites. Completely distributed ratings (everyone having a full copy) are fine for V0.1, but eventually there will have to be a more scalable model that doesn't require everyone have a full copy.

There will be a default heuristic for getting a users trust based on your trust ratings, but it will be changeable. For example, some users might place a lot of trust in the ratings of those they trust, while others may think trust is less transitive. Users having different heuristics brings up issues with finding mutually agreeable mediators (you don't know who they trust), but there are many solutions. Assuming everyone has the same heuristic is sufficient for V0.1 though.

There are DoS problems with this (eg. a user rating everyone in the network or a bunch of faux users), but they will be covered in the Broadcasting section.

### Web of Trust Search


Based on listings in a category you are interested in, your local web of trust will determine the trust-rating of each merchant in the list you are viewing. Indexing your Web of Trust should take O(R) time, where R is the number of ratings. The time needed to actually rate will be determined by the heuristics algorithm.

### Mutually Agreed Upon Mediators

Find an agreed upon mediator, or set of mediators using the Web of Trust. Bitcoin scripts allow for you to choose N mediators and have the transaction spent to an address determined by either you and the merchant together (both parties are happy) or M of those N mediators along with either a buyer or merchant (whoever the M mediators side with).

For example, Bob could decide to sell me a TV. We might find 5 mediators we trust and require a 3/5ths vote. If Bob and I agree (he got paid and I got my TV) we both sign a transaction paying him. If we don't agree and he has a stronger case according to 3/5ths of the mediators, Bob and 3/5ths of the mediators will sign a transaction paying him. If I have a stronger case, 3/5th of the mediators and I will sign a transaction paying to myself and I will receive a refund.

### Listing Propogation

Unlike ratings, listings are not permenant. There probably will be significantly less listings than ratings. For V0.1 everyone will have every rating, but to scale, the same scheme used for the WoT will probably be used.

Listings themselves will not be propogated, only the title and information used to get the rest of the listing will be. This is to prevent DoS.

The listing metadata will be formatted in this way:
{Sellers ECDSA Key (32 bytes), Title (up to 64 bytes), Timestamp (8 bytes), Category (4 bytes)}

Based on the listings data, a listing can be requested from one of the hosts. Listings themselves can contain text and/or an image. To prevent DoS attacks against the hosts, they will request either a proof of work (hashcash) or proof of stake (proof of ownership of Bitcoins).

### Hosting

A host is someone who decides to host listings for the network (and after V0.2 possibly do static web hosting). They are paid for their services by sellers however, they are sellers themselves because they sell the hosting service.

A "proof of data transfer" is impossible without risking a sybil attack. This means it is up to the seller (who is actually a customer for the hosting seller) to audit them and notify the mediators if they are cheating.

### Private Messaging

In V0.1, a Bitmessage-like network (or Bitmessage itself) will be used for private messaging.

V0.2 may include the option to let users have PM accounts with hosters. The problems this brings up are difficulty auditing, problems with centralization and reduced anonymity. For these reasons Bitmessage may be the better solution.

### Broadcasting

Client Operation
================

(General Client Workings Here)
