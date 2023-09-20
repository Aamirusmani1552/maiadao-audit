# ✨ So you want to run an audit

This `README.md` contains a set of checklists for our audit collaboration.

Your audit will use two repos: 
- **an _audit_ repo** (this one), which is used for scoping your audit and for providing information to wardens
- **a _findings_ repo**, where issues are submitted (shared with you after the audit) 

Ultimately, when we launch the audit, this repo will be made public and will contain the smart contracts to be reviewed and all the information needed for audit participants. The findings repo will be made public after the audit report is published and your team has mitigated the identified issues.

Some of the checklists in this doc are for **C4 (🐺)** and some of them are for **you as the audit sponsor (⭐️)**.

---

# Repo setup

## ⭐️ Sponsor: Add code to this repo

- [ ] Create a PR to this repo with the below changes:
- [ ] Provide a self-contained repository with working commands that will build (at least) all in-scope contracts, and commands that will run tests producing gas reports for the relevant contracts.
- [ ] Make sure your code is thoroughly commented using the [NatSpec format](https://docs.soliditylang.org/en/v0.5.10/natspec-format.html#natspec-format).
- [ ] Please have final versions of contracts and documentation added/updated in this repo **no less than 48 business hours prior to audit start time.**
- [ ] Be prepared for a 🚨code freeze🚨 for the duration of the audit — important because it establishes a level playing field. We want to ensure everyone's looking at the same code, no matter when they look during the audit. (Note: this includes your own repo, since a PR can leak alpha to our wardens!)


---

## ⭐️ Sponsor: Edit this `README.md` file

- [ ] Modify the contents of this `README.md` file. Describe how your code is supposed to work with links to any relevent documentation and any other criteria/details that the C4 Wardens should keep in mind when reviewing. ([Here's a well-constructed example.](https://github.com/code-423n4/2022-08-foundation#readme))
- [ ] Review the Gas award pool amount. This can be adjusted up or down, based on your preference - just flag it for Code4rena staff so we can update the pool totals across all comms channels.
- [ ] Optional / nice to have: pre-record a high-level overview of your protocol (not just specific smart contract functions). This saves wardens a lot of time wading through documentation.
- [ ] [This checklist in Notion](https://code4rena.notion.site/Key-info-for-Code4rena-sponsors-f60764c4c4574bbf8e7a6dbd72cc49b4#0cafa01e6201462e9f78677a39e09746) provides some best practices for Code4rena audits.

## ⭐️ Sponsor: Final touches
- [ ] Review and confirm the details in the section titled "Scoping details" and alert Code4rena staff of any changes.
- [ ] Check that images and other files used in this README have been uploaded to the repo as a file and then linked in the README using absolute path (e.g. `https://github.com/code-423n4/yourrepo-url/filepath.png`)
- [ ] Ensure that *all* links and image/file paths in this README use absolute paths, not relative paths
- [ ] Check that all README information is in markdown format (HTML does not render on Code4rena.com)
- [ ] Remove any part of this template that's not relevant to the final version of the README (e.g. instructions in brackets and italic)
- [ ] Delete this checklist and all text above the line below when you're ready.

---

# Maia DAO audit details
- Total Prize Pool: $100,000 USDC 
  - HM awards: $69,712.50 USDC 
  - Analysis awards: $4,225 USDC 
  - QA awards: $2,112.50 USDC 
  - Bot Race awards: $6,337.50 USDC 
  - Gas awards: $2,112.50 USDC 
  - Judge awards: $9,000 USDC 
  - Lookout awards: $6,000 USDC 
  - Scout awards: $500 USDC 
- Join [C4 Discord](https://discord.gg/code4rena) to register
- Submit findings [using the C4 form](https://code4rena.com/contests/2023-09-Maia/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts September 22, 2023 20:00 UTC 
- Ends September 29, 2023 20:00 UTC 

## Automated Findings / Publicly Known Issues

Automated findings output for the audit can be found [here](https://github.com/code-423n4/2023-09-maia/bot-report.md) within 24 hours of audit opening.

*Note for C4 wardens: Anything included in the automated findings output is considered a publicly known issue and is ineligible for awards.*

### Publicly Known Issues

- Some chains may demand different block timestamp settings.
- Using your Virtual Account as refundee for a the creation of cross-chain messages with or without settlement attached as well as retrySettlement or retryDeposit will result in gas refunds of excess gas on branch chains being inaccesible since the refundee is not an EOA able to claim receive the funds on any branhc chain.
- Floating pragma, code will be depoyed with 0.8.19.
- There are contracts that allow to renounce ownership and do not override renounceOwnership function from solady library.
- Our protocol has permissionless factories where anyonce can create with for example deposit poison erc20 tokens in ports or create malicious routers. While contracts generated by these are not in scope, if it does affect other contracts or other balances, it is in scope.

# Overview

Maia DAO V2 Ecosytem docs section that explains the business logic and technical references for Ulysses Protocol, can be found [here](https://v2-docs.maiadao.io/protocols/Ulysses/introduction/).

### Ulysses

**[Ulysses](https://v2-docs.maiadao.io/protocols/Ulysses/introduction)** scope for this audit focuses on Ulysses Omnichain our Liquidity and Execution Platform built on top of Layer Zero.

This can be divided in two main features:

1. **[Virtualized liquidity](https://v2-docs.maiadao.io/protocols/Ulysses/overview/omnichain/virtual-liquidity)** is achieved by connecting [Ports](https://v2-docs.maiadao.io/protocols/Ulysses/overview/omnichain/ports) within a Pool and Spoke architecture, comprising both the Root Chain and multiple Branch Chains. These contracts are responsible for managing token balances and address mappings across environments. In addition, means that an asset deposited from a specific chain, is recognized as a different asset from the "same" asset but from a different chain (ex: arb ETH is different from mainnet ETH).

2. **Arbitary Cross-Chain Execution** is facilitated by an expandable set of routers such as the Multicall Root Router that can be permissionlessly deployed through the Bridge Agent Factories. For more insight on Bridge Agents, please refer to our documentation [here](https://v2-docs.maiadao.io/protocols/Ulysses/overview/omnichain/bridge-agents). Our [Virtual Account](https://v2-docs.maiadao.io/protocols/Ulysses/overview/omnichain/virtual-accounts) contract simplifies remote asset management and interaction within the Root chain.

## Areas of Concern

While this audit has all the Ulysses Omnichain components and features in scope, there are specific concerns that we would like to highlight for the wardens to pay special attention to. These are:

- BranchPort's Strategy Token and Port Strategy related functions.
- Omnichain balance accounting.
- Omnichain execution management aspects, particularly related to transaction nonce retry, as well as the retrieve and redeem patterns:
  1. `srChain` settlement and deposits should either have `status` set to `STATUS_SUCCESS` and `STATUS_FAILED` depending on their redeemability by the user on the source.
  2. `dstChain` settlement and deposit execution should have `executionState` set to `STATUS_READY`, `STATUS_DONE` or `STATUS_RETRIEVE` according to user input fallback and destination execution outcome.

## Links

##### **Previous audits:**

Previous Audits by Zellic and Code 4rena can be found in the [audits](https://github.com/code-423n4/2023-05-maia/tree/main/audits) folder.
There are three audits, two of them featuring Ulysses:

- [Zellic Audit](https://github.com/code-423n4/2023-05-maia/tree/main/audits/Ulysses%20Protocol%20May%202023%20-%20Zellic%20Audit%20Report.pdf)
- [Code 4rena Contest](https://code4rena.com/reports/2023-05-maia)

##### **Other links:**

- **[Documentation](https://v2-docs.maiadao.io/)**
- **[Website](https://maiadao.io/)**
- **[Twitter](https://twitter.com/MaiaDAOEco)**
- **[Discord](https://discord.gg/maiadao)**

# Scope

[ ⭐️ SPONSORS: add scoping and technical details here ]

- [ ] In the table format shown below, provide the name of each contract and:
  - [ ] source lines of code (excluding blank lines and comments) in each _For line of code counts, we recommend running prettier with a 100-character line length, and using [cloc](https://github.com/AlDanial/cloc)._
  - [ ] external contracts called in each
  - [ ] libraries used in each

_List all files in scope in the table below (along with hyperlinks) -- and feel free to add notes here to emphasize areas of focus._

| Contract                                                   | SLOC | Purpose                | Libraries used                                           |
| ---------------------------------------------------------- | ---- | ---------------------- | -------------------------------------------------------- |
| [contracts/folder/sample.sol](contracts/folder/sample.sol) | 123  | This contract does XYZ | [`@openzeppelin/*`](https://openzeppelin.com/contracts/) |

## Out of scope

_List any files/contracts that are out of scope for this audit._

# Additional Context

### Describe any novel or unique curve logic or mathematical models implemented in the contracts
  - Branch / Root Bridge Agent and Bridge Agent Executor packed payload decoding and encoding.

### Please list specific ERC20 that your protocol is anticipated to interact with. Could be "any" (literally anything, fee on transfer tokens, ERC777 tokens and so forth) or a list of tokens you envision using on launch.
  - Arbitrum's deployment of UniswapV3 and Balancer.

### Please list specific ERC721 that your protocol is anticipated to interact with.
  - Virtual Account should be able to keep and use UniswapV3 NFT's.

### Which blockchains will this code be deployed to, and are considered in scope for this audit?
  - Root contracts are to be deploye on Arbitrum and Branch contracts in several L1 and L2 networks such as Ethereum mainnet, Polygon, Base and Optimism

### Please list all trusted roles (e.g. operators, slashers, pausers, etc.), the privileges they hold, and any conditions under which privilege escalation is expected/allowable:
  - Only our governance has access to key admin state changing functions present in the `RootPort` and `CoreRootRouter` and the Root Bridge Agent deployer (referred to in the codebase as manager) is responsible for allowing new branch chains to connect to their Root Bridge Agent in order to prevent griefing.

### In the event of a DOS, could you outline a minimum duration after which you would consider a finding to be valid? This question is asked in the context of most systems' capacity to handle DoS attacks gracefully for a certain period.
  - Unless there is the need to upgrade and migrate any component of Ulysses via governance ( e.g. Bridge Agents or Core Routers) downtime should be negligeble to ensure assets are available at any time to their different users.

### Is any part of your implementation intended to conform to any EIP's? If yes, please list the contracts in this format:
  - `ERC20hTokenBranch`: Should comply with `ERC20/EIP20`
  - `ERC20hTokenRoot`: Should comply with `ERC20/EIP20`

## Attack ideas (Where to look for bugs)

_List specific areas to address - see [this blog post](https://medium.com/code4rena/the-security-council-elections-within-the-arbitrum-dao-a-comprehensive-guide-aa6d001aae60#9adb) for an example_

- Double spending of deposit and settlement nonces / assets (Bridge Agents and Bridge Agent Executors).
- Griefing of user deposits and settlements (Bridge Agents).
- Bricking of Bridge Agent and subsequent Omnichian dApps that rely on it.
- Circumventing Bridge Agent's encoding rules to manipulate remote chain state.

## Main invariants

- The total balance of any given Virtualized Liquidity Token should never be greater than the amount of Underlying Tokens deposited in the asset's origin chain Branch Port.

## Scoping Details

[ ⭐️ SPONSORS: please confirm/edit the information below. ]

```
- If you have a public code repo, please share it here:
- How many contracts are in scope?:  50
- Total SLoC for these contracts?:  4266
- How many external imports are there?: 33
- How many separate interfaces and struct definitions are there for the contracts within scope?:  42
- Does most of your code generally use composition or inheritance?:   Inheritance
- How many external calls?:   17
- What is the overall line coverage percentage provided by your tests?: 67%
- Is this an upgrade of an existing system?: False
- Check all that apply (e.g. timelock, NFT, AMM, ERC20, rollups, etc.): Uses L2, Multi-Chain, Side-Chain, ERC-20 Token, Timelock function
- Is there a need to understand a separate part of the codebase / get context in order to audit this part of the protocol?:   Yes
- Please describe required context:   Layerzero Messaging layer, namely Endpoint contract: https://github.com/LayerZero-Labs/LayerZero/blob/main/contracts/Endpoint.sol
- Does it use an oracle?:  No
- Describe any novel or unique curve logic or mathematical models your code uses: None
- Is this either a fork of or an alternate implementation of another project?:   No
- Does it use a side-chain?: True
- Describe any specific areas you would like addressed: Please try to break token deposits and settlements patterns - retry, retrieve, and redeem - to avoid double spending, reentrancy and race conditions. Ensure proper asset management of different ports. Encoding and decoding of cross-chain payloads/data on bridge agents and routers.
```

# Tests

_Provide every step required to build the project from a fresh git clone, as well as steps to run the tests with a gas report._

_Note: Many wardens run Slither as a first pass for testing. Please document any known errors with no workaround._

# Tests

**Here is an example of a full script to run the first time you build the contracts in both Windows and Linux:**

- Remove `.example` from the provided `.env` file and edit the uncommented `RPC` and `RPC_API_KEY` values to your preferences. These values will be used by our fork testing suite.

```bash
forge install
forge build
forge test --gas-report
forge snapshot --diff
```

Default gas prixe is 10,000, but you can change it by adding `--gas-price <gas price>` to the command or by setting the `gas_price` property in the [foundry.toml](https://github.com/code-423n4/2023-05-maia/tree/main/foundry.toml) file.

Tests don't compile with --via-ir, but contracts do and will be deployed with --via-ir. Compilation settings that will be used are in [hardhat.config.ts](https://github.com/code-423n4/2023-05-maia/tree/main/hardhat.config.ts).

### Install and First Build

Install libraries using forge and compile contracts.

```bash
forge install
forge build
```

## Slither

If you encounter any issues, please update slither to [0.9.3](https://github.com/crytic/slither/releases/tag/0.9.3), the latest version at the moment.

To run [slither](https://github.com/crytic/slither) from root, please specify the src directory.

```bash
slither "./src/*"
```

We have a [slither config](https://github.com/code-423n4/2023-05-maia/tree/main/slither.config.json) file that turns on optimization and adds forge remappings.