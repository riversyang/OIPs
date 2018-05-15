# Olympus Labs: Index Architecture Design (Draft 0.1.5)



Jerome Chen, Gerdinand Hardeman created: 2018-03-06, last updated: 2018-03-26

Translated by riversyang, last updated: 2018-05-15



### 1. 背景

### 2. 总体设计

### 3. 组件详情

### 4. 在平台上使用 MOT token

### 5. 后续工作





## 背景

Olympus is a groundbreaking financial ecosystem that defines the protocol for cryptocurrency-based financial products. The Olympus ecosystem provides investors with a comprehensive financial marketplace filled with financial products, services, and applications that serve their investment needs. Olympus Labs provides the tools for investors to construct a well-diversified portfolio, to hedge their downside risk, and to make positive returns in all market conditions.
Olympus 是一个突破性的金融生态系统，它为基于加密货币的金融产品定义了一套协议。
Olympus 生态系统为投资者们提供了一个由金融产品、服务和应用程序所组成的综合性的金融市场。
Olympus Labs 为投资者们提供了一系列工具，可以用来构建多元化的投资组合，对冲下跌风险，并在所有市场条件下获得积极的回报。

In this document, we will describe the architecture of the Index product, the first financial product on the platform. Firstly, we will make a high-level design describing the general ideas of the components, then we will go into details for each of the described components, including their roles, responsibilities, and the interactions amongst the component modules.
在本文中，我们将描述指数（Index）产品的架构，它是平台上的第一个金融产品。
首先，我们将进行总体设计，描述组件的一般思想；然后我们将讨论每个组件的详情，包括它们的角色、职责和组件模块之间的交互。

Next, we will list the potential use cases of the native token of the Olympus Protocol, MOT, and discuss the advantages and disadvantages of these use cases. Finally, we will describe some possibilities which might be extended in the near future.
接下来，我们将列出 Olympus 协议的原生 token —— MOT 的潜在应用，并讨论这些用例的优点和缺点。
最后，我们将描述一些可能在不久的将来进行的扩展。


## 总体设计

An index is an indicator or measure of something, and in finance, it typically refers to a statistical measure of change in a securities market. In the case of financial markets, stock and bond market indexes consist of a hypothetical portfolio of securities representing a particular market or a segment of it. 
指数（index）是指某事物的指标或度量，在金融领域，它通常指证券市场变化的统计度量。
在金融市场中，股票和债券市场指数是由代表着特定市场或其一部分的假想的证券投资组合所构成的。

On the Olympus Labs platform, it doesn't exactly mean the index on a traditional platform, instead, it tries to create a different mode of the index based on the blockchain. More specifically, an index on the Olympus platform means the combination of different tokens and their respective weights.
在 Olympus Labs 平台上，我们尝试创建一种基于区块链的崭新模式的指数产品，而不是像在传统平台中那样。
更具体地说，Olympus 平台上的指数意味着一种由不同 token 和它们各自的权重所构成的组合。

The goals of this design is create an architecture that satisfies the following:
这种设计的目标是创建一种满足以下条件的架构：

1. 可伸缩（Scalable）
2. 可扩展（Extensible）
3. 面向开发者（Developer friendly）
4. 去中心化（Decentralized）
5. 去中心化自制组织（DAO）

To meet the goals listed above, the system needs to be designed in a flexible/extensible way. Hence the need for modular component design. The main components of the system are the following:
为了达到以上目标，系统需要以灵活、可扩展的方式来设计，因而需要将组件模块化。系统中的主要组件有：

1. 兑换合约及其适配器（Exchange provider and its adapters）
2. 策略合约及其客户端（Strategy provider and its clients）
3. 价格合约及其预言机（Price provider and its oracles）
4. 核心智能合约（Core smart contract）

Their relationships are shown in the diagram below.
它们的关系如下图所示。

