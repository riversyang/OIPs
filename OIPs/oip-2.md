# Olympus Labs: Financial Component Based Architecture Design

Jerome Chen created: 2018-06-14, last updated:2018-06-20

### 1. Background

### 2. Overall design

### 3. Example workflow

### 4. Communication between protocols and components

### 5. Interfaces

### 6. MOT Tokenomics



## Background

Olympus Labs is a protocol for developing tokenized cryptocurrency financial products such as indices, funds, lending products, futures, options, and more. We build the core components of financial products, such as exchange, rebalance, and fee calculation, and financial products can be created by assembling these core components.

Olympus Labs aims to power the next generation fintech DApp ecosystem through the Olympus Protocol. We provide APIs/SDKs for developers and existing applications to integrate our protocol and incorporate tokenized financial products and services to their users, both investors and investment managers. 

A new architecture based on modularized and layered design thinking is needed in order to support such an ecosystem. Therefore, the [original architecture](https://github.com/Olympus-Labs/OIPs/blob/master/OIPs/oip-1.md) needs to be upgraded. This document lays out the new architecture, which is highly reusable, extensible, and scalable, and easy for developers build and integrate. 



## Overall design

The new architecture takes on a layered design, with each layer being built around the layer within. There are 4 layers in the Olympus Ecosystem, with each inner layer performing functionalities that support the development of each outer layer.


#### Layer 1: MOT

The first layer of the ecosystem is MOT, the native token in the Olympus Ecosystem. MOT is a utility token used for accessing Olympus core components.

#### Layer 2: Core Components

The second layer of the ecosystem is Olympus core components. We have conducted a deep analysis of the financial products that exist in the financial world and have reduced them to their core functionalities. These core functionalities are common across various financial products and thus we have developed highly reusable functionality modules, what we call core components. Each core component performs an essential function, such as allowing the buying/selling of tokens, calculating the management fee, conducting risk control, amongst others. The key benefit of these core components is that they can be easily assembled together to create new types of financial products such as indices, funds, lending products, options, and futures, and enables the creation of novel financial products that do not yet exist.

#### Layer 3: Financial Protocols

​The third layer of the ecosystem is Financial Protocols. This layer consists of the financial product templates created using the core components, and thus we see how an inner layer supports the development of an outer layer. There are two types of financial product templates that form this layer. The first type of financial product templates is the base template for each financial product type, for example the base template for an index. It is the most basic template for that product type and serves as the standard. All templates of that type of financial product must meet the requirements of the base template in order to qualify as a template of that financial product. 

The second type of financial product templates is custom templates. These templates can be created either by taking a base template and adding/modifying core components or by assembling core components from the ground up. Financial product templates can be easily created in this manner as the product creator only needs to focus on the product logic without having to think about the development. For example, if the product creator is creating a fund and need to calculate his/her management fee, he/she can do so by simply integrating the fee calculation component, without having to build that functionality him/herself.

#### Layer 4: Financial Applications

The fourth layer of the ecosystem is financial applications, or the DApp layer. The DApp layer of financial applications will be powered by the Olympus Protocol. We envision two main categories of DApps in this layer: 1. DApps that serve producers of financial products (investment managers and product creators) and 2. DApps that serve consumers of financial products (consumers and investors). The Olympus Protocol supports fintech applications such as wallets, portfolio tracking applications, investment manager applications for creating and managing financial products, financial product marketplaces, financial product exchanges, and much more. Through our APIs/SDKs, both existing applications and developers building new applications can integrate Olympus Protocol to provide the next generation of financial product offerings and services to their users.

The following diagram shows the 4 layers of the Olympus Ecosystem:


![Application Layers](../assets/Application-Layers.png)

## Example workflow

Let's use an example to see how the ecosystem works. Here we define 3 user roles:

1.  Financial product template developer

2.  Fund manager

3.  Fund investor

To start, a smart contract developer sees the opportunity in developing financial products for the Olympus Ecosystem, so he/she uses the Olympus core components and the base financial product template for a fund to create his own fund template, heretofore known as “Own Fund Template”, thus becoming a financial product template developer. He/she then registers Own Fund Template in the Olympus Labs Template List, a list of all financial product templates built on the Olympus Protocol that have chosen to register.  


![Developer](../assets/Developer.png)

Now that Own Fund Template is in the Template List, it can be made available to fund managers through a DApp that allows fund managers to create and manage funds, heretofore known as “Management Portal”. In the Management Portal, the fund manager can see a range of fund templates available and the features and customizations that each template supports. Let’s assume that he likes the features of Own Fund Template, then he can choose it, add in his specifications, such as the management fee percentage that he will charge, and deploy an instance of Own Fund Template. The product that he deploys is his fund, specifically a tokenized fund in the form of a smart contract, heretofore known as “Own Fund Instance”. He then manages this fund/smart contract by using the Management Portal, conducting operations such as buying/selling cryptocurrencies.


![Fund Manager](../assets/Fund-Manager.png)

When Own Fund Instance is deployed on the Ethereum Network, it registers to the Olympus Products List, a list of all tokenized financial products based on the Olympus Protocol that have chosen to register. Own Fund Instance can now be made available for investors to invest through any consumer facing DApp such as wallets.

The fund investor can choose to invest in this fund by simply clicking the buy button on the client app / portal. The fund investor sends his investment, for example ETH, and receives the fund’s tokens. Since the fund’s tokens are a tokenized product that meets ERC20 standards, the fund’s tokens will automatically be sent to the fund investor’s wallet.


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

The Derivative Interface implements ERC20 standard, Ownable component, and the ComponentCointainerInterface. Implementation of the ERC20 standard allows interoperability of the tokenized product as it can be traded like any ERC20 token and can be stored in wallets or be listed on exchanges. The Ownable component gives the creator the permission control to conduct advanced operations such as setting up components.

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

The Derivative Interface is powerful because it can be used to derive protocols for a number of tokenized financial products, such as fund, index, stable coin, lending, future or option. Below is an example implementation. 


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

MOT plays a critical role in the system architecture as it is the utility token used to access the Olympus core components. Each core component has its own fee model. There are three main types of fee models: 

1.  Based on transaction volume
2.  Based on subscription
3.  Based on calls

The exchange core component fits into the first fee model while the rest of the components can adopt either the second or third fee model. 

A key implication of building the fee and the usage of MOT into the core components is that the value of MOT will be tied directly to the success of Olympus Labs, specifically the success of the Olympus Protocol and the Olympus Ecosystem. It means that when a new developer builds a new financial product template or a DApp that integrates the Olympus Protocol, demand for MOT increases. It means that when a wallet offers Olympus products to their users, demand for MOT increases. It means that when investment managers create funds and financial products and offers these products to their investors, the demand for MOT increases. In essence, any development or progress that increases the usage of the Olympus Protocol increases the demand for MOT. 

Another benefit to building the fees into the core components and deducting the fees directly from the funds is that no user, neither investment manager nor investor, need to pay any direct fees to be in the Olympus Ecosystem. This simplifies the user experience so that for the users, creating or buying a tokenized Olympus financial product is no different than creating or buying any ERC20 token.

The fees generated from providing the core components will be directed to further development of the project, namely on developing the Olympus Protocol and for building the Olympus Ecosystem, which will in turn generate more demand for MOT, thus creating a virtuous cycle of sustainable development.
