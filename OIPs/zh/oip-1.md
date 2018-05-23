# Olympus Labs: Index Architecture Design (Draft 0.1.5)



Jerome Chen, Gerdinand Hardeman created: 2018-03-06, last updated: 2018-03-26

Translated by riversyang, last updated: 2018-05-15



### 1. 背景

### 2. 总体设计

### 3. 组件详情

### 4. 在平台上使用 MOT token

### 5. 后续工作





## 背景

Olympus 是一个突破性的金融生态系统，它为基于加密货币的金融产品定义了一套协议。
Olympus 生态系统为投资者们提供了一个由金融产品、服务和应用程序所组成的综合性的金融市场。
Olympus Labs 为投资者们提供了一系列工具，可以用来构建多元化的投资组合，对冲下跌风险，并在所有市场条件下获得积极的回报。

在本文中，我们将描述指数（Index）产品的架构，它是平台上的第一个金融产品。
首先，我们将进行总体设计，描述组件的一般思想；然后我们将讨论每个组件的详情，包括它们的角色、职责和组件模块之间的交互。

接下来，我们将列出 Olympus 协议的原生 token —— MOT 的潜在应用，并讨论这些用例的优点和缺点。
最后，我们将描述一些可能在不久的将来进行的扩展。


## 总体设计

指数（index）是指某事物的指标或度量，在金融领域，它通常指证券市场变化的统计度量。
在金融市场中，股票和债券市场指数是由代表着特定市场或其一部分的假想的证券投资组合所构成的。

在 Olympus Labs 平台上，我们尝试创建一种基于区块链的崭新模式的指数产品，而不是像在传统平台中那样。
更具体地说，Olympus 平台上的指数意味着一种由不同 token 和它们各自的权重所构成的组合。

这种设计的目标是创建一种满足以下条件的架构：

1. 可伸缩（Scalable）
2. 可扩展（Extensible）
3. 面向开发者（Developer friendly）
4. 去中心化（Decentralized）
5. 去中心化自制组织（DAO）

为了达到以上目标，系统需要以灵活、可扩展的方式来设计，因而需要将组件模块化。系统中的主要组件有：

1. 兑换合约及其适配器（Exchange provider and its adapters）
2. 策略合约及其客户端（Strategy provider and its clients）
3. 价格合约及其预言机（Price provider and its oracles）
4. 核心智能合约（Core smart contract）

它们的关系如下图所示。

