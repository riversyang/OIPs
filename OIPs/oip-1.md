# Olympus Labs: Index Architecture Design (Draft 0.1.5)



Jerome Chen, created: 2018-03-06, last updated: 2018-03-11



### 1. Background

### 2. High level design

### 3. Component details.

### 4. Use of MOT token on the platform. 

### 5. What's next?





## Background

Olympus is a groundbreaking financial ecosystem that defines the protocol for cryptocurrency-based financial products. The Olympus ecosystem provides investors with a comprehensive financial marketplace filled with financial products, services, and applications that serve their investment needs. Olympus Labs provides the tools for investors to construct a well diversified portfolio, to hedge their downside risk, and to make positive returns in all market conditions.

In this document, we will describe the architecture of Index, the first financial product on the platform. Firstly, we will make a high level design describing the general ideas of the components, then we will go into details for each of the described components, including their roles, responsiblities and how the interactions are done among them.

Next, we will list possiblities of the usage of MOT token on the platform, advantages and disadvantages. Finally, we will describe some possibilities which might be extended in the near future.



## High Level Design

An index is an indicator or measure of something, and in finance, it typically refers to a statistical measure of change in a securities market. In the case of financial markets, stock and bond market indices consist of a hypothetical portfolio of securities representing a particular market or a segment of it. 

On Olympus Labs platform, it doesn't exactly mean the index on tranditional platform, instead, it tries to create a different mode of index on blockchains basis. More specifically, an index on Olympus platform means the combination of different tokens and the weights of them.

The goals of this design is to make it a product which is:

1. Scalable
2. Extensible
3. Developer friendly.
4. Decentralized.
5. DAO.

To be able to make goals listed above, the system needs to be designed in a such a flexible/extensible way, that needs to be build with the combination of different components, they are:

1. Exchange provider and its adapters.
2. Strategy provider and its clients.
3. Price provider and its oracles.
4. Core smart contract.

Their relationships are like the diagram shown below.

