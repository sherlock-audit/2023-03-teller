
# Teller Protocol V2 contest details

- Join [Sherlock Discord](https://discord.gg/MABEWyASkp)
- Submit findings using the issue page in your private contest repo (label issues as med or high)
- [Read for more details](https://docs.sherlock.xyz/audits/watsons)

# Resources

- [Teller V2 Gitbook](https://docs.teller.org/teller-v2/)

# On-chain context
 

**Some pointers for filling out the section below:**  
ERC20/ERC721/ERC777/FEE-ON-TRANSFER/REBASING TOKENS:  
*Which tokens do you expect will interact with the smart contracts? Please note that these answers have a significant impact on the issues that will be submitted by Watsons. Please list specific tokens (ETH, USDC, DAI) where possible, otherwise "Any"/"None" type answers are acceptable as well.*

Any tokens are acceptable and this is one area where research is needed to ensure that functionality is not taken advantage of and that assumptions are not broken in the protocol. This includes tokens (principal or collateral) which attempt to use re-entrancy to break assumptions. 


ADMIN:

Even the Admins/owners should not be able to steal funds from the protocol (assuming no changes to solidity code of course).

*Admin/owner of the protocol/contracts.
Label as TRUSTED, If you **don't** want to receive issues about the admin of the contract being able to steal funds. 
If you want to receive issues about the Admin of the contract being able to steal funds, label as RESTRICTED & list specific acceptable/unacceptable actions for the admins.*

EXTERNAL ADMIN:
Even the Admins/owners should not be able to steal funds from the protocol (assuming no changes to solidity code of course).

*These are admins of the protocols your contracts integrate with (if any). 
If you **don't** want to receive issues about this Admin being able to steal funds or result in loss of funds, label as TRUSTED
If you want to receive issues about this admin being able to steal or result in loss of funds, label as RESTRICTED.*
 
```
DEPLOYMENT: mainnet, arbitrum, optimism, polygon, base
ERC20: any
ERC721: any 
ERC777: none
ERC1155: any 
FEE-ON-TRANSFER: any
REBASING TOKENS: none (not supported)
ADMIN: restricted
EXTERNAL-ADMINS: n/a
```


Please answer the following questions to provide more context: 
### Q: Are there any additional protocol roles? If yes, please explain in detail:
1) The roles
2) The actions those roles can take 
3) Outcomes that are expected from those roles 
4) Specific actions/outcomes NOT intended to be possible for those roles

A: 
1) Roles include market owners, borrowers, lenders and liquidators. 

2) Market owners own markets and can control settings such as acceptable interest rates and loan durations within a market.  Each loan that is submitted must belong to a 'market' specific by a 'marketId' and the loan will derive some of its parameters from that.   Borrowers can 'submitbid' which defines a new potential loan with committed collateral (approved but not deposited).  Lenders can accept those submitted bids which transfers the requested principal to the borrower, locks the borrowers collateral into escrow, and activates the loan. 

3)  When the loan is fully repaid, the borrower can withdraw the collateral.  If the loan becomes defaulted instead, then the lender has a 24 hour grace period to claim the collateral (losing the principal) and after that, anyone is allowed to liquidate the loan to pay the borrower the rest of the principal+interest that the borrower failed to repay and claim the collateral for themselves.  

4) Collateral should not be able to withdrawn except for in those ways by the borrower, lender, or liquidator.  Collateral should not be able to be permanently locked/burned in the contracts (loans will always eventually default and be liquidateable). 

Market owners should NOT be able to race-condition attack borrowers or lenders by changing market settings while bids are being submitted or accepted (while tx are in mempool).  Care has been taken to ensure that this is not possible (similar in theory to sandwich attacking but worse as if possible it could cause unexpected and non-consentual interest rate on a loan) and further-auditing of this is welcome.  The best way to defend against this is to allow borrowers and lenders to specify such loan parameters in their TX such that they are explicitly consenting to them in the tx and then reverting if the market settings conflict with those tx arguments. 



NOTE: The owner of the TellerV2 contract(s) is not able to directly withdraw collateral (assuming no contract upgrade) nor are they able to affect loan statuses or committed capital (assuming no contract upgrade).  One ultimate goal when contract logic can be proven to be sound is to revoke ownership of contracts and make them non-upgradeable after that point. 

___
### Q: Is the code/contract expected to comply with any EIPs? Are there specific assumptions around adhering to those EIPs that Watsons should be aware of?

