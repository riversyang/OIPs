# Olympus Labs: Index Architecture Design (Draft 0.1.5)



Jerome Chen, Gerdinand Hardeman created: 2018-03-06, last updated: 2018-03-26



### 1. Background

### 2. High level design

### 3. Component details.

### 4. Use of MOT token on the platform. 

### 5. What's next?





## Background

Olympus is a groundbreaking financial ecosystem that defines the protocol for cryptocurrency-based financial products. The Olympus ecosystem provides investors with a comprehensive financial marketplace filled with financial products, services, and applications that serve their investment needs. Olympus Labs provides the tools for investors to construct a well-diversified portfolio, to hedge their downside risk, and to make positive returns in all market conditions.

In this document, we will describe the architecture of the Index product, the first financial product on the platform. Firstly, we will make a high-level design describing the general ideas of the components, then we will go into details for each of the described components, including their roles, responsibilities, and the interactions amongst the component modules.

Next, we will list the potential use cases of the native token of the Olympus Protocol, MOT, and discuss the advantages and disadvantages of these use cases. Finally, we will describe some possibilities which might be extended in the near future.



## High Level Design

An index is an indicator or measure of something, and in finance, it typically refers to a statistical measure of change in a securities market. In the case of financial markets, stock and bond market indexes consist of a hypothetical portfolio of securities representing a particular market or a segment of it. 

On the Olympus Labs platform, it doesn't exactly mean the index on a traditional platform, instead, it tries to create a different mode of the index based on the blockchain. More specifically, an index on the Olympus platform means the combination of different tokens and their respective weights.

The goals of this design is create an architecture that satisfies the following:

1. Scalable
2. Extensible
3. Developer friendly
4. Decentralized
5. DAO

To meet the goals listed above, the system needs to be designed in a flexible/extensible way. Hence the need for modular component design. The main components of the system are the following:

1. Exchange provider and its adapters.
2. Strategy provider and its clients.
3. Price provider and its oracles.
4. Core smart contract.

Their relationships are shown in the diagram below.

![Component View](https://raw.githubusercontent.com/Olympus-Labs/OIPs/master/assets/Component%20View.png)


The general concept is to have the core smart contract interact with the other smart contract modules. In this way, the 3rd party clients, such as wallet and applications, can connect with the Olympus Protocol and the entire Olympus Ecosystem by simply interacting with the core smart contract. 


## Component Details

As described above, the system consists of 4 components:

##### Core Smart Contract

The core smart contract is responsible for calling all of the other smart contracts. It accepts requests from the end user and splits those requests to the relevant smart contracts. The core smart contract should be the only component that the end user interacts with directly. All other processes flow from the core smart contract behind the scenes and does not interact directly with the end user.

For the index product, the main features should include:

1. Retrieving available indexes and their properties. 
2. Retrieving token prices and calculating the index price.
3. Buying tokens included in an index from exchanges or buying the tokenized index.
4. Cancel the order if it takes too long or the price becomes inappropriate.
5. Get notified when the order status changes.


So the interface of the core smart contract should look like the following:

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

The Price Smart Contract plays the role of providing prices for tokens, and therefore the price of the indexes, to the user as references when buying indexes.

This smart contract has two main functions: 1. provide data to the core smart contract and 2. retrieve data feed from its oracles. To the core smart contract, it should deliver the tokens that are supported and their respective prices as well as core properties such as the frequency of price udpate. For the oracles, it should provide a clear method for oracles to integrate. 

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

The strategy smart contract holds all of the financial products created by  organizations and individual experts. similar to the price smart contract, the strategy smart contract interacts with 2 parties as well, the core smart contract and the financial product creators. 

To the core smart contract, it gives a list of the available financial products and its properties. As for the financial product cretors, it provides an interface for them to create and manage their financial products.

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



For strategy editors, it can be extended to different platforms, like Web-based, Windows/Mac application or a separate app. In the future, it can also provide the end user with the possibility to create their own strategy.



##### Exchange Smart Contract and adapters

The exchange smart contract is responsible for handling all of the actual order placements on the exchanges. It includes one smart contract and several adapters.

This smart contract interacts with the core smart contract and the adapters to make sure orders will be correctly processed and directed to the exchanges according to certain algorithms. There are currently 2 categories of exchanges, Decentralized Exchanges (DEX) and centralized exchanges. We will be using sub smart contracts to communicate with the DEXes and oracles for the centralized exchanges.

The exchange provider should distributes the trade orders to the exchanges based on an optimization algorithmn that takes into account factors such as price, market depth, and reputation of the exchange. It should also have events for when a certain order is placed and for when it is processed/completed. The smart contract also provides an interface that allows the user to cancel the order manually if necessary. 

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

The interaction between the components described above and the buying process is visualized in the sequence diagram below. 

![OlympusSequence](https://raw.githubusercontent.com/Olympus-Labs/OIPs/master/assets/OlympusSequence.png)



## Use Of MOT in the Olympus Ecosystem

MOT (Mount Olympus Tokens) is the native token in the Olympus Ecosystem and plays a very important role. MOT is used as the fee in the Olympus Ecosystem. From the end user perspective, they pay the fee associated with purchasing a financial product using MOT. From the product creators perspective, they receive their fee in the form of MOT. Similarly, price oracles, third party applications, and exchanges, could potentially receive a part of the fee paid by the end user based on their contribution to the Olympus Ecosystem.


## What's Next?

If we use the idea of epics to conceptualize the development roadmap, this is epic 1, the beginning. The future holds many more possibilities. 

1. Tokenization.
   For each index or fund, programmatically create a smart contract and token to represent this index or fund. When users buy it, a certain amount of these tokens will be transferred into the users' wallet instead of the underlyimg tokens. 
2. DAO
   It's always interesting to think about how to grow/develop the community in an organic manner. The key is to create a system with reward / punishment mechanisms by using the MOT token.
3. Rebalancing.
   What we have created so far is only buying, and we will be developing the ability for product creators to rebalance their porfolios.
4. Financial product marketplace.
   Researching different projects require a lot of knowledge and effort. That deserves respect and needs to be honoured. By paying MOT to the financial product creators, it attracts more professionals to develop on the Olympus Platform.
5. Tools for product creators.
  We will be creating a web portal for product creators to easily create and manage their financial products, as well as raise funding for their funds.
