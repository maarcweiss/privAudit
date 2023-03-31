# Contest225 contest details
- Total Prize Pool: $80,000 USDC
  - HM awards: $42,500 USDC 
  - QA report awards: $5,000 USDC
  - Gas report awards: $2,500 USDC 
  - Judge awards: $10,500 USDC 
  - Lookout awards: $4,000 USDC
  - Scout awards: $500 USDC 
  - Mitigation review contest: $15,000 USDC
- Join [C4 Discord](https://discord.gg/code4rena) to register
- Submit findings [using the C4 form](https://code4rena.com/contests/2023-03-contest-225-contest/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts March 27, 2023 20:00 UTC
- Ends April 03, 2023 20:00 UTC

## Automated Findings / Publicly Known Issues

Automated findings output for the contest can be found [here](https://gist.github.com/Picodes/8cba2f31a33998b5c94f1803f6dd6f0a) within an hour of contest opening.

*Note for C4 wardens: Anything included in the automated findings output is considered a publicly known issue and is ineligible for awards.*

# Overview

This contest is for an ETH lending protocol with loans collateralized by NFTs. Loan offers are signed off-chain by potential lenders and taken on-chain by borrowers who lock their NFT in the protocol as collateral. When an offer is taken, a lien is created. Aside from a few flows, the protocol will make all transfers using ETH deposited in the `Pool`.

Lenders are able to create multiple liens from a singe loan offer by adjusting the `totalAmount`, `minAmount`, and `maxAmount`.
`totalAmount` determines the most ETH the offer is able to provide in loans.
`minAmount` is the minimum amount that must be taken in each loan.
`maxAmount` is the most that can be taken in each loan.

They will also choose the `rate` which is the interest rate in bips. The protocol uses compound interest.

## Offer Verification and Management
Besides the lender signature, offers are checked for cancellations and expiration. There are several methods to cancel an offer.

Users can cancel the salt of their order to ensure it cannot be taken.

Users can increment their user nonce which will cancel all orders with this nonce.

Users can opt-in to off-chain cancellations by setting the `oracle` address in their LoanOffer. This will require an oracle signature in addition to the user signature. This is an oracle we will maintain to allow users to cancel off-chain through our service. Oracle signatures are signed with a recent block number, which is checked to be within `blockRange` of the block at the time of execution. This guarantees that the oracle signature is recent and will expire. Only approved oracles are valid.

## Refinancing
There are a few methods lenders or borrowers can use to refinance their side of the lien.

### Instant Refinance
Allows a lender to replace their loan with a different loan offer with an interest rate that's at least as good as theirs.

### Auction Refinance
A lender can auction off their lien if they want to get rid of it. They will begin a Dutch Auction on the interest rate of the lien that increases over the course of the auction. It will begin at 0% and progress to 1000% interest over the course of the `AUCTION_DURATION`. At any point, another lender can take over the loan at the auction rate or better. If the auction concludes and no new lender has taken over the lien, the lien is defaulted and the lender can `seize` the collateral.

### Borrower Refinance
Allows a borrower to replace their loan with a different loan offer. If the loan offer is more than the previous, then they will repay the original loan and withdraw the surplus. If it is less than the previous, then the borrower will have to supplement the difference.

## Marketplace
There are several high-level flows that allow borrowers to perform composite interactions with this protocol and NFT marketplaces. These are

### Borrow to Buy
Take a loan offer and use the funds to purchase an NFT.

### Buy to Borrow
Take a flash loan to purchase an NFT and use that NFT to take a loan offer; the loan repays the flash loan.

### Take Bid
Allow a borrower to take a bid on a locked NFT and use the funds to repay the loan.

### Buy Locked
Allow a borrower to sell a locked NFT and use the funds to repay the loan. The borrower will create and sign a `SellOffer` that offers the token held in the lien for a certain price. See Offer Verification and Management for details on how the offers are verified.

# Scope

| Contract | SLOC | Purpose | Libraries used |  
| ----------- | ----------- | ----------- | ----------- |
| [contracts/Core.sol](https://github.com/code-423n4/2023-03-contest225/blob/main/contracts/Core.sol) | 424 | Core protocol logic | [`transmissions11/solmate`](https://github.com/transmissions11/solmate) |
| [contracts/lib/Signatures.sol](https://github.com/code-423n4/2023-03-contest225/blob/main/contracts/lib/Signatures.sol) | 224 | Offer signature logic |  |
| [contracts/OfferController.sol](https://github.com/code-423n4/2023-03-contest225/blob/main/contracts/OfferController.sol) | 69 | Offer management functions | [`transmissions11/solmate`](https://github.com/transmissions11/solmate) [`@openzeppelin/*`](https://openzeppelin.com/contracts/) |
| [contracts/lib/Structs.sol](https://github.com/code-423n4/2023-03-contest225/blob/main/contracts/lib/Structs.sol) | 44 | Structs |  |
| [contracts/lib/ExchangeStructs.sol](https://github.com/code-423n4/2023-03-contest225/blob/main/contracts/lib/ExchangeStructs.sol) | 42 | Exchange Structs |  |
| [contracts/lib/Errors.sol](https://github.com/code-423n4/2023-03-contest225/blob/main/contracts/lib/Errors.sol) | 28 | Errors |  |
| [contracts/interfaces/*](https://github.com/code-423n4/2023-03-contest225/blob/main/contracts/interfaces) | 65 | Interfaces |  |

## Out of Scope

[contracts/Pool.sol](https://github.com/code-423n4/2023-03-contest225/blob/main/contracts/Pool.sol)

# Additional Context

This is the function used for the Dutch Auction to compute the interest rate limit.
[Dutch Auction Interest Rate](https://www.desmos.com/calculator/7ef4rtuzsh)

## Scoping Details 
```
- If you have a public code repo, please share it here:
- How many contracts are in scope?:  10
- Total SLoC for these contracts?:  1000
- How many external imports are there?: 2
- How many separate interfaces and struct definitions are there for the contracts within scope?:  about 3 interfaces and 6 structs
- Does most of your code generally use composition or inheritance?: Inheritance
- How many external calls?:   2
- What is the overall line coverage percentage provided by your tests?:  90
- Is there a need to understand a separate part of the codebase / get context in order to audit this part of the protocol?:   false
- Please describe required context:   n/a
- Does it use an oracle?:  Others; Custom order signature oracle
- Does the token conform to the ERC20 standard?:  
- Are there any novel or unique curve logic or mathematical models?: None
- Does it use a timelock function?:  
- Is it an NFT?: No
- Does it have an AMM?:   false
- Is it a fork of a popular project?:  No
- Does it use rollups?:   no
- Is it multi-chain?:  no
- Does it use a side-chain?: false
```

# Tests
Use Node v16

Install packages
```
yarn install
```

Copy environment variables and set `MAINNET_RPC_URL` with an infura url
```
cp .env.config .env
```

Run tests
```
yarn test
```

Run tests with gas report
```
yarn test:gas-report
```

Run slither
```
yarn slither
```
