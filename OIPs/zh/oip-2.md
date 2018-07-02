# Olympus Labs: Component-based Financial Products Architecture

Jerome Chen created: 2018-06-14, last updated:2018-06-20

### 1. Background
### 1. 背景

### 2. Overall design
### 2. 总体设计

### 3. Example workflow
### 3. 示例流程

### 4. Communication between protocols and components
### 4. 协议和组件间的通信

### 5. Interfaces
### 5. 接口

### 6. MOT Tokenomics
### 6. MOT 通证经济



## Background
## 背景

Olympus Labs is a protocol for developing tokenized cryptocurrency financial products such as indices, funds, lending products, futures, options, and more. We build the core components of financial products, such as exchange, rebalance, and fee calculation, and financial products can be created by assembling these core components.
Olympus Labs 是为了开发通证化的加密货币金融产品（tokenized cryptocurrency financial products）的一种协议，这些金融产品包括指数、基金、贷款产品、期货和期权等等。我们构建了一些类似于兑换、再平衡以及费用计算之类的核心组件，而金融产品则可以通过组合使用这些核心组件来构建。

Olympus Labs aims to power the next generation fintech DApp ecosystem through the Olympus Protocol. We provide APIs/SDKs for developers and existing applications to integrate our protocol and offer tokenized financial products and services to their users, both investors and investment managers.
Olympus Labs 旨在通过 Olympus 协议来为下一代的金融科技 DApp 生态系统赋能。我们为开发者和既有应用程序提供了 APIs/SDKs 来集成我们的协议，并且为他们的用户，诸如投资者和投资经理人们提供通证化的金融产品和服务。