![Component View](https://github.com/Olympus-Labs/protocol-architecture/raw/master/Component%20View.png)



The general idea is that by using a smart contract as the core and strategy, pricing and exchange as providers to make it fulfils the requirements described above. All the providers should register itself to the core smart contract and all the apps / 3rd-party clients only interact with the core smart contract. In this case, one entrance only for the index product to make the system architecture clearer.



## Component Details

As described above, the system consists of 4 components:

##### Core smart contract

The core smart contract helds responsiblity to accept requests from end user, calculate requests and split it into multiple requests and distribute them to different parties. It should be the only entrance that end user awares of and all the other providers should be behind the scene.

For the index product, the main features should include:

1. Retrieving available indexes and their properties. 

2. Retrieving token prices.

3. Buying tokens included in an Index from exchanges.

4. Cancel the order if it takes too long or the price becomes inappropriate.

5. Get notified when the order status is changed.

   ​

So the interface of the core smart contract should look like below:

```javascript
enum ProviderType{
    Strategy,
    Pricing,
    Exchange,
}
    
struct ProviderStatistic {
    uint counter,
}    

struct ERC20Token {
    string symbol;
    address address;
    uint decimal;
}    

contract Provider {
    string name,
    ProviderType type,
    string description,
    map(string => boolean) properties,
    ProviderStatistic statistics,
}
```

```javascript
contract OlymplusLabsCore is Ownable {
    event IndexOrderUpdated (string orderId);
    
    struct IndexOrder {
        string id;
        string strategyId;
        map(address => uint) tokenQuantities;
        OrderStatus status;
        uint dateCreated;
        uint dateCompleted;
    }
    
    enum OrderStatus {
        Placed,
        PartiallyCompleted,
        Completed,
        Cancelled,
        Errored,
    }
    
    // Forward to Strategy smart contract.
    function getStrategies() public returns (uint[] ids, string[] names, string[] descriptions);
    function getStrategy(string strategyId) public returns (string name, address[] tokens, uint[] weights);
    // Forward to Price smart contract.
    function getPrices(uint strategyId) return (uint[] prices);
    // Send to Exchange smart contract after validation and splitted to sub orders.
    function buyIndex(uint strategyId, uint amountInEther, uint[] stopLimits, address depositAddress) public returns (string orderId);
        
    // For app/3rd-party clients to check details / status.
    function getIndexOrder(string orderId) public returns (IndexOrder);
    function getIndexStatus (string orderId) public returns (OrderStatus status);
    function cancelOrder(string orderId) public returns (boolean success);
}
```



##### Price Smart Contract and oracles

Price smart contract part in the system plays the role of providing prices for tokens to the user as references when buying indexes, it can be extended to providing also the instant value of certain Index in the future.

It should interact with 2 parts, providing data to the core smart contract and retrieve data feed from its oracles. To the core smart contract, it should tell which tokens are supported and what the prices are, it should also expose its properties like how often the prices get updated. For the oracles, it should public a method for the oracles to feed in.

```javascript
contract PriceProvider is Provider, Ownable {
    // For now, all price are ETH based.
    event PriceUpdated(uint timeUpdated);
    
    // To core smart contract
    function getSupportedTokens() external returns (address[] tokenAddresses);
    function getPrices(address[] tokenAddresses) external returns (uint[] prices);
    function getPrice(address tokenAddress) external returns (uint);
    
    // TO Oracles. msg.sender is the address of that Oracle.
    function updatePrices(address[] tokenAddresses, uint[] prices) external return (boolean success);
}
```



##### Strategy Smart Contract and clients

The strategy smart contract holds all the strategies provided by different organizations or individual experts. The same as the Price smart contract, it interacts with 2 parties as well, the core smart contract and its clients. 

To the core smart contract, it gives a list of the available strategies and its properties, like if it's private and its price for instance. As for the clients, it exposes an interface for them to control their own strategies according to their expertise and investigations to make the profit maximized.

For now, a strategy consists of:

1. Tokens in it.
2. Weights of the tokens in it.
3. Private or not (Future).
4. Price in MOT if it's private (Future).
5. Dynamic code for adjusting strategy if certain circumstances change. (Future)
6. ...

```javascript
contract StrategyProvider is Provider, Ownable {
    event StrategyChanged(string strategyId);
    
    // To core smart contract
    function getStrategies() external returns (uint[] ids, string[] names, string[] descriptions);
    function getStrategy(string strategyId) external returns (string name, address owner, string description, address[] tokens, uint[] weights);
    
    // To clients
    function createStrategy(string name, string description, address[] tokenAddresses, uint[] weights, boolean private, uint priceInMot) public returns (uint strategyId);
    function updateStrategy(uint strategyId, string description, address[] tokenAddresses, uint[] weights, boolean private, uint priceInMot) public returns (boolean success);
}
```



For strategy editors, it can be extended to different platforms, like Web based, Windows/Mac application or a separate  app. In the future, it can also enable end user the possilbity to create their own strategy.



##### Exchange provider and adapters

This part is responsible for handling all the actual order placements on the exchanges. It includes one smart contract and serveral adapters.

The smart contract interacts with the core smart contract and the adapters to make sure orders will be correctly processed and put onto exchanges according to cert algorithms. There are 2 kinds of exchanges at this moment, DEX and centrailized exchanges. We will be using sub smart contracts to communicate with the DEXes and oracles for the tranditional ones.

The exchange provider should determine which exchange it should take for the order sent from the core smart contract base on price / reputation of the exchanges. It should also have evens when a certain order is placed and when it's processed / completed. it also exposes an interface that if an order takes too long then a user could cancel it manually.

```javascript
contract ExchangeProvider is Ownable, Provider {
    event OrderStatusChanged(string orderId, MarketOrderStatus status);
    
    enum MarketOrderStatus {
        Pending,
        Placed,
        Completed,
        Cancelled,
        Errored,
    }
        
    struct MarketOrder {
	    address buyer;
        ERC20Token token;
        uint quantity;
        uint timestamp;
    }
    
    function getExchanges() external returns (uint[] ids, string[] names);
    function getSupportedTokens(uint exchangeId) external returns (address[] tokenAddresses, string[] names, string[] symbols);
    function getMarketPrices(address[] tokenAddresses) external returns (uint[]);
    function placeOrder(address tokenAddress, uint quantity, uint price, address depositAddress) external returns (string orderId);
    function cancelOrder(string orderId) external returns (boolean success);
}
```



##### Summary

The interaction between the components described above and the buying process is described as below sequence diagram. 

![OlympusSequence](https://github.com/Olympus-Labs/protocol-architecture/raw/master/OlympusSequence.png)



## Use Of MOT on the platform

MOT plays an very important role on OlympusLabs platform, there are several possibilities to give the MOT token value.

For the indexes, the value for the user is that they can use the smart contract service to exchange (for example) their Ether into the indexes through all the possible exchange contracts, without having to manually split their money and create accounts on different exchanges and get benefit from the financial experts by following certain strategies. 

1. Permit to buy OL financial products.

   This is to require the user to have a certain amount of MOT before they are allowed to buy financial products from our platform.

   It promots the use of MOT, yet the downside of this is it limits the size of the active community (depending on the on the amount of MOT defined) which can make use of the Olympus platform.

2. Used as fee on the platform.

   User pay MOT for a fee when interacting with the financical products. A part of this fee will be rewarding the oracles / providers because they give the platform added value. 

3. Voting process.

   By holding certain amount of MOT, user is permited to vote on providers/ oracles. And the award might have relationships with the votes. Most likely this possibility on its own is not a goal for users, as it doesn't directly reward them with anything. For this possibility, a rewards mechanism still needs to be researched.

4. Buy private strategies.

   User can unlock certain indexes with MOT. This price in MOT could be decided by the providers for the Strategy Smart Contract. This might involve privacy solutions such as ZK-STARKS or Homomorphic encryption implemented.

All of these possibilities can be used together, but the technology required for the last suggestion is still in active development in the blockchain space, so this should be implemented at a later stage. However, this possibility brings much to the table in terms of incentives for the providers to provide more strategies, which in turn attracts more strategy consumers. Similar work on this is being done by the Enigma decentralized Data Marketplace.



## What's Next?

Future is bright and interesting, the archeteture we described above is just a beginning. Actually, if we use epics concept, it should be just the Epic 1. For the future there are much more to imagine and to extend:

1. Tokenization.

   For each index, there is a potential to programmatically create smart contract behind and make a token to represent, When users buys it, a certain mount of this tokens will be transfered into user's wallet instead of the bought tokens, they will be held/locked into the smart contract that user can actually exchange them back after a certain locking period depending on different strategies.

2. DAO

   It's always intresting to think about how to make the community to grow / develop without too much interference. The key is to create a system with rewarding / punishment machanism by using the MOT token.

3. Rebalacing.

   What we have created so far is only buying, in reality, the market changes rapiddly, thus it's interesting to think about assets got rebalanced when certain circumstances are changed.

4. Strategy marketplace.

   Researching different projects require a lot of knoweledge and efforts. That deserves respect and needs to be honored. By paying MOT to the strategy provider, it attracts more profestionals to get involved to Olympus Platform and make the community bigger.

5. Tooling for strategy organizations / managers.

   On tranditional financial industry, there are a lot of toolings for different parties / roles. And on blockchain, it's just almost empty yet. By creating the index product, it makes it possible to touch this market by creating toolings for the organizaitoins / funding managers from different angels.