![Component View](https://raw.githubusercontent.com/Olympus-Labs/OIPs/master/assets/Component%20View.png)


总体思想就是通过核心智能合约（core smart contract）来与其他智能合约模块进行交互。
通过这种方式，像钱包、应用程序这样的第三方客户端，就可以通过简单地与核心智能合约交互来与 Olympus 协议和整个 Olympus 生态系统建立联系。


## 组件详情

如上文锁述，系统由 4 个组件所组成：

#### 核心智能合约（Core Smart Contract）

核心智能合约是来负责调用其他智能合约的，它负责接收最终用户的请求，并将其拆分成对相关智能合约的一批请求。
核心智能合约应该是最终用户唯一需要直接进行交互的组件。这个场景中从核心智能合约所发出的其他处理过程都不会直接与最终用户交互。

对于指数（index）产品，其主要特性包括：

1. 获取有效的指数和它们的相关属性
2. 获取 token 价格并计算指数价格
3. 从交易所购买指数中包含的特定 token 或者直接购买由 token 构成的指数
4. 在长时间未购买成功或价格不再合适时取消订单
5. 当订单状态变动时获得通知


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



#### 价格智能合约及其预言机

价格智能合约充当了在用户购买指数产品时为其提供 token 价格乃至指数产品价格的角色。

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



#### 策略智能合约及其客户端

策略智能合约保存了所有组织和独立专家创建的金融产品。与价格智能合约类似，策略智能合约也会与两个主体进行交互——核心智能合约以及金融产品创建人。

对于核心智能合约，它会提供一个可用的金融产品列表及其相关属性。对于金融产品创建人，它则为他们提供了一些创建和管理金融产品的接口。

目前，一个策略由以下一些内容组成：

1. 其中的 token
2. 其中 token 的权重
3. 策略是否为私有（未来）
4. 如果策略为私有，则其应有一个以 MOT 为单位的价格（未来）
5. 当经济情况变动时用来调整策略的动态代码（未来）
6. ...

```javascript
contract StrategyProvider is Provider, Ownable {
    event StrategyChanged(string strategyId);
    
    // 供核心智能合约使用
    function getStrategies() external returns (uint[] ids, string[] names, string[] descriptions);
    function getStrategy(string strategyId) external returns (string name, address owner, string description, address[] tokens, uint[] weights);
    
    // 供客户端使用
    function createStrategy(string name, string description, address[] tokenAddresses, uint[] weights, boolean private, uint priceInMot) public returns (uint strategyId);
    function updateStrategy(uint strategyId, string description, address[] tokenAddresses, uint[] weights, boolean private, uint priceInMot) public returns (boolean success);
}
```



对于策略编辑者而言，它可以扩展到不同的平台上，比如基于网络的 Windows/Mac 应用程序或者一个独立的应用程序。
未来，它也可以给最终用户提供创建属于他们自己的策略的可能性。



#### 兑换智能合约及其适配器

兑换智能合约的职责是负责处理实际交易中的所有实际订单。它包含了一个智能合约和多个适配器。

这个智能合约会与核心智能合约以及适配器进行交互，来确保订单被正确的处理，并基于特定的规则发送到交易所。
交易所有两种类型：去中心化交易所（DEX）和中心化交易所。
我们会使用子智能合约来与 DEX 们交互，并通过预言机来与中心化交易所交互。

交易所适配器会基于一种优化的策略将交易请求分发到交易所中，这是一种考虑了诸如价格、市场深度以及交易所声誉等账目因素的规则。
它也会产生一些针对特定交易的发布、处理/完成这些操作的事件。这个合约还给用户提供了一个可以在必要的时候手工取消交易的接口。

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



#### 小结

下面的流程图可视化地介绍了上文提到的组件间的交互以及购买流程。

![OlympusSequence](https://raw.githubusercontent.com/Olympus-Labs/OIPs/master/assets/OlympusSequence.png)



## MOT 在 Olympus 生态系统中的作用

MOT（Mount Olympus Tokens）是 Olympus 生态系统中的原生 token，并扮演了非常重要的角色。它是作为 Olympus 生态系统中的费用而存在的。
从最终用户角度，他们需要使用 MOT 来订购某个金融产品。对于产品创建人，他们以 MOT 的形式来收取服务费。
与此类似，价格预言机、第三方应用程序以及交易所都可以基于他们对 Olympus 生态系统的贡献收取一部分由最终用户提供的使用费。


## 后续工作

如果我们以宏大的概念来描述开发路线图，那么以上这些仅仅是第一步，是开始。未来则拥有更多的可能性。

1. 代币化（Tokenization）
   对每一个指数（index）或基金（fund），我们可以通过程序创建一个智能合约和 token 来作为它的代理。当用户买进这个产品时，以这个特定的 token 为单位的资金会发送到用户的钱包，而不是用户购买的产品里所涉及的实际的 token。
2. 去中心化自制组织（DAO）
   考虑如果以一种绿色的方式发展/开发社区总是非常有趣的。这里的关键是如何利用 MOT token 来创建一个具有激励/惩罚规则的系统。
3. 再平衡（Rebalancing）
   我们目前建立的只是购买机制，我们也将为产品创建者们开发用于再平衡他们的投资组合的功能。
4. 金融产品市场（Financial product marketplace）
   研究不同的项目需要大量的知识和工作，这值得尊重和嘉奖。我们可以通过向金融产品创建人支付 MOT 来吸引更多的专业人士共同在 Olympus 平台上开发产品。
5. 产品创建人工具（Tools for product creators）
   我们将会为产品创建人构建一个门户网站，以使他们可以简单的创建和管理他们的金融产品、为他们的基金募集资金。
