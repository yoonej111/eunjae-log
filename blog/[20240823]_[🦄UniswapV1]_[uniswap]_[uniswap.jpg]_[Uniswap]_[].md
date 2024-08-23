# Uniswap

유니스왑 V1에서 V4까지 차례대로 어떻게 변화해왔는지 살펴보도록 하자.

### AMM(Automated Market Maker), CPMM(Constant Product Market Maker)

유니스왑을 이해하려면, 먼저 AMM이 무엇을 의미하는지 알아볼 필요가 있다. AMM이란 자동화된 마켓 메이커로, 매수자와 매도자간의 거래를 탈중앙화된 방식으로 자동화된 것을 의미한다. 자동화된 알고리즘을 통해 자산 가격을 조절하며, 유동성 제공자들이 자산을 풀에 예치함으로써 거래가 이루어진다.

AMM을 구성하는 주요 요소는 가격 결정 알고리즘, 유동성 공급자, 토큰 페어, 토큰을 스왑하는 사용자이다. 여기서 가격 결정 알고리즘이 가장 중요한 요소이며, 유니스왑은 AMM 모델 중 어떠한 곱이 항상 동일하게 유지되는 CPMM(Constant Product Market Maker) 을 사용한다.

* 토큰 페어: 거래되는 토큰 쌍을 의미한다. 예를 들어, 거래소에서 USDC로 ETH를 구매하거나 ETH를 USDC로 판매하는 것은 모두 ETH/USDC 페어에서 거래를 하게 되는 것이다. 이 토큰 페어로 구성된 유동성을 풀이라고 한다.
* 유동성 공급자: 토큰 페어를 이루는 두가지 토큰에 대한 유동성을 풀에 공급하는 주체를 의미한다. 이렇게 공급된 유동성은 AMM의 가격 결정 알고리즘에 따른 가격으로 거래된다.
* 트레이더: 토큰을 거래하는 주체이다. 유동성 공급자가 조성한 유동성을 바탕으로 거래한다.

이제 가장 중요한 요소인 가격 결정 알고리즘에 살펴보자. 유니스왑에서 사용하는 가격 결정 알고리즘인 CPMM은, X*Y = K로 표현된다. X와 Y는 토큰 페어의 각 토큰(x,y)의 리저브 수량을 의마하는데, K는 X와 Y를 곱한 값을 의미한다. 예를 들어, 풀에 공급된 X와 Y의 수량이 각각 10,500 이라면 k는 5,000이 된다.
CPMM에서 모든 거래는 거래후에 변경된 변경된 X와 Y의 곱이 항상 일정하게 K로 유지되어야 하며, 이 곱의 값이 일정하게 유지되도록 X,Y의 변화량이 이에 맞게 결정된다. 예를 들어, X를 지불하여 유동성 풀에 넣고 Y를 구매하면, X의 양은 줄어들고 Y의 양은 늘어난다. 이런 구매가 반복된다면, 동일한 X로 구매할 수 있는 Y의 양이 줄어들게 된다. 이로 인해 y의 가격은 상승하고 x의 가격은 하락하게 된다.

![image](img/blog/v1.png)

위 이미지를 보면 조금 더 쉽게 이해할 수 있다. 현재 유니스왑의 ETH/OMG 풀에는 10 ETH와 500 OMG 의 유동성이 공급되어 있고, 이 때 상수 K는 5000 이 된다. 이는 유동성이 추가로 공급되거나 제거될 때만 변경되고, 거래가 계속 일어나더라도 변경되지 않는다.

위의 예시에서는, Buyer가 1 ETH를 지불하고 OMG를 구매하려고 하는데, 이 Buyer가 1ETH로 구매할 수 있는 OMG의 양은 45.5가 된다. 여기서 의문점이 하나 생길 수 있다. 거래 이전의 1ETH의 가격은 5OMG 였는데, 막상 거래를 하고 보니 1ETH 당 45.5OMG로 거래가 되었다는 점이다. 이처럼 처음에 의도했던 가격과 실제 거래 가격 사이에 차이가 발생하는 것을 슬리피지라고 한다. 유니스왑의 CPMM은 그 특성상 슬리피지가 발생할 수 밖에 없게 되어 있고, 풀에 공급된 유동성의 양이 클수록 상대적인 슬리피지 규모는 점점 줄어들게 된다.

* 초기 상태: ETH/OMG 풀에 10 ETH와 500 OMG 가 들어 있다. 이 때, 1 ETH는 500 OMG 이다.
* 거래 발생: 구매자가 1 ETH를 추가하고, OMG 를 빼가려고 한다. 유동성 풀에는 11ETH와 500 OMG가 있게 된다. 하지만, 거래 후 풀 내 자산 비율이 고정된 곱을 유지해야하므로 50 OMG를 출금할 수 없다. 실제로는 11 ETH 와 (5000/11) 454.5 OMG가 풀에 남게 되고, 45.5 OMG가 출금된다.

