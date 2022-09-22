---
title: "Data Exchange"
description: "The Data Exchange Algorithm of IPFS"
draft: false
menu:
    curriculum:
        parent: "curriculum-ipfs"
weight: 130
category: lecture
level:
- deep
---

## IPFS Data
IPFS differs greatly in the way that it stores, shares, and retrieves files. Instead of having clients rely on servers, IPFS allows peers to connect and search for one another in an efficient manner to exchange data directly.

## Bitswap
_See the full set of resources [on the ResNetLab Tutorials page](https://research.protocol.ai/tutorials/resnetlab-on-tour)_

#### Content Exchange | ResNetLabs on Tour

{{< youtube jaGkTW2xacE >}}

#### Beyond Bitswap | ResNetLabs on Tour <!-- Presenter?-->

<!-- Add introduction here -->

{{< youtube cXl-tzX24VQ >}}

<!-- Add summarizing points -->

## Graphsync

#### Future of Decentralized Data Transfer | Hannah Howard

<!-- Need an intro paragraph -->

{{< youtube Qtt21TItPI4 >}}

<!-- Summarizing points -->

### Deep Dive into IPNS

IPNS is a self-certifying mutable pointer. Meaning any name that gets published is signed by a private key and anyone else can verify that it was signed by that peer with just the _name_. This self-certifying nature gives IPNS a number of super-powers not present in consensus systems (DNS, blockchain identifiers, etc.). Check out the [IPNS spec](https://github.com/ipfs/specs/blob/main/IPNS.md#ipns-record) to learn more about IPNS records.

<!-- Some notable ones include: mutable link information can come from anywhere, not just a particular service/system, and it is very fast and easy to confirm a link is authentic. IPNS names are encapsulated by a data structure called an _IPNS Record_ which contains the CID you are pointing to, the expiration date of a name and the sequence number which is incremented whenever the record is updated.  -->

#### How it Works

* **Publishing -** When you first publish an IPNS name, you create a brand new record containing the CID to point to, validity timestamp, and a cryptographic signature of the record created with the private key to establish you are the owner and certify the name.
<!-- This gets published to your local datastore, then the DHT.
Whenever you update a record, including to refresh its validity, you create a new record based on the previous, update the CID pointer if needed, increment the sequence number, then sign and publish it to the network. -->
* **Searching -** [Kubo](https://github.com/ipfs/kubo) uses the DHT to find peers that will have the queried record. This method has gotten faster over the years, but is still limited by the speed of DHT resolution itself. Alternative transports can be used to improve resolution speed (see [PubSub](#pubsub--ipns)).
* **Validity -** A record is valid for 24 hours by default, but you can change its `validity` to be longer. When someone has your record, but it is expired, they will have to go to DHT to find a new, valid version of your record.
<!-- Issues with validity don't arise if you're the original publisher of a record:  Kubo will periodically update validity and publish the updated IPNS records to the local record store and to DHT to keep it alive. DHT nodes will drop stored values after 24 hours, no matter what the validity. For this reason, for the IPNS name to be resolvable, the record needs to be continuously republished. -->
* **Keys -** A _name_ is a hash of a public key in a key pair. By default, the first public key used by Kubo is the same as the one for identifying your peer ([PeerID](https://docs.ipfs.tech/concepts/glossary/#peer-id)). You can [generate new key pairs with Kubo](https://docs.ipfs.tech/reference/kubo/cli/#ipfs-key-gen) and use them to create additional IPNS records.

<!--  
#### Common Pitfalls
* Resolving IPNS over DHT may be slow. This is due to having the `sequence` number in a record. If there are multiple versions of a name, then you want to make sure that you have the latest version. And you do this by performing a time-bound search through the DHT for peers that hold your records and checking a quorum for  the latest one. Kubo will spend up to a minute to try to find at least 16 peers to form a quorum.
* Users trying to use IPNS from [JavaScript in web browsers](https://github.com/ipfs/js-ipfs/blob/master/docs/BROWSERS.md) run into a variety of issues here related to how web browsers are isolated from the p2p network due to transport restrictions. To get around this problem, you would delegate the work out to a local or remote node. Another workaround would be using [public gateways](https://docs.ipfs.tech/concepts/ipfs-gateway/#public-gateways) for resolving [IPNS records and IPFS CIDs](https://protocol-labs.gitbook.io/launchpad-curriculum/launchpad-learning-resources/ipfs/ipfs-gateways#path-gateway).
* Only the holder of the corresponding private key can update an IPNS record, e.g. to extend its validity. Regardless, any peer can act as a host and provide the record to the network if that record is valid.
* When users share private keys and publish from multiple nodes, the issue of ["sequence number collision"](https://discuss.ipfs.tech/t/ipns-beyond-the-basics-no-ipns-pinning-service-any-docs-on-this/13424/2) would occur (among various others). This would be when two people updating a shared record have the same `sequence` number, but different CIDs. How would a receiver of said record know which one is correct?
* Safe Vs. Old problem. This is a question around practicality on IPNS content. If I accidentally let my record expire, do I want my users to fetch and resolve the data anyways (aka serve **old** data) or not at all (**safe**). Currently, the default is to be safe, and not serve any data. This is an ongoing and situational conversation, you can [read about it on GitHub](https://github.com/ipfs/kubo/issues/1958#issuecomment-444201606)

Sources: [Overview of IPNS by Adin](https://pl-strflt.notion.site/IPNS-Overview-and-FAQ-071b9b14f12045ea842a7d51cfb47dff#0963fe6b470a4c55b1929146c360dc95), [IPNS Spec](https://github.com/ipfs/specs/blob/main/IPNS.md), [Discuss forum for IPFS](https://discuss.ipfs.tech/t/how-do-i-make-my-ipns-records-live-longer/14768/17?u=lidel)
-->

### DNSLink

[DNSLink](https://docs.ipfs.tech/concepts/glossary/#dnslink) is a standard for a mutable pointer system that stores its links in DNS TXT records corresponding to a given domain name. With DNSLink you can use a mutable pointer like a DNS domain name to point to content-addressed data in IPFS.

The benefits of DNSLink include:
- Human-readable mutable pointers like `/ipns/libp2p.io`.
- DNSLink leverages the distributed architecture of DNS to enable internet-scale mutable names or pointers which interoperate with IPFS.
- In many cases, resolving a DNSLink pointer is faster than IPNS.

To create a mutable pointer with DNSLink, you need:
- a DNS domain name you control
- A CID or an IPNS name to link to

> Note that while using DNSLink to link to an IPNS name is possible, it's often counterintuitive, because you are using a mutable pointer (DNS record) to point to another mutable pointer (IPNS name).

<!--  
For example, `libp2p.io` has a TXT DNS record `_dnslink.libp2p.io` with the value `dnslink=/ipfs/bafybeibwlrl7olq5sggibzucp5s6y5n22numfpyjlzntnuvgs5zt2umjuu`. This DNS record can be _mutated_ at any point to point to a new CID.

Finally, there are several common ways to resolve DNSLink names:
- With an IPFS gateway using the `/ipns/[DNSLink]` resolution scheme, e.g. https://ipfs.io/ipns/libp2p.io or https://cloudflare-ipfs.com/ipns/libp2p.io. Note that the `/ipns` is misleading – and should probably be prefixed with `/dnslink`, given that you are not resolving an IPNS name.
- Directly via the domain, but this requires some additional DNS configuration as explained in the [DNSLink docs](https://dnslink.dev/#example-ipfs-gateway).

-->

**You can follow tutorials and read more at [https://dnslink.dev](https://dnslink.dev/)**


#### Quick explanation of dnslink in IPFS

{{< youtube YxKZFeDvcBs >}}

Source: [Introduction to DNSlink](https://dnslink.dev/#introduction), [IPFS docs on DNSLink](https://docs.ipfs.tech/concepts/dnslink/)
