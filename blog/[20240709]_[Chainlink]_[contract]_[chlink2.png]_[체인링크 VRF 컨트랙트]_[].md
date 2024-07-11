# ⛓️ 체인링크 VRF 컨트랙트: 심층 분석 

체인링크는 VRF는 2년 가까이 참여하고 있는 프로젝트에서 사용하고 있는 서비스라 익숙한 편이다. 처음에 체인링크 VRF를 접하면서 간단한 스터디를 진행하기는 했지만 예쁘게? 정리해둔 문서가 없어서 CCIP 를 공부하는 김에 VRF 컨트랙트도 가볍게 정리해보가자 한다.
체인링크 VRF는 Verified Random Function 으로 간단히 말해 블록체인 상에서 검증 가능한 랜덤 난수를 제공하는 서비스이다. 이는 블록체인 상에서 랜덤 난수를 필요로 하는 DeFi, 확률 아이템을 가지고 있는 게임, NFT 등 다양한 분야에서 사용할 수 있다.

그럼 왜 랜덤 난수 생성을 위해 체인링크 VRF 를 사용해야 할까? 블록체인 랜덤 난수 생성에는 몇 가지 문제점이 있기 때문이다.
1) 첫 째로, 예측 가능성이다. 블록체인은 투명성과 공개성을 중시하기 때문에 모든 트랜잭션과 블록 정보가 공개된다. 이로 인해, 만약 특정한 알고리즘이나 메서드를 통해 랜덤 난수가 생성된다면 그 과정을 분석해 난수를 예측할 가능성이 생기게 된다
2) 두 번째는, 조작 가능성이다. 블록체인은 분산 시스템으로 각 노드는 블록을 생성할 때 경제적 인센티브를 받게 되는데, 만약 블록 생성 과정에서 특정 난수 값이 유리하게 작용할 수 있다면 채굴자들은 자신에게 유리한 난수를 생성하기 위해 블록을 조작할 수 있다.

keccak 함수를 사용하여 솔리디티 자체적으로 난수 생성이 불가능한 것은 아니지만 예측 가능한 랜덤 난수는, 난수의 기능을 하지 못한다고도 볼 수 있다. 이제 체인링크 VRF COORDINATOR, CONSUMEMR 컨트랙트를 살펴보자. 


#### 1. 어떻게 랜덤난수를 받아와 사용하는가?

![keccak256](img/blog/image.png)

체인링크 VRF는 어떻게 이용하면 되는지는 아래와 같이 간단히 정리할 수 있다. 참고로, VRF 사용 방식에는 Direct Funding 방식과 구독 방식이 있는데 위 예제는 구독 방식이다.

1) 서비스 주체자는 Subscription Manager 를 통해 Subscription 을 생성한다.
2) Subscripstion 이 생성되면, 랜덤 난수를 받아와야 할 메서드가 있는 컨슈머 컨트랙트를 내가 만든 Subscription 에 등록한다.
3) 해당 Subscription 에 LINK 를 충전해둔다. 호출될 때마다, 자동으로 가스비가 차감된다.
4) 일반 유저가 컨슈머 컨트랙트를 통해 VRF Cooridinator 컨트랙트에 random words를 요청한다.
5) 요청을 받으면 VRF Cooridinator 는 이벤트를 발생시켜, random words 를 VRF Service 에 요청한다.
6) VRF Service 는 검증 가능한 랜덤 난수를 Coordinator에 전달하며, 동시에 VRF Coordinator는 해당 난수를 컨슈머 컨트랙트에 전달해준다.


간단히 정리하자면 프로세스는 위와 같다. 여기서 의문점은 VRF Service 일 것이다. 아무래도 온체인이 아니라 오프체인에서 돌아가는 프로세스이기 때문이다. 체인링크가 오라클 이슈를 어떻게 해결했는지에 대한 기술적인 부분은 앞으로 차차 알아가보도록 하자.
의문점 의외에도 아쉬운 부분이 한 가지 있다. 위 과정이 사실상 2 step 이라는 것이다. 요청 -> 랜덤난수 리턴이 하나의 트랜잭션이면 좋으련만, 2 step으로 되어 있어서 요청 트랜잭션 성공 후 랜덤 난수 리턴 트랜잭션이 실패로 끝나게 되는 경우가 발생할 수 있다. 게다가, 랜덤 난수 리턴만 수동으로 다시 발생시켜 줄 방법이 없다.
이 부분이 개인적으로는 조금 아쉬운 부분이다.


#### 2. VRF COORDINATOR

VRFCoordinatorV2.sol 컨트랙트는은 VRF 요청과 이행을 관리하는 체인링크의 스마트 컨트랙트이다. 이는 VRF 요청을 처리하고, 결과를 검증하며, LINK 토큰을 통해 비용을 관리하며, 주요 기능은 아래와 같다.

* **난수 생성:** 난수 생성 알고리즘을 사용하여 난수를 생성한다.
* **요청 관리:** 여러 VRF Consumer 컨트랙트로부터 난수 요청을 관리하고 처리한다.
* **응답 전송:** 생성된 난수와 난수 생성 과정을 증명하는 데이터를 VRF Consumer 컨트랙트에 전송한다.
* **오라클 네트워크 관리:** 탈중앙화 오라클 네트워크를 관리하고, 난수 생성 과정의 투명성을 보장한다.
* **보안 관리:** 난수 생성 과정의 보안을 유지하고, 악의적인 행위를 방지한다.

그럼 코드를 하나하나 뜯어 보면서 위 기능들을 살펴보자. 코드는 아래 github 에서 확인 가능하다.
[VRF Coorinator Github Link](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.8/vrf/VRFCoordinatorV2.sol)

1. constructor

```solidity
constructor(address link, address blockhashStore, address linkEthFeed) ConfirmedOwner(msg.sender) {
    LINK = LinkTokenInterface(link);
    LINK_ETH_FEED = AggregatorV3Interface(linkEthFeed);
    BLOCKHASH_STORE = BlockhashStoreInterface(blockhashStore);
}
```
