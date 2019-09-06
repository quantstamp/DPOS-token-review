<!-- markdownlint-disable MD033 -->

# DPOS token

This smart contract audit was prepared by [Quantstamp](https://www.quantstamp.com/), the protocol for securing smart contracts.


## Executive Summary

| Category                  | Description                                                                                                                                                                                                                   |
| :------------------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Type                      | Token Distribution Contracts                                                                                                                                                                                                  |
| Auditor(s)                | Ed Zulkoski, Senior Security Engineer<br/>John Bender, Senior Research Engineer<br/>Sung-Shine Lee, Research Engineer<br/>                                                                                                    |
| Timeline                  | 2019-08-09 through 2019-09-05                                                                                                                                                                                                 |
| EVM                       | Constantinople                                                                                                                                                                                                                |
| Language(s)               | Solidity                                                                                                                                                                                                                      |
| Method(s)                 | Architecture Review, Unit Testing, Functional Testing, Computer-Aided Verification, Manual Review                                                                                                                             |
| Specification(s)          | DPOS Staking Token Rewards and Emission Model: <https://forum.poa.network/t/dpos-staking-token-rewards-and-emission-model/2469><br/>Whitepaper: <https://forum.poa.network/uploads/short-url/cReUNVzFKxnnKk2gN8FTMfyuoFc.pdf> |
| Source code               | Repository: [dpos-token](https://github.com/poanetwork/dpos-token)<br/>Commit: [9f45465](https://github.com/poanetwork/dpos-token/commit/9f45465e5bdb9d0b09ea3e4ed5907cb1661e160e)                                            |
| Total Issues              | 7                                                                                                                                                                                                                             |
| High Risk Issues          | 0                                                                                                                                                                                                                             |
| Medium Risk Issues        | 1                                                                                                                                                                                                                             |
| Low Risk Issues           | 0                                                                                                                                                                                                                             |
| Informational Risk Issues | 3                                                                                                                                                                                                                             |
| Undetermined Risk Issues  | 3                                                                                                                                                                                                                             |

| Severity Level | Explanation                                                                                                                                                                                                   |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| High           | The issue puts a large number of users’ sensitive information at risk, or is reasonably likely to lead to catastrophic impact for client’s reputation or serious financial implications for client and users. |
| Medium         | The issue puts a subset of users’ sensitive information at risk, would be detrimental for the client’s reputation if exploited, or is reasonably likely to lead to moderate financial impact.                 |
| Low            | The risk is relatively small and could not be exploited on a recurring basis, or is a risk that the client has indicated is low-impact in view of the client’s business circumstances.                        |
| Informational  | The issue does not pose an immediate threat to continued operation or usage, but is relevant for security best practices, software engineering best practices, or defensive redundancy.                       |
| Undetermined   | The impact of the issue is uncertain.                                                                                                                                                                         |

### Goals

This report focused on evaluating security of smart contracts, as requested by the DPOS token team. Specific questions to answer:

- Ensure that no funds can be locked or stolen
- Check that the ERC677BridgeToken adheres to the ERC677 specification
- Ensure that the distribution contract distributes tokens as defined in the specification

### Changelog

- Date: 2019-08-15 - Initial Report
- Date: 2019-09-05 - Report updated based on commit [97080db](https://github.com/poanetwork/dpos-token/commit/97080db0c3c918fe1c9fb59cb7755d1d3540ef5b).

### Overall Assessment

The DPOS token and distribution contracts are well written and properly documented. The POA Network team has addressed all concerns from our initial report as of commit [97080db](https://github.com/poanetwork/dpos-token/commit/97080db0c3c918fe1c9fb59cb7755d1d3540ef5b).

-----

## Quantstamp Audit Breakdown

Quantstamp's objective was to evaluate the repository for security-related issues, code quality, and adherence to specification and best practices.
Possible issues we looked for included (but are not limited to):

- Transaction-ordering dependence
- Timestamp dependence
- Mishandled exceptions and call stack limits
- Unsafe external calls
- Integer overflow / underflow
- Number rounding errors
- Reentrancy and cross-function vulnerabilities
- Denial of service / logical oversights
- Access control
- Centralization of power
- Business logic contradicting the specification
- Code clones, functionality duplication
- Gas usage
- Arbitrary token minting  

### Methodology

The Quantstamp auditing process follows a routine series of steps:

1. Code review that includes the following:
    1. Review of the specifications, sources, and instructions provided to Quantstamp to make sure we understand the size, scope, and functionality of the smart contract.
    2. Manual review of code, which is the process of reading source code line-by-line in an attempt to identify potential vulnerabilities.
    3. Comparison to specification, which is the process of checking whether the code does what the specifications, sources, and instructions provided to Quantstamp describe.
2. Testing and automated analysis that includes the following:
    1. Test coverage analysis, which is the process of determining whether the test cases are actually covering the code and how much code is exercised when we run those test cases.  
    2. Symbolic execution, which is analyzing a program to determine what inputs cause each part of a program to execute.
3. Best practices review, which is a review of the smart contracts to improve efficiency, effectiveness, clarify, maintainability, security, and control based on the established industry and academic practices, recommendations, and research.
4. Specific, itemized, and actionable recommendations to help you take steps to secure your smart contracts.

### Toolset

The below notes outline the setup and steps performed in the process of this audit.

### Setup

Tool setup:

- [Truffle](https://truffleframework.com/) v5.0.31
- [Ganache](https://truffleframework.com/ganache) v2.7.0
- [solidity-coverage](https://github.com/sc-forks/solidity-coverage) v0.6.4
- [Oyente](https://github.com/melonproject/oyente) v0.2.7
- [Mythril](https://github.com/ConsenSys/mythril) v0.20.4
- [truffle-flattener](https://github.com/alcuadrado/truffle-flattener) v1.4.0
- [MAIAN](https://github.com/MAIAN-tool/MAIAN) commit sha: ab387e1
- [Securify](https://github.com/eth-sri/securify) commit sha: 13d4784 
- [NodeJS](https://nodejs.org/en/) v8.16.0
      
## Assessment

### Findings
        
#### Gas Usage / `for` Loop Concerns

**Status:** Fixed

**Contract(s) affected:** `Distribution.sol`


**Severity:** Medium

**Description:** Gas usage is a main concern for smart contract developers and users, since high gas costs may prevent users from wanting to use the smart contract. Even worse, some gas usage issues may prevent the contract from providing services entirely. For example, if a `for` loop requires too much gas to exit, then it may prevent the contract from functioning correctly entirely. It is best to break such loops into individual functions as possible.


**Exploit Scenario:** If the list of `privateOfferingParticipants` is too large, any loop over its members, such as in `_distributeTokensForPrivateOffering()`, may fail.

**Recommendation:** Ensure that the list of `privateOfferingParticipants` is small enough to avoid gas limits. If needed, create new functions that perform transfers to only a portion of the list at a time, so that the full set of transfers can be broken into several transactions.

***Update:*** This has been fixed through the new `PrivateOfferingDistribution` contract, which uses a pull-strategy for user withdrawals.

#### Possible Simplification and Readability Improvements

**Status:** Fixed

**Contract(s) affected:** `ERC677BridgeToken.sol`, `Distribution.sol`


**Severity:** Informational

**Description:** 
* Overall: use uint256 instead of uint;
* ERC677BridgeToken.sol: in `_contractFallback()` make the return of `success` explicit with a return-statement;
* Distribution.sol: in `_updatePoolData()`, make the return of `remainder` explicit;
* Distribution.sol: in `initialize()`, can `tokensLeft` for PUBLIC_OFFERING and EXCHANGE_RELATED_ACTIVITIES just be set to zero instead of subtracting? A similar occurrence exists in `unlockRewardForStaking()`;
* Distribution.sol: the `initialized` modifier may not be necessary/desired in `changePoolAddress()`.


**Recommendation:** Please consider implementing the above refactoring suggestions.

#### Transfers to `address(0)` can be accomplished using `ERC20Burnable`

**Status:** Fixed

**Contract(s) affected:** `ERC677BridgeToken.sol`, `Distribution.sol`


**Severity:** Informational

**Description:** The `ERC20` contract was slightly modified to allow for transfers to `address(0)`. While there is no inherent vulnerability with this, similar functionality could be obtained with `ERC20Burnable`, or alternatively transferring to an unclaimed non-zero address (e.g., `0xdead`).

This can potentially occur in `transferAndCall()` and `_distributeTokensForPrivateOffering()`.

**Recommendation:** Please add documentation on the design decisions for these changes to the `ERC20` contract. Consider using `ERC20Burnable` if possible.

***Update:*** Due to external requirements of the POA network, the total supply cannot be decremented, and therefore `ERC20Burnable` cannot be used in this context.

#### Centralization of Power

**Status:** Fixed

**Contract(s) affected:** `ERC677BridgeToken.sol`


**Severity:** Informational

**Description:** Smart contracts will often have `owner` variables to designate the person with special privileges to make modifications to the smart contract. However, this centralization of power needs to be made clear to the users, especially depending on the level of privilege the contract allows to the owner.

**Recommendation:** Add documentation informing users of the roles and responsibilities of the contract owner in the POA network, particularly for the `claimTokens()` function.

***Update:*** Documentation has been added to `claimTokens()`. A description of user roles has been added to the `README.md`.

#### Unspecified use of forced sends

**Status:** Fixed

**Contract(s) affected:** `ERC677BridgeToken.sol`, `Sacrifice.sol`


**Severity:** Undetermined

**Description:** In `claimTokens()`, if an ether transfer fails, a forced transfer occurs through the use of `Sacrifice` invoking `selfdestruct()`. It is not clear why this pattern is used or needed in this function.

**Recommendation:** Please document the design needs of this forced-send pattern. If the `_to` address is known in advance and is known to not fail upon receiving ether, the `send()` can be converted to a `transfer()` and the forced-send can be removed.

***Update:*** The POA network team has added documentation this is to ensure coins can definitely be sent to the receiver.

#### Unclear use of `transfer` followed by `transferFrom`

**Status:** Fixed

**Contract(s) affected:** `Distribution.sol`


**Severity:** Undetermined

**Description:** In `unlockRewardForStaking()`, it is not clear why there is a transfer to the `REWARD_FOR_STAKING` address, immediately followed by a `transferFrom` to the `bridgeAddress`.

**Recommendation:** Add a comment specifying the need for the two actions, or alternatively consider a direct `transfer` from the `Distribution` contract to `bridgeAddress`.

***Update:*** The POA network team has clarified to us that the bridge contract's behavior is dependent upon these transfers to behave as expected. Documentation has been added to `unlockRewardForStaking()` accordingly.

#### `_callAfterTransfer()` is invoked from `transfer()` and `transferFrom()`

**Status:** Fixed

**Contract(s) affected:** `ERC677BridgeToken.sol`


**Severity:** Undetermined

**Description:** There may exist scenarios where it is desirable to only transfer tokens to a contract without invoking `onTransferCall()`. In such a case, the call to `_callAfterTransfer()` within `transfer()` and `transferFrom()` may not be desirable. Note that the reference [ERC677Token implementation](https://github.com/smartcontractkit/LinkToken/blob/master/contracts/ERC677Token.sol) does not override these functions.

**Recommendation:** Determine if it is necessary to invoke `onTransferCall()` when invoking `transfer()`. Remove the invocation of `_callAfterTransfer()` from within `transfer()` and `transferFrom()` if necessary.

***Update:*** The POA network team has clarified to us that `callAfterTransfer()` is needed here to handle bridge-specific requirements.



### Test Results

#### Test Suite Results

```
  Contract: Distribution
    makeInstallment
      ✓ should make all installments (ECOSYSTEM_FUND) - 1 (5379ms)
      ✓ should make all installments (ECOSYSTEM_FUND) - 2 (time past more than cliff) (4339ms)
      ✓ should make all installments (ECOSYSTEM_FUND) - 3 (time past more than cliff + all installments) (221ms)
      ✓ should make all installments (FOUNDATION_REWARD) - 1 (1920ms)
      ✓ should make all installments (FOUNDATION_REWARD) - 2 (time past more than cliff) (1548ms)
      ✓ should make all installments (FOUNDATION_REWARD) - 3 (time past more than cliff + all installments) (170ms)
      ✓ should make all installments (PRIVATE_OFFERING) - 1 (1862ms)
      ✓ should make all installments (PRIVATE_OFFERING) - 2 (time past more than cliff) (1522ms)
      ✓ should make all installments (PRIVATE_OFFERING) - 3 (time past more than cliff + all installments) (177ms)
      ✓ cannot make installment if not initialized (486ms)
      ✓ cannot make installment for wrong pool (138ms)
      ✓ should revert if no installments available (112ms)

  Contract: Distribution
    constructor
      ✓ should be created (92ms)
      ✓ cannot be created with wrong values (299ms)
    preInitialize
      ✓ should be pre-initialized (221ms)
      ✓ cannot be pre-initialized with not a token address
      ✓ cannot be pre-initialized twice (81ms)
      ✓ cannot be initialized with wrong token (71ms)
      ✓ should fail if not an owner
    initialize
      ✓ should be initialized (165ms)
      ✓ cannot be initialized if not pre-initialized (149ms)
      ✓ cannot be initialized twice (96ms)
      ✓ should fail if not an owner first 90 days
      ✓ can be initialized by anyone after 90 days (104ms)
      ✓ should be initialized right after setting Private Offering participants (360ms)
      ✓ cannot be initialized if Private Offering participants are not set (414ms)
    unlockRewardForStaking
      ✓ should be unlocked (166ms)
      ✓ should be unlocked if time past more than cliff (112ms)
      ✓ should fail if bridge address is not set (498ms)
      ✓ should fail if tokens are not approved (72ms)
      ✓ cannot be unlocked before time (54ms)
      ✓ cannot be unlocked if not initialized (196ms)
      ✓ cannot be unlocked twice (128ms)
    changePoolAddress
      ✓ should be changed (97ms)
      ✓ should fail if wrong pool (66ms)
      ✓ should fail if not authorized (64ms)
      ✓ should fail if invalid address
    setBridgeAddress
      ✓ should be set (43ms)
      ✓ should fail if not a contract
      ✓ should fail if not an owner

  Contract: PrivateOfferingDistribution
    addParticipants
      ✓ should be added (63ms)
      ✓ should be added 50 participants (893ms)
      ✓ cannot be added with wrong values (293ms)
      ✓ cannot be added if finalized (139ms)
      ✓ should be added and validated (250 participants) (4717ms)
      ✓ should fail if not an owner
    finalizeParticipants
      ✓ should be finalized (92ms)
      ✓ should be finalized with sum of stakes which is less than the whole pool stake (94ms)
      ✓ should be finalized with 250 participants (4643ms)
      ✓ cannot be finalized twice (114ms)
      ✓ should fail if not an owner (85ms)
    initialize
      ✓ should be initialized
      ✓ should fail if sender is not a distribution address
      ✓ cannot be initialized twice (63ms)
      ✓ should fail if not finalized (99ms)
    setDistributionAddress
      ✓ should be set (41ms)
      ✓ cannot be set twice (55ms)
      ✓ should fail if not an owner
      ✓ should fail if not the Distribution contract address (235ms)
    withdraw
      ✓ should be withdrawn (497ms)
      ✓ should be withdrawn 10 times by 20 participants (19631ms)
      ✓ should be withdrawn in random order (1938ms)
      ✓ cannot be withdrawn by not participant (391ms)
      ✓ cannot be withdrawn when no tokens available (493ms)
    burn
      ✓ should be burnt (405ms)
      ✓ should be burnt after withdrawals (2261ms)
      ✓ cannot be burnt by not an owner (403ms)
      ✓ cannot be burnt if zero address stake is zero (380ms)
      ✓ cannot be burnt when no tokens available (476ms)
    onTokenTransfer
      ✓ should be called (42ms)
      ✓ should fail if "from" value is not the distribution contract
      ✓ should fail if caller is not the token contract

  Contract: Token
    constructor
      ✓ should be created (287ms)
      ✓ should fail if invalid address (505ms)
    setBridgeContract
      ✓ should set (47ms)
      ✓ should fail if invalid or wrong address (53ms)
      ✓ should fail if not an owner
    transferAndCall
      ✓ should transfer and call (93ms)
      ✓ should fail if wrong custom data (56ms)
    transfer
      ✓ should transfer (39ms)
      ✓ should fail if recipient is bridge contract (107ms)
    transferFrom
      ✓ should transfer (74ms)
      ✓ should fail if recipient is bridge contract (129ms)
    claimTokens
      ✓ should claim tokens (52ms)
      ✓ should fail if invalid recipient (58ms)
      ✓ should fail if not an owner
      ✓ should claim eth (125ms)
      ✓ should claim eth to non-payable contract (189ms)
    renounceOwnership
      ✓ should fail (not implemented) (228ms)


  89 passing (1m)
```

#### Code Coverage

```
File                               |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
-----------------------------------|----------|----------|----------|----------|----------------|
 contracts/                        |    99.38 |    97.44 |    96.43 |    99.39 |                |
  Distribution.sol                 |      100 |    97.73 |      100 |      100 |                |
  IDistribution.sol                |      100 |      100 |      100 |      100 |                |
  IPrivateOfferingDistribution.sol |      100 |      100 |      100 |      100 |                |
  PrivateOfferingDistribution.sol  |    97.83 |    97.06 |    92.31 |    97.92 |            128 |
 contracts/Token/                  |    85.37 |    71.43 |    77.78 |    85.54 |                |
  ERC20.sol                        |    63.64 |    41.67 |    53.85 |    63.64 |... 202,233,234 |
  ERC677BridgeToken.sol            |      100 |    83.33 |      100 |      100 |                |
  IERC677BridgeToken.sol           |      100 |      100 |      100 |      100 |                |
  Sacrifice.sol                    |      100 |      100 |      100 |      100 |                |
-----------------------------------|----------|----------|----------|----------|----------------|
All files                          |    94.63 |    88.33 |    87.27 |    94.74 |                |
-----------------------------------|----------|----------|----------|----------|----------------|

```

**Update: fixed.** The `claimTokens()` function is not properly covered. Unit tests should be added for this and other uncovered cases.
### Automated Analyses


#### Oyente

Repository: <https://github.com/melonproject/oyente>

Oyente is a symbolic execution tool that analyzes the bytecode of Ethereum smart contracts. It checks if a contract features any of the predefined vulnerabilities before the contract gets deployed on the blockchain.

##### Oyente Findings

Oyente does not currently support `0.5.0^` contracts, hence it was not used during analysis.

#### Mythril

Repository: <https://github.com/ConsenSys/mythril>

Mythril is a security analysis tool for Ethereum smart contracts. It uses concolic analysis, taint analysis and control flow checking to detect a variety of security vulnerabilities.

##### Mythril Findings

Mythril reported no vulnerabilities.

#### MAIAN

Repository: <https://github.com/MAIAN-tool/MAIAN>

MAIAN is a tool for automatic detection of trace vulnerabilities in Ethereum smart contracts. It processes a contract's bytecode to build a trace of transactions to find and confirm bugs.

##### MAIAN Findings

Maian reported no vulnerabilities.
#### Securify

Repository: <https://github.com/eth-sri/securify>
##### Securify Findings

Securify reported several "Locked Ether" vulnerabilities, however manual inspection determined that the reported lines were all false positives.

-----
  
## Adherence to Specification

The smart contracts appear to correctly adhere to the provided specification.

## Code Documentation

The code has excellent documentation.

## Adherence to Best Practices

**Update: fixed.** The smart contracts generally follow best practices, with exception of some minor suggestions above in the `Possible Simplification and Readability Improvements` listing.

-----

## Appendix

### File Signatures
The following are the SHA‌-256 hashes of the audited contracts and/or test files. A smart contract or file with a different SHA‌-256 hash has been modified, intentionally or otherwise, after the audit. You are cautioned that a different SHA-256 hash could be (but is not necessarily) an indication of a changed condition or potential vulnerability that was not within the scope of the audit.

#### Contracts


```
contracts/Migrations.sol: df24290ca011483abee2f487aa0dfe076cd763dbb2f0b89bc9401920d6f54ac2
contracts/PrivateOfferingDistribution.sol: 908b056b0e0b0f16ea8ced8129ae84240202af289faa3116c23358ec31fb2180
contracts/IDistribution.sol: d660b7df371cb3170dfed5bac247a886126a17d183fdddbd6ac69127370ca3d0
contracts/Distribution.sol: 67b8ecdbae7862abd40c47de6f307955c763ce3d6c78fbc82a2207daafc63ecf
contracts/IPrivateOfferingDistribution.sol: 411144a54401f040617b3c43e53a18929a22005a82898c8681d95313c3c16e2a
contracts/Token/ERC20.sol: ab190ca812ccae5dc8eeec71323350de0a6e16d4c7d3b8893c18e3ffee2a37f0
contracts/Token/Sacrifice.sol: 566502351be8da6093b9437199e4a3f4830134a8a893a830c2f223cce5e676ab
contracts/Token/IERC677BridgeToken.sol: 148f37a81fb221be3aff9a510a3b094c943db68a607c57aed1df1af6309192f4
contracts/Token/ERC677BridgeToken.sol: d53361a22323efbec20a36562551de44a4a7c889e027f42851b01af2196a11b9
```


#### Tests


```
test/constants.js: 2520790c697bd154e4509658d53b5ac2078dddb4deacb32708ae88879e892857
test/token.test.js: 6ce59cd60cca2b7e5541186586c1f007b7488b50d99af69365081bbc1aa13bab
test/distribution.test.js: 3a6ffe86e2bbceb608cb4231367ba04f3083920439ae50388e1a69674d23f82f
test/distribution.installments.test.js: 1b53e6d3ea3f3be56c568c6455d78b95e721f0e1f5c2c776224278fe02684ffd
test/privateOfferingDistribution.test.js: 3fdfb6468347677d45e974c7c2ae6a77891cbf64e5a03375db93284c3762630d
test/helpers/ganache.js: 64cf58d95eed1af19e81a0817a33e6c1b4effd08087a6a4546e9c1161ac557e1
test/mocks/RecipientMock.sol: 5d3c5e8d95eb0468a587ae528b45c0c14a5f32bd6b7e607b71dbdb36e2b784eb
test/mocks/EmptyContract.sol: 1327b357f64c029edc121903762af479bc8c177fd36b3c2c31593f2c9bc136c9
test/mocks/PrivateOfferingDistributionMock.sol: 12c120997aeba261349c57d990272d094f87bde81790922be95cb3b655a64a89
test/mocks/TokenMock.sol: f468321f1baa3cc25bce69c08a40d86abdd6462114b1c5bfec9c4df1690b1d60
test/mocks/DistributionMock.sol: c9b180309153ab2a10f7a3184c1e3beaa14a25822d07b0ae4a21cf492c8c8c39
test/mocks/BridgeTokenMock.sol: 6cbb131029106e189417ec5094ce794777cc3effee3f094372d08e97c0e25348
```



#### Steps

Steps taken to run the tools:

- Installed Truffle: `npm install -g truffle`
- Installed Ganache: `npm install -g ganache-cli`
  - Installed the solidity-coverage tool (within the project's root directory): `npm install --save-dev solidity-coverage`
- Ran the coverage tool from the project's root directory: `./node_modules/.bin/solidity-coverage`
- Flattened the source code using `truffle-flattener` to accommodate the auditing tools.
- Installed the Mythril tool from Pypi: `pip3 install mythril`
- Ran the Mythril tool on each contract: `myth -x path/to/contract`  
- Ran the Securify tool `java -Xmx6048m -jar securify-0.1.jar -fs contract.sol`  
- Installed the Oyente tool from Docker: `docker pull luongnguyen/oyente`
- Migrated files into Oyente (root directory): `docker run -v $(pwd):/tmp -it luongnguyen/oyente`
- Ran the Oyente tool on each contract: `cd /oyente/oyente && python oyente.py /tmp/path/to/contract`
- Ran the MAIAN tool on each contract: `cd maian/tool/ && python3 maian.py -s path/to/contract contract`

- Installed NodeJS: https://nodejs.org/en/download/


## About Quantstamp

Quantstamp is a Y Combinator-backed company that helps to secure smart contracts at scale using computer-aided reasoning tools, with a mission to help boost adoption of this exponentially growing technology.

Quantstamp’s team boasts decades of combined experience in formal verification, static analysis, and software verification. Collectively, our individuals have over 500 Google scholar citations and numerous published papers. In its mission to proliferate development and adoption of blockchain applications, Quantstamp is also developing a new protocol for smart contract verification to help smart contract developers and projects worldwide to perform cost-effective smart contract security audits.
To date, Quantstamp has helped to secure hundreds of millions of dollars of transaction value in smart contracts and has assisted dozens of blockchain projects globally with its white glove security auditing services. As an evangelist of the blockchain ecosystem, Quantstamp assists core infrastructure projects and leading community initiatives such as the Ethereum Community Fund to expedite the adoption of blockchain technology.

Finally, Quantstamp’s dedication to research and development in the form of collaborations with leading academic institutions such as National University of Singapore and MIT (Massachusetts Institute of Technology) reflects Quantstamp’s commitment to enable world-class smart contract innovation.


### Timeliness of content

The content contained in the report is current as of the date appearing on the report and is subject to change without notice, unless indicated otherwise by Quantstamp; however, 
Quantstamp does not guarantee or warrant the accuracy, timeliness, or completeness of any report you access using the internet or other means, and assumes no obligation to update any information following publication.
publication.

### Notice of Confidentiality

This report, including the content, data, and underlying methodologies, are subject to the confidentiality and feedback provisions in your agreement with Quantstamp. 
These materials arenot to be disclosed, extracted, copied, or distributed except to the extent expressly authorized by Quantstamp.

### Links to other websites

You may, through hypertext or other computer links, gain access to web sites operated by persons other than Quantstamp, Inc. (Quantstamp). Such hyperlinks are provided for your reference and convenience only, and are the exclusive responsibility of such web sites&apos; owners. 
You agree that Quantstamp are not responsible for the content or operation of such web sites, and that Quantstamp shall have no liability to you or any other person or entity for the use of third-party web sites. 
Except as described below, a hyperlink from this web site to another web site does not imply or mean that Quantstamp endorses the content on that web site or the operator or operations of that site. 
You are solely responsible for determining the extent to which you may use any content at any other web sites to which you link from the report. 
Quantstamp assumes no responsibility for the use of third-party software on the website and shall have no liability whatsoever to any person or entity for the accuracy or completeness of any outcome generated by such.

### Disclaimer

This report is based on the scope of materials and documentation provided for a limited review at the time provided. Results may not be complete nor inclusive of all vulnerabilities. 
The review and this report are provided on an as-is, where-is, and as-available basis. 
You agree that your access and/or use, including but not limited to any associated services, products, protocols, platforms, content, and materials, will be at your sole risk. 
Cryptographic tokens are emergent technologies and carry with them high levels of technical risk and uncertainty. 
The Solidity language itself and other smart contract languages remain under development and are subject to unknown risks and flaws. 
The review does not extend to the compiler layer, or any other areas beyond Solidity or the smart contract programming language, or other programming aspects that could present security risks. 
You may risk loss of tokens, Ether, and/or other loss. A report is not an endorsement (or other opinion) of any particular project or team, and the report does not guarantee the security of any particular project. 
A report does not consider, and should not be interpreted as considering or having any bearing on, the potential economics of a token, token sale or any other
product, service or other asset. No third party should rely on the reports in any way, including for the purpose of making any decisions to buy or sell any token, product, service or other asset.
To the fullest extent permitted by law, we disclaim all warranties, express or implied, in connection with this report, its content, and the related services and products and your use thereof, including,
without limitation, the implied warranties of merchantability, fitness for a particular purpose, and non-infringement.  
We do not warrant, endorse, guarantee, or assume responsibility for any product or service advertised or offered by a third party through the product, any open source or third party software, code,
libraries, materials, or information linked to, called by, referenced by or accessible through the report, its content, and the related services and products, any hyperlinked website, or any
website or mobile application featured in any banner or other advertising, and we will not be a party to or in any way be responsible for monitoring any transaction between you and any
third-party providers of products or services. As with the purchase or use of a product or service through any medium or in any environment, you should use your best judgment and exercise caution where appropriate. You may risk loss of QSP tokens or other loss.
FOR AVOIDANCE OF DOUBT, THE REPORT, ITS CONTENT, ACCESS, AND/OR USAGE THEREOF, INCLUDING ANY ASSOCIATED SERVICES OR MATERIALS, SHALL NOT BE CONSIDERED OR RELIED UPON AS ANY FORM OF FINANCIAL, INVESTMENT, TAX, LEGAL, REGULATORY, OR OTHER ADVICE.

