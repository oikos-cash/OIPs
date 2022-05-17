---
Oip: 4
network: BNB Chain
title: oUSD Shorts
status: Draft
author: Manuel Corona (@triloger)
discussions-to: https://discord.com/invite/VVDu6Er
created: 2022-04-16
updated: N/A
type: Governance
---

## Simple Summary

<!--"If you can't explain it simply, you don't understand it well enough." Simply describe the outcome the proposed changes intends to achieve. This should be non-technical and accessible to a casual community member.-->

Allow users to short synthetic assets against oUSD.

## Abstract

<!--A short (~200 word) description of the proposed change, the abstract should clearly describe the proposed change. This is what *will* be done if the OIP is implemented, not *why* it should be done or *how* it will be done. If the OIP proposes deploying a new contract, write, "we propose to deploy a new contract that will do x".-->

In traditional finance, assets can be borrowed and sold short to gain inverse price exposure. The borrower is legally obligated to repurchase the asset in the future and return it. With the introduction of fixed debt loans in Oikos, we provide a shorting mechanism which relies on overcollateralization in the absence of legal enforcement. By depositing oUSD collateral, users can borrow and short sell synthetic assets (synths). To retrieve their collateral, they must repurchase the borrowed synth and return it.

This mechanism provides an alternative to inverse Synths for users seeking inverse price exposure. While these positions require collateral, they return the oUSD proceeds of the sale to the shorter, which can then be deployed productively throughout DEFI.

## Motivation

<!--This is the problem statement. This is the *why* of the OIP. It should clearly explain *why* the current state of the protocol is inadequate.  It is critical that you explain *why* the change is needed, if the OIP proposes changing how something is calculated, you must address *why* the current calculation is inaccurate or wrong. This is not the place to describe how the OIP will address the issue!-->

The current inverse Synth mechanism provides inverse price exposure but has limitations.

- Purging and resetting inverse Synths involves a significant amount of manual labour and owner risk. It also requires token holders to reset their positions and new rewards contracts to be deployed.
- The cost of buying a unit decreases as the price approaches the upper limit, which can be abused by front runners.
- The inverse Synth token cannot be used productively outside of the system.

## Specification

<!--The specification should describe the syntax and semantics of any new feature, there are five sections
1. Overview
2. Rationale
3. Technical Specification
4. Test Cases
5. Configurable Values
-->

### Overview

Shorts are fixed debt, over-collateralised loans that are structurally identical to those used in our multicollateral loan system. To open a short, the user must deposit oUSD collateral. They choose the synth they wish to short and the size of the position, subject to collateralisation requirements. However, instead of issuing the shorted currency, the contract converts the value to oUSD and issues that to the user, which represents the act of selling the asset short. A simple example illustrates this process.

1. User deposits 10000 oUSD and elects to short 10 oBNB.
2. A loan is created for the user for 10 oBNB.
3. The user is issued 5000 oUSD.
4. To reclaim the 10000 oUSD collateral, they must return 10 oBNB to the contract.

As mentioned, this position is represented internally as a loan, denominated in the currency that was short sold. In this example, the following loan would be stored.

| Symbol    | Description                  | Example    |
| --------- | ---------------------------- | ---------- |
| \\(c\\)   | Collateral locked            | 10000 oUSD |
| \\(p_c\\) | USD price of the collateral  | $1         |
| \\(s\\)   | Synth borrowed               | 10 oBNB    |
| \\(p_s\\) | USD price of the synth       | $500       |
| \\(I\\)   | Interest accrued on the loan | 0.01 oBNB  |

In this case, 5000 oUSD was issued to the shorter. Generally, quantity of oUSD issued is:

\\(q \ := s p_s\\)

As with any loan, the collateralisation ratio of the short position can be determined by:

\\( r \ := \frac{p_c \ c}{p_s \ (s + I)}\\)

A separate contract which accepts oUSD as collateral will be responsible for the issuance and management of all short positions. This contract supports the same pattern of interactions as described in [OIP 97](https://OIPs.oikos.io/OIPs/OIP-97).

#### Debt Pool and Interest

An oUSD short is a directional bet on the price of a synth. Similar to iSynths, the payoff for the position varies inversely with the price of the synth.

For any given synth, the ratio between its supply (long) and the supply of its corresponding iSynth (short) determines the skew. If the skew is 0, price fluctuations will not effect the debt pool as any increase in debt from one side will be offset by a corresponding decrease on the other side. From the perspective of the debt pool, this is an optimal position as it minimises risk.

To encourage balance, a positive interest rate will be charged on synths where there is greater short exposure than long exposure. This interest rate will increase in proportion with the skew. On the other hand, when long exposure exceeds short, the interest rate will be 0. Additionally, we may propose to incentivise taking positions that reduce skew through rewards programs, which are currently active for certain iSynths.

To define the interest rate, we modify some definitions from <!--[OIP 80](https://OIPs.oikos.io/OIPs/OIP-80). -->

| Symbol | Description | Definition | Notes |
| \\(Q_L\\) | Long supply | - | The total supply of the synth. |
| \\(Q_S\\) | Short supply | - | The total short supply of the synth. |
| \\(K\\) | Synth skew | \\(K \ := \ Q_S - Q_L\\) | The excess supply on one side or the other. When the skew is positive, shorts outweigh longs; when it is negative, longs outweigh shorts. When \\(K = 0\\), the synth is perfectly balanced. |
| \\(W\\) | Proportional skew | \\[W \ := \ \frac{K}{Q_L + Q_S}\\] | The skew as a fraction of the total amount of long and short. |
| \\(b\\) | Base short rate | | A configurable parameter that reflects the cost of borrowing. |
| \\(i\\) | Instantaneous interest rate | \\[i \ := \ max(W + b, 0) \\] | When the market is long skewed, it will be free to short. |

Interest accrues continuously, and is computed using the accumulated funding method described in <!--[OIP 80](https://OIPs.oikos.io/OIPs/OIP-80). -->

#### Liquidations

Shorts will be eligible for liquidation if their collateralisation ratio falls below the minimum required by the contract. Liquidations will be incentivised via a penalty which is paid out of the users collateral. The liquidation mechanism is the same as described in <!--[OIP 15](https://OIPs.oikos.cash/OIPs/OIP-15).-->

### Rationale

<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

### Technical Specification

<!--The technical specification should outline the public API of the changes proposed. That is, changes to any of the interfaces Synthetix currently exposes or the creations of new ones.-->

A new contract `CollateralShort.sol` which inherits from `CollateralErc20.sol` and implements the following API.

```solidity

function open(uint collateral, uint amount, bytes32 currency) external;

function close(uint id) external;

function deposit(address borrower, uint id, uint collateral) external;

function withdraw(uint id, uint amount) external;

function repay(address borrower, uint id, uint amount) external;

function draw(uint id, uint amount) external;

function liquidate(address borrower, uint id, uint amount) external;

```

### Test Cases

Test cases included with the implementation.

### Configurable Values (Via OCCP)

For the oUSD short contract, the following values must be set.

- `synths` the synths which can be shorted.
- `minCratio` the minimum collateralisation ratio before becoming eligible for liquidation.
- `minCollateral` the minimum collateral required to open a position.
- `issueFeeRate` the fee for opening a short.

#### Proposed Initial Values

The following values are proposed as the initial configuration.

`CollateralShort.sol`

- `synths` oBTC, oBNB
- `minCratio` 150%
- `minCollateral` 500
- `issueFeeRate` 0

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).