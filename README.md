# Chainlink Plasm Development Node	

This folder contains all files required to run a Chainlink Node for the Plasm Dusty Testnet. You will need `docker` and `docker-compose` 

## Contract Deployment

All Chainlink v6 contracts required (including LinkToken, Oracle, AccessControlledAggregator and EACAggregatorProxy) have been tested, verified and working on Dusty. For the test we did an ETH/USD price aggregator with only one Oracle node. You can view the already deployed ones here

```
LinkToken - 0xD3896BDD73E61a4275e27F660dDF095522F0a1D3
Oracle - 0xa5f33f7d680E596d2f13D1e12434C4de8204C678
AccessControlAggregator - 0xDcaD20CA76D7F0df84e50CdD80AeA43cEEA980fc
EACAggregatorProxy - 0x8581D1D42b5Cb8d6fA2A901ebbdD0BD4f19aB9f6
```

Deploying these contracts required no code modifications, and can be deployed using remix and following the [Ethereum Contract on Dusty Network tutorial](https://docs.plasmnet.io/build/smart-contracts/ethereum-virtual-machine/ethereum-contract-on-dusty-network).

> Note that no $PLD is required to interact with the Dusty network, thus we have set the `ETH_GAS_PRICE_DEFAULT` environment variable to zero. This should be changed for the Plasm Main Network.

## Configure

There are four files to configure in this folder, `.env.chainlink`, `.env.postgres`, `data/password` and `data/api`. All files should already be present in this folder, just configure them to your liking. 

* `.env.chainlink` - All environment variables for the Chainlink Node. The ones present are the ones used for internal testing and validated as working on Dusty.
* `.env.postgres` - All environment variables for the Postgres Database server. 
* `data` - This folder is mapped via docker to `/chainlink` in the Chainlink Node container
* `data/password` - The password used by the Chainlink Node to lock the wallet
* `data/api` - The password used by the Chainlink Node used for login and API calls

For the postgres database password, please make sure they match in both `.env.chainlink` and `.env.postgres`. 

## External Adapters

For price aggregation, a script is included to automatically clone the [External Adapters JS Repo](https://github.com/smartcontractkit/external-adapters-js.git) and build the following external adapters

* Cryptocompare
* metalsapi
* alphavantage

You may use the script `build_adapters.sh` inside this folder to clone and build these, or you may choose to build external adapters yourself. Be sure to include the External Adapter container inside the `docker-compose.yml` file. It should look something like this

### Adding External Adapters

To add an external adapter to the `docker-compose.yml` file, add a section for your external adapter at the bottom, like so

```
  chainlink-external-cryptocompare:
    image: cryptocompare-adapter:latest
    ports:
      - "8080:8080"
    environment:
      API_KEY: <API_KEY_HERE>
```

then, add the container as a requirement to the `chainlink` container, like so

```
services:
  chainlink:
    image: smartcontract/chainlink:latest
    depends_on:
      - chainlink-db
      - chainlink-external-cryptocompare
      ...more external adapters containers here...
```

### Configuring External Adapter in Node

To configure the external adapter in the chainlink node, simply point to the name of the container and the port it's exposing, like so

![example bridge](https://i.ibb.co/0Mj82pZ/msedge-LHpxwubo72.png) 

## Running

To run, simply run `docker-compose up` inside this folder after configuring. This will launch all required containers

To stop, simply run `docker-compose down` or use `CTRL + C` 