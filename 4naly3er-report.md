# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | `a = a + b` is more gas effective than `a += b` for state variables (excluding arrays and mappings) | 10 |
| [GAS-2](#GAS-2) | Use assembly to check for `address(0)` | 1 |
| [GAS-3](#GAS-3) | Using bools for storage incurs overhead | 1 |
| [GAS-4](#GAS-4) | Cache array length outside of loop | 41 |
| [GAS-5](#GAS-5) | Use calldata instead of memory for function arguments that do not get mutated | 6 |
| [GAS-6](#GAS-6) | For Operations that will not overflow, you could use unchecked | 434 |
| [GAS-7](#GAS-7) | Avoid contract existence checks by using low level calls | 1 |
| [GAS-8](#GAS-8) | State variables only set in the constructor should be declared `immutable` | 1 |
| [GAS-9](#GAS-9) | Functions guaranteed to revert when called by normal users can be marked `payable` | 3 |
| [GAS-10](#GAS-10) | `++i` costs less gas compared to `i++` or `i += 1` (same for `--i` vs `i--` or `i -= 1`) | 1 |
| [GAS-11](#GAS-11) | Using `private` rather than `public` for constants, saves gas | 17 |
| [GAS-12](#GAS-12) | Use shift right/left instead of division/multiplication if possible | 5 |
| [GAS-13](#GAS-13) | Superfluous event fields | 2 |
| [GAS-14](#GAS-14) | Use of `this` instead of marking as `public` an `external` function | 1 |
| [GAS-15](#GAS-15) | Increments/decrements can be unchecked in for-loops | 59 |
| [GAS-16](#GAS-16) | Use != 0 instead of > 0 for unsigned integer comparison | 11 |
| [GAS-17](#GAS-17) | `internal` functions not called by the contract should be removed | 6 |
### <a name="GAS-1"></a>[GAS-1] `a = a + b` is more gas effective than `a += b` for state variables (excluding arrays and mappings)
This saves **16 gas per instance.**

*Instances (10)*:
```solidity
File: ./src/ccip/FeeQuoter.sol

665:         tokenTransferFeeUSDWei += uint256(destChainConfig.defaultTokenFeeUSDCents) * 1e16;

666:         tokenTransferGas += destChainConfig.defaultTokenDestGasOverhead;

667:         tokenTransferBytesOverhead += Pool.CCIP_LOCK_OR_BURN_V1_RET_BYTES;

687:       tokenTransferGas += transferFeeConfig.destGasOverhead;

688:       tokenTransferBytesOverhead += transferFeeConfig.destBytesOverhead;

694:         tokenTransferFeeUSDWei += minFeeUSDWei;

700:         tokenTransferFeeUSDWei += maxFeeUSDWei;

704:       tokenTransferFeeUSDWei += bpsFeeUSDWei;

```

```solidity
File: ./src/ccip/MultiAggregateRateLimiter.sol

109:           value += _getTokenValue(tokenAmounts[i]);

```

```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

253:         expectedDataLength +=

```

### <a name="GAS-2"></a>[GAS-2] Use assembly to check for `address(0)`
*Saves 6 gas per instance*

*Instances (1)*:
```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

218:       s_oracles[ocrPluginType][oracle] = Oracle(uint8(i), role);

```

### <a name="GAS-3"></a>[GAS-3] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (1)*:
```solidity
File: ./src/ccip/rmn/RMNRemote.sol

80:   mapping(address signer => bool exists) private s_signers; // for more gas efficient verify.

```

### <a name="GAS-4"></a>[GAS-4] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (41)*:
```solidity
File: ./src/ccip/FeeQuoter.sol

431:     for (uint256 i = 0; i < feeTokensToRemove.length; ++i) {

436:     for (uint256 i = 0; i < feeTokensToAdd.length; ++i) {

486:     for (uint256 i; i < tokenPriceFeedUpdates.length; ++i) {

508:     for (uint256 i = 0; i < feeds.length; ++i) {

617:     for (uint256 i = 0; i < premiumMultiplierWeiPerEthArgs.length; ++i) {

798:     for (uint256 i = 0; i < tokenTransferFeeConfigArgs.length; ++i) {

802:       for (uint256 j = 0; j < tokenTransferFeeConfigArg.tokenTransferFeeConfigs.length; ++j) {

822:     for (uint256 i = 0; i < tokensToUseDefaultFeeConfigs.length; ++i) {

966:     for (uint256 i = 0; i < onRampTokenTransfers.length; ++i) {

1021:     for (uint256 i = 0; i < destChainConfigArgs.length; ++i) {

```

```solidity
File: ./src/ccip/MultiAggregateRateLimiter.sol

107:       for (uint256 i = 0; i < tokenAmounts.length; ++i) {

165:     for (uint256 i = 0; i < rateLimiterUpdates.length; ++i) {

230:     for (uint256 i = 0; i < removes.length; ++i) {

239:     for (uint256 i = 0; i < adds.length; ++i) {

```

```solidity
File: ./src/ccip/NonceManager.sol

132:     for (uint256 i = 0; i < previousRampsArgs.length; ++i) {

```

```solidity
File: ./src/ccip/capability/CCIPHome.sol

574:     for (uint256 i = 0; i < chainSelectorRemoves.length; ++i) {

587:     for (uint256 i = 0; i < chainConfigAdds.length; ++i) {

```

```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

132:     for (uint256 i; i < ocrConfigArgs.length; ++i) {

202:     for (uint256 i = 0; i < oracleAddresses.length; ++i) {

212:     for (uint256 i = 0; i < oracleAddresses.length; ++i) {

```

```solidity
File: ./src/ccip/offRamp/OffRamp.sol

305:         for (uint256 tokenIndex = 0; tokenIndex < message.tokenAmounts.length; ++tokenIndex) {

349:     for (uint256 i = 0; i < reports.length; ++i) {

745:     for (uint256 i = 0; i < sourceTokenAmounts.length; ++i) {

810:     for (uint256 i = 0; i < commitReport.merkleRoots.length; ++i) {

935:     for (uint256 i = 0; i < s_sourceChainSelectors.length(); ++i) {

955:     for (uint256 i = 0; i < sourceChainConfigUpdates.length; ++i) {

```

```solidity
File: ./src/ccip/onRamp/OnRamp.sol

217:     for (uint256 i = 0; i < message.tokenAmounts.length; ++i) {

238:     for (uint256 i = 0; i < newMessage.tokenAmounts.length; ++i) {

366:     for (uint256 i = 0; i < destChainConfigArgs.length; ++i) {

432:     for (uint256 i = 0; i < allowlistConfigArgsItems.length; ++i) {

440:           for (uint256 j = 0; j < allowlistConfigArgs.addedAllowlistedSenders.length; ++j) {

454:       for (uint256 j = 0; j < allowlistConfigArgs.removedAllowlistedSenders.length; ++j) {

508:     for (uint256 i = 0; i < feeTokens.length; ++i) {

```

```solidity
File: ./src/ccip/rmn/RMNHome.sol

349:     for (uint256 i = 0; i < staticConfig.nodes.length; ++i) {

350:       for (uint256 j = i + 1; j < staticConfig.nodes.length; ++j) {

```

```solidity
File: ./src/ccip/rmn/RMNRemote.sol

121:     for (uint256 i = 0; i < signatures.length; ++i) {

145:     for (uint256 i = 1; i < newConfig.signers.length; ++i) {

157:     for (uint256 i = s_config.signers.length; i > 0; --i) {

162:     for (uint256 i = 0; i < newConfig.signers.length; ++i) {

213:     for (uint256 i = 0; i < subjects.length; ++i) {

237:     for (uint256 i = 0; i < subjects.length; ++i) {

```

### <a name="GAS-5"></a>[GAS-5] Use calldata instead of memory for function arguments that do not get mutated
When a function with a `memory` array is called externally, the `abi.decode()` step has to use a for-loop to copy each index of the `calldata` to the `memory` index. Each iteration of this for-loop costs at least 60 gas (i.e. `60 * <mem_array>.length`). Using `calldata` directly bypasses this loop. 

If the array is passed to an `internal` function which passes the array to another internal function where the array is modified and therefore `memory` is used in the `external` call, it's still more gas-efficient to use `calldata` when the `external` function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one. 

 *Saves 60 gas per instance*

*Instances (6)*:
```solidity
File: ./src/ccip/interfaces/IFeeQuoter.sol

34:     bytes memory extraArgs,

```

```solidity
File: ./src/ccip/interfaces/IMessageInterceptor.sol

17:     Client.Any2EVMMessage memory message

23:   function onOutboundMessage(uint64 destChainSelector, Client.EVM2AnyMessage memory message) external;

```

```solidity
File: ./src/ccip/interfaces/IRMNRemote.sol

21:     Internal.MerkleRoot[] memory merkleRoots,

22:     Signature[] memory signatures

```

```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

133:       _setOCR3Config(ocrConfigArgs[i]);

```

### <a name="GAS-6"></a>[GAS-6] For Operations that will not overflow, you could use unchecked

*Instances (434)*:
```solidity
File: ./src/ccip/FeeQuoter.sol

4: import {IReceiver} from "../keystone/interfaces/IReceiver.sol";

5: import {ITypeAndVersion} from "../shared/interfaces/ITypeAndVersion.sol";

6: import {IFeeQuoter} from "./interfaces/IFeeQuoter.sol";

7: import {IPriceRegistry} from "./interfaces/IPriceRegistry.sol";

9: import {KeystoneFeedsPermissionHandler} from "../keystone/KeystoneFeedsPermissionHandler.sol";

10: import {KeystoneFeedDefaultMetadataLib} from "../keystone/lib/KeystoneFeedDefaultMetadataLib.sol";

11: import {AuthorizedCallers} from "../shared/access/AuthorizedCallers.sol";

12: import {AggregatorV3Interface} from "./../shared/interfaces/AggregatorV3Interface.sol";

13: import {Client} from "./libraries/Client.sol";

14: import {Internal} from "./libraries/Internal.sol";

15: import {Pool} from "./libraries/Pool.sol";

16: import {USDPriceWith18Decimals} from "./libraries/USDPriceWith18Decimals.sol";

18: import {EnumerableSet} from "../vendor/openzeppelin-solidity/v5.0.2/contracts/utils/structs/EnumerableSet.sol";

62:     address dataFeedAddress; // ─╮ Price feed contract. Can be address(0) to indicate no feed is configured.

63:     uint8 tokenDecimals; //      │ Decimals of the token, used for both keystone and price feed decimal multiplications.

64:     bool isEnabled; // ──────────╯ Whether the token is configured to receive keystone and/or price feed updates.

69:     address sourceToken; // Source token to update feed for.

70:     TokenPriceFeedConfig feedConfig; // Feed config update data.

77:     uint96 maxFeeJuelsPerMsg; // ─╮ Maximum fee that can be charged for a message.

78:     address linkToken; // ────────╯ LINK token address.

86:     address token; //       Token address.

87:     uint224 price; // ────╮ Price of the token in USD with 18 decimals.

88:     uint32 timestamp; // ─╯ Timestamp of the price update.

93:     bool isEnabled; // ──────────────────────────╮ Whether this destination chain is enabled.

94:     uint16 maxNumberOfTokensPerMsg; //           │ Maximum number of distinct ERC20 tokens transferred per message.

95:     uint32 maxDataBytes; //                      │ Maximum data payload size in bytes.

96:     uint32 maxPerMsgGasLimit; //                 │ Maximum gas limit for messages targeting EVMs.

97:     uint32 destGasOverhead; //                   │ Gas charged on top of the gasLimit to cover destination chain costs.

98:     uint16 destGasPerPayloadByte; //             │ Destination chain gas charged each byte of `data` payload.

99:     uint32 destDataAvailabilityOverheadGas; //   │ Data availability gas charged for overhead costs e.g. for OCR.

100:     uint16 destGasPerDataAvailabilityByte; //    │ Gas units charged per byte of message data that needs availability.

101:     uint16 destDataAvailabilityMultiplierBps; // │ Multiplier for data availability gas, multiples of bps, or 0.0001.

103:     uint16 defaultTokenFeeUSDCents; //           │ Default token fee charged per token transfer.

104:     uint32 defaultTokenDestGasOverhead; // ──────╯ Default gas charged to execute a token transfer on the destination chain.

105:     uint32 defaultTxGasLimit; //─────────────────╮ Default gas limit for a tx.

106:     uint64 gasMultiplierWeiPerEth; //            │ Multiplier for gas costs, 1e18 based so 11e17 = 10% extra cost.

107:     uint32 networkFeeUSDCents; //                │ Flat network fee to charge for messages, multiples of 0.01 USD.

108:     uint32 gasPriceStalenessThreshold; //        │ The amount of time a gas price can be stale before it is considered invalid (0 means disabled).

109:     bool enforceOutOfOrder; //                   │ Whether to enforce the allowOutOfOrderExecution extraArg value to be true.

110:     bytes4 chainFamilySelector; // ──────────────╯ Selector that identifies the destination chain's family. Used to determine the correct validations to perform for the dest chain.

117:     uint64 destChainSelector; // Destination chain selector.

118:     DestChainConfig destChainConfig; // Config to update for the chain selector.

123:     uint32 minFeeUSDCents; // ────╮ Minimum fee to charge per token transfer, multiples of 0.01 USD.

124:     uint32 maxFeeUSDCents; //     │ Maximum fee to charge per token transfer, multiples of 0.01 USD.

125:     uint16 deciBps; //            │ Basis points charged on token transfers, multiples of 0.1bps, or 1e-5.

126:     uint32 destGasOverhead; //    │ Gas charged to execute the token transfer on the destination chain.

128:     uint32 destBytesOverhead; //  │ the destination pool. Must be >= Pool.CCIP_LOCK_OR_BURN_V1_RET_BYTES.

129:     bool isEnabled; // ───────────╯ Whether this token has custom transfer fees.

135:     address token; // Token address.

136:     TokenTransferFeeConfig tokenTransferFeeConfig; // Struct to hold the transfer fee configuration for token transfers.

141:     uint64 destChainSelector; // Destination chain selector.

142:     TokenTransferFeeConfigSingleTokenArgs[] tokenTransferFeeConfigs; // Array of token transfer fee configurations.

148:     uint64 destChainSelector; // ─╮ Destination chain selector.

149:     address token; // ────────────╯ Token address.

154:     address token; // // ──────────────────╮ Token address.

155:     uint64 premiumMultiplierWeiPerEth; // ─╯ Multiplier for destination chain specific premiums.

163:   string public constant override typeAndVersion = "FeeQuoter 1.6.0-dev";

245:     if (block.timestamp - tokenPrice.timestamp < i_tokenPriceStalenessThreshold) {

279:     for (uint256 i = 0; i < length; ++i) {

343:     return (fromTokenAmount * _getValidatedTokenPrice(fromToken)) / _getValidatedTokenPrice(toToken);

397:       uint256 timePassed = block.timestamp - gasPrice.timestamp;

431:     for (uint256 i = 0; i < feeTokensToRemove.length; ++i) {

436:     for (uint256 i = 0; i < feeTokensToAdd.length; ++i) {

456:     for (uint256 i = 0; i < tokenUpdatesLength; ++i) {

465:     for (uint256 i = 0; i < gasUpdatesLength; ++i) {

486:     for (uint256 i; i < tokenPriceFeedUpdates.length; ++i) {

508:     for (uint256 i = 0; i < feeds.length; ++i) {

564:       premiumFee = uint256(destChainConfig.networkFeeUSDCents) * 1e16;

593:         destChainConfig.destGasOverhead + (message.data.length * destChainConfig.destGasPerPayloadByte) + tokenTransferGas

594:           + _parseEVMExtraArgsFromBytes(message.extraArgs, destChainConfig).gasLimit

595:       ) * destChainConfig.gasMultiplierWeiPerEth;

600:     return ((premiumFee * s_premiumMultiplierWeiPerEth[message.feeToken]) + executionCost + dataAvailabilityCost)

601:       / feeTokenPrice;

617:     for (uint256 i = 0; i < premiumMultiplierWeiPerEthArgs.length; ++i) {

659:     for (uint256 i = 0; i < numberOfTokens; ++i) {

665:         tokenTransferFeeUSDWei += uint256(destChainConfig.defaultTokenFeeUSDCents) * 1e16;

666:         tokenTransferGas += destChainConfig.defaultTokenDestGasOverhead;

667:         tokenTransferBytesOverhead += Pool.CCIP_LOCK_OR_BURN_V1_RET_BYTES;

684:         bpsFeeUSDWei = (tokenPrice._calcUSDValueFromTokenAmount(tokenAmount.amount) * transferFeeConfig.deciBps) / 1e5;

687:       tokenTransferGas += transferFeeConfig.destGasOverhead;

688:       tokenTransferBytesOverhead += transferFeeConfig.destBytesOverhead;

692:       uint256 minFeeUSDWei = uint256(transferFeeConfig.minFeeUSDCents) * 1e16;

694:         tokenTransferFeeUSDWei += minFeeUSDWei;

698:       uint256 maxFeeUSDWei = uint256(transferFeeConfig.maxFeeUSDCents) * 1e16;

700:         tokenTransferFeeUSDWei += maxFeeUSDWei;

704:       tokenTransferFeeUSDWei += bpsFeeUSDWei;

727:     uint8 excessDecimals = dataFeedDecimal + tokenDecimal;

731:       rebasedVal = feedValue / (10 ** (excessDecimals - FEE_BASE_DECIMALS));

733:       rebasedVal = feedValue * (10 ** (FEE_BASE_DECIMALS - excessDecimals));

760:     uint256 dataAvailabilityLengthBytes = Internal.MESSAGE_FIXED_BYTES + messageDataLength

761:       + (numberOfTokens * Internal.MESSAGE_FIXED_BYTES_PER_TOKEN) + tokenTransferBytesOverhead;

765:     uint256 dataAvailabilityGas = (dataAvailabilityLengthBytes * destChainConfig.destGasPerDataAvailabilityByte)

766:       + destChainConfig.destDataAvailabilityOverheadGas;

770:     return ((dataAvailabilityGas * dataAvailabilityGasPrice) * destChainConfig.destDataAvailabilityMultiplierBps) * 1e14;

798:     for (uint256 i = 0; i < tokenTransferFeeConfigArgs.length; ++i) {

802:       for (uint256 j = 0; j < tokenTransferFeeConfigArg.tokenTransferFeeConfigs.length; ++j) {

822:     for (uint256 i = 0; i < tokensToUseDefaultFeeConfigs.length; ++i) {

966:     for (uint256 i = 0; i < onRampTokenTransfers.length; ++i) {

1021:     for (uint256 i = 0; i < destChainConfigArgs.length; ++i) {

```

```solidity
File: ./src/ccip/MultiAggregateRateLimiter.sol

4: import {ITypeAndVersion} from "../shared/interfaces/ITypeAndVersion.sol";

5: import {IFeeQuoter} from "./interfaces/IFeeQuoter.sol";

6: import {IMessageInterceptor} from "./interfaces/IMessageInterceptor.sol";

8: import {AuthorizedCallers} from "../shared/access/AuthorizedCallers.sol";

9: import {EnumerableMapAddresses} from "./../shared/enumerable/EnumerableMapAddresses.sol";

10: import {Client} from "./libraries/Client.sol";

11: import {RateLimiter} from "./libraries/RateLimiter.sol";

12: import {USDPriceWith18Decimals} from "./libraries/USDPriceWith18Decimals.sol";

34:     uint64 remoteChainSelector; // ─╮ Remote chain selector for which to update the rate limit token mapping.

35:     address localToken; // ─────────╯ Token on the chain on which the multi-ARL is deployed.

40:     LocalRateLimitToken localTokenArgs; // Local token update args scoped to one remote chain.

41:     bytes remoteToken; // Token on the remote chain, for OnRamp - dest, or OffRamp - source.

46:     uint64 remoteChainSelector; // ─╮ Remote chain selector to set config for.

47:     bool isOutboundLane; // ────────╯ If set to true, represents the outbound message lane (OnRamp), and the inbound message lane otherwise (OffRamp)

48:     RateLimiter.Config rateLimiterConfig; // Rate limiter config to set

53:     RateLimiter.TokenBucket inboundLaneBucket; // Bucket for the inbound lane (remote -> local).

54:     RateLimiter.TokenBucket outboundLaneBucket; // Bucket for the outbound lane (local -> remote).

57:   string public constant override typeAndVersion = "MultiAggregateRateLimiter 1.6.0-dev";

107:       for (uint256 i = 0; i < tokenAmounts.length; ++i) {

109:           value += _getTokenValue(tokenAmounts[i]);

165:     for (uint256 i = 0; i < rateLimiterUpdates.length; ++i) {

215:     for (uint256 i = 0; i < tokenCount; ++i) {

230:     for (uint256 i = 0; i < removes.length; ++i) {

239:     for (uint256 i = 0; i < adds.length; ++i) {

```

```solidity
File: ./src/ccip/NonceManager.sol

4: import {ITypeAndVersion} from "../shared/interfaces/ITypeAndVersion.sol";

5: import {IEVM2AnyOnRamp} from "./interfaces/IEVM2AnyOnRamp.sol";

6: import {INonceManager} from "./interfaces/INonceManager.sol";

8: import {AuthorizedCallers} from "../shared/access/AuthorizedCallers.sol";

20:     address prevOnRamp; // Previous onRamp.

21:     address prevOffRamp; // Previous offRamp.

27:     uint64 remoteChainSelector; // ──╮ Chain selector.

28:     bool overrideExistingRamps; // ──╯ Whether to override existing ramps.

29:     PreviousRamps prevRamps; // Previous on/off ramps.

32:   string public constant override typeAndVersion = "NonceManager 1.6.0-dev";

52:     uint64 outboundNonce = _getOutboundNonce(destChainSelector, sender) + 1;

87:     uint64 inboundNonce = _getInboundNonce(sourceChainSelector, sender) + 1;

132:     for (uint256 i = 0; i < previousRampsArgs.length; ++i) {

```

```solidity
File: ./src/ccip/capability/CCIPHome.sol

4: import {ICapabilityConfiguration} from "../../keystone/interfaces/ICapabilityConfiguration.sol";

6: import {INodeInfoProvider} from "../../keystone/interfaces/INodeInfoProvider.sol";

7: import {ITypeAndVersion} from "../../shared/interfaces/ITypeAndVersion.sol";

9: import {Ownable2StepMsgSender} from "../../shared/access/Ownable2StepMsgSender.sol";

10: import {Internal} from "../libraries/Internal.sol";

12: import {IERC165} from "../../vendor/openzeppelin-solidity/v5.0.2/contracts/interfaces/IERC165.sol";

13: import {EnumerableSet} from "../../vendor/openzeppelin-solidity/v5.0.2/contracts/utils/structs/EnumerableSet.sol";

100:     bytes32 p2pId; // Peer2Peer connection ID of the oracle.

101:     bytes signerKey; // On-chain signer public key.

102:     bytes transmitterKey; // On-chain transmitter public key. Can be set to empty bytes to represent that the node is a signer but not a transmitter.

110:     Internal.OCRPluginType pluginType; // ─╮ The plugin that the configuration is for.

111:     uint64 chainSelector; //               │ The (remote) chain that the configuration is for.

112:     uint8 FRoleDON; //                     │ The "big F" parameter for the role DON.

113:     uint64 offchainConfigVersion; // ──────╯ The version of the exec offchain configuration.

114:     bytes offrampAddress; // The remote chain offramp address.

115:     bytes rmnHomeAddress; // The home chain RMN home address.

116:     OCR3Node[] nodes; // Keys & IDs of nodes part of the role DON.

117:     bytes offchainConfig; // The offchain configuration for the OCR3 plugin. Protobuf encoded.

129:     bytes32[] readers; // The P2P IDs of the readers for the chain. These IDs must be registered in the capabilities registry.

130:     uint8 fChain; // The fault tolerance parameter of the chain.

131:     bytes config; // The chain configuration. This is kept intentionally opaque so as to add fields in the future if needed.

140:   string public constant override typeAndVersion = "CCIPHome 1.6.0-dev";

143:   uint256 private constant PREFIX = 0x000a << (256 - 16); // 0x000a00..00

149:   uint256 private constant PREFIX_MASK = type(uint256).max << (256 - 16); // 0xFFFF00..00

207:     bytes32[] calldata, // nodes.

211:     uint64, // config count.

247:     uint32 /* donId */

294:     for (uint256 i = 0; i < MAX_CONCURRENT_CONFIGS; ++i) {

353:     uint32 newVersion = ++s_currentVersion;

487:     if (numberOfNodes <= 3 * FRoleDON) revert FTooHigh();

491:     for (uint256 i = 0; i < numberOfNodes; ++i) {

497:         nonZeroTransmitters++;

511:     uint256 minTransmittersLength = 3 * fChain + 1;

542:     uint256 startIndex = pageIndex * pageSize;

545:       return new ChainConfigArgs[](0); // Return an empty array if pageSize is 0 or pageIndex is out of bounds.

548:     uint256 endIndex = startIndex + pageSize;

553:     ChainConfigArgs[] memory paginatedChainConfigs = new ChainConfigArgs[](endIndex - startIndex);

555:     for (uint256 i = startIndex; i < endIndex; ++i) {

557:       paginatedChainConfigs[i - startIndex] =

574:     for (uint256 i = 0; i < chainSelectorRemoves.length; ++i) {

587:     for (uint256 i = 0; i < chainConfigAdds.length; ++i) {

```

```solidity
File: ./src/ccip/interfaces/IFeeQuoter.sol

4: import {Client} from "../libraries/Client.sol";

5: import {Internal} from "../libraries/Internal.sol";

6: import {IPriceRegistry} from "./IPriceRegistry.sol";

```

```solidity
File: ./src/ccip/interfaces/IMessageInterceptor.sol

4: import {Client} from "../libraries/Client.sol";

```

```solidity
File: ./src/ccip/interfaces/IRMNRemote.sol

4: import {Internal} from "../libraries/Internal.sol";

```

```solidity
File: ./src/ccip/libraries/Client.sol

8:     address token; // token address on the local chain.

9:     uint256 amount; // Amount of tokens.

13:     bytes32 messageId; // MessageId corresponding to ccipSend on source.

14:     uint64 sourceChainSelector; // Source chain selector.

15:     bytes sender; // abi.decode(sender) if coming from an EVM chain.

16:     bytes data; // payload sent in original message.

17:     EVMTokenAmount[] destTokenAmounts; // Tokens and their amounts in their destination chain representation.

22:     bytes receiver; // abi.encode(receiver address) for dest EVM chains.

23:     bytes data; // Data payload.

24:     EVMTokenAmount[] tokenAmounts; // Token transfers.

25:     address feeToken; // Address of feeToken. address(0) means you will send msg.value.

26:     bytes extraArgs; // Populate this with _argsToBytes(EVMExtraArgsV2).

```

```solidity
File: ./src/ccip/libraries/Internal.sol

4: import {MerkleMultiProof} from "../libraries/MerkleMultiProof.sol";

16:   uint16 internal constant MAX_RET_BYTES = 4 + 4 * 32;

30:     address sourceToken; // Source token.

31:     uint224 usdPerToken; // 1e18 USD per 1e18 of the smallest token denomination.

37:     uint64 destChainSelector; // Destination chain selector.

38:     uint224 usdPerUnitGas; // 1e18 USD per smallest unit (e.g. wei) of destination chain gas.

43:     uint224 value; // ──────╮ Value in uint224, packed.

44:     uint32 timestamp; // ───╯ Timestamp of the most recent price update.

63:     uint32 destGasAmount; // The amount of gas available for the releaseOrMint and balanceOf calls on the offRamp

69:     uint64 sourceChainSelector; // Source chain selector for which the report is submitted.

83:   uint256 public constant MESSAGE_FIXED_BYTES = 32 * 15;

95:   uint256 public constant MESSAGE_FIXED_BYTES_PER_TOKEN = 32 * (4 + (3 + 2));

198:     bytes32 messageId; // Unique identifier for the message, generated with the source chain's encoding scheme (i.e. not necessarily abi.encoded).

199:     uint64 sourceChainSelector; // ──╮ the chain selector of the source chain, note: not chainId.

200:     uint64 destChainSelector; //     │ the chain selector of the destination chain, note: not chainId.

201:     uint64 sequenceNumber; //        │ sequence number, not unique across lanes.

202:     uint64 nonce; // ────────────────╯ nonce for this lane for this sender, not unique across senders/lanes.

216:     uint256 amount; // Amount of tokens.

226:     address destTokenAddress; // ─╮ Address of destination token

227:     uint32 destGasAmount; //──────╯ The amount of gas available for the releaseOrMint and transfer calls on the offRamp.

232:     uint256 amount; // Amount of tokens.

239:     RampMessageHeader header; // Message header.

240:     bytes sender; // sender address on the source chain.

241:     bytes data; // arbitrary data payload supplied by the message sender.

242:     address receiver; // receiver address on the destination chain.

243:     uint256 gasLimit; // user supplied maximum gas amount available for dest chain execution.

244:     Any2EVMTokenTransfer[] tokenAmounts; // array of tokens and amounts to transfer.

251:     RampMessageHeader header; // Message header.

252:     address sender; // sender address on the source chain.

253:     bytes data; // arbitrary data payload supplied by the message sender.

254:     bytes receiver; // receiver address on the destination chain.

255:     bytes extraArgs; // destination-chain specific extra args, such as the gasLimit for EVM chains.

256:     address feeToken; // fee token.

257:     uint256 feeTokenAmount; // fee token amount.

258:     uint256 feeValueJuels; // fee amount in Juels.

259:     EVM2AnyTokenTransfer[] tokenAmounts; // array of tokens and amounts to transfer.

270:     uint64 sourceChainSelector; // Remote source chain selector that the Merkle Root is scoped to

271:     bytes onRampAddress; //        Generic onramp address, to support arbitrary sources; for EVM, use abi.encode

272:     uint64 minSeqNr; // ─────────╮ Minimum sequence number, inclusive

273:     uint64 maxSeqNr; // ─────────╯ Maximum sequence number, inclusive

274:     bytes32 merkleRoot; //         Merkle root covering the interval & source chain messages

```

```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

4: import {Ownable2StepMsgSender} from "../../shared/access/Ownable2StepMsgSender.sol";

5: import {ITypeAndVersion} from "../../shared/interfaces/ITypeAndVersion.sol";

50:     uint8 F; // ─────────────────────────────╮ maximum number of faulty/dishonest oracles the system can tolerate.

51:     uint8 n; //                              │ number of configured signers.

52:     bool isSignatureVerificationEnabled; // ─╯ if true, requires signers and verifies signatures on transmission.

69:     uint8 index; // ─╮ Index of oracle in s_signers/s_transmitters.

70:     Role role; // ───╯ Role of the address which mapped to this struct.

75:     ConfigInfo configInfo; //  latest OCR config.

77:     address[] signers; //      addresses oracles use to sign the reports.

78:     address[] transmitters; // addresses oracles use to transmit the reports.

83:     bytes32 configDigest; // The new config digest.

84:     uint8 ocrPluginType; // ─────────────────╮ OCR plugin type to update config for.

85:     uint8 F; //                              │ Maximum number of faulty/dishonest oracles.

86:     bool isSignatureVerificationEnabled; // ─╯ If true, requires signers and verifies signatures on transmission.

87:     address[] signers; // signing address of each oracle.

88:     address[] transmitters; // the address the oracle sends transactions from.

103:   uint16 private constant TRANSMIT_MSGDATA_CONSTANT_LENGTH_COMPONENT_NO_SIGNATURES = 4 // function selector.

104:     + 3 * 32 // 3 words containing reportContext.

105:     + 32 // word containing start location of abiencoded report value.

106:     + 32; // word containing length of report.

110:   uint16 private constant TRANSMIT_MSGDATA_EXTRA_CONSTANT_LENGTH_COMPONENT_FOR_SIGNATURES = 32 // word containing location start of abiencoded rs value.

111:     + 32 // word containing start location of abiencoded ss value.

112:     + 32 // rawVs value.

113:     + 32 // word containing length rs.

114:     + 32; // word containing length of ss.

132:     for (uint256 i; i < ocrConfigArgs.length; ++i) {

169:       if (signers.length <= 3 * ocrConfigArgs.F) revert InvalidConfig(InvalidConfigErrorType.F_TOO_HIGH);

202:     for (uint256 i = 0; i < oracleAddresses.length; ++i) {

212:     for (uint256 i = 0; i < oracleAddresses.length; ++i) {

249:       uint256 expectedDataLength = uint256(TRANSMIT_MSGDATA_CONSTANT_LENGTH_COMPONENT_NO_SIGNATURES) + report.length;

253:         expectedDataLength +=

254:           TRANSMIT_MSGDATA_EXTRA_CONSTANT_LENGTH_COMPONENT_FOR_SIGNATURES + rs.length * 32 + ss.length * 32;

285:         if (rs.length != configInfo.F + 1) revert WrongNumberOfSignatures();

313:     for (uint256 i; i < numberOfSignatures; ++i) {

315:       address signer = ecrecover(hashedReport, uint8(rawVs[i]) + 27, rs[i], ss[i]);

```

```solidity
File: ./src/ccip/offRamp/OffRamp.sol

4: import {ITypeAndVersion} from "../../shared/interfaces/ITypeAndVersion.sol";

5: import {IAny2EVMMessageReceiver} from "../interfaces/IAny2EVMMessageReceiver.sol";

6: import {IFeeQuoter} from "../interfaces/IFeeQuoter.sol";

7: import {IMessageInterceptor} from "../interfaces/IMessageInterceptor.sol";

8: import {INonceManager} from "../interfaces/INonceManager.sol";

9: import {IPoolV1} from "../interfaces/IPool.sol";

10: import {IRMNRemote} from "../interfaces/IRMNRemote.sol";

11: import {IRouter} from "../interfaces/IRouter.sol";

12: import {ITokenAdminRegistry} from "../interfaces/ITokenAdminRegistry.sol";

14: import {CallWithExactGas} from "../../shared/call/CallWithExactGas.sol";

15: import {Client} from "../libraries/Client.sol";

16: import {Internal} from "../libraries/Internal.sol";

17: import {MerkleMultiProof} from "../libraries/MerkleMultiProof.sol";

18: import {Pool} from "../libraries/Pool.sol";

19: import {MultiOCR3Base} from "../ocr/MultiOCR3Base.sol";

21: import {IERC20} from "../../vendor/openzeppelin-solidity/v5.0.2/contracts/token/ERC20/IERC20.sol";

22: import {ERC165Checker} from "../../vendor/openzeppelin-solidity/v5.0.2/contracts/utils/introspection/ERC165Checker.sol";

23: import {EnumerableSet} from "../../vendor/openzeppelin-solidity/v5.0.2/contracts/utils/structs/EnumerableSet.sol";

95:     uint64 chainSelector; // ────╮ Destination chainSelector

96:     IRMNRemote rmnRemote; // ────╯ RMN Verification Contract

97:     address tokenAdminRegistry; // Token admin registry address

98:     address nonceManager; // Nonce manager address

103:     IRouter router; // ───╮ Local router to use for messages coming from this source chain.

104:     bool isEnabled; //    │ Flag whether the source chain is enabled or not.

105:     uint64 minSeqNr; // ──╯ The min sequence number expected for future messages.

106:     bytes onRamp; // OnRamp address on the source chain.

112:     IRouter router; // ────────────╮  Local router to use for messages coming from this source chain.

113:     uint64 sourceChainSelector; // │  Source chain selector of the config to update.

114:     bool isEnabled; // ────────────╯  Flag whether the source chain is enabled or not.

115:     bytes onRamp; // OnRamp address on the source chain.

121:     address feeQuoter; // ─────────────────────────────╮ FeeQuoter address on the local chain.

122:     uint32 permissionLessExecutionThresholdSeconds; // │ Waiting time before manual execution is enabled.

123:     bool isRMNVerificationDisabled; // ────────────────╯ Flag whether the RMN verification is disabled or not.

124:     address messageInterceptor; // Optional, validates incoming messages (zero address = no interceptor).

130:     Internal.PriceUpdates priceUpdates; // Collection of gas and price updates to commit.

131:     Internal.MerkleRoot[] merkleRoots; // Collection of merkle roots per source chain to commit.

132:     IRMNRemote.Signature[] rmnSignatures; // RMN signatures on the merkle roots.

138:     uint256 receiverExecutionGasLimit; // Overrides EVM2EVMMessage.gasLimit.

139:     uint32[] tokenGasOverrides; // Overrides EVM2EVMMessage.sourceTokenData.destGasAmount, length must be same as tokenAmounts.

143:   string public constant override typeAndVersion = "OffRamp 1.6.0-dev";

209:   uint256 private constant MESSAGE_EXECUTION_STATE_MASK = (1 << MESSAGE_EXECUTION_STATE_BIT_WIDTH) - 1;

223:           >> ((sequenceNumber % 128) * MESSAGE_EXECUTION_STATE_BIT_WIDTH)

238:     uint256 offset = (sequenceNumber % 128) * MESSAGE_EXECUTION_STATE_BIT_WIDTH;

246:     s_executionStates[sourceChainSelector][sequenceNumber / 128] = bitmap;

257:     return s_executionStates[sourceChainSelector][sequenceNumber / 128];

278:     for (uint256 reportIndex = 0; reportIndex < numReports; ++reportIndex) {

288:       for (uint256 msgIndex = 0; msgIndex < numMsgs; ++msgIndex) {

305:         for (uint256 tokenIndex = 0; tokenIndex < message.tokenAmounts.length; ++tokenIndex) {

349:     for (uint256 i = 0; i < reports.length; ++i) {

395:       for (uint256 i = 0; i < numMsgs; ++i) {

420:     for (uint256 i = 0; i < numMsgs; ++i) {

445:           (block.timestamp - timestampCommitted) > s_dynamicConfig.permissionLessExecutionThresholdSeconds;

522:         gasStart - gasleft()

685:       (uint256 balancePost,) = _getBalanceOfReceiver(receiver, localToken, gasLeft - gasUsedReleaseOrMint);

688:       if (balancePost < balancePre || balancePost - balancePre != localAmount) {

723:     return (abi.decode(returnData, (uint256)), gasLimit - gasUsed);

745:     for (uint256 i = 0; i < sourceTokenAmounts.length; ++i) {

810:     for (uint256 i = 0; i < commitReport.merkleRoots.length; ++i) {

836:       sourceChainConfig.minSeqNr = root.maxSeqNr + 1;

935:     for (uint256 i = 0; i < s_sourceChainSelectors.length(); ++i) {

955:     for (uint256 i = 0; i < sourceChainConfigUpdates.length; ++i) {

```

```solidity
File: ./src/ccip/onRamp/OnRamp.sol

4: import {ITypeAndVersion} from "../../shared/interfaces/ITypeAndVersion.sol";

5: import {IEVM2AnyOnRampClient} from "../interfaces/IEVM2AnyOnRampClient.sol";

6: import {IFeeQuoter} from "../interfaces/IFeeQuoter.sol";

7: import {IMessageInterceptor} from "../interfaces/IMessageInterceptor.sol";

8: import {INonceManager} from "../interfaces/INonceManager.sol";

9: import {IPoolV1} from "../interfaces/IPool.sol";

10: import {IRMNRemote} from "../interfaces/IRMNRemote.sol";

11: import {IRouter} from "../interfaces/IRouter.sol";

12: import {ITokenAdminRegistry} from "../interfaces/ITokenAdminRegistry.sol";

14: import {Ownable2StepMsgSender} from "../../shared/access/Ownable2StepMsgSender.sol";

15: import {Client} from "../libraries/Client.sol";

16: import {Internal} from "../libraries/Internal.sol";

17: import {Pool} from "../libraries/Pool.sol";

18: import {USDPriceWith18Decimals} from "../libraries/USDPriceWith18Decimals.sol";

20: import {IERC20} from "../../vendor/openzeppelin-solidity/v4.8.3/contracts/token/ERC20/IERC20.sol";

21: import {SafeERC20} from "../../vendor/openzeppelin-solidity/v4.8.3/contracts/token/ERC20/utils/SafeERC20.sol";

22: import {EnumerableSet} from "../../vendor/openzeppelin-solidity/v5.0.2/contracts/utils/structs/EnumerableSet.sol";

62:     uint64 chainSelector; // ────╮ Source chain selector.

63:     IRMNRemote rmnRemote; // ────╯ RMN remote address.

64:     address nonceManager; //       Nonce manager address.

65:     address tokenAdminRegistry; // Token admin registry address.

71:     address feeQuoter; // FeeQuoter address.

72:     bool reentrancyGuardEntered; // Reentrancy protection.

73:     address messageInterceptor; // Optional message interceptor to validate messages. Zero address = no interceptor.

74:     address feeAggregator; // Fee aggregator address.

75:     address allowlistAdmin; // authorized admin to add or remove allowed senders.

82:     uint64 sequenceNumber; // ──╮ The last used sequence number.

83:     bool allowlistEnabled; //   │ True if the allowlist is enabled.

84:     IRouter router; // ─────────╯ Local router address  that is allowed to send messages to the destination chain.

85:     EnumerableSet.AddressSet allowedSendersList; // The list of addresses allowed to send messages.

92:     uint64 destChainSelector; // ─╮ Destination chain selector.

93:     IRouter router; //            │ Source router address.

94:     bool allowlistEnabled; // ────╯ True if the allowlist is enabled.

99:     uint64 destChainSelector; // ──╮ Destination chain selector.

100:     bool allowlistEnabled; // ─────╯ True if the allowlist is enabled.

101:     address[] addedAllowlistedSenders; // list of senders to be added to the allowedSendersList.

102:     address[] removedAllowlistedSenders; // list of senders to be removed from the allowedSendersList.

106:   string public constant override typeAndVersion = "OnRamp 1.6.0-dev";

154:     return s_destChainConfigs[destChainSelector].sequenceNumber + 1;

199:         sequenceNumber: ++destChainConfig.sequenceNumber,

210:       feeValueJuels: 0, // calculated later.

217:     for (uint256 i = 0; i < message.tokenAmounts.length; ++i) {

238:     for (uint256 i = 0; i < newMessage.tokenAmounts.length; ++i) {

298:       destExecData: "" // This is set in the processPoolReturnData function.

366:     for (uint256 i = 0; i < destChainConfigArgs.length; ++i) {

432:     for (uint256 i = 0; i < allowlistConfigArgsItems.length; ++i) {

440:           for (uint256 j = 0; j < allowlistConfigArgs.addedAllowlistedSenders.length; ++j) {

454:       for (uint256 j = 0; j < allowlistConfigArgs.removedAllowlistedSenders.length; ++j) {

471:   function getPoolBySourceToken(uint64, /*destChainSelector*/ IERC20 sourceToken) public view returns (IPoolV1) {

477:     uint64 // destChainSelector

508:     for (uint256 i = 0; i < feeTokens.length; ++i) {

```

```solidity
File: ./src/ccip/rmn/RMNHome.sol

4: import {ITypeAndVersion} from "../../shared/interfaces/ITypeAndVersion.sol";

6: import {Ownable2StepMsgSender} from "../../shared/access/Ownable2StepMsgSender.sol";

78:     bytes32 peerId; //            Used for p2p communication.

79:     bytes32 offchainPublicKey; // Observations are signed with this public key, and are only verified offchain.

83:     uint64 chainSelector; // ─╮ The Source chain selector.

84:     uint64 f; // ─────────────╯ Maximum number of faulty observers; f+1 observers required to agree on an observation for this source chain.

85:     uint256 observerNodesBitmap; // ObserverNodesBitmap & (1<<i) == (1<<i) iff StaticConfig.nodes[i] is an observer for this source chain.

92:     bytes offchainConfig; // Offchain configuration for RMN nodes.

98:     bytes offchainConfig; // Offchain configuration for RMN nodes.

110:   string public constant override typeAndVersion = "RMNHome 1.6.0-dev";

113:   uint256 private constant PREFIX = 0x000b << (256 - 16); // 0x000b00..00.

115:   uint256 private constant PREFIX_MASK = type(uint256).max << (256 - 16); // 0xFFFF00..00.

166:     for (uint256 i = 0; i < MAX_CONCURRENT_CONFIGS; ++i) {

222:     uint32 newVersion = ++s_currentVersion;

295:     for (uint256 i = 0; i < MAX_CONCURRENT_CONFIGS; ++i) {

349:     for (uint256 i = 0; i < staticConfig.nodes.length; ++i) {

350:       for (uint256 j = i + 1; j < staticConfig.nodes.length; ++j) {

368:     for (uint256 i = 0; i < numberOfSourceChains; ++i) {

371:       for (uint256 j = i + 1; j < numberOfSourceChains; ++j) {

380:       if (bitmap & (type(uint256).max >> (256 - numberOfNodes)) != bitmap) {

385:       for (; bitmap != 0; ++observersCount) {

386:         bitmap &= bitmap - 1;

390:       if (observersCount < 2 * currentSourceChain.f + 1) {

```

```solidity
File: ./src/ccip/rmn/RMNRemote.sol

4: import {ITypeAndVersion} from "../../shared/interfaces/ITypeAndVersion.sol";

5: import {IRMNRemote} from "../interfaces/IRMNRemote.sol";

7: import {Ownable2StepMsgSender} from "../../shared/access/Ownable2StepMsgSender.sol";

8: import {EnumerableSet} from "../../shared/enumerable/EnumerableSetWithBytes16.sol";

9: import {Internal} from "../libraries/Internal.sol";

43:     address onchainPublicKey; // ─╮ For signing reports.

44:     uint64 nodeIndex; // ─────────╯ Maps to nodes in home chain config, should be strictly increasing.

49:     bytes32 rmnHomeContractConfigDigest; // Digest of the RMNHome contract config.

50:     Signer[] signers; // List of signers.

51:     uint64 f; // Max number of faulty RMN nodes; f+1 signers are required to verify a report, must configure 2f+1 signers in total.

57:     uint256 destChainId; //                 To guard against chain selector misconfiguration.

58:     uint64 destChainSelector; //  ────────╮ The chain selector of the destination chain.

59:     address rmnRemoteContractAddress; // ─╯ The address of this contract.

60:     address offrampAddress; //              The address of the offramp on the same chain as this contract.

61:     bytes32 rmnHomeContractConfigDigest; // The digest of the RMNHome contract config.

62:     Internal.MerkleRoot[] merkleRoots; //   The dest lane updates.

68:   string public constant override typeAndVersion = "RMNRemote 1.6.0-dev";

80:   mapping(address signer => bool exists) private s_signers; // for more gas efficient verify.

103:     if (signatures.length < s_config.f + 1) revert ThresholdNotMet();

121:     for (uint256 i = 0; i < signatures.length; ++i) {

145:     for (uint256 i = 1; i < newConfig.signers.length; ++i) {

146:       if (!(newConfig.signers[i - 1].nodeIndex < newConfig.signers[i].nodeIndex)) {

152:     if (newConfig.signers.length < 2 * newConfig.f + 1) {

157:     for (uint256 i = s_config.signers.length; i > 0; --i) {

158:       delete s_signers[s_config.signers[i - 1].onchainPublicKey];

162:     for (uint256 i = 0; i < newConfig.signers.length; ++i) {

170:     uint32 newConfigCount = ++s_configCount;

213:     for (uint256 i = 0; i < subjects.length; ++i) {

237:     for (uint256 i = 0; i < subjects.length; ++i) {

```

### <a name="GAS-7"></a>[GAS-7] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including `EXTCODESIZE` (**100 gas**), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence

*Instances (1)*:
```solidity
File: ./src/ccip/onRamp/OnRamp.sol

510:       uint256 feeTokenBalance = feeToken.balanceOf(address(this));

```

### <a name="GAS-8"></a>[GAS-8] State variables only set in the constructor should be declared `immutable`
Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around **20 000 gas** per variable) and replace the expensive storage-reading operations (around **2100 gas** per reading) to a less expensive value reading (**3 gas**)

*Instances (1)*:
```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

122:   /// @notice Sets offchain reporting protocol configuration incl. participating oracles.

```

### <a name="GAS-9"></a>[GAS-9] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

*Instances (3)*:
```solidity
File: ./src/ccip/capability/CCIPHome.sol

520:   function _onlySelfCall() internal view {

```

```solidity
File: ./src/ccip/rmn/RMNHome.sol

265:   function promoteCandidateAndRevokeActive(bytes32 digestToPromote, bytes32 digestToRevoke) external onlyOwner {

294:   function setDynamicConfig(DynamicConfig calldata newDynamicConfig, bytes32 currentDigest) external onlyOwner {

```

### <a name="GAS-10"></a>[GAS-10] `++i` costs less gas compared to `i++` or `i += 1` (same for `--i` vs `i--` or `i -= 1`)
Pre-increments and pre-decrements are cheaper.

For a `uint256 i` variable, the following is true with the Optimizer enabled at 10k:

**Increment:**

- `i += 1` is the most expensive form
- `i++` costs 6 gas less than `i += 1`
- `++i` costs 5 gas less than `i++` (11 gas less than `i += 1`)

**Decrement:**

- `i -= 1` is the most expensive form
- `i--` costs 11 gas less than `i -= 1`
- `--i` costs 5 gas less than `i--` (16 gas less than `i -= 1`)

Note that post-increments (or post-decrements) return the old value before incrementing or decrementing, hence the name *post-increment*:

```solidity
uint i = 1;  
uint j = 2;
require(j == i++, "This will be false as i is incremented after the comparison");
```
  
However, pre-increments (or pre-decrements) return the new value:
  
```solidity
uint i = 1;  
uint j = 2;
require(j == ++i, "This will be true as i is incremented before the comparison");
```

In the pre-increment case, the compiler has to create a temporary variable (when used) for returning `1` instead of `2`.

Consider using pre-increments and pre-decrements where they are relevant (meaning: not where post-increments/decrements logic are relevant).

*Saves 5 gas per instance*

*Instances (1)*:
```solidity
File: ./src/ccip/capability/CCIPHome.sol

497:         nonZeroTransmitters++;

```

### <a name="GAS-11"></a>[GAS-11] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (17)*:
```solidity
File: ./src/ccip/FeeQuoter.sol

159:   uint256 public constant FEE_BASE_DECIMALS = 36;

161:   uint256 public constant KEYSTONE_PRICE_DECIMALS = 18;

163:   string public constant override typeAndVersion = "FeeQuoter 1.6.0-dev";

```

```solidity
File: ./src/ccip/MultiAggregateRateLimiter.sol

57:   string public constant override typeAndVersion = "MultiAggregateRateLimiter 1.6.0-dev";

```

```solidity
File: ./src/ccip/NonceManager.sol

32:   string public constant override typeAndVersion = "NonceManager 1.6.0-dev";

```

```solidity
File: ./src/ccip/capability/CCIPHome.sol

140:   string public constant override typeAndVersion = "CCIPHome 1.6.0-dev";

```

```solidity
File: ./src/ccip/libraries/Client.sol

30:   bytes4 public constant EVM_EXTRA_ARGS_V1_TAG = 0x97a657c9;

43:   bytes4 public constant EVM_EXTRA_ARGS_V2_TAG = 0x181dcf10;

```

```solidity
File: ./src/ccip/libraries/Internal.sol

50:   uint8 public constant GAS_PRICE_BITS = 112;

83:   uint256 public constant MESSAGE_FIXED_BYTES = 32 * 15;

95:   uint256 public constant MESSAGE_FIXED_BYTES_PER_TOKEN = 32 * (4 + (3 + 2));

159:   uint256 public constant PRECOMPILE_SPACE = 1024;

263:   bytes4 public constant CHAIN_FAMILY_SELECTOR_EVM = 0x2812d52c;

```

```solidity
File: ./src/ccip/offRamp/OffRamp.sol

143:   string public constant override typeAndVersion = "OffRamp 1.6.0-dev";

```

```solidity
File: ./src/ccip/onRamp/OnRamp.sol

106:   string public constant override typeAndVersion = "OnRamp 1.6.0-dev";

```

```solidity
File: ./src/ccip/rmn/RMNHome.sol

110:   string public constant override typeAndVersion = "RMNHome 1.6.0-dev";

```

```solidity
File: ./src/ccip/rmn/RMNRemote.sol

68:   string public constant override typeAndVersion = "RMNRemote 1.6.0-dev";

```

### <a name="GAS-12"></a>[GAS-12] Use shift right/left instead of division/multiplication if possible
While the `DIV` / `MUL` opcode uses 5 gas, the `SHR` / `SHL` opcode only uses 3 gas. Furthermore, beware that Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting. Eventually, overflow checks are never performed for shift operations as they are done for arithmetic operations. Instead, the result is always truncated, so the calculation can be unchecked in Solidity version `0.8+`
- Use `>> 1` instead of `/ 2`
- Use `>> 2` instead of `/ 4`
- Use `<< 3` instead of `* 8`
- ...
- Use `>> 5` instead of `/ 2^5 == / 32`
- Use `<< 6` instead of `* 2^6 == * 64`

TL;DR:
- Shifting left by N is like multiplying by 2^N (Each bits to the left is an increased power of 2)
- Shifting right by N is like dividing by 2^N (Each bits to the right is a decreased power of 2)

*Saves around 2 gas + 20 for unchecked per instance*

*Instances (5)*:
```solidity
File: ./src/ccip/libraries/Internal.sol

16:   uint16 internal constant MAX_RET_BYTES = 4 + 4 * 32;

```

```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

104:     + 3 * 32 // 3 words containing reportContext.

254:           TRANSMIT_MSGDATA_EXTRA_CONSTANT_LENGTH_COMPONENT_FOR_SIGNATURES + rs.length * 32 + ss.length * 32;

```

```solidity
File: ./src/ccip/offRamp/OffRamp.sol

246:     s_executionStates[sourceChainSelector][sequenceNumber / 128] = bitmap;

257:     return s_executionStates[sourceChainSelector][sequenceNumber / 128];

```

### <a name="GAS-13"></a>[GAS-13] Superfluous event fields
`block.timestamp` and `block.number` are added to event information by default so adding them manually wastes gas

*Instances (2)*:
```solidity
File: ./src/ccip/FeeQuoter.sol

49:   event UsdPerUnitGasUpdated(uint64 indexed destChain, uint256 value, uint256 timestamp);

50:   event UsdPerTokenUpdated(address indexed token, uint256 value, uint256 timestamp);

```

### <a name="GAS-14"></a>[GAS-14] Use of `this` instead of marking as `public` an `external` function
Using `this.` is like making an expensive external call. Consider marking the called function as public

*Saves around 2000 gas per instance*

*Instances (1)*:
```solidity
File: ./src/ccip/offRamp/OffRamp.sol

537:     try this.executeSingleMessage(message, offchainTokenData, tokenGasOverrides) {}

```

### <a name="GAS-15"></a>[GAS-15] Increments/decrements can be unchecked in for-loops
In Solidity 0.8+, there's a default overflow check on unsigned integers. It's possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

[ethereum/solidity#10695](https://github.com/ethereum/solidity/issues/10695)

The change would be:

```diff
- for (uint256 i; i < numIterations; i++) {
+ for (uint256 i; i < numIterations;) {
 // ...  
+   unchecked { ++i; }
}  
```

These save around **25 gas saved** per instance.

The same can be applied with decrements (which should use `break` when `i == 0`).

The risk of overflow is non-existent for `uint256`.

*Instances (59)*:
```solidity
File: ./src/ccip/FeeQuoter.sol

279:     for (uint256 i = 0; i < length; ++i) {

431:     for (uint256 i = 0; i < feeTokensToRemove.length; ++i) {

436:     for (uint256 i = 0; i < feeTokensToAdd.length; ++i) {

456:     for (uint256 i = 0; i < tokenUpdatesLength; ++i) {

465:     for (uint256 i = 0; i < gasUpdatesLength; ++i) {

486:     for (uint256 i; i < tokenPriceFeedUpdates.length; ++i) {

508:     for (uint256 i = 0; i < feeds.length; ++i) {

617:     for (uint256 i = 0; i < premiumMultiplierWeiPerEthArgs.length; ++i) {

659:     for (uint256 i = 0; i < numberOfTokens; ++i) {

798:     for (uint256 i = 0; i < tokenTransferFeeConfigArgs.length; ++i) {

802:       for (uint256 j = 0; j < tokenTransferFeeConfigArg.tokenTransferFeeConfigs.length; ++j) {

822:     for (uint256 i = 0; i < tokensToUseDefaultFeeConfigs.length; ++i) {

966:     for (uint256 i = 0; i < onRampTokenTransfers.length; ++i) {

1021:     for (uint256 i = 0; i < destChainConfigArgs.length; ++i) {

```

```solidity
File: ./src/ccip/MultiAggregateRateLimiter.sol

107:       for (uint256 i = 0; i < tokenAmounts.length; ++i) {

165:     for (uint256 i = 0; i < rateLimiterUpdates.length; ++i) {

215:     for (uint256 i = 0; i < tokenCount; ++i) {

230:     for (uint256 i = 0; i < removes.length; ++i) {

239:     for (uint256 i = 0; i < adds.length; ++i) {

```

```solidity
File: ./src/ccip/NonceManager.sol

132:     for (uint256 i = 0; i < previousRampsArgs.length; ++i) {

```

```solidity
File: ./src/ccip/capability/CCIPHome.sol

294:     for (uint256 i = 0; i < MAX_CONCURRENT_CONFIGS; ++i) {

491:     for (uint256 i = 0; i < numberOfNodes; ++i) {

555:     for (uint256 i = startIndex; i < endIndex; ++i) {

574:     for (uint256 i = 0; i < chainSelectorRemoves.length; ++i) {

587:     for (uint256 i = 0; i < chainConfigAdds.length; ++i) {

```

```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

132:     for (uint256 i; i < ocrConfigArgs.length; ++i) {

202:     for (uint256 i = 0; i < oracleAddresses.length; ++i) {

212:     for (uint256 i = 0; i < oracleAddresses.length; ++i) {

313:     for (uint256 i; i < numberOfSignatures; ++i) {

```

```solidity
File: ./src/ccip/offRamp/OffRamp.sol

278:     for (uint256 reportIndex = 0; reportIndex < numReports; ++reportIndex) {

288:       for (uint256 msgIndex = 0; msgIndex < numMsgs; ++msgIndex) {

305:         for (uint256 tokenIndex = 0; tokenIndex < message.tokenAmounts.length; ++tokenIndex) {

349:     for (uint256 i = 0; i < reports.length; ++i) {

395:       for (uint256 i = 0; i < numMsgs; ++i) {

420:     for (uint256 i = 0; i < numMsgs; ++i) {

745:     for (uint256 i = 0; i < sourceTokenAmounts.length; ++i) {

810:     for (uint256 i = 0; i < commitReport.merkleRoots.length; ++i) {

935:     for (uint256 i = 0; i < s_sourceChainSelectors.length(); ++i) {

955:     for (uint256 i = 0; i < sourceChainConfigUpdates.length; ++i) {

```

```solidity
File: ./src/ccip/onRamp/OnRamp.sol

217:     for (uint256 i = 0; i < message.tokenAmounts.length; ++i) {

238:     for (uint256 i = 0; i < newMessage.tokenAmounts.length; ++i) {

366:     for (uint256 i = 0; i < destChainConfigArgs.length; ++i) {

432:     for (uint256 i = 0; i < allowlistConfigArgsItems.length; ++i) {

440:           for (uint256 j = 0; j < allowlistConfigArgs.addedAllowlistedSenders.length; ++j) {

454:       for (uint256 j = 0; j < allowlistConfigArgs.removedAllowlistedSenders.length; ++j) {

508:     for (uint256 i = 0; i < feeTokens.length; ++i) {

```

```solidity
File: ./src/ccip/rmn/RMNHome.sol

166:     for (uint256 i = 0; i < MAX_CONCURRENT_CONFIGS; ++i) {

295:     for (uint256 i = 0; i < MAX_CONCURRENT_CONFIGS; ++i) {

349:     for (uint256 i = 0; i < staticConfig.nodes.length; ++i) {

350:       for (uint256 j = i + 1; j < staticConfig.nodes.length; ++j) {

368:     for (uint256 i = 0; i < numberOfSourceChains; ++i) {

371:       for (uint256 j = i + 1; j < numberOfSourceChains; ++j) {

385:       for (; bitmap != 0; ++observersCount) {

```

```solidity
File: ./src/ccip/rmn/RMNRemote.sol

121:     for (uint256 i = 0; i < signatures.length; ++i) {

145:     for (uint256 i = 1; i < newConfig.signers.length; ++i) {

157:     for (uint256 i = s_config.signers.length; i > 0; --i) {

162:     for (uint256 i = 0; i < newConfig.signers.length; ++i) {

213:     for (uint256 i = 0; i < subjects.length; ++i) {

237:     for (uint256 i = 0; i < subjects.length; ++i) {

```

### <a name="GAS-16"></a>[GAS-16] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (11)*:
```solidity
File: ./src/ccip/FeeQuoter.sol

559:     if (numberOfTokens > 0) {

573:     if (destChainConfig.destDataAvailabilityMultiplierBps > 0) {

674:       if (transferFeeConfig.deciBps > 0) {

```

```solidity
File: ./src/ccip/MultiAggregateRateLimiter.sol

113:       if (value > 0) tokenBucket._consume(value, address(0));

```

```solidity
File: ./src/ccip/offRamp/OffRamp.sol

561:     if (message.tokenAmounts.length > 0) {

787:       if (commitReport.merkleRoots.length > 0) {

793:     if (commitReport.priceUpdates.tokenPriceUpdates.length > 0 || commitReport.priceUpdates.gasPriceUpdates.length > 0)

```

```solidity
File: ./src/ccip/onRamp/OnRamp.sol

438:       if (allowlistConfigArgs.addedAllowlistedSenders.length > 0) {

458:       if (allowlistConfigArgs.removedAllowlistedSenders.length > 0) {

512:       if (feeTokenBalance > 0) {

```

```solidity
File: ./src/ccip/rmn/RMNRemote.sol

157:     for (uint256 i = s_config.signers.length; i > 0; --i) {

```

### <a name="GAS-17"></a>[GAS-17] `internal` functions not called by the contract should be removed
If the functions are required by an interface, the contract should inherit from that interface and use the `override` keyword

*Instances (6)*:
```solidity
File: ./src/ccip/libraries/Client.sol

36:   function _argsToBytes(

54:   function _argsToBytes(

```

```solidity
File: ./src/ccip/libraries/Internal.sol

106:   function _hash(Any2EVMRampMessage memory original, bytes32 metadataHash) internal pure returns (bytes32) {

129:   function _hash(EVM2AnyRampMessage memory original, bytes32 metadataHash) internal pure returns (bytes32) {

165:   function _validateEVMAddress(

```

```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

231:     // NOTE: If these parameters are changed, expectedMsgDataLength and/or TRANSMIT_MSGDATA_CONSTANT_LENGTH_COMPONENT

```


## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Replace `abi.encodeWithSignature` and `abi.encodeWithSelector` with `abi.encodeCall` which keeps the code typo/type safe | 2 |
| [NC-2](#NC-2) | Array indices should be referenced via `enum`s rather than via numeric literals | 2 |
| [NC-3](#NC-3) | Use `string.concat()` or `bytes.concat()` instead of `abi.encodePacked` | 1 |
| [NC-4](#NC-4) | `constant`s should be defined rather than using magic numbers | 19 |
| [NC-5](#NC-5) | Control structures do not follow the Solidity Style Guide | 99 |
| [NC-6](#NC-6) | Default Visibility for constants | 2 |
| [NC-7](#NC-7) | Consider disabling `renounceOwnership()` | 5 |
| [NC-8](#NC-8) | Event missing indexed field | 1 |
| [NC-9](#NC-9) | Function ordering does not follow the Solidity style guide | 1 |
| [NC-10](#NC-10) | Functions should not be longer than 50 lines | 48 |
| [NC-11](#NC-11) | Change int to int256 | 2 |
| [NC-12](#NC-12) | Change uint to uint256 | 2 |
| [NC-13](#NC-13) | Lack of checks in setters | 1 |
| [NC-14](#NC-14) | Lines are too long | 1 |
| [NC-15](#NC-15) | Missing Event for critical parameters change | 1 |
| [NC-16](#NC-16) | Use a `modifier` instead of a `require/if` statement for a special `msg.sender` actor | 6 |
| [NC-17](#NC-17) | Adding a `return` statement when the function defines a named return variable, is redundant | 3 |
| [NC-18](#NC-18) | Take advantage of Custom Error's return value property | 71 |
| [NC-19](#NC-19) | Contract does not follow the Solidity style guide's suggested layout ordering | 3 |
| [NC-20](#NC-20) | Use Underscores for Number Literals (add an underscore every 3 digits) | 1 |
| [NC-21](#NC-21) | Internal and private variables and functions names should begin with an underscore | 2 |
| [NC-22](#NC-22) | Event is missing `indexed` fields | 2 |
| [NC-23](#NC-23) | Variables need not be initialized to zero | 62 |
### <a name="NC-1"></a>[NC-1] Replace `abi.encodeWithSignature` and `abi.encodeWithSelector` with `abi.encodeCall` which keeps the code typo/type safe
When using `abi.encodeWithSignature`, it is possible to include a typo for the correct function signature.
When using `abi.encodeWithSignature` or `abi.encodeWithSelector`, it is also possible to provide parameters that are not of the correct type for the function.

To avoid these pitfalls, it would be best to use [`abi.encodeCall`](https://solidity-by-example.org/abi-encode/) instead.

*Instances (2)*:
```solidity
File: ./src/ccip/libraries/Client.sol

39:     return abi.encodeWithSelector(EVM_EXTRA_ARGS_V1_TAG, extraArgs);

57:     return abi.encodeWithSelector(EVM_EXTRA_ARGS_V2_TAG, extraArgs);

```

### <a name="NC-2"></a>[NC-2] Array indices should be referenced via `enum`s rather than via numeric literals

*Instances (2)*:
```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

248:       // one byte per entry in _report

297:   /// @param ocrPluginType OCR plugin type to transmit report for.

```

### <a name="NC-3"></a>[NC-3] Use `string.concat()` or `bytes.concat()` instead of `abi.encodePacked`
Solidity version 0.8.4 introduces `bytes.concat()` (vs `abi.encodePacked(<bytes>,<bytes>)`)

Solidity version 0.8.12 introduces `string.concat()` (vs `abi.encodePacked(<str>,<str>), which catches concatenation errors (in the event of a `bytes` data mixed in the concatenation)`)

*Instances (1)*:
```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

289:       bytes32 h = keccak256(abi.encodePacked(keccak256(report), reportContext));

```

### <a name="NC-4"></a>[NC-4] `constant`s should be defined rather than using magic numbers
Even [assembly](https://github.com/code-423n4/2022-05-opensea-seaport/blob/9d7ce4d08bf3c3010304a0476a785c70c0e90ae7/contracts/lib/TokenTransferrer.sol#L35-L39) can benefit from using readable constants instead of hex/numeric literals

*Instances (19)*:
```solidity
File: ./src/ccip/capability/CCIPHome.sol

487:     if (numberOfNodes <= 3 * FRoleDON) revert FTooHigh();

511:     uint256 minTransmittersLength = 3 * fChain + 1;

```

```solidity
File: ./src/ccip/libraries/Internal.sol

168:     if (encodedAddress.length != 32) revert InvalidEVMAddress(encodedAddress);

```

```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

104:     + 3 * 32 // 3 words containing reportContext.

105:     + 32 // word containing start location of abiencoded report value.

106:     + 32; // word containing length of report.

111:     + 32 // word containing start location of abiencoded ss value.

112:     + 32 // rawVs value.

113:     + 32 // word containing length rs.

114:     + 32; // word containing length of ss.

169:       if (signers.length <= 3 * ocrConfigArgs.F) revert InvalidConfig(InvalidConfigErrorType.F_TOO_HIGH);

254:           TRANSMIT_MSGDATA_EXTRA_CONSTANT_LENGTH_COMPONENT_FOR_SIGNATURES + rs.length * 32 + ss.length * 32;

315:       address signer = ecrecover(hashedReport, uint8(rawVs[i]) + 27, rs[i], ss[i]);

```

```solidity
File: ./src/ccip/offRamp/OffRamp.sol

223:           >> ((sequenceNumber % 128) * MESSAGE_EXECUTION_STATE_BIT_WIDTH)

238:     uint256 offset = (sequenceNumber % 128) * MESSAGE_EXECUTION_STATE_BIT_WIDTH;

246:     s_executionStates[sourceChainSelector][sequenceNumber / 128] = bitmap;

257:     return s_executionStates[sourceChainSelector][sequenceNumber / 128];

```

```solidity
File: ./src/ccip/rmn/RMNHome.sol

390:       if (observersCount < 2 * currentSourceChain.f + 1) {

```

```solidity
File: ./src/ccip/rmn/RMNRemote.sol

152:     if (newConfig.signers.length < 2 * newConfig.f + 1) {

```

### <a name="NC-5"></a>[NC-5] Control structures do not follow the Solidity Style Guide
See the [control structures](https://docs.soliditylang.org/en/latest/style-guide.html#control-structures) section of the Solidity Style Guide

*Instances (99)*:
```solidity
File: ./src/ccip/FeeQuoter.sol

6: import {IFeeQuoter} from "./interfaces/IFeeQuoter.sol";

110:     bytes4 chainFamilySelector; // ──────────────╯ Selector that identifies the destination chain's family. Used to determine the correct validations to perform for the dest chain.

155:     uint64 premiumMultiplierWeiPerEth; // ─╯ Multiplier for destination chain specific premiums.

216:     if (

318:     if (!s_destChainConfigs[destChainSelector].isEnabled) revert DestinationChainNotEnabled(destChainSelector);

354:     if (tokenPrice.timestamp == 0 || tokenPrice.value == 0) revert TokenNotSupported(token);

542:     if (!destChainConfig.isEnabled) revert DestinationChainNotEnabled(destChainSelector);

543:     if (!s_feeTokens.contains(message.feeToken)) revert FeeTokenNotSupported(message.feeToken);

855:     if (evmExtraArgs.gasLimit > uint256(destChainConfig.maxPerMsgGasLimit)) revert MessageGasLimitTooHigh();

940:     if (msgFeeJuels > i_maxFeeJuelsPerMsg) revert MessageFeeTooHigh(msgFeeJuels, i_maxFeeJuelsPerMsg);

1029:       if (

```

```solidity
File: ./src/ccip/MultiAggregateRateLimiter.sol

5: import {IFeeQuoter} from "./interfaces/IFeeQuoter.sol";

47:     bool isOutboundLane; // ────────╯ If set to true, represents the outbound message lane (OnRamp), and the inbound message lane otherwise (OffRamp)

113:       if (value > 0) tokenBucket._consume(value, address(0));

141:     uint224 pricePerToken = IFeeQuoter(s_feeQuoter).getTokenPrice(tokenAmount.token).value;

142:     if (pricePerToken == 0) revert PriceNotFoundForToken(tokenAmount.token);

```

```solidity
File: ./src/ccip/capability/CCIPHome.sol

131:     bytes config; // The chain configuration. This is kept intentionally opaque so as to add fields in the future if needed.

220:     if (

462:     if (cfg.chainSelector == 0) revert ChainSelectorNotSet();

472:     if (!s_remoteChainSelectors.contains(cfg.chainSelector)) revert ChainSelectorNotFound(cfg.chainSelector);

486:     if (numberOfNodes > MAX_NUM_ORACLES) revert TooManySigners();

487:     if (numberOfNodes <= 3 * FRoleDON) revert FTooHigh();

545:       return new ChainConfigArgs[](0); // Return an empty array if pageSize is 0 or pageIndex is out of bounds.

```

```solidity
File: ./src/ccip/interfaces/IRMNRemote.sol

19:   function verify(

```

```solidity
File: ./src/ccip/libraries/Client.sol

15:     bytes sender; // abi.decode(sender) if coming from an EVM chain.

```

```solidity
File: ./src/ccip/libraries/Internal.sol

168:     if (encodedAddress.length != 32) revert InvalidEVMAddress(encodedAddress);

198:     bytes32 messageId; // Unique identifier for the message, generated with the source chain's encoding scheme (i.e. not necessarily abi.encoded).

255:     bytes extraArgs; // destination-chain specific extra args, such as the gasLimit for EVM chains.

```

```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

52:     bool isSignatureVerificationEnabled; // ─╯ if true, requires signers and verifies signatures on transmission.

86:     bool isSignatureVerificationEnabled; // ─╯ If true, requires signers and verifies signatures on transmission.

142:     if (ocrConfigArgs.F == 0) revert InvalidConfig(InvalidConfigErrorType.F_MUST_BE_POSITIVE);

150:       configInfo.isSignatureVerificationEnabled = ocrConfigArgs.isSignatureVerificationEnabled;

158:     if (transmitters.length > MAX_NUM_ORACLES) revert InvalidConfig(InvalidConfigErrorType.TOO_MANY_TRANSMITTERS);

159:     if (transmitters.length == 0) revert InvalidConfig(InvalidConfigErrorType.NO_TRANSMITTERS);

168:       if (signers.length > MAX_NUM_ORACLES) revert InvalidConfig(InvalidConfigErrorType.TOO_MANY_SIGNERS);

169:       if (signers.length <= 3 * ocrConfigArgs.F) revert InvalidConfig(InvalidConfigErrorType.F_TOO_HIGH);

172:       if (signers.length < transmitters.length) revert InvalidConfig(InvalidConfigErrorType.TOO_MANY_TRANSMITTERS);

217:       if (oracle == address(0)) revert OracleCannotBeZeroAddress();

257:       if (msg.data.length != expectedDataLength) revert WrongMessageLength(expectedDataLength, msg.data.length);

272:       if (

285:         if (rs.length != configInfo.F + 1) revert WrongNumberOfSignatures();

286:         if (rs.length != ss.length) revert SignaturesOutOfRegistration();

290:       _verifySignatures(ocrPluginType, h, rs, ss, rawVs);

302:   function _verifySignatures(

318:       if (oracle.role != Role.Signer) revert UnauthorizedSigner();

319:       if (signed & (0x1 << oracle.index) != 0) revert NonUniqueSignatures();

326:     if (i_chainID != block.chainid) revert ForkedChain(i_chainID, block.chainid);

```

```solidity
File: ./src/ccip/offRamp/OffRamp.sol

6: import {IFeeQuoter} from "../interfaces/IFeeQuoter.sol";

64:   error SignatureVerificationRequiredInCommitPlugin();

65:   error SignatureVerificationNotAllowedInExecutionPlugin();

96:     IRMNRemote rmnRemote; // ────╯ RMN Verification Contract

123:     bool isRMNVerificationDisabled; // ────────────────╯ Flag whether the RMN verification is disabled or not.

181:     if (

276:     if (numReports != gasLimitOverrides.length) revert ManualExecutionGasLimitMismatch();

286:       if (numMsgs != msgGasLimitOverrides.length) revert ManualExecutionGasLimitMismatch();

343:     if (reports.length == 0) revert EmptyBatch();

376:     if (numMsgs == 0) revert EmptyReport(report.sourceChainSelector);

377:     if (numMsgs != report.offchainTokenData.length) revert UnexpectedTokenData();

416:     uint256 timestampCommitted = _verify(sourceChainSelector, hashedLeaves, report.proofs, report.proofFlagBits);

417:     if (timestampCommitted == 0) revert RootNotCommitted(sourceChainSelector);

429:       if (

474:           if (

559:     if (msg.sender != address(this)) revert CanOnlySelfCall();

597:     if (

606:     if (!success) revert ReceiverError(returnData);

673:     if (!success) revert TokenHandlingError(returnData);

714:     if (!success) revert TokenHandlingError(returnData);

788:         i_rmnRemote.verify(address(this), commitReport.merkleRoots, commitReport.rmnSignatures);

793:     if (commitReport.priceUpdates.tokenPriceUpdates.length > 0 || commitReport.priceUpdates.gasPriceUpdates.length > 0)

802:         IFeeQuoter(dynamicConfig.feeQuoter).updatePrices(commitReport.priceUpdates);

806:         if (commitReport.merkleRoots.length == 0) revert StaleCommitReport();

829:       if (merkleRoot == bytes32(0)) revert InvalidRoot();

865:   function _verify(

879:     bool isSignatureVerificationEnabled = s_ocrConfigs[ocrPluginType].configInfo.isSignatureVerificationEnabled;

884:         revert SignatureVerificationRequiredInCommitPlugin();

893:         revert SignatureVerificationNotAllowedInExecutionPlugin();

```

```solidity
File: ./src/ccip/onRamp/OnRamp.sol

6: import {IFeeQuoter} from "../interfaces/IFeeQuoter.sol";

83:     bool allowlistEnabled; //   │ True if the allowlist is enabled.

94:     bool allowlistEnabled; // ────╯ True if the allowlist is enabled.

100:     bool allowlistEnabled; // ─────╯ True if the allowlist is enabled.

128:     if (

166:     if (s_dynamicConfig.reentrancyGuardEntered) revert ReentrancyGuardReentrantCall();

173:     if (originalSender == address(0)) revert RouterMustSetOriginalSender();

182:     if (msg.sender != address(destChainConfig.router)) revert MustBeCalledByRouter();

227:     (newMessage.feeValueJuels, isOutOfOrderExecution, convertedExtraArgs, destExecDataPerToken) = IFeeQuoter(

272:     if (tokenAndAmount.amount == 0) revert CannotSendZeroTokens();

336:     if (

495:     if (i_rmnRemote.isCursed(bytes16(uint128(destChainSelector)))) revert CursedByRMN(destChainSelector);

497:     return IFeeQuoter(s_dynamicConfig.feeQuoter).getValidatedFee(destChainSelector, message);

```

```solidity
File: ./src/ccip/rmn/RMNHome.sol

79:     bytes32 offchainPublicKey; // Observations are signed with this public key, and are only verified offchain.

85:     uint256 observerNodesBitmap; // ObserverNodesBitmap & (1<<i) == (1<<i) iff StaticConfig.nodes[i] is an observer for this source chain.

```

```solidity
File: ./src/ccip/rmn/RMNRemote.sol

51:     uint64 f; // Max number of faulty RMN nodes; f+1 signers are required to verify a report, must configure 2f+1 signers in total.

80:   mapping(address signer => bool exists) private s_signers; // for more gas efficient verify.

86:     if (localChainSelector == 0) revert ZeroValueNotAllowed();

95:   function verify(

103:     if (signatures.length < s_config.f + 1) revert ThresholdNotMet();

123:       if (signerAddress == address(0)) revert InvalidSignature();

124:       if (prevAddress >= signerAddress) revert OutOfOrderSignatures();

125:       if (!s_signers[signerAddress]) revert UnexpectedSigner();

```

### <a name="NC-6"></a>[NC-6] Default Visibility for constants
Some constants are using the default visibility. For readability, consider explicitly declaring them as `internal`.

*Instances (2)*:
```solidity
File: ./src/ccip/rmn/RMNRemote.sol

14: bytes16 constant LEGACY_CURSE_SUBJECT = 0x01000000000000000000000000000000;

19: bytes16 constant GLOBAL_CURSE_SUBJECT = 0x01000000000000000000000000000001;

```

### <a name="NC-7"></a>[NC-7] Consider disabling `renounceOwnership()`
If the plan for your project does not include eventually giving up all ownership control, consider overwriting OpenZeppelin's `Ownable`'s `renounceOwnership()` function in order to disable it.

*Instances (5)*:
```solidity
File: ./src/ccip/capability/CCIPHome.sol

67: contract CCIPHome is Ownable2StepMsgSender, ITypeAndVersion, ICapabilityConfiguration, IERC165 {

```

```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

8: abstract contract MultiOCR3Base is ITypeAndVersion, Ownable2StepMsgSender {

```

```solidity
File: ./src/ccip/onRamp/OnRamp.sol

27: contract OnRamp is IEVM2AnyOnRampClient, ITypeAndVersion, Ownable2StepMsgSender {

```

```solidity
File: ./src/ccip/rmn/RMNHome.sol

59: contract RMNHome is Ownable2StepMsgSender, ITypeAndVersion {

```

```solidity
File: ./src/ccip/rmn/RMNRemote.sol

22: contract RMNRemote is Ownable2StepMsgSender, ITypeAndVersion, IRMNRemote {

```

### <a name="NC-8"></a>[NC-8] Event missing indexed field
Index event fields make the field more quickly accessible [to off-chain tools](https://ethereum.stackexchange.com/questions/40396/can-somebody-please-explain-the-concept-of-event-indexing) that parse events. This is especially useful when it comes to filtering based on an address. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Where applicable, each `event` should use three `indexed` fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three applicable fields, all of the applicable fields should be indexed.

*Instances (1)*:
```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

18:   event ConfigSet(uint8 ocrPluginType, bytes32 configDigest, address[] signers, address[] transmitters, uint8 F);

```

### <a name="NC-9"></a>[NC-9] Function ordering does not follow the Solidity style guide
According to the [Solidity style guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions), functions should be laid out in the following order :`constructor()`, `receive()`, `fallback()`, `external`, `public`, `internal`, `private`, but the cases below do not follow this pattern

*Instances (1)*:
```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

1: 
   Current order:
   external setOCR3Configs
   internal _setOCR3Config
   internal _afterOCR3ConfigSet
   internal _clearOracleRoles
   internal _assignOracleRoles
   internal _transmit
   internal _verifySignatures
   internal _whenChainNotForked
   external latestConfigDetails
   
   Suggested order:
   external setOCR3Configs
   external latestConfigDetails
   internal _setOCR3Config
   internal _afterOCR3ConfigSet
   internal _clearOracleRoles
   internal _assignOracleRoles
   internal _transmit
   internal _verifySignatures
   internal _whenChainNotForked

```

### <a name="NC-10"></a>[NC-10] Functions should not be longer than 50 lines
Overly complex code can make understanding functionality more difficult, try to further modularize your code to ensure readability 

*Instances (48)*:
```solidity
File: ./src/ccip/FeeQuoter.sol

411:   function getFeeTokens() external view returns (address[] memory) {

430:   function _applyFeeTokensUpdates(address[] memory feeTokensToRemove, address[] memory feeTokensToAdd) private {

501:   function onReport(bytes calldata metadata, bytes calldata report) external {

838:   function _validateDestFamilyAddress(bytes4 chainFamilySelector, bytes memory destAddress) internal pure {

1052:   function getStaticConfig() external view returns (StaticConfig memory) {

```

```solidity
File: ./src/ccip/MultiAggregateRateLimiter.sol

257:   function getFeeQuoter() external view returns (address feeQuoter) {

```

```solidity
File: ./src/ccip/NonceManager.sol

62:   function getOutboundNonce(uint64 destChainSelector, address sender) external view returns (uint64) {

66:   function _getOutboundNonce(uint64 destChainSelector, address sender) private view returns (uint64) {

105:   function getInboundNonce(uint64 sourceChainSelector, bytes calldata sender) external view returns (uint64) {

109:   function _getInboundNonce(uint64 sourceChainSelector, bytes calldata sender) private view returns (uint64) {

```

```solidity
File: ./src/ccip/capability/CCIPHome.sol

192:   function getCapabilityRegistry() external view returns (address) {

273:   function getActiveDigest(uint32 donId, Internal.OCRPluginType pluginType) public view returns (bytes32) {

279:   function getCandidateDigest(uint32 donId, Internal.OCRPluginType pluginType) public view returns (bytes32) {

369:   function revokeCandidate(uint32 donId, Internal.OCRPluginType pluginType, bytes32 configDigest) external {

447:   function _getActiveIndex(uint32 donId, Internal.OCRPluginType pluginType) private view returns (uint32) {

451:   function _getCandidateIndex(uint32 donId, Internal.OCRPluginType pluginType) private view returns (uint32) {

532:   function getNumChainConfigurations() external view returns (uint256) {

540:   function getAllChainConfigs(uint256 pageIndex, uint256 pageSize) external view returns (ChainConfigArgs[] memory) {

```

```solidity
File: ./src/ccip/interfaces/IMessageInterceptor.sol

23:   function onOutboundMessage(uint64 destChainSelector, Client.EVM2AnyMessage memory message) external;

```

```solidity
File: ./src/ccip/interfaces/INonceManager.sol

10:   function getIncrementedOutboundNonce(uint64 destChainSelector, address sender) external returns (uint64);

```

```solidity
File: ./src/ccip/interfaces/IRMNRemote.sol

27:   function getCursedSubjects() external view returns (bytes16[] memory subjects);

```

```solidity
File: ./src/ccip/libraries/Internal.sol

106:   function _hash(Any2EVMRampMessage memory original, bytes32 metadataHash) internal pure returns (bytes32) {

129:   function _hash(EVM2AnyRampMessage memory original, bytes32 metadataHash) internal pure returns (bytes32) {

```

```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

201:   function _clearOracleRoles(uint8 ocrPluginType, address[] memory oracleAddresses) internal {

211:   function _assignOracleRoles(uint8 ocrPluginType, address[] memory oracleAddresses, Role role) internal {

```

```solidity
File: ./src/ccip/offRamp/OffRamp.sol

325:   function execute(bytes32[3] calldata reportContext, bytes calldata report) external {

847:   function getLatestPriceSequenceNumber() external view returns (uint64) {

856:   function getMerkleRoot(uint64 sourceChainSelector, bytes32 root) external view returns (uint256) {

906:   function getStaticConfig() external view returns (StaticConfig memory) {

917:   function getDynamicConfig() external view returns (DynamicConfig memory) {

932:   function getAllSourceChainConfigs() external view returns (uint64[] memory, SourceChainConfig[] memory) {

```

```solidity
File: ./src/ccip/onRamp/OnRamp.sol

309:   function getStaticConfig() external view returns (StaticConfig memory) {

320:   function getDynamicConfig() external view returns (DynamicConfig memory dynamicConfig) {

471:   function getPoolBySourceToken(uint64, /*destChainSelector*/ IERC20 sourceToken) public view returns (IPoolV1) {

```

```solidity
File: ./src/ccip/rmn/RMNHome.sol

144:   function getConfigDigests() external view returns (bytes32 activeConfigDigest, bytes32 candidateConfigDigest) {

149:   function getActiveDigest() external view returns (bytes32) {

154:   function getCandidateDigest() public view returns (bytes32) {

265:   function promoteCandidateAndRevokeActive(bytes32 digestToPromote, bytes32 digestToRevoke) external onlyOwner {

294:   function setDynamicConfig(DynamicConfig calldata newDynamicConfig, bytes32 currentDigest) external onlyOwner {

313:   function _calculateConfigDigest(bytes memory staticConfig, uint32 version) internal view returns (bytes32) {

324:   function _getActiveIndex() private view returns (uint32) {

328:   function _getCandidateIndex() private view returns (uint32) {

366:   function _validateDynamicConfig(DynamicConfig memory dynamicConfig, uint256 numberOfNodes) internal pure {

```

```solidity
File: ./src/ccip/rmn/RMNRemote.sol

177:   function getVersionedConfig() external view returns (uint32 version, Config memory config) {

183:   function getLocalChainSelector() external view returns (uint64 localChainSelector) {

189:   function getReportDigestHeader() external pure returns (bytes32 digestHeader) {

246:   function getCursedSubjects() external view returns (bytes16[] memory subjects) {

251:   function isCursed() external view returns (bool) {

```

### <a name="NC-11"></a>[NC-11] Change int to int256
Throughout the code base, some variables are declared as `int`. To favor explicitness, consider changing all instances of `int` to `int256`

*Instances (2)*:
```solidity
File: ./src/ccip/libraries/Internal.sol

63:     uint32 destGasAmount; // The amount of gas available for the releaseOrMint and balanceOf calls on the offRamp

227:     uint32 destGasAmount; //──────╯ The amount of gas available for the releaseOrMint and transfer calls on the offRamp.

```

### <a name="NC-12"></a>[NC-12] Change uint to uint256
Throughout the code base, some variables are declared as `uint`. To favor explicitness, consider changing all instances of `uint` to `uint256`

*Instances (2)*:
```solidity
File: ./src/ccip/libraries/Internal.sol

169:     uint256 encodedAddressUint = abi.decode(encodedAddress, (uint256));

170:     if (encodedAddressUint > type(uint160).max || encodedAddressUint < PRECOMPILE_SPACE) {

```

### <a name="NC-13"></a>[NC-13] Lack of checks in setters
Be it sanity checks (like checks against `0`-values) or initial setting checks: it's best for Setter functions to have them

*Instances (1)*:
```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

132:     for (uint256 i; i < ocrConfigArgs.length; ++i) {
           _setOCR3Config(ocrConfigArgs[i]);
         }
       }
     
       /// @notice Sets offchain reporting protocol configuration incl. participating oracles for a single OCR plugin type.

```

### <a name="NC-14"></a>[NC-14] Lines are too long
Usually lines in source code are limited to [80](https://softwareengineering.stackexchange.com/questions/148677/why-is-80-characters-the-standard-limit-for-code-width) characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over [164](https://github.com/aizatto/character-length) characters, the lines below should be split when they reach that length

*Instances (1)*:
```solidity
File: ./src/ccip/FeeQuoter.sol

110:     bytes4 chainFamilySelector; // ──────────────╯ Selector that identifies the destination chain's family. Used to determine the correct validations to perform for the dest chain.

```

### <a name="NC-15"></a>[NC-15] Missing Event for critical parameters change
Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

*Instances (1)*:
```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

132:     for (uint256 i; i < ocrConfigArgs.length; ++i) {
           _setOCR3Config(ocrConfigArgs[i]);
         }
       }
     
       /// @notice Sets offchain reporting protocol configuration incl. participating oracles for a single OCR plugin type.

```

### <a name="NC-16"></a>[NC-16] Use a `modifier` instead of a `require/if` statement for a special `msg.sender` actor
If a function is supposed to be access-controlled, a `modifier` should be used instead of a `require/if` statement for more readability.

*Instances (6)*:
```solidity
File: ./src/ccip/capability/CCIPHome.sol

214:     if (msg.sender != i_capabilitiesRegistry) {

521:     if (msg.sender != address(this)) {

```

```solidity
File: ./src/ccip/offRamp/OffRamp.sol

559:     if (msg.sender != address(this)) revert CanOnlySelfCall();

```

```solidity
File: ./src/ccip/onRamp/OnRamp.sol

182:     if (msg.sender != address(destChainConfig.router)) revert MustBeCalledByRouter();

426:     if (msg.sender != owner()) {

427:       if (msg.sender != s_dynamicConfig.allowlistAdmin) {

```

### <a name="NC-17"></a>[NC-17] Adding a `return` statement when the function defines a named return variable, is redundant

*Instances (3)*:
```solidity
File: ./src/ccip/libraries/Client.sol

36:   function _argsToBytes(
        EVMExtraArgsV1 memory extraArgs
      ) internal pure returns (bytes memory bts) {
        return abi.encodeWithSelector(EVM_EXTRA_ARGS_V1_TAG, extraArgs);

54:   function _argsToBytes(
        EVMExtraArgsV2 memory extraArgs
      ) internal pure returns (bytes memory bts) {
        return abi.encodeWithSelector(EVM_EXTRA_ARGS_V2_TAG, extraArgs);

```

```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

330:   /// @param ocrPluginType OCR plugin type to return config details for.
       /// @return ocrConfig OCR config for the plugin type.
       function latestConfigDetails(
         uint8 ocrPluginType
       ) external view returns (OCRConfig memory ocrConfig) {
         return s_ocrConfigs[ocrPluginType];
       }
     }
     

```

### <a name="NC-18"></a>[NC-18] Take advantage of Custom Error's return value property
An important feature of Custom Error is that values such as address, tokenID, msg.value can be written inside the () sign, this kind of approach provides a serious advantage in debugging and examining the revert details of dapps such as tenderly.

*Instances (71)*:
```solidity
File: ./src/ccip/FeeQuoter.sol

220:       revert InvalidStaticConfig();

376:       revert DataFeedValueOutOfUint224Range();

737:       revert DataFeedValueOutOfUint224Range();

855:     if (evmExtraArgs.gasLimit > uint256(destChainConfig.maxPerMsgGasLimit)) revert MessageGasLimitTooHigh();

859:       revert ExtraArgOutOfOrderExecutionMustBeTrue();

889:     revert InvalidExtraArgsTag();

```

```solidity
File: ./src/ccip/MultiAggregateRateLimiter.sol

171:         revert ZeroChainSelectorNotAllowed();

245:         revert ZeroAddressNotAllowed();

277:       revert ZeroAddressNotAllowed();

```

```solidity
File: ./src/ccip/NonceManager.sol

143:           revert PreviousRampAlreadySet();

```

```solidity
File: ./src/ccip/capability/CCIPHome.sol

181:       revert ZeroAddressNotAllowed();

215:       revert OnlyCapabilitiesRegistryCanCall();

238:         revert(add(retData, 0x20), returndatasize())

373:       revert RevokingZeroDigestNotAllowed();

399:       revert NoOpStateTransitionNotAllowed();

462:     if (cfg.chainSelector == 0) revert ChainSelectorNotSet();

464:       revert InvalidPluginType();

467:       revert OfframpAddressCannotBeZero();

470:       revert RMNHomeAddressCannotBeZero();

486:     if (numberOfNodes > MAX_NUM_ORACLES) revert TooManySigners();

487:     if (numberOfNodes <= 3 * FRoleDON) revert FTooHigh();

522:       revert CanOnlySelfCall();

596:         revert FChainMustBePositive();

```

```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

217:       if (oracle == address(0)) revert OracleCannotBeZeroAddress();

278:         revert UnauthorizedTransmitter();

285:         if (rs.length != configInfo.F + 1) revert WrongNumberOfSignatures();

286:         if (rs.length != ss.length) revert SignaturesOutOfRegistration();

318:       if (oracle.role != Role.Signer) revert UnauthorizedSigner();

319:       if (signed & (0x1 << oracle.index) != 0) revert NonUniqueSignatures();

```

```solidity
File: ./src/ccip/offRamp/OffRamp.sol

185:       revert ZeroAddressNotAllowed();

189:       revert ZeroChainSelectorNotAllowed();

276:     if (numReports != gasLimitOverrides.length) revert ManualExecutionGasLimitMismatch();

286:       if (numMsgs != msgGasLimitOverrides.length) revert ManualExecutionGasLimitMismatch();

343:     if (reports.length == 0) revert EmptyBatch();

377:     if (numMsgs != report.offchainTokenData.length) revert UnexpectedTokenData();

559:     if (msg.sender != address(this)) revert CanOnlySelfCall();

806:         if (commitReport.merkleRoots.length == 0) revert StaleCommitReport();

829:       if (merkleRoot == bytes32(0)) revert InvalidRoot();

884:         revert SignatureVerificationRequiredInCommitPlugin();

893:         revert SignatureVerificationNotAllowedInExecutionPlugin();

960:         revert ZeroChainSelectorNotAllowed();

964:         revert ZeroAddressNotAllowed();

984:         revert ZeroAddressNotAllowed();

1012:       revert ZeroAddressNotAllowed();

1043:     revert();

```

```solidity
File: ./src/ccip/onRamp/OnRamp.sol

132:       revert InvalidConfig();

166:     if (s_dynamicConfig.reentrancyGuardEntered) revert ReentrancyGuardReentrantCall();

173:     if (originalSender == address(0)) revert RouterMustSetOriginalSender();

182:     if (msg.sender != address(destChainConfig.router)) revert MustBeCalledByRouter();

272:     if (tokenAndAmount.amount == 0) revert CannotSendZeroTokens();

339:     ) revert InvalidConfig();

428:         revert OnlyCallableByOwnerOrAllowlistAdmin();

479:     revert GetSupportedTokensFunctionalityRemovedCheckAdminRegistry();

```

```solidity
File: ./src/ccip/rmn/RMNHome.sol

244:       revert RevokingZeroDigestNotAllowed();

267:       revert NoOpStateTransitionNotAllowed();

345:       revert OutOfBoundsNodesLength();

352:           revert DuplicatePeerId();

355:           revert DuplicateOffchainPublicKey();

373:           revert DuplicateSourceChain();

381:         revert OutOfBoundsObserverNodeIndex();

391:         revert NotEnoughObservers();

```

```solidity
File: ./src/ccip/rmn/RMNRemote.sol

86:     if (localChainSelector == 0) revert ZeroValueNotAllowed();

101:       revert ConfigNotSet();

103:     if (signatures.length < s_config.f + 1) revert ThresholdNotMet();

123:       if (signerAddress == address(0)) revert InvalidSignature();

124:       if (prevAddress >= signerAddress) revert OutOfOrderSignatures();

125:       if (!s_signers[signerAddress]) revert UnexpectedSigner();

141:       revert ZeroValueNotAllowed();

147:         revert InvalidSignerOrder();

153:       revert NotEnoughSigners();

164:         revert DuplicateOnchainPublicKey();

```

### <a name="NC-19"></a>[NC-19] Contract does not follow the Solidity style guide's suggested layout ordering
The [style guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout) says that, within a contract, the ordering should be:

1) Type declarations
2) State variables
3) Events
4) Modifiers
5) Functions

However, the contract(s) below do not follow this ordering

*Instances (3)*:
```solidity
File: ./src/ccip/libraries/Client.sol

1: 
   Current order:
   StructDefinition.EVMTokenAmount
   StructDefinition.Any2EVMMessage
   StructDefinition.EVM2AnyMessage
   VariableDeclaration.EVM_EXTRA_ARGS_V1_TAG
   StructDefinition.EVMExtraArgsV1
   FunctionDefinition._argsToBytes
   VariableDeclaration.EVM_EXTRA_ARGS_V2_TAG
   StructDefinition.EVMExtraArgsV2
   FunctionDefinition._argsToBytes
   
   Suggested order:
   VariableDeclaration.EVM_EXTRA_ARGS_V1_TAG
   VariableDeclaration.EVM_EXTRA_ARGS_V2_TAG
   StructDefinition.EVMTokenAmount
   StructDefinition.Any2EVMMessage
   StructDefinition.EVM2AnyMessage
   StructDefinition.EVMExtraArgsV1
   StructDefinition.EVMExtraArgsV2
   FunctionDefinition._argsToBytes
   FunctionDefinition._argsToBytes

```

```solidity
File: ./src/ccip/libraries/Internal.sol

1: 
   Current order:
   ErrorDefinition.InvalidEVMAddress
   VariableDeclaration.GAS_FOR_CALL_EXACT_CHECK
   VariableDeclaration.MAX_RET_BYTES
   VariableDeclaration.MAX_BALANCE_OF_RET_BYTES
   StructDefinition.PriceUpdates
   StructDefinition.TokenPriceUpdate
   StructDefinition.GasPriceUpdate
   StructDefinition.TimestampedPackedUint224
   VariableDeclaration.GAS_PRICE_BITS
   StructDefinition.SourceTokenData
   StructDefinition.ExecutionReport
   VariableDeclaration.MESSAGE_FIXED_BYTES
   VariableDeclaration.MESSAGE_FIXED_BYTES_PER_TOKEN
   VariableDeclaration.ANY_2_EVM_MESSAGE_HASH
   VariableDeclaration.EVM_2_ANY_MESSAGE_HASH
   FunctionDefinition._hash
   FunctionDefinition._hash
   VariableDeclaration.PRECOMPILE_SPACE
   FunctionDefinition._validateEVMAddress
   EnumDefinition.MessageExecutionState
   EnumDefinition.OCRPluginType
   StructDefinition.RampMessageHeader
   StructDefinition.EVM2AnyTokenTransfer
   StructDefinition.Any2EVMTokenTransfer
   StructDefinition.Any2EVMRampMessage
   StructDefinition.EVM2AnyRampMessage
   VariableDeclaration.CHAIN_FAMILY_SELECTOR_EVM
   StructDefinition.MerkleRoot
   
   Suggested order:
   VariableDeclaration.GAS_FOR_CALL_EXACT_CHECK
   VariableDeclaration.MAX_RET_BYTES
   VariableDeclaration.MAX_BALANCE_OF_RET_BYTES
   VariableDeclaration.GAS_PRICE_BITS
   VariableDeclaration.MESSAGE_FIXED_BYTES
   VariableDeclaration.MESSAGE_FIXED_BYTES_PER_TOKEN
   VariableDeclaration.ANY_2_EVM_MESSAGE_HASH
   VariableDeclaration.EVM_2_ANY_MESSAGE_HASH
   VariableDeclaration.PRECOMPILE_SPACE
   VariableDeclaration.CHAIN_FAMILY_SELECTOR_EVM
   EnumDefinition.MessageExecutionState
   EnumDefinition.OCRPluginType
   StructDefinition.PriceUpdates
   StructDefinition.TokenPriceUpdate
   StructDefinition.GasPriceUpdate
   StructDefinition.TimestampedPackedUint224
   StructDefinition.SourceTokenData
   StructDefinition.ExecutionReport
   StructDefinition.RampMessageHeader
   StructDefinition.EVM2AnyTokenTransfer
   StructDefinition.Any2EVMTokenTransfer
   StructDefinition.Any2EVMRampMessage
   StructDefinition.EVM2AnyRampMessage
   StructDefinition.MerkleRoot
   ErrorDefinition.InvalidEVMAddress
   FunctionDefinition._hash
   FunctionDefinition._hash
   FunctionDefinition._validateEVMAddress

```

```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

1: 
   Current order:
   VariableDeclaration.MAX_NUM_ORACLES
   EventDefinition.ConfigSet
   EventDefinition.Transmitted
   EnumDefinition.InvalidConfigErrorType
   ErrorDefinition.InvalidConfig
   ErrorDefinition.WrongMessageLength
   ErrorDefinition.ConfigDigestMismatch
   ErrorDefinition.ForkedChain
   ErrorDefinition.WrongNumberOfSignatures
   ErrorDefinition.SignaturesOutOfRegistration
   ErrorDefinition.UnauthorizedTransmitter
   ErrorDefinition.UnauthorizedSigner
   ErrorDefinition.NonUniqueSignatures
   ErrorDefinition.OracleCannotBeZeroAddress
   ErrorDefinition.StaticConfigCannotBeChanged
   StructDefinition.ConfigInfo
   EnumDefinition.Role
   StructDefinition.Oracle
   StructDefinition.OCRConfig
   StructDefinition.OCRConfigArgs
   VariableDeclaration.s_ocrConfigs
   VariableDeclaration.s_oracles
   VariableDeclaration.TRANSMIT_MSGDATA_CONSTANT_LENGTH_COMPONENT_NO_SIGNATURES
   VariableDeclaration.TRANSMIT_MSGDATA_EXTRA_CONSTANT_LENGTH_COMPONENT_FOR_SIGNATURES
   VariableDeclaration.i_chainID
   FunctionDefinition.constructor
   FunctionDefinition.setOCR3Configs
   FunctionDefinition._setOCR3Config
   FunctionDefinition._afterOCR3ConfigSet
   FunctionDefinition._clearOracleRoles
   FunctionDefinition._assignOracleRoles
   FunctionDefinition._transmit
   FunctionDefinition._verifySignatures
   FunctionDefinition._whenChainNotForked
   FunctionDefinition.latestConfigDetails
   
   Suggested order:
   VariableDeclaration.MAX_NUM_ORACLES
   VariableDeclaration.s_ocrConfigs
   VariableDeclaration.s_oracles
   VariableDeclaration.TRANSMIT_MSGDATA_CONSTANT_LENGTH_COMPONENT_NO_SIGNATURES
   VariableDeclaration.TRANSMIT_MSGDATA_EXTRA_CONSTANT_LENGTH_COMPONENT_FOR_SIGNATURES
   VariableDeclaration.i_chainID
   EnumDefinition.InvalidConfigErrorType
   EnumDefinition.Role
   StructDefinition.ConfigInfo
   StructDefinition.Oracle
   StructDefinition.OCRConfig
   StructDefinition.OCRConfigArgs
   ErrorDefinition.InvalidConfig
   ErrorDefinition.WrongMessageLength
   ErrorDefinition.ConfigDigestMismatch
   ErrorDefinition.ForkedChain
   ErrorDefinition.WrongNumberOfSignatures
   ErrorDefinition.SignaturesOutOfRegistration
   ErrorDefinition.UnauthorizedTransmitter
   ErrorDefinition.UnauthorizedSigner
   ErrorDefinition.NonUniqueSignatures
   ErrorDefinition.OracleCannotBeZeroAddress
   ErrorDefinition.StaticConfigCannotBeChanged
   EventDefinition.ConfigSet
   EventDefinition.Transmitted
   FunctionDefinition.constructor
   FunctionDefinition.setOCR3Configs
   FunctionDefinition._setOCR3Config
   FunctionDefinition._afterOCR3ConfigSet
   FunctionDefinition._clearOracleRoles
   FunctionDefinition._assignOracleRoles
   FunctionDefinition._transmit
   FunctionDefinition._verifySignatures
   FunctionDefinition._whenChainNotForked
   FunctionDefinition.latestConfigDetails

```

### <a name="NC-20"></a>[NC-20] Use Underscores for Number Literals (add an underscore every 3 digits)

*Instances (1)*:
```solidity
File: ./src/ccip/libraries/Internal.sol

159:   uint256 public constant PRECOMPILE_SPACE = 1024;

```

### <a name="NC-21"></a>[NC-21] Internal and private variables and functions names should begin with an underscore
According to the Solidity Style Guide, Non-`external` variable and function names should begin with an [underscore](https://docs.soliditylang.org/en/latest/style-guide.html#underscore-prefix-for-non-external-functions-and-variables)

*Instances (2)*:
```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

94:   /// @notice OCR plugin type => signer OR transmitter address mapping.

97:   // Constant-length components of the msg.data sent to transmit.

```

### <a name="NC-22"></a>[NC-22] Event is missing `indexed` fields
Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

*Instances (2)*:
```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

18:   event ConfigSet(uint8 ocrPluginType, bytes32 configDigest, address[] signers, address[] transmitters, uint8 F);

23:   event Transmitted(uint8 indexed ocrPluginType, bytes32 configDigest, uint64 sequenceNumber);

```

### <a name="NC-23"></a>[NC-23] Variables need not be initialized to zero
The default value for variables is zero, so initializing them to zero is superfluous.

*Instances (62)*:
```solidity
File: ./src/ccip/FeeQuoter.sol

279:     for (uint256 i = 0; i < length; ++i) {

431:     for (uint256 i = 0; i < feeTokensToRemove.length; ++i) {

436:     for (uint256 i = 0; i < feeTokensToAdd.length; ++i) {

456:     for (uint256 i = 0; i < tokenUpdatesLength; ++i) {

465:     for (uint256 i = 0; i < gasUpdatesLength; ++i) {

508:     for (uint256 i = 0; i < feeds.length; ++i) {

556:     uint256 premiumFee = 0;

557:     uint32 tokenTransferGas = 0;

558:     uint32 tokenTransferBytesOverhead = 0;

569:     uint256 dataAvailabilityCost = 0;

617:     for (uint256 i = 0; i < premiumMultiplierWeiPerEthArgs.length; ++i) {

659:     for (uint256 i = 0; i < numberOfTokens; ++i) {

671:       uint256 bpsFeeUSDWei = 0;

675:         uint224 tokenPrice = 0;

798:     for (uint256 i = 0; i < tokenTransferFeeConfigArgs.length; ++i) {

802:       for (uint256 j = 0; j < tokenTransferFeeConfigArg.tokenTransferFeeConfigs.length; ++j) {

822:     for (uint256 i = 0; i < tokensToUseDefaultFeeConfigs.length; ++i) {

966:     for (uint256 i = 0; i < onRampTokenTransfers.length; ++i) {

1021:     for (uint256 i = 0; i < destChainConfigArgs.length; ++i) {

```

```solidity
File: ./src/ccip/MultiAggregateRateLimiter.sol

107:       for (uint256 i = 0; i < tokenAmounts.length; ++i) {

165:     for (uint256 i = 0; i < rateLimiterUpdates.length; ++i) {

215:     for (uint256 i = 0; i < tokenCount; ++i) {

230:     for (uint256 i = 0; i < removes.length; ++i) {

239:     for (uint256 i = 0; i < adds.length; ++i) {

```

```solidity
File: ./src/ccip/NonceManager.sol

132:     for (uint256 i = 0; i < previousRampsArgs.length; ++i) {

```

```solidity
File: ./src/ccip/capability/CCIPHome.sol

171:   uint32 private s_currentVersion = 0;

294:     for (uint256 i = 0; i < MAX_CONCURRENT_CONFIGS; ++i) {

489:     uint256 nonZeroTransmitters = 0;

491:     for (uint256 i = 0; i < numberOfNodes; ++i) {

574:     for (uint256 i = 0; i < chainSelectorRemoves.length; ++i) {

587:     for (uint256 i = 0; i < chainConfigAdds.length; ++i) {

```

```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

202:     for (uint256 i = 0; i < oracleAddresses.length; ++i) {

212:     for (uint256 i = 0; i < oracleAddresses.length; ++i) {

310:     uint256 signed = 0;

```

```solidity
File: ./src/ccip/offRamp/OffRamp.sol

278:     for (uint256 reportIndex = 0; reportIndex < numReports; ++reportIndex) {

288:       for (uint256 msgIndex = 0; msgIndex < numMsgs; ++msgIndex) {

305:         for (uint256 tokenIndex = 0; tokenIndex < message.tokenAmounts.length; ++tokenIndex) {

349:     for (uint256 i = 0; i < reports.length; ++i) {

395:       for (uint256 i = 0; i < numMsgs; ++i) {

420:     for (uint256 i = 0; i < numMsgs; ++i) {

745:     for (uint256 i = 0; i < sourceTokenAmounts.length; ++i) {

810:     for (uint256 i = 0; i < commitReport.merkleRoots.length; ++i) {

935:     for (uint256 i = 0; i < s_sourceChainSelectors.length(); ++i) {

955:     for (uint256 i = 0; i < sourceChainConfigUpdates.length; ++i) {

```

```solidity
File: ./src/ccip/onRamp/OnRamp.sol

217:     for (uint256 i = 0; i < message.tokenAmounts.length; ++i) {

238:     for (uint256 i = 0; i < newMessage.tokenAmounts.length; ++i) {

366:     for (uint256 i = 0; i < destChainConfigArgs.length; ++i) {

432:     for (uint256 i = 0; i < allowlistConfigArgsItems.length; ++i) {

440:           for (uint256 j = 0; j < allowlistConfigArgs.addedAllowlistedSenders.length; ++j) {

454:       for (uint256 j = 0; j < allowlistConfigArgs.removedAllowlistedSenders.length; ++j) {

508:     for (uint256 i = 0; i < feeTokens.length; ++i) {

```

```solidity
File: ./src/ccip/rmn/RMNHome.sol

128:   uint32 private s_currentVersion = 0;

134:   uint32 private s_activeConfigIndex = 0;

166:     for (uint256 i = 0; i < MAX_CONCURRENT_CONFIGS; ++i) {

295:     for (uint256 i = 0; i < MAX_CONCURRENT_CONFIGS; ++i) {

349:     for (uint256 i = 0; i < staticConfig.nodes.length; ++i) {

368:     for (uint256 i = 0; i < numberOfSourceChains; ++i) {

384:       uint256 observersCount = 0;

```

```solidity
File: ./src/ccip/rmn/RMNRemote.sol

121:     for (uint256 i = 0; i < signatures.length; ++i) {

162:     for (uint256 i = 0; i < newConfig.signers.length; ++i) {

213:     for (uint256 i = 0; i < subjects.length; ++i) {

237:     for (uint256 i = 0; i < subjects.length; ++i) {

```


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Use of `ecrecover` is susceptible to signature malleability | 2 |
| [L-2](#L-2) | `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()` | 1 |
| [L-3](#L-3) | `decimals()` is not a part of the ERC-20 standard | 1 |
| [L-4](#L-4) | Division by zero not prevented | 3 |
| [L-5](#L-5) | `domainSeparator()` isn't protected against replay attacks in case of a future chain split  | 2 |
| [L-6](#L-6) | Loss of precision | 1 |
| [L-7](#L-7) | Solidity version 0.8.20+ may not work on other chains due to `PUSH0` | 2 |
| [L-8](#L-8) | File allows a version of solidity that is susceptible to an assembly optimizer bug | 2 |
| [L-9](#L-9) | Consider using OpenZeppelin's SafeCast library to prevent unexpected overflows when downcasting | 1 |
| [L-10](#L-10) | Unsafe solidity low-level call can cause gas grief attack | 1 |
| [L-11](#L-11) | Use of ecrecover is susceptible to signature malleability | 2 |
### <a name="L-1"></a>[L-1] Use of `ecrecover` is susceptible to signature malleability
The built-in EVM precompile `ecrecover` is susceptible to signature malleability, which could lead to replay attacks.
References:  <https://swcregistry.io/docs/SWC-117>,  <https://swcregistry.io/docs/SWC-121>, and  <https://medium.com/cryptronics/signature-replay-vulnerabilities-in-smart-contracts-3b6f7596df57>.
While this is not immediately exploitable, this may become a vulnerability if used elsewhere.

*Instances (2)*:
```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

315:       address signer = ecrecover(hashedReport, uint8(rawVs[i]) + 27, rs[i], ss[i]);

```

```solidity
File: ./src/ccip/rmn/RMNRemote.sol

122:       signerAddress = ecrecover(digest, ECDSA_RECOVERY_V, signatures[i].r, signatures[i].s);

```

### <a name="L-2"></a>[L-2] `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`
Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). "Unless there is a compelling reason, `abi.encode` should be preferred". If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).
If all arguments are strings and or bytes, `bytes.concat()` should be used instead

*Instances (1)*:
```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

293:     emit Transmitted(ocrPluginType, configDigest, uint64(uint256(reportContext[1])));

```

### <a name="L-3"></a>[L-3] `decimals()` is not a part of the ERC-20 standard
The `decimals()` function is not a part of the [ERC-20 standard](https://eips.ethereum.org/EIPS/eip-20), and was added later as an [optional extension](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/IERC20Metadata.sol). As such, some valid ERC20 tokens do not support this interface, so it is unsafe to blindly cast all tokens to this interface, and then call this function.

*Instances (1)*:
```solidity
File: ./src/ccip/FeeQuoter.sol

379:       _calculateRebasedValue(dataFeedContract.decimals(), priceFeedConfig.tokenDecimals, uint256(dataFeedAnswer));

```

### <a name="L-4"></a>[L-4] Division by zero not prevented
The divisions below take an input parameter which does not have any zero-value checks, which may lead to the functions reverting when zero is passed.

*Instances (3)*:
```solidity
File: ./src/ccip/FeeQuoter.sol

343:     return (fromTokenAmount * _getValidatedTokenPrice(fromToken)) / _getValidatedTokenPrice(toToken);

601:       / feeTokenPrice;

731:       rebasedVal = feedValue / (10 ** (excessDecimals - FEE_BASE_DECIMALS));

```

### <a name="L-5"></a>[L-5] `domainSeparator()` isn't protected against replay attacks in case of a future chain split 
Severity: Low.
Description: See <https://eips.ethereum.org/EIPS/eip-2612#security-considerations>.
Remediation: Consider using the [implementation](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/EIP712.sol#L77-L90) from OpenZeppelin, which recalculates the domain separator if the current `block.chainid` is not the cached chain ID.
Past occurrences of this issue:
- [Reality Cards Contest](https://github.com/code-423n4/2021-06-realitycards-findings/issues/166)
- [Swivel Contest](https://github.com/code-423n4/2021-09-swivel-findings/issues/98)
- [Malt Finance Contest](https://github.com/code-423n4/2021-11-malt-findings/issues/349)

*Instances (2)*:
```solidity
File: ./src/ccip/libraries/Internal.sol

111:         MerkleMultiProof.LEAF_DOMAIN_SEPARATOR,

134:         MerkleMultiProof.LEAF_DOMAIN_SEPARATOR,

```

### <a name="L-6"></a>[L-6] Loss of precision
Division by large numbers may result in the result being zero, due to solidity not supporting fractions. Consider requiring a minimum amount for the numerator to ensure that it is always larger than the denominator

*Instances (1)*:
```solidity
File: ./src/ccip/FeeQuoter.sol

731:       rebasedVal = feedValue / (10 ** (excessDecimals - FEE_BASE_DECIMALS));

```

### <a name="L-7"></a>[L-7] Solidity version 0.8.20+ may not work on other chains due to `PUSH0`
The compiler for Solidity 0.8.20 switches the default target EVM version to [Shanghai](https://blog.soliditylang.org/2023/05/10/solidity-0.8.20-release-announcement/#important-note), which includes the new `PUSH0` op code. This op code may not yet be implemented on all L2s, so deployment on these chains will fail. To work around this issue, use an earlier [EVM](https://docs.soliditylang.org/en/v0.8.20/using-the-compiler.html?ref=zaryabs.com#setting-the-evm-version-to-target) [version](https://book.getfoundry.sh/reference/config/solidity-compiler#evm_version). While the project itself may or may not compile with 0.8.20, other projects with which it integrates, or which extend this project may, and those projects will have problems deploying these contracts/libraries.

*Instances (2)*:
```solidity
File: ./src/ccip/libraries/Client.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: ./src/ccip/libraries/Internal.sol

2: pragma solidity ^0.8.4;

```

### <a name="L-8"></a>[L-8] File allows a version of solidity that is susceptible to an assembly optimizer bug
In solidity versions 0.8.13 and 0.8.14, there is an [optimizer bug](https://github.com/ethereum/solidity-blog/blob/499ab8abc19391be7b7b34f88953a067029a5b45/_posts/2022-06-15-inline-assembly-memory-side-effects-bug.md) where, if the use of a variable is in a separate `assembly` block from the block in which it was stored, the `mstore` operation is optimized out, leading to uninitialized memory. The code currently does not have such a pattern of execution, but it does use `mstore`s in `assembly` blocks, so it is a risk for future changes. The affected solidity versions should be avoided if at all possible.

*Instances (2)*:
```solidity
File: ./src/ccip/libraries/Client.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: ./src/ccip/libraries/Internal.sol

2: pragma solidity ^0.8.4;

```

### <a name="L-9"></a>[L-9] Consider using OpenZeppelin's SafeCast library to prevent unexpected overflows when downcasting
Downcasting from `uint256`/`int256` in Solidity does not revert on overflow. This can result in undesired exploitation or bugs, since developers usually assume that overflows raise errors. [OpenZeppelin's SafeCast library](https://docs.openzeppelin.com/contracts/3.x/api/utils#SafeCast) restores this intuition by reverting the transaction when such an operation overflows. Using this library eliminates an entire class of bugs, so it's recommended to use it always. Some exceptions are acceptable like with the classic `uint256(uint160(address(variable)))`

*Instances (1)*:
```solidity
File: ./src/ccip/libraries/Internal.sol

173:     return address(uint160(encodedAddressUint));

```

### <a name="L-10"></a>[L-10] Unsafe solidity low-level call can cause gas grief attack
Using the low-level calls of a solidity address can leave the contract open to gas grief attacks. These attacks occur when the called contract returns a large amount of data.

So when calling an external contract, it is necessary to check the length of the return data before reading/copying it (using `returndatasize()`).

*Instances (1)*:
```solidity
File: ./src/ccip/capability/CCIPHome.sol

234:     (bool success, bytes memory retData) = address(this).call(update);

```

### <a name="L-11"></a>[L-11] Use of ecrecover is susceptible to signature malleability
The built-in EVM precompile ecrecover is susceptible to signature malleability, which could lead to replay attacks.Consider using OpenZeppelin’s ECDSA library instead of the built-in function.

*Instances (2)*:
```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

315:       address signer = ecrecover(hashedReport, uint8(rawVs[i]) + 27, rs[i], ss[i]);

```

```solidity
File: ./src/ccip/rmn/RMNRemote.sol

122:       signerAddress = ecrecover(digest, ECDSA_RECOVERY_V, signatures[i].r, signatures[i].s);

```


## Medium Issues


| |Issue|Instances|
|-|:-|:-:|
| [M-1](#M-1) | Centralization Risk for trusted owners | 22 |
| [M-2](#M-2) | Lack of EIP-712 compliance: using `keccak256()` directly on an array or struct variable | 3 |
| [M-3](#M-3) | Direct `supportsInterface()` calls may cause caller to revert | 3 |
### <a name="M-1"></a>[M-1] Centralization Risk for trusted owners

#### Impact:
Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates or drain funds.

*Instances (22)*:
```solidity
File: ./src/ccip/FeeQuoter.sol

422:   ) external onlyOwner {

477:   ) external onlyOwner {

608:   ) external onlyOwner {

789:   ) external onlyOwner {

1013:   ) external onlyOwner {

```

```solidity
File: ./src/ccip/MultiAggregateRateLimiter.sol

164:   ) external onlyOwner {

229:   ) external onlyOwner {

266:   ) external onlyOwner {

```

```solidity
File: ./src/ccip/NonceManager.sol

131:   ) external onlyOwner {

```

```solidity
File: ./src/ccip/capability/CCIPHome.sol

572:   ) external onlyOwner {

```

```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

131:   ) external onlyOwner {

```

```solidity
File: ./src/ccip/offRamp/OffRamp.sol

946:   ) external onlyOwner {

1002:   ) external onlyOwner {

```

```solidity
File: ./src/ccip/onRamp/OnRamp.sol

328:   ) external onlyOwner {

358:   ) external onlyOwner {

```

```solidity
File: ./src/ccip/rmn/RMNHome.sol

208:   ) external onlyOwner returns (bytes32 newConfigDigest) {

242:   ) external onlyOwner {

265:   function promoteCandidateAndRevokeActive(bytes32 digestToPromote, bytes32 digestToRevoke) external onlyOwner {

294:   function setDynamicConfig(DynamicConfig calldata newDynamicConfig, bytes32 currentDigest) external onlyOwner {

```

```solidity
File: ./src/ccip/rmn/RMNRemote.sol

139:   ) external onlyOwner {

212:   ) public onlyOwner {

236:   ) public onlyOwner {

```

### <a name="M-2"></a>[M-2] Lack of EIP-712 compliance: using `keccak256()` directly on an array or struct variable
Directly using the actual variable instead of encoding the array values goes against the EIP-712 specification https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md#definition-of-encodedata. 
**Note**: OpenSea's [Seaport's example with offerHashes and considerationHashes](https://github.com/ProjectOpenSea/seaport/blob/a62c2f8f484784735025d7b03ccb37865bc39e5a/reference/lib/ReferenceGettersAndDerivers.sol#L130-L131) can be used as a reference to understand how array of structs should be encoded.

*Instances (3)*:
```solidity
File: ./src/ccip/libraries/Internal.sol

124:         keccak256(abi.encode(original.tokenAmounts))

147:         keccak256(abi.encode(original.tokenAmounts)),

```

```solidity
File: ./src/ccip/ocr/MultiOCR3Base.sol

293:     emit Transmitted(ocrPluginType, configDigest, uint64(uint256(reportContext[1])));

```

### <a name="M-3"></a>[M-3] Direct `supportsInterface()` calls may cause caller to revert
Calling `supportsInterface()` on a contract that doesn't implement the ERC-165 standard will result in the call reverting. Even if the caller does support the function, the contract may be malicious and consume all of the transaction's available gas. Call it via a low-level [staticcall()](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/f959d7e4e6ee0b022b41e5b644c79369869d8411/contracts/utils/introspection/ERC165Checker.sol#L119), with a fixed amount of gas, and check the return code, or use OpenZeppelin's [`ERC165Checker.supportsInterface()`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/f959d7e4e6ee0b022b41e5b644c79369869d8411/contracts/utils/introspection/ERC165Checker.sol#L36-L39).

*Instances (3)*:
```solidity
File: ./src/ccip/offRamp/OffRamp.sol

599:         || !message.receiver.supportsInterface(type(IAny2EVMMessageReceiver).interfaceId)

640:     if (localPoolAddress == address(0) || !localPoolAddress.supportsInterface(Pool.CCIP_POOL_V1)) {

```

```solidity
File: ./src/ccip/onRamp/OnRamp.sol

278:     if (address(sourcePool) == address(0) || !sourcePool.supportsInterface(Pool.CCIP_POOL_V1)) {

```
