                      
       /     /   ▶ Hord   
      / --- /      Ordinal indexing engine based on Chainhook.
     /     /       Build indexes, standards and protocols on top of Ordinals and Inscriptions (BRC20, etc).
                  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[![Introduction](https://img.shields.io/badge/%23-%20Introduction%20-orange?labelColor=gray)](#Introduction)
&nbsp;&nbsp;&nbsp;&nbsp;[![Features](https://img.shields.io/badge/%23-Features-orange?labelColor=gray)](#Features)
&nbsp;&nbsp;&nbsp;&nbsp;[![Getting started](https://img.shields.io/badge/%23-Quick%20Start-orange?labelColor=gray)](#Quick-start)
&nbsp;&nbsp;&nbsp;&nbsp;[![Documentation](https://img.shields.io/badge/%23-Documentation-orange?labelColor=gray)](#Documentation)
&nbsp;&nbsp;&nbsp;&nbsp;[![Contribute](https://img.shields.io/badge/%23-Contribute-orange?labelColor=gray)](#Contribute)

***

# Introduction

The [Ordinal Theory](https://trustmachines.co/glossary/ordinal-theory) is a protocol aiming at attributing unique identifiers to minted satoshis (sats). With its numbering scheme, satoshis can be **inscribed** with arbitrary content (aka **inscriptions**), creating bitcoin-native digital artifacts more commonly known as NFTs. Inscriptions do not require a sidechain or separate token, which makes it attractive for new entrants to adopt, extend, and use. These inscribed sats can be transferred using Bitcoin transactions, sent to Bitcoin addresses, and held in Bitcoin UTXOs. In all respects, these transactions, addresses, and UTXOs are normal Bitcoin transactions, addresses, and UTXOs, except that to send individual sats, transactions must control the order and value of inputs and outputs per Ordinal Theory.

Now that we discussed Ordinal Theory, let's dive into what **hord** attempts to solve for developers.

The **hord** is an indexer designed to help developers build new re-org-resistant applications on top of the Ordinal Theory. This indexer will make it easy for protocol developers and users of those protocols to trace and discover the ownership of Ordinal's inscriptions, along with a wealth of information about each inscription.

The **hord** uses [Chainhook SDK](https://github.com/hirosystems/chainhook/tree/develop/components/chainhook-sdk) from the [Chainhook](https://github.com/hirosystems/chainhook/tree/develop) project, which is a re-org-aware transaction indexing engine for Stacks and Bitcoin. The SDK is designed with first-class event-driven principles, so it helps developers extract transactions from blocks efficiently and keeps a consistent view of the chain state.

With **hord**, Bitcoin developers can reliably implement feature-rich protocols and business models utilizing  _near-real-time_ Ordinals inscriptions and transfers events.

# Quick Start

## Installing `hord` from source

```console 
$ git clone https://github.com/hirosystems/hord.git
$ cd hord
$ cargo hord-install
```

## Getting started with `hord`

### Explore Ordinal activities in your terminal

Once `hord` is installed, Ordinals activities scanning can simply be performed using the following command:
```console
$ hord scan blocks 767430 767753 --mainnet
Inscription 6fb976ab49dcec017f1e201e84395983204ae1a7c2abf7ced0a85d692e442799i0 revealed at block #767430 (ordinal_number 1252201400444387, inscription_number 0)
Inscription 26482871f33f1051f450f2da9af275794c0b5f1c61ebf35e4467fb42c2813403i0 revealed at block #767753 (ordinal_number 727624168684699, inscription_number 1) 
```

In this command, an interval of blocks to scan (starting at block `767430`, ending at block `767753`) is being provided. `hord` will display inscriptions and transfers activities occurring in the range of the specified blocks. 

The activity for a given inscription can be retrieved using the following command:

```console
$ hord scan inscription 6fb976ab49dcec017f1e201e84395983204ae1a7c2abf7ced0a85d692e442799i0 --mainnet
Inscription 6fb976ab49dcec017f1e201e84395983204ae1a7c2abf7ced0a85d692e442799i0 revealed at block #767430 (ordinal_number 1252201400444387, inscription_number 0)
Transfered in transaction bc4c30829a9564c0d58e6287195622b53ced54a25711d1b86be7cd3a70ef61ed at block 785396
```

---
### Stream Ordinal activities to an indexer

`hord` is designed to help developers extracting ordinals activities (inscriptions and transfers) from the Bitcoin chain and streaming these activities to their indexer / web application.

In order to get started, a `bitcoind` instance with access to the RPC methods `getblockhash` and `getblock` must be running. The RPC calls latency will directly impact the speed of the scans.

*Note: the configuration of a `bitcoind` instance is out of scope for this guide.*

Assuming: 

`1)` a `bitcoind` node correctly configured and 

`2)` a local HTTP server running on port `3000` exposing a `POST /api/events` endpoint, 

A configuration file `Hord.toml` can be generated using the command:

```console
$ hord config new --mainnet
✔ Generated config file Hord.toml
```

After adjusting the `Hord.toml` settings to make them match the `bitcoind` configuration, the following command can be ran:  

```
$ hord scan blocks 767430 767753 --post-to=http://localhost:3000/api/events --config-path=./Hord.toml
```

`hord` will retrieve the full Ordinals activities (including the inscriptions content) and send all these informations to the `http://localhost:3000/api/events` HTTP POST endpoint. 

---
### Run `hord` as a service for streaming blocks

`hord` can be ran as a service for streaming and processing new blocks appended to the Bitcoin blockchain.

```console
$ hord service start --post-to=http://localhost:3000/api/events --config-path=./Hord.toml
```

New `http-post` endpoints can also be added dynamically by spinning up a redis server and adding the following section in the `Hord.toml` configuration file:

```toml
[http_api]
http_port = 20456
database_uri = "redis://localhost:6379/"
```

Running `hord` with the command

```console
$ hord service start --config-path=./Hord.toml
```

will spin up a HTTP API for managing events destinations.

A comprehensive OpenAPI specification explaining how to interact with this HTTP REST API can be found [here](https://github.com/hirosystems/chainhook/blob/develop/docs/chainhook-openapi.json).