![Component View](https://raw.githubusercontent.com/Olympus-Labs/OIPs/master/assets/Component%20View.png)


The general concept is to have the core smart contract interact with the other smart contract modules. In this way, the 3rd party clients, such as wallet and applications, can connect with the Olympus Protocol and the entire Olympus Ecosystem by simply interacting with the core smart contract. 
总体思想就是通过核心智能合约（core smart contract）来与其他智能合约模块进行交互。
通过这种方式，像钱包、应用程序这样的第三方客户端，就可以通过简单地与核心智能合约交互来与 Olympus 协议和整个 Olympus 生态系统建立联系。


## 组件详情

As described above, the system consists of 4 components:
如上文锁述，系统由 4 个组件所组成：

##### 核心智能合约（Core Smart Contract）

The core smart contract is responsible for calling all of the other smart contracts. It accepts requests from the end user and splits those requests to the relevant smart contracts. The core smart contract should be the only component that the end user interacts with directly. All other processes flow from the core smart contract behind the scenes and does not interact directly with the end user.
核心智能合约是来负责调用其他智能合约的，它负责接收最终用户的请求，并将其拆分成对相关智能合约的一批请求。
核心智能合约应该是最终用户唯一需要直接进行交互的组件。这个场景中从核心智能合约所发出的其他处理过程都不会直接与最终用户交互。

For the index product, the main features should include:
对于指数（index）产品，其主要特性包括：

1. Retrieving available indexes and their properties. 
2. Retrieving token prices and calculating the index price.
3. Buying tokens included in an index from exchanges or buying the tokenized index.
4. Cancel the order if it takes too long or the price becomes inappropriate.
5. Get notified when the order status changes.
1. 获取有效的指数和它们的相关属性
2. 获取 token 价格并计算指数价格
3. 从交易所购买指数中包含的特定 token 或者直接购买由 token 构成的指数
4. 在长时间未购买成功或价格不再合适时取消订单
5. 当订单状态变动时获得通知


So the interface of the core smart contract should look like the following:
所以核心智能合约的接口应该大致如下：

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
    
    // 转到策略智能合约
    function getStrategies() public returns (uint[] ids, string[] names, string[] descriptions);
    function getStrategy(string strategyId) public returns (string name, address[] tokens, uint[] weights);
    // 转到价格智能合约
    function getPrices(uint strategyId) return (uint[] prices);
    // 在校验并分割为子订单之后发送到兑换智能合约
    function buyIndex(uint strategyId, uint amountInEther, uint[] stopLimits, address depositAddress) public returns (string orderId);
        
    // 供应用程序、第三方客户端来检查详情或状态
    function getIndexOrder(string orderId) public returns (IndexOrder);
    function getIndexStatus (string orderId) public returns (OrderStatus status);
    function cancelOrder(string orderId) public returns (boolean success);
}
```



##### 价格智能合约及其预言机

The Price Smart Contract plays the role of providing prices for tokens, and therefore the price of the indexes, to the user as references when buying indexes.
价格智能合约充当了在用户购买指数产品时为其提供 token 价格乃至指数产品价格的角色。

This smart contract has two main functions: 1. provide data to the core smart contract and 2. retrieve data feed from its oracles. To the core smart contract, it should deliver the tokens that are supported and their respective prices as well as core properties such as the frequency of price udpate. For the oracles, it should provide a clear method for oracles to integrate. 
这个智能合约有两个主要的函数：1、为核心智能合约提供数据；2、从其预言机获取数据。
它会为核心智能合约提供它所支持的 token 的价格以及诸如价格更新频率这样的关键属性。
它也给预言机提供了一个明确的函数来更新价格。

```javascript
contract PriceProvider is Provider, Ownable {
    // 目前，所有价格都是基于 ETH 的。
    event PriceUpdated(uint timeUpdated);
    
    // 供核心智能合约使用
    function getSupportedTokens() external returns (address[] tokenAddresses);
    function getPrices(address[] tokenAddresses) external returns (uint[] prices);
    function getPrice(address tokenAddress) external returns (uint);
    
    // 供预言机使用。msg.sender 就是预言机合约的地址。
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
2. DAO.
   It's always interesting to think about how to grow/develop the community in an organic manner. The key is to create a system with reward / punishment mechanisms by using the MOT token.
3. Rebalancing.
   What we have created so far is only buying, and we will be developing the ability for product creators to rebalance their porfolios.
4. Financial product marketplace.
   Researching different projects require a lot of knowledge and effort. That deserves respect and needs to be honoured. By paying MOT to the financial product creators, it attracts more professionals to develop on the Olympus Platform.
5. Tools for product creators.
  We will be creating a web portal for product creators to easily create and manage their financial products, as well as raise funding for their funds.
