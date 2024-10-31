

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

#### Token issuer is malicious

* Any malicious token issuer can negatively impact transactions that contain their token.
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
  * For example, fee-on-transfer tokens won't work because they require multiple hops in the CCIP contracts.
  * They will still be able to register their token, as this process is fully permissionless. This could negatively impact transactions containing these tokens.
  * Pool `releaseOrMint` is bound by a per-ramp `maxPoolReleaseOrMintGas`, and Token transfer is bound by a per-ramp `maxTokenTransferGas`. As a result, pools or tokens that are extremely gas-intensive may not be supported. Similarly, pool data that can be relayed from source to destination is bound as well.
* Not every token will be able to permissionlessly register.
  * There will be tokens that do not expose the `owner` and don't have the `getCCIPAdmin` function.
  * These can still be onboarded in the same way all current tokens are onboarded: manually through the CCIP Owner.

#### Other

* CCIP Owner is a trusted role.
  * As mentioned previously, the multi-signature contract structure includes the entire DON and therefore inherits the security of the entire DON.
* Solidity 0.8.24 is used and CCIP will be deployed on various chains that don't support the newer Solidity features.
  * We explicitly compile with the Paris hard fork to ensure compatibility.
* The aggregate rate limiter only works for tokens that have prices, as it's denominated in USD.
  * Self-serve tokens will not have prices, as it would be impractical to acquire accurate price sources for any arbitrary token, and writing these prices onchain would be economically unsustainable, and easily exploitable.
  * This means that tokens default to **not** be included in the aggregate rate limiter.
* Router API `getSupportedTokens` is now deprecated, calling it will revert.
  * There is no longer an official API that returns all supported tokens for a given destination in 1 call.
  * One should iterate through tokens via `getAllConfiguredTokens` in TokenAdminRegistry and then call `isSupportedChain` on its token pool.

# Overview

[ ⭐️ SPONSORS: add info here ]

## Links

- **Previous audits:**  None made public
- **Documentation:** https://docs.chain.link/ccip
- **Website:** https://chain.link/
- **X/Twitter:** https://twitter.com/chainlink
- **Discord:** https://chain.link/discord

---

# Scope

[ ⭐️ SPONSORS: list all files in scope in the table below (along with hyperlinks) -- and feel free to add notes to emphasize areas of focus. ]

### Files in scope
- ✅ Last row of the table should be Total: SLOC
- ✅ SCOUTS: Have the sponsor review and confirm in text the details in the section titled "Scoping Q amp; A"

