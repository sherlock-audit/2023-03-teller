# Teller V2 Protocol 

Teller Protocol is a decentralized, non-custodial lending book that allows users to lend and borrow any crypto asset without price-based liquidation.


## Pre-requisite software

1. Install NodeJS v16+

2. Install rust, 'foundry' and 'forge'


## Unit Testing

1. Set required ENV variables (see .env.template)
DEFAULT_NETWORK=hardhat 
HARDHAT_DEPLOY_FORK=goerli
GOERLI_RPC_URL= ...

2. Install dependencies with 'yarn install'

3. Run tests with 'yarn test' 

4. Display test coverage report with 'yarn coverage' or 'yarn coverage-report'



## Forge docs 

 > forge doc --serve 


### Troubleshooting 

git submodule add https://github.com/foundry-rs/forge-std packages/contracts/lib/forge-std