A: The contracts are expected to comply with the ERC20, ERC721 and ERC1155 EIPs of which any can be used as loan collateral.  Only ERC20 tokens can be used as loan principal.  During the audit, please verify that rebasing ERC20 tokens and other weird tokens will not interfere with the protocol assumptions.  If a rebasing/weird token breaks just the loan that it is in, we want to know about it but that is bad but largely OK (not hyper critical) since the borrower and lender both agreed to that asset manually beforehand and, really, shouldnt have.  If a rebasing/weird token in a loan can break OTHER loans that is a major bug that we definitely need to be aware of.  

The Teller contracts also adhere to ERC2771 meta transactions standard which is used in TellerV2Context in order to override 'msgSender' using an address appended to calldata.  This is used for borrowers and lenders to be able to pre-approve a 'trusted market forwarder' contract to submit and accept bids on their behalf.  This ERC2771 context is implemented using a forked version of OpenZeppelins standard non-upgradeable ERC2771 but references the upgradeable ContextUpgradeable so as to not break storage slots as this was amended after initial deployment. 


___

### Q: Please list any known issues/acceptable risks that should not result in a valid finding.

A: Known issue 1: Collateral assets that can be 'paused' for transfer do exhibit a risk since they may not be able to be withdrawn from the loans as collateral. Furthermore, collateral assets that can be made non-transferrable can actually 'poison' a collateral vault and make a loan non-liquidateable since a liquidateLoan call would revert.  


____
### Q: Please provide links to previous audits (if any).
A: n/a

___

### Q: Are there any off-chain mechanisms or off-chain procedures for the protocol (keeper bots, input validation expectations, etc)? 

A: When creating loan commitments, it is important to understand how decimals are expanded for calculating ratios such as maxPrincipalPerCollateralAmount and so typescript libraries are provided for that purpose.  However Teller avoids the use of any centralized mechanisms or on-chain oracles in order to remain as decentralized as possible.  

_____

### Q: In case of external protocol integrations, are the risks of an external protocol pausing or executing an emergency withdrawal acceptable? If not, Watsons will submit issues related to these situations that can harm your protocol's functionality. 

A: [ACCEPTABLE] 

Collateral assets that can be 'paused' for transfer do exhibit a risk since they may not be able to be withdrawn from the loans as collateral. Furthermore, collateral assets that can be made non-transferrable can actually 'poison' a collateral vault and make a loan non-liquidateable since a liquidateLoan call would revert.  Therefore collateral assets which have transferrability which can be paused should not be used as collateral.  Furthermore, the liquidateLoan method in particular should be restructured so that it does not intrinsically withdraw collateral but instead just switches the loan to a 'liquidated' state which allows the liquidator to then with a separate transaction claim the collateral.  


# Audit scope


[teller-protocol-v2 @ cb66c9e348cdf1fd6d9b0416a49d663f5b6a693c](https://github.com/teller-protocol/teller-protocol-v2/tree/cb66c9e348cdf1fd6d9b0416a49d663f5b6a693c)
- [teller-protocol-v2/packages/contracts/contracts/TellerV2.sol](teller-protocol-v2/packages/contracts/contracts/TellerV2.sol)
- [teller-protocol-v2/packages/contracts/contracts/TellerV2Context.sol](teller-protocol-v2/packages/contracts/contracts/TellerV2Context.sol)
- [teller-protocol-v2/packages/contracts/contracts/CollateralManager.sol](teller-protocol-v2/packages/contracts/contracts/CollateralManager.sol)
- [teller-protocol-v2/packages/contracts/contracts/escrow/CollateralEscrowV1.sol](teller-protocol-v2/packages/contracts/contracts/escrow/CollateralEscrowV1.sol)
- [teller-protocol-v2/packages/contracts/contracts/LenderCommitmentForwarder.sol](teller-protocol-v2/packages/contracts/contracts/LenderCommitmentForwarder.sol)
- [teller-protocol-v2/packages/contracts/contracts/LenderManager.sol](teller-protocol-v2/packages/contracts/contracts/LenderManager.sol)



# About Teller V2 Protocol
The Teller Protocol operates as decentralized software, enabling unsecured DeFi digital asset lending and borrowing through an open order-book model.
Through the protocol, borrowers can bridge off-chain data onto on-chain loan requests. Those requesting assets propose a loan request, and those supplying assets commit those assets to loan requests of their choosing. Lenders who agree to loan terms requested by borrowers, based on the data provided or required, transact directly.
Information appended to a loan request is at the borrower's discretion. This may include details from a borrower’s financial stature, social status, identity, or other relevant data. The Teller Protocol is data agnostic and does not have an opinion on the user. The user can gather any type of data from third parties.