| Contract | SLOC | Purpose | Libraries used |  
| ----------- | ----------- | ----------- | ----------- |
| [contracts/folder/sample.sol](https://github.com/code-423n4/repo-name/blob/contracts/folder/sample.sol) | 123 | This contract does XYZ | [`@openzeppelin/*`](https://openzeppelin.com/contracts/) |

### Files out of scope
✅ SCOUTS: List files/directories out of scope

## Scoping Q &amp; A

### General questions
### Are there any ERC20's in scope?: Yes

✅ SCOUTS: If the answer above 👆 is "Yes", please add the tokens below 👇 to the table. Otherwise, update the column with "None".

Any (all possible ERC20s)
LINK, wrapped native (ex WETH, WPOL)

### Are there any ERC777's in scope?: No

✅ SCOUTS: If the answer above 👆 is "Yes", please add the tokens below 👇 to the table. Otherwise, update the column with "None".



### Are there any ERC721's in scope?: No

✅ SCOUTS: If the answer above 👆 is "Yes", please add the tokens below 👇 to the table. Otherwise, update the column with "None".



### Are there any ERC1155's in scope?: No

✅ SCOUTS: If the answer above 👆 is "Yes", please add the tokens below 👇 to the table. Otherwise, update the column with "None".



✅ SCOUTS: Once done populating the table below, please remove all the Q/A data above.

| Question                                | Answer                       |
| --------------------------------------- | ---------------------------- |
| ERC20 used by the protocol              |       🖊️             |
| Test coverage                           | ✅ SCOUTS: Please populate this after running the test coverage command                          |
| ERC721 used  by the protocol            |            🖊️              |
| ERC777 used by the protocol             |           🖊️                |
| ERC1155 used by the protocol            |              🖊️            |
| Chains the protocol will be deployed on | Ethereum,Arbitrum,Avax,Base,BSC,Optimism,Polygon,zkSync,Otherany and all EVM chains  |

### ERC20 token behaviors in scope

| Question                                                                                                                                                   | Answer |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| [Missing return values](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#missing-return-values)                                                      |   In scope  |
| [Fee on transfer](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#fee-on-transfer)                                                                  |  In scope  |
| [Balance changes outside of transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#balance-modifications-outside-of-transfers-rebasingairdrops) | In scope    |
| [Upgradeability](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#upgradable-tokens)                                                                 |   In scope  |
| [Flash minting](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#flash-mintable-tokens)                                                              | In scope    |
| [Pausability](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#pausable-tokens)                                                                      | In scope    |
| [Approval race protections](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#approval-race-protections)                                              | In scope    |
| [Revert on approval to zero address](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-approval-to-zero-address)                            | In scope    |
| [Revert on zero value approvals](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-approvals)                                    | In scope    |
| [Revert on zero value transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-transfers)                                    | In scope    |
| [Revert on transfer to the zero address](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-transfer-to-the-zero-address)                    | In scope    |
| [Revert on large approvals and/or transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-large-approvals--transfers)                  | In scope    |
| [Doesn't revert on failure](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#no-revert-on-failure)                                                   |  In scope   |
| [Multiple token addresses](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-transfers)                                          | In scope    |
| [Low decimals ( < 6)](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#low-decimals)                                                                 |   In scope  |
| [High decimals ( > 18)](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#high-decimals)                                                              | In scope    |
| [Blocklists](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#tokens-with-blocklists)                                                                | In scope    |

### External integrations (e.g., Uniswap) behavior in scope:


| Question                                                  | Answer |
| --------------------------------------------------------- | ------ |
| Enabling/disabling fees (e.g. Blur disables/enables fees) | No   |
| Pausability (e.g. Uniswap pool gets paused)               |  No   |
| Upgradeability (e.g. Uniswap gets upgraded)               |   No  |


### EIP compliance checklist
N/A

✅ SCOUTS: Please format the response above 👆 using the template below👇

| Question                                | Answer                       |
| --------------------------------------- | ---------------------------- |
| src/Token.sol                           | ERC20, ERC721                |
| src/NFT.sol                             | ERC721                       |


# Additional context

## Main invariants

* Commitments must contain sequential message requests from source chains (cannot skip messages)
* messages must be executed in order per user, when enabled. Out of order execution is possible only when specifically requested by the message requester
* users can always manually execute their messages on destination chains once the commitment is posted

✅ SCOUTS: Please format the response above 👆 so its not a wall of text and its readable.

## Attack ideas (where to focus for bugs)
- DoS: for example a condition that would block all lanes because of a single lane issue
- Security check bypass due to the new multi lane nature of the contracts: for example using the data from one lane to execute a message on another lane
- Things that we might have missed on the chain agnostic side: for example a non EVM chain integration that would somehow break the current security model
- Hashing collisions
- Replay attacks messages/price reports (same chain/cross chain)
- Upgrades leading to unintentional side effects
- Nonce ordering not honoured
- Nonce ordering leading to stuck messages
- Multi-source batch getting stuck (one source blocking another)
- Unintentional interactions between 1.5 self-serve pools and V2 multi-ramps
- Over/under payment / billing bugs
- Wasting NOP gas
- Unrecoverable states in the CCIPConfig state transitions

✅ SCOUTS: Please format the response above 👆 so its not a wall of text and its readable.

## All trusted roles in the protocol

Contract owners

✅ SCOUTS: Please format the response above 👆 using the template below👇

| Role                                | Description                       |
| --------------------------------------- | ---------------------------- |
| Owner                          | Has superpowers                |
| Administrator                             | Can change fees                       |

## Describe any novel or unique curve logic or mathematical models implemented in the contracts:

N/A

✅ SCOUTS: Please format the response above 👆 so its not a wall of text and its readable.

## Running tests

Prerequisites:
  * node 20.13.1
  * pnpm 9.4.0
  * foundryup (https://book.getfoundry.sh/getting-started/installation)\

* cd contracts/
* pnpm install
* foundryup --version nightly-3ff3d0562215bca620e07c5c4c154eec8da0f04b
* forge test
* forge snapshot

✅ SCOUTS: Please format the response above 👆 using the template below👇

✅ SCOUTS: Add a screenshot of your terminal showing the gas report
✅ SCOUTS: Add a screenshot of your terminal showing the test coverage

## Miscellaneous
Employees of Chainlink and employees' family members are ineligible to participate in this audit.

Code4rena's rules cannot be overridden by the contents of this README. In case of doubt, please check with C4 staff.
