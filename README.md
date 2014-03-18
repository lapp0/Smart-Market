Smart-Market
============

Decentralized marketplace that automatically ranks potential-sellers and mediators based on a public and audit-able web of trust.

Compiled Downloads
==================

Will be here.


Motivation
==========

Bitcoins greatest accomplishment was removing trust from third parties and allowing people to exchange money without a central authority. The goal of Smart Market is to, again, remove the trust you need in a central authority (eg. an auctioning website, or payment processor), have no down time and allow you to make "smart"-trades.

Features
========

Once the following features are complete, Smart Market will be on V0.1:
* Distributed web of trust
* Web of trust search to find high rated sellers
* Web of trust search to find mutually agreeable mediators
* Propogation of listing metadata
* (De)centralized hosting of actual listings
* Private messaging

Design
======

As this project grows, I may find it is safer to implement certain features in different ways, therefore, the following design ideas are subject to change.

Distributed Web of Trust
------------------------

The distributed web of trust will have both private and public ratings. Private ratings only effect locally effect how your web of trust acts locally, while public ratings are broadcasted and used by others to determine trust.

The trust rating will be minimal, following the format:
{Rated user keyhash (20 bytes), Rating user key (32 bytes), Rating (1 byte), Signature(72 bytes)}

Each rating will be a little over 100bytes. At a rate of 8 ratings per kb, there can be "only" 8 million ratings per GB. This doesn't scale to the trade volume of major auctioning and sales websites. Completely distributed ratings are fine for V0.1, but eventually there will have to be a more scalable model that will be similar to Bitcoins SPV clients.

There will be a default heuristic for getting a users trust based on your trust ratings, but it will be changeable. For example, some users might place a lot of trust in the ratings of those they trust, while others may think trust is less transitive. Users having different heuristics brings up issues with finding mutually agreeable mediators (you don't know who they trust), but there are many solutions. Assuming everyone has the same heuristic is sufficient for V0.1 though.

There are DoS problems with this (eg. a user rating everyone in the network or a bunch of faux users), but they will be covered in the Broadcasting section.

Web of Trust Search
-------------------

Based on listings in a category you are interested in, your local web of trust will determine the trust-rating of each seller in the list you are viewing. Once your Web of Trust is indexed, this should take O(log n) time.

Mutually Agreed Upon Mediators
------------------------------

Find an agreed upon mediator, or set of mediators using the Web of Trust. Bitcoin scripts allow for you to choose N mediators and have the transaction spent to an address determined by either you and the seller together (both parties are happy) or M of those N mediators along with either a buyer or seller (whoever the M mediators side with).

For example, Bob could decide to sell me a TV. We might find 5 mediators we trust and require a 3/5ths vote. If Bob and I agree (he got paid and I got my TV) we both sign a transaction paying him. If we don't agree and he has a stronger case according to 3/5ths of the mediators, 3/5ths of the mediators and he will sign a transaction paying him. If I have a stronger case, the mediators will do the same and I will receive a refund.