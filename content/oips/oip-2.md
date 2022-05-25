---
oip: 2
network: Binance Smart Chain
title: Staking on Polygon (MATIC)
status: Draft
author: Albert R.
created: 'March 23, 2022'
type: Governance
---

## Simple Summary
The ability to stake OKS tokens on the Polygon network before transitioning to a fully multi-chain presence.

## Abstract
We propose the launch of a Polygon <-> Binance Smart Chain portal leveraging the ChainPort technology. Once bridged to the Polygon network, OKS tokens will be stakeable on a smart contract and will earn OKS rewards disbursed by the Oikos Treasury.  

## Motivation
Engage users in multi-chain activities before the upcoming upgrade and reward those who participate with extra rewards. 

## Specification
## Overview
ChainPort is a easy to use, trustless bridge with acceptable levels of security. Oikos leverages this technology to expand our tokens' staking abilities to the Polygon network. The process consists of several steps which will be outlined in this OIP.
We would like to note that this is a transitionary step toward a full multi-chain presence and as such it will be subject to frequent changes.  


## ChainPort
The user interface offered at https://app.chainport.io allows users to bridge their OKS to Polygon in a few steps. The process follows the classic ERC20 approval and transfer flow, at the end of which OKS on BSC are locked into a smart contract and an equivalent amount is minted on the [Polygon side](https://polygonscan.com/address/0xAA59eec0E39505104c6Ada6fEC944aef04E1E759).

<br /><img src="https://raw.githubusercontent.com/oikos-cash/assets/master/media/wall.png" width="350"><br /><i>ChainPort user interface</i><br />

## Staking
Users will be able to stake their OKS on Polygon via the upcoming multi-chain portal. Rewards will be distributed weekly and escrowed for one year.
The exact amount of rewards distributed will be decided in a future Treasury Council call. 

## Implementation
A preliminary implementation is in the works. This is a live document. We will update all information as they become available.