그럼 유동성의 규모가 커질수록 슬리피지가 줄어드는지 한 번 계산해보자.

현재 ETH/DAI에 아래와 같이 유동성이 공급되어 있다고 가정해보고, 10 ETH 를 DAI 로 스왑해보면서 수치를 비교해보자.

1) 
##### 거래 전
* **Pair:** ETH/DAI
* **Liquidity:** 100ETH / 1000 DAI
* **k:** 100000

##### 거래 후
* **Liquidity:** 110ETH / 909.1 DAI
* **Buyer에게 지급한 금액:** 90.9 DAI

2)
##### 거래 전
* **Pair:** ETH/DAI
* **Liquidity:** 1000ETH / 10000 DAI
* **k:** 10000000

##### 거래 후
* **Liquidity:** 1010ETH / 9901 DAI
* **Buyer에게 지급한 금액:** 99 DAI


이렇게 계산하다보면, 유동성이 큰 풀일 수록 처음 계산했던 100 DAI에 가까워지는 것을 확인할 수 있다.


### 스마트 컨트랙트를 한 번 살펴보자.

기존 컨트랙트 코드는 vyper로 작성[https://github.com/Uniswap/v1-contracts]되어 있는 듯 하여, 비슷하게 작성되어 있는 이 코드[https://github.com/Uniswap/old-solidity-contracts]를 한 번 분석해보자. 

#### Uniswap Factory

유니스왑 팩토리는 유니스왑 거래 계약의 레지스트리 역할을 담당한다. 즉, 팩토리로 유니스왑 시스템에 추가된 ERC20 토큰 및 거래 주소를 조회할 수 있다.

##### **1.상속**

```javascript
contract UniswapFactory is FactoryInterface
```

* UniswapFactory는 FactoryInterface 라는 인터페이스를 상속받음.

##### **2.이벤트**

```javascript
event ExchangeLaunch(address indexed exchange, address indexed token);
```

* ExchangeLaunch 이벤트를 정의한다.
* 여기서 배운 것이 indexed 인데, indexed가 지정된 필드는 이더리움 블록체인의 이벤트 로그에 인덱스가 생성되어, 특정 값을 빠르게 찾을 수 있도록 도와준다고 한다. 


##### **3.launchExchange** 

```javascript
function launchExchange(address _token) public returns (address exchange) {
    require(tokenToExchange[_token] == address(0)); // 토큰에 대한 exchange가 이미 생성되어 있는지 확인
    require(_token != address(0) && _token != address(this));  // 유효한 토큰 주소인지 확인
    
    UniswapExchange newExchange = new UniswapExchange(_token);  // 새로운 exchange 인스턴스 생성
    tokenList.push(_token);  // 토큰 목록에 추가
    tokenToExchange[_token] = newExchange;  // 토큰과 exchange 연결
    exchangeToToken[newExchange] = _token;  // exchange와 토큰 연결
    
    ExchangeLaunch(newExchange, _token);  // 이벤트 발생
    return newExchange;  // 새로 생성된 exchange 주소 반환
}
```
* 특정 토큰풀을 거래 리스트에 등록하는 함수


##### **4.getExchangeCount** 

```javascript
function getExchangeCount() public view returns (uint exchangeCount) {
    return tokenList.length;
}
```

* 등록되어 있는 tokenList의 개수를 조회하는 함수


##### **5.tokenToExchangeLookup** 

```javascript
function tokenToExchangeLookup(address _token) public view returns (address exchange) {
    return tokenToExchange[_token];
}
```

* 토큰 주소로 exhange 주소 조회하는 함수

##### **6.exchangeToTokenLookup** 

```javascript
function exchangeToTokenLookup(address _exchange) public view returns (address token) {
    return exchangeToToken[_exchange];
}
```

* exhange 주소로 토큰 주소 조회하는 함수


#### Uniswap Exchange
유니스왑(Uniswap) V1 프로토콜의 핵심 구성 요소 중 하나로, 특정 토큰과 이더(ETH) 간의 자동화된 거래(교환)를 처리하고, 유동성 제공자를 위한 기능을 포함한다.


##### **1.이벤트**

```javascript
event EthToTokenPurchase(address indexed buyer, uint256 indexed ethIn, uint256 indexed tokensOut);
event TokenToEthPurchase(address indexed buyer, uint256 indexed tokensIn, uint256 indexed ethOut);
event Investment(address indexed liquidityProvider, uint256 indexed sharesPurchased);
event Divestment(address indexed liquidityProvider, uint256 indexed sharesBurned);
```

* 각종 필요한 이벤트들을 정의


##### **2.Consutructor**

```javascript
function UniswapExchange(address _tokenAddress) public {
    tokenAddress = _tokenAddress;
    factoryAddress = msg.sender;
    token = ERC20Interface(tokenAddress);
    factory = FactoryInterface(factoryAddress);
}
```

* 생성할 토큰의 주소를 전달받아 위와 같이 필요한 값들을 설정한다. 신기한 구성이네,,