A new architecture based on modularized and layered design thinking is needed in order to support such an ecosystem. Therefore, the [original architecture](https://github.com/Olympus-Labs/OIPs/blob/master/OIPs/oip-1.md) needs to be upgraded. This document lays out the new architecture, which is highly reusable, extensible, and scalable, and easy for developers build and integrate. 
要支持这样一个生态系统，我们需要一个模块化的、分层设计的新架构。所以我们需要升级 [原先的设计](https://github.com/Olympus-Labs/OIPs/blob/master/OIPs/oip-1.md) 。这份文档展示了一个高可用、可伸缩、可扩展且易于开发者构建和集成的新架构。


## Overall design
## 总体设计

The new architecture takes on a layered design, with each layer being built around the layer within. There are a total of 4 layers in the Olympus Ecosystem, with each inner layer performing functionalities that support the development of each outer layer.
新的架构是基于一种嵌套的分层架构设计的，每一层都是基于包含在其内的一层构建的。Olympus 生态系统总共包含了 4 个层次，每一层都为其外的一层提供了相应的功能。


#### Layer 1: MOT
#### 第一层：MOT

The first layer of the ecosystem is MOT, the native token in the Olympus Ecosystem. MOT is a utility token used for accessing Olympus core components.
生态系统的第一层是 Olympus 生态系统的原生通证 MOT，它是用来访问 Olympus 核心组件的功能性通证。

#### Layer 2: Core Components
#### 第二层：核心组件

The second layer of the ecosystem is Olympus core components. We have conducted a deep analysis of the financial products that exist in the financial world and have reduced them to their core functionalities. These core functionalities are common across various financial products and thus we have developed highly reusable functionality modules, what we call core components. Each core component performs an essential function, such as allowing the buying/selling of tokens, calculating the management fee, conducting risk control, amongst others. The key benefit of these core components is that they can be easily assembled together to create new types of financial products such as indices, funds, lending products, options, and futures, and enables the creation of novel financial products that do not yet exist.
生态系统的第二层是核心组件。基于我们对金融世界中的既存金融产品的深入分析，我们将它们缩减为若干核心功能；这些核心功能可以服务于不同的金融产品，因而我们开发了称为“核心组件”的高重用性功能模块。每一个核心组件会提供一个基本功能，比如允许通证的买卖、计算管理费用、引导风险控制等等。这些核心组件可以用来组合创建出诸如指数、基金、贷款产品、期货和期权之类的金融产品，同时它们也可以用来构建全新的金融产品。

#### Layer 3: Financial Protocols
#### 第三层：金融协议

​The third layer of the ecosystem is Financial Protocols. This layer consists of the financial product templates created using the core components, and thus we see how an inner layer supports the development of an outer layer. There are two types of financial product templates that exist in this layer. The first type of financial product templates is the base template for each financial product type, such as the base template for an index. It is the most basic template for that product type and serves as the standard. All templates of that type of financial product must meet the requirements of the base template in order to qualify as a template of that financial product type. 
生态系统的第三层是金融协议。这一层由一些使用核心组件创建的金融产品模版所构成，这就是我们看到的内层将支持外层的开发。这一层中存在两种金融产品模版。第一种是像指数这样的金融产品基础模版，它是作为某些产品类型的基础模版并作为标准来使用的。这些金融产品类型的所有模版都必须与基础模版相吻合，以便它们可以作为对应类型的金融产品模版来运作。

The second type of financial product templates is custom templates. These templates can be created either by taking a base template and adding/modifying core components or by assembling core components from the ground up. Financial product templates can be easily created in this manner as the product creator only needs to focus on the product logic without having to develop the underlying functionalities. For example, if the product creator is creating a fund and need to calculate his/her management fee, he/she can do so by simply integrating the fee calculation component, without having to build that functionality him/herself.
第二种模版就是定制模版。这些模版可以通过增加/修改基础模版中的核心组件来构建，或者重新组合核心组件来构建。产品创建人可以仅关注产品的逻辑而不用去开发基础功能，金融产品模版则可以通过这种方式简单的构建出来。例如，如果产品创建人正在创建一个需要计算他/她的管理费的基金，他/她可以简单地集成费用计算组件，而不用去自己构建那种功能。

#### Layer 4: Financial Applications
#### 第四层：金融应用程序

The fourth layer of the ecosystem is financial applications, or the DApp layer. The DApp layer of financial applications will be powered by the Olympus Protocol. We envision two main categories of DApps in this layer: 1. DApps that serve producers of financial products (investment managers and product creators) and 2. DApps that serve consumers of financial products (consumers and investors). The Olympus Protocol supports fintech applications such as wallets, portfolio tracking applications, market analysis tools, investment manager applications for creating and managing financial products, financial product marketplaces, financial product exchanges, and much more. Through our APIs/SDKs, both existing applications and developers building new applications can integrate Olympus Protocol to provide the next generation of financial product offerings and services to their users.
生态系统的第四层就是金融应用程序，或 DApp 层。金融应用程序的 DApp 层将由 Olympus 协议所赋能。在这层中，我们预想了两大类 DApp：1、服务于金融产品出品人（投资精力和产品创建者）的 DApp，和 2、服务于金融产品消费者（消费者和投资人）的 DApp。Olympus 协议支持诸如钱包、投资组合跟踪应用、市场分析工具、用于创建和管理金融产品的投资管理应用、金融产品市场、金融产品交易所等等的金融科技应用。无论是既有应用还是构建新应用的开发者都可以通过使用我们的 APIs/SDKs 来集成 Olympus 协议，为他们的用户提供下一代金融产品和服务。

The following diagram shows the 4 layers of the Olympus Ecosystem:
下图展示了 Olympus 生态系统的 4 个层级：


![Application Layers](../assets/Application-Layers.png)

## Example workflow
## 示例流程

Let's use an example to see how the ecosystem works. Here we define 3 user roles:
让我们通过一个示例来看看生态系统是如何工作的。这里我们定义三个用户角色：

1.  Financial product template developer

2.  Fund manager

3.  Fund investor

To start, a smart contract developer sees opportunity in developing financial products for the Olympus Ecosystem, so he/she uses the Olympus core components and the base template for a fund to create his own fund template, heretofore known as “Own Fund Template”, thus becoming a financial product template developer. He/she then registers Own Fund Template in the Olympus Labs Template List, a list of all financial product templates built on the Olympus Protocol that have chosen to register.  


![Developer](../assets/Developer.png)

Now that Own Fund Template is in the Template List, it can be made available to fund managers through any DApp that allows fund managers to create and manage funds, heretofore known as “Management Portal”. In the Management Portal, the fund manager can see a range of fund templates available and the features and customizations that each template supports. Let’s assume that he likes the features of Own Fund Template, then he can choose it, add in his specifications, such as the management fee percentage that he will charge, and deploy an instance of Own Fund Template. The product that he deploys is his fund, specifically a tokenized fund in the form of a smart contract, heretofore known as “Own Fund Instance”. He then manages this fund/smart contract by using the Management Portal, conducting operations such as buying/selling cryptocurrencies.


![Fund Manager](../assets/Fund-Manager.png)

When Own Fund Instance is deployed on the Ethereum Network, it registers to the Olympus Products List, a list of all tokenized financial products based on the Olympus Protocol that have chosen to register. Own Fund Instance can now be made available for investors to invest through any consumer facing DApp such as wallets.

The fund investor can choose to invest in this fund by buying it on a DApp. The fund investor sends his investment, for example ETH, and receives the fund’s tokens in return. Since the fund’s tokens are a tokenized product that meets ERC20 standards, the fund’s tokens will automatically be sent to the fund investor’s wallet.


![Investor](../assets/Investor.png)



## Communication between core components and financial protocols (financial product templates)

Let’s take a look at the communication between the core components and the financial product template by using the Own Fund Instance as an example, and do so from both the perspective of the fund manager and the fund investor.

Own Fund Instance, which is based on the Own Fund Template, consists of multiple core components. In this example, it consists of the whitelist, risk-control, redeem, and exchange components. The figures below illustrates the interactions that occur based on some example actions performed by the fund manager and fund investor.


![Example investement](../assets/Example-investement.png)





# Interfaces

This section explains how a financial product template is constructed from a technical perspective.

1. Component Interface
First, each core component implements its own ComponentInterface, which describes the component and its function.


![Component Interfaces](../assets/Component-Interfaces.png)

Below is a sample definition

```javascript
pragma solidity 0.4.24;


contract ComponentInterface {
    string public name;
    string public description;
    string public category;
    string public version;
}
```

2. Derivative Interface

As financial product templates are constructed by assembling core components, we have defined the Derivative Interface as follows:


![Derivative Interfaces](../assets/Derivative-Interfaces.png)

3. Component Container Interface

The component container interface enables financial products to dynamically interchange core components that make up that product, which is a very powerful feature. Core components of the same type, for example calculating management fees, can be easily exchanged. Let’s take the management fee that fund managers charge in return for managing the fund as an example. Let’s say that when the fund was created, it had used a fee component that charged a fixed monthly management fee. Later on, the fund manager wants to charge the management fee based on a percentage of assets. This can be easily done by simply switching the original fee component to a new fee component by calling the setComponent function. This gives unparalleled flexibility to investment managers to create, manage, and adapt their investment funds and investment products.


```javascript
pragma solidity 0.4.24;


contract ComponentContainerInterface {
    mapping (string => address) components;

    event ComponentUpdated (string _name, address _componentAddress);

    function setComponent(string _name, address _componentAddress) public returns (bool success);
    function getComponentByName(string name) external view returns (address);
}
```

The Derivative Interface implements ERC20 standard, Ownable component, and the ComponentCointainerInterface. Implementation of the ERC20 standard allows interoperability of the tokenized product as it can be traded in the same way as any ERC20 token and can be stored in wallets or be listed on exchanges. The Ownable component gives the creator the permission control to conduct advanced operations such as setting up components.

```javascript
pragma solidity 0.4.24;

import "zeppelin-solidity/contracts/token/ERC20/ERC20.sol";
import "../libs/Ownable.sol";
import "./ComponentContainerInterface.sol";


contract DerivativeInterface is ERC20, Ownable, ComponentContainerInterface {

    enum DerivativeStatus { Active, Paused, Closed }
    enum DerivativeType { Index, Fund }

    string public description;
    string public category;
    string public version;
    DerivativeType public fundType;

    address[] public tokens;
    DerivativeStatus public status;

    // invest, redeem is done in transfer.
    function invest() public payable returns(bool success);
    function changeStatus(DerivativeStatus _status) public returns(bool);
    function getPrice() public view returns(uint);
}
```

The Derivative Interface is powerful because it can be used to derive protocols for new tokenized financial products, such as fund, index, stable coin, lending, future or option. Below is an example implementation. 


```javascript
pragma solidity 0.4.24;

import "./DerivativeInterface.sol";


contract IndexInterface is DerivativeInterface {
    uint[] public weights;
    bool public supportRebalance;

    // this should be called until it returns true.
    function rebalance() public returns (bool success);
}
```

```javascript
pragma solidity 0.4.24;

import "./DerivativeInterface.sol";


contract FundInterface is DerivativeInterface {
    function buyTokens(string _exchangeId, ERC20[] _tokens, uint[] _amounts, uint[] _rates)
        public returns(bool success);

    function sellTokens(string _exchangeId, ERC20[] _tokens, uint[] _amounts, uint[] _rates)
        public returns(bool success);
}
```



## MOT Tokenomics

MOT plays a critical role in the system architecture as it is the utility token used to access the Olympus core components. Each core component has its own fee model. There are three main types of fee models in consideration: 

1.  Based on transaction volume
2.  Based on subscription
3.  Based on calls

The exchange core component fits into the first fee model while the rest of the components can adopt either the second or third fee model. 

A key implication of building the fee and the usage of MOT into the core components is that the value of MOT will be tied directly to the success of Olympus Labs, specifically the success of the Olympus Protocol and the Olympus Ecosystem. It means that when a new developer builds a new financial product template or a DApp that integrates the Olympus Protocol, demand for MOT increases. It means that when a wallet offers Olympus products to their users, demand for MOT increases. It means that when investment managers create funds and financial products and offers these products to their investors, the demand for MOT increases. In essence, any development or progress that increases the usage of the Olympus Protocol increases the demand for MOT. 

Another benefit to building the fees into the core components and deducting the fees directly from the funds is the end user does not need to pay direct fees to be in the Olympus Ecosystem. This simplifies the user experience so that for the users, creating or buying a tokenized Olympus financial product is no different than creating or buying any ERC20 token.

The fees generated from providing the core components will be directed to further development of the project, namely on developing the Olympus Protocol and for building the Olympus Ecosystem, which will in turn generate more demand for MOT, thus creating a virtuous cycle of sustainable development.
