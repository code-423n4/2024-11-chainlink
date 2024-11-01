

# Chainlink audit details
- Total Prize Pool: $235,000 in USDC
  - HM awards: $167,500 in USDC
  - QA awards: $7,000 in USDC
  - Judge awards: $15,000 in USDC
  - Validator awards: $10,000 in USDC 
  - Scout awards: $500 USDC
  - Mitigation Review: $35,000 in USDC (*Details to be confirmed.*)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts November 1, 2024 20:00 UTC
- Ends November 25, 2024 20:00 UTC

## Automated Findings / Publicly Known Issues

_Note for C4 wardens: Anything included in this `Automated Findings / Publicly Known Issues` section is considered a publicly known issue and is ineligible for awards._

The 4naly3er report can be found [here](https://github.com/code-423n4/2024-11-chainlink/blob/main/4naly3er-report.md).

### Billing

- Billing is not perfectly exact, as it doesn't have to be.
  - Token and gas prices do not happen as regularly as normal Chainlink data feeds.
  - A lot of billing logic is fuzzy, and we tend to slightly overcharge, e.g. we assume all bytes in the data payload are non-zero.
  - This fuzzy billing with a slight overcharge means we don't have to be perfect down to the single wei like some exchange contracts would have to be.
  - This means findings about insignificant effects on the fee charged, e.g. rounding errors, might not be marked valid.
- Billing is meant to cover the commit and execution cost of a single, not-batched message. 
  - If multiple messages are batched into a single merkle root and/or execution transaction the protocol will make some profit. 
  - v1.6 has focussed on making batching more likely by only having a single offRamp per chain, enabling multi-source batching.


#### Token issuer is malicious
NOTE: Token pools are out of scope for this audit. Their effects on the on/offRamps are not. 

* Any malicious token issuer can negatively impact transactions that contain their token.
  * Pools can siphon gas from users that send tokens belonging to that pool.
* Any malicious token issuer can make transactions that contain their token fail forever, leading to the loss of any funds in that transaction.
  * Users should be cautious about sending tokens they do not trust.
  * Sending a single token per transaction will prevent any contagion to other tokens.
  * On some chains, there are methods through which a malicious token issuer can cause an entire transaction to revert without being caught by the try-catch, such as overflowing ZK proof size on EVM-compatible zkRollups. In such edge cases, a malicious token issuer can block following transactions from the same sender at the destination when not using out-of-order execution, potentially causing unbounded loss.
* Anyone can list any token with any name or affiliation. The CCIP Owner has zero control over what is listed and cannot modify or remove any tokens. This means there could be tokens listed with names or affiliations that are not endorsed by the CCIP Owner or Chainlink Labs.
* Malicious tokens or token pools will be able to re-enter.
  * This should not have any negative impact on the CCIP system.
  * This does allow them to re-order some events, but never to double spend.

#### Token issuer responsibility

* When a token issuer misconfigures their own tokens/pools, users could potentially not receive these tokens.
  * The CCIP Owner has no special access to resolve misconfigurations, only the token issuer can do so.
* Token issuers can deploy token pools without rate limits, which is different from today where every token pool is rate-limited.

#### Compatibility

* Some tokens will not work.
  * For example, fee-on-transfer tokens won't work because we require multiple hops in the CCIP contracts.
  * They will still be able to register their token, as this process is fully permissionless. This could negatively impact transactions containing these tokens.
  * Some issuers may choose to exempt their token pool from the fee-on-transfer logic, which could make fee-on-transfer tokens functional within CCIP depending on their exact contract logic.
* Not every token will be able to permissionlessly register.
  * There will be tokens that do not expose the `owner` and don't have the `getCCIPAdmin` function.
    * These can still be onboarded in the same way all current tokens are onboarded: manually through the CCIP Owner.
  * A pools `releaseOrMint` function is, by default, bound by a `defaultTokenDestGasOverhead`. As a result, pools or tokens that require more gas for this action will have to have a custom gas override set. This cannot be done in a self-serve way.

#### Other

* CCIP Owner is a trusted role.
  * The multi-signature contract structure includes the entire DON and therefore inherits the security of the entire DON.
* Solidity 0.8.24 is used and CCIP will be deployed on various chains that don't support the newer Solidity features.
  * We explicitly compile with the Paris hard fork to ensure compatibility.
* The aggregate rate limiter only works for tokens that have prices, as it's denominated in USD.
  * Self-serve tokens will not have prices, as it would be impractical to acquire accurate price sources for any arbitrary token, and writing these prices onchain would be economically unsustainable, and easily exploitable.
  * This means that tokens default to **not** be included in the aggregate rate limiter.
* Router API `getSupportedTokens` is now deprecated, calling it will revert.
  * There is no longer an official API that returns all supported tokens for a given destination in 1 call.
  * One should iterate through tokens via `getAllConfiguredTokens` in TokenAdminRegistry and then call `isSupportedChain` on its token pool.
* Partial token price updates may block other price updates
  * When this happens, the offchain code will re-submit prices for the price updates that have become stale.
* Potential execution of ordered messages out of order through reentrancy pattern
  * When a messages with nonces `n` and `n+1` are UNTOUCHED but the execution DON has not executed them for the entire `permissionLessExecutionThresholdSeconds`, they become executable.
  * When reentrancy is in the execution of `n` to execute `n+1`, the ExecutionStateChanged event will be emitted in the order `n+1` and then `n`. 
* When setting `permissionLessExecutionThresholdSeconds` there is an increased risk of MEV extraction.
* Stale prices can be used from the FeeQuoter in some circumstances, as clearly defined in the code.
* There's increased risk of rounding errors when working with tokens with >36 decimals in the FeeQuoter.

* **Please note: where the docs and code conflict, the code should be taken as source of truth.**
# Overview

Chainlink CCIP is a cross-chain messaging protocol with native support for token transfers and security in depth. You can reference our [public docs](https://docs.chain.link/ccip/) for more background information.

We are asking you to audit our v1.5 => v1.6 upgrade.

The main goal of CCIP v1.6 is to address the current limitations with regards to scalability by moving from a single on/off ramp per lane, to a single on/off ramp per chain, shared across lanes and chain families (sometimes referred to as a multi-lane architecture). By making the contracts non lane specific in v1.6, there are no more contracts to be deployed for new unidirectional lane, only configs must be set.

Another goal is to convert CCIP v1.6 contracts to be chain family agnostic (i.e. not tied to EVM), enabling bridging across different chain families (ex EVM <>=> Solana).

## Links

- **Previous audits:**  None made public
- **Documentation:** https://docs.chain.link/ccip
  * Please see the [Documentation File](https://github.com/code-423n4/2024-11-chainlink/blob/main/CCIP%201.6%20Documentation.pdf)
- **Website:** https://chain.link/
- **X/Twitter:** https://twitter.com/chainlink
- **Discord:** https://chain.link/discord

---

# Scope

The contracts in scope for this audit are both the ramps and all of their helper contracts. This means token pools are explicitly out of scope; they have not changed since the latest 1.5 audit.

 * _[scope.txt](https://github.com/code-423n4/2024-11-chainlink/blob/main/scope.txt)_
### Files in scope
| Contract                                                                                                                                             |   SLOC   | Purpose                         |
| ---------------------------------------------------------------------------------------------------------------------------------------------------- |:--------:|:------------------------------- |
| [contracts/src/ccip/FeeQuoter.sol](https://github.com/code-423n4/2024-11-chainlink/blob/main/src/ccip/FeeQuoter.sol)                                           |   527    | Contains all fee logic          |
| [contracts/src/ccip/MultiAggregateRateLimiter.sol](https://github.com/code-423n4/2024-11-chainlink/blob/main/src/ccip/MultiAggregateRateLimiter.sol)           |   155    | Rate limiter                    |
| [contracts/src/ccip/NonceManager.sol](https://github.com/code-423n4/2024-11-chainlink/blob/main/src/ccip/NonceManager.sol)                                     |    83    | Handling nonces                 |
| [contracts/src/ccip/capability/CCIPHome.sol](https://github.com/code-423n4/2024-11-chainlink/blob/main/src/ccip/capability/CCIPHome.sol)                       |   310    | Home chain OCR config contract  |
| [contracts/src/ccip/interfaces/IFeeQuoter.sol](https://github.com/code-423n4/2024-11-chainlink/blob/main/src/ccip/interfaces/IFeeQuoter.sol)                   |    6    | FeeQuoter interface             |
| [contracts/src/ccip/interfaces/IMessageInterceptor.sol](https://github.com/code-423n4/2024-11-chainlink/blob/main/src/ccip/interfaces/IMessageInterceptor.sol) |    5     | MessageInterceptor interface    |
| [contracts/src/ccip/interfaces/INonceManager.sol](https://github.com/code-423n4/2024-11-chainlink/blob/main/src/ccip/interfaces/INonceManager.sol)             |    3     | NonceManager interface          |
| [contracts/src/ccip/interfaces/IRMN.sol](https://github.com/code-423n4/2024-11-chainlink/blob/main/src/ccip/interfaces/IRMN.sol)                               |    7    | RMN interface                   |
| [contracts/src/ccip/interfaces/IRMNRemote.sol](https://github.com/code-423n4/2024-11-chainlink/blob/main/src/ccip/interfaces/IRMNRemote.sol)                   |    8    | RMNRemote interface             |
| [contracts/src/ccip/libraries/Client.sol](https://github.com/code-423n4/2024-11-chainlink/blob/main/src/ccip/libraries/Client.sol)                             |    36    | Customer facing library         |
| [contracts/src/ccip/libraries/Internal.sol](https://github.com/code-423n4/2024-11-chainlink/blob/main/src/ccip/libraries/Internal.sol)                         |   150    | Internal library                |
| [contracts/src/ccip/ocr/MultiOCR3Base.sol](https://github.com/code-423n4/2024-11-chainlink/blob/main/src/ccip/ocr/MultiOCR3Base.sol)                           |   178    | OCR3 base contract              |
| [contracts/src/ccip/offRamp/OffRamp.sol](https://github.com/code-423n4/2024-11-chainlink/blob/main/src/ccip/offRamp/OffRamp.sol)                               |   571    | Main destination chain contract |
| [contracts/src/ccip/onRamp/OnRamp.sol](https://github.com/code-423n4/2024-11-chainlink/blob/main/src/ccip/onRamp/OnRamp.sol)                                   |   305    | Main source chain contract      |
| [contracts/src/ccip/rmn/RMNHome.sol](https://github.com/code-423n4/2024-11-chainlink/blob/main/src/ccip/rmn/RMNHome.sol)                                       |   194    | RMN contract on the home chain  |
| [contracts/src/ccip/rmn/RMNRemote.sol](https://github.com/code-423n4/2024-11-chainlink/blob/main/src/ccip/rmn/RMNRemote.sol)                                   |   159    | RMN contract on each chain      |
| **Total**                                                                                                                                            | **2697** |                                 |

### Files out of scope
Any files not listed above are OOS.


## Scoping Q &amp; A

### General questions

| Question                                | Answer                                                                                                                                                                                                                                                                   |
|-----------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ERC20 used by the protocol              | Any (all possible ERC20s), LINK, wrapped native (ex WETH, WPOL). NOTE: some tokens might require a custom pool to properly function. This is out of scope for this audit.                                                                                                |
| Test coverage                           | 99.57%                                                                                                                                                                                                                                                                   |
| ERC721 used  by the protocol            | None                                                                                                                                                                                                                                                                     |
| ERC777 used by the protocol             | None                                                                                                                                                                                                                                                                     |
| ERC1155 used by the protocol            | None                                                                                                                                                                                                                                                                     |
| Chains the protocol will be deployed on | Ethereum, Arbitrum, Avax, Base, BSC, Optimism, Polygon, zkSync, Other any and all EVM chains. Some chains require special configs (like setting OOO to be enforced). There is no guarantee that every chain that claims to be EVM is supported by the current contracts. |


### External integrations (e.g., Uniswap) behavior in scope:


| Question                                                  | Answer |
|-----------------------------------------------------------|--------|
| Enabling/disabling fees (e.g. Blur disables/enables fees) | No     |
| Pausability (e.g. Uniswap pool gets paused)               | No     |
| Upgradeability (e.g. Uniswap gets upgraded)               | No     |


### EIP compliance
None



# Additional context

## Main invariants

* Messages are only successfully executed once.
* Commitments must contain sequential message requests from source chains (cannot skip messages)
* Message execution must be *attempted* in nonce order, when enabled. Out of order execution is possible when specifically configured by the message sender.
  * When a message fails, the nonce is still incremented. This means there is no guarantee of *successful* message execution ordering.
* Users can manually execute their messages on destination chains once the commitment is posted
  * There is a waiting period before users can manually execute to give the Execution DON time to execute it first.



## Attack ideas (where to focus for bugs)
- DoS: for example a condition that would block all lanes because of a single lane issue
- Security check bypass due to the new multi-lane nature of the contracts: for example using the data from one lane to execute a message on another lane
- Things that we might have missed on the chain agnostic side: for example a non EVM chain integration that would somehow break the current security model
  - NOTE: not all exotic chains have been taking into consideration when designing CCIP. There will inevitably be chains that won't work. Findings are judged on how realistic it would be for CCIP to expand to that chain.
  - The FeeQuoter currently does not contain non-evm logic. It is intended to be upgraded to support non-evm chains. Limitations in billing that could be solved through a new FeeQuoter are therefore out of scope. 
- Hashing collisions
- Replay attacks messages/price reports (same chain/cross chain)
- Upgrades leading to unintentional side effects
- Nonce ordering not honoured
- Nonce ordering leading to stuck messages
- Multi-source batch getting stuck (one source blocking another)
- Unintentional interactions between 1.5 self-serve pools and V1.6 multi-ramps
- Over/under payment / billing bugs
- Wasting NOP gas, leading to losses for NOPs
- Unrecoverable states in the CCIPConfig state transitions



## All trusted roles in the protocol

- CCIP Owner Multisig which owns all contracts.


## Describe any novel or unique curve logic or mathematical models implemented in the contracts:

N/A

## Running tests

Prerequisites:
  * foundryup (https://book.getfoundry.sh/getting-started/installation)

```bash
git clone --recurse https://github.com/code-423n4/2024-11-chainlink.git
foundryup --version nightly-3ff3d0562215bca620e07c5c4c154eec8da0f04b
forge test
```

For gas profiling:
```bash
forge snapshot
```

For test coverage:
```bash
forge coverage --no-match-coverage "(keystone|liquiditymanager|mocks|vendor|shared|pools|test|applications|tokenAdminRegistry|USDPriceWith18Decimals|libraries/RateLimiter.sol|MerkleMultiProof|ARMProxy|Router)"
```

Coverage table:
| File                                   | % Lines           | % Statements       | % Branches       | % Funcs           |
|----------------------------------------|-------------------|--------------------|------------------|-------------------|
| src/ccip/FeeQuoter.sol                 | 100.00% (213/213) | 100.00% (302/302)  | 100.00% (45/45)  | 100.00% (38/38)   |
| src/ccip/MultiAggregateRateLimiter.sol | 100.00% (57/57)   | 100.00% (82/82)    | 100.00% (15/15)  | 100.00% (13/13)   |
| src/ccip/NonceManager.sol              | 100.00% (36/36)   | 100.00% (45/45)    | 100.00% (7/7)    | 100.00% (9/9)     |
| src/ccip/capability/CCIPHome.sol       | 99.20% (124/125)  | 99.46% (184/185)   | 96.97% (32/33)   | 100.00% (22/22)   |
| src/ccip/libraries/Client.sol          | 100.00% (2/2)     | 100.00% (4/4)      | 100.00% (0/0)    | 100.00% (2/2)     |
| src/ccip/libraries/Internal.sol        | 100.00% (7/7)     | 100.00% (13/13)    | 100.00% (2/2)    | 100.00% (3/3)     |
| src/ccip/ocr/MultiOCR3Base.sol         | 100.00% (57/57)   | 100.00% (90/90)    | 100.00% (22/22)  | 100.00% (9/9)     |
| src/ccip/offRamp/OffRamp.sol           | 99.13% (228/230)  | 99.05% (314/317)   | 98.53% (67/68)   | 100.00% (28/28)   |
| src/ccip/onRamp/OnRamp.sol             | 100.00% (93/93)   | 100.00% (129/129)  | 100.00% (20/20)  | 100.00% (17/17)   |
| src/ccip/rmn/RMNHome.sol               | 98.46% (64/65)    | 99.10% (110/111)   | 94.44% (17/18)   | 100.00% (14/14)   |
| src/ccip/rmn/RMNRemote.sol             | 100.00% (52/52)   | 100.00% (86/86)    | 100.00% (14/14)  | 100.00% (13/13)   |
| Total                                  | 99.57% (933/937)  | 99.63% (1359/1364) | 98.77% (241/244) | 100.00% (168/168) |



## Miscellaneous
Employees of Chainlink and employees' family members are ineligible to participate in this audit.

Code4rena's rules cannot be overridden by the contents of this README. In case of doubt, please check with C4 staff.
