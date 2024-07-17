# ⛓️ 체인링크 VRF 컨트랙트: 심층 분석 

체인링크는 VRF는 2년 가까이 참여하고 있는 프로젝트에서 사용하고 있는 서비스라 익숙한 편이다. 처음에 체인링크 VRF를 접하면서 간단한 스터디를 진행하기는 했지만 예쁘게? 정리해둔 문서가 없어서 CCIP 를 공부하는 김에 VRF 컨트랙트도 가볍게 정리해보가자 한다.
체인링크 VRF는 Verified Random Function 으로 간단히 말해 블록체인 상에서 검증 가능한 랜덤 난수를 제공하는 서비스이다. 이는 블록체인 상에서 랜덤 난수를 필요로 하는 DeFi, 확률 아이템을 가지고 있는 게임, NFT 등 다양한 분야에서 사용할 수 있다.

그럼 왜 랜덤 난수 생성을 위해 체인링크 VRF 를 사용해야 할까? 블록체인 랜덤 난수 생성에는 몇 가지 문제점이 있기 때문이다.
1) 첫 째로, 예측 가능성이다. 블록체인은 투명성과 공개성을 중시하기 때문에 모든 트랜잭션과 블록 정보가 공개된다. 이로 인해, 만약 특정한 알고리즘이나 메서드를 통해 랜덤 난수가 생성된다면 그 과정을 분석해 난수를 예측할 가능성이 생기게 된다
2) 두 번째는, 조작 가능성이다. 블록체인은 분산 시스템으로 각 노드는 블록을 생성할 때 경제적 인센티브를 받게 되는데, 만약 블록 생성 과정에서 특정 난수 값이 유리하게 작용할 수 있다면 채굴자들은 자신에게 유리한 난수를 생성하기 위해 블록을 조작할 수 있다.

keccak 함수를 사용하여 솔리디티 자체적으로 난수 생성이 불가능한 것은 아니지만 예측 가능한 랜덤 난수는, 난수의 기능을 하지 못한다고도 볼 수 있다. 이제 체인링크 VRF COORDINATOR, CONSUMEMR 컨트랙트의 함수들을 살펴보자. 


#### 1. 어떻게 랜덤난수를 받아와 사용하는가?

![image](img/blog/image.png)

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

##### **1. constructor**

```javascript
constructor(address link, address blockhashStore, address linkEthFeed) ConfirmedOwner(msg.sender) {
    LINK = LinkTokenInterface(link);
    LINK_ETH_FEED = AggregatorV3Interface(linkEthFeed);
    BLOCKHASH_STORE = BlockhashStoreInterface(blockhashStore);
}
```

**input 매개변수**
* **link:** LINK 토큰 컨트랙트의 주소이다.
* **blockhashStore:** BlockhashStore 컨트랙트의 주소이다. BlockhashStore는 블록체인의 블록 해시를 저장하고 조회하는 기능을 제공하는 컨트랙트인데, 블록 해시는 블록체인의 블록을 고유하게 식별하는 값으로, 스마트 컨트랙트 내에서 과거 블록의 해시를 안전하게 저장하고 나중에 참조할 수 있도록 한다.
[BlockhashStore](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.8/vrf/dev/BlockhashStore.sol)
* **linkEthFeed:** Chainlink ETH 가격 피드 컨트랙트의 주소이다. linkEthfeed는 체인링크의 오라클 네트워크를 통해 ETH와 LINK 간의 최신 환율 정보를 제공하는 컨트랙트이다. 이를 통해 스마트 컨트랙트가 실시간으로 LINK와 ETH 간의 환율을 조회하고 사용할 수 있게 된다.
[AggregatorV3Interface](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.8/shared/interfaces/AggregatorV3Interface.sol)


##### **2. registerProvingKey**

```javascript
function registerProvingKey(address oracle, uint256[2] calldata publicProvingKey) external onlyOwner {
    bytes32 kh = hashOfKey(publicProvingKey);
    if (s_provingKeys[kh] != address(0)) {
      revert ProvingKeyAlreadyRegistered(kh);
    }
    s_provingKeys[kh] = oracle;
    s_provingKeyHashes.push(kh);
    emit ProvingKeyRegistered(kh, oracle);
}
```

이 함수는 오라클이 사용할 수 있는 공개 증명키(Public Proving Key) 를 등록하는 데 사용되는 함수이다.

**input 매개변수**
* **oracle:** 사용할 oracle 의 주소이다.
* **publicProvingKey:** 공개증명 키이다. 이는 VRF 작업을 수행할 때 사용되는 키이며, 변경되지 않는 읽기 전용 데이터인 'calldata'로 지정되어 함수 호출 시 전달되는 입력 데이터로 취급된다.

**함수 설명**
* **registerProvingKey** 함수는 external 로 선언되어 외부에서만 호출이 가능하며 onlyOnwer modifier로 지정되어 있어 컨트랙트 소유자만이 호출할 수 있다.
* **hashOfKey** 함수를 사용하여 **publicProvingKey** 로부터 해시를 계산해 kh에 저장한다.
* 위에서 계산된 hash가 이미 등록되 키인지 확인하고 (**s_provingKeys[kh] != address(0)**), 그렇다면 **ProvingKeyAlreadyRegistered** 에러를 발생시킨다.
* 등록되지 않은 키라면 해시된 키를 키 값으로 oracle 주소를 저장하고, 해시된 키를 **s_provingKeyHashes** 배열에 추가한다.
* 위 과정이 완료되었다면, 이벤트를 발생시켜 키가 성공적으로 등록되었음을 알린다. (**emit ProvingKeyRegistered(kh, oracle)**)


##### **3. hashOfKey**

```javascript
function hashOfKey(uint256[2] memory publicKey) public pure returns (bytes32) {
    return keccak256(abi.encode(publicKey));
}
```

publicKey 를 입력받아 keccak256을 돌려 반환하는 함수이다.

**input 매개변수**
* **publicKey:** 공개증명 키이다. 

**함수 설명**
* keccak256을 사용하면 어떤 값이 들어와도 고정된 길이의 값이 리턴된다. keccak256에 대한 내용은 이전에 적어둔 글에서도 확인 가능하다.
[Keccak256을](https://yoonej111.github.io/eunjae-log/?post=%5B20240706%5D_%5BKeccak256%5D_%5Balgorithm%5D_%5Balgorithm.jpg%5D_%5Bkeccak256%EC%9D%98+%EC%9E%91%EB%8F%99%EC%9B%90%EB%A6%AC%5D_%5B%5D.md)


##### **4. deregisterProvingKey**

```javascript
function deregisterProvingKey(uint256[2] calldata publicProvingKey) external onlyOwner {
    bytes32 kh = hashOfKey(publicProvingKey);
    address oracle = s_provingKeys[kh];
    if (oracle == address(0)) {
      revert NoSuchProvingKey(kh);
    }
    delete s_provingKeys[kh];
    for (uint256 i = 0; i < s_provingKeyHashes.length; i++) {
      if (s_provingKeyHashes[i] == kh) {
        bytes32 last = s_provingKeyHashes[s_provingKeyHashes.length - 1];
        // Copy last element and overwrite kh to be deleted with it
        s_provingKeyHashes[i] = last;
        s_provingKeyHashes.pop();
      }
    }
    emit ProvingKeyDeregistered(kh, oracle);
}
```

이 함수는 등록된 provingKey를 해제하는 역할을 한다. 이 과정에서 해시 계산, 키 존재 여부 확인, 매핑 삭제, 배열 요소 삭제 등 이벤트 발생 등의 단계를 거친다.

**input 매개변수**
* **publicProvingKey:** 컨트랙트에서 삭제하고자 하는 publicProvingKey를 입력해준다.

**함수 설명**
* **deregisterProvingKey** 함수도 external, onlyOwner로 지정되어 있어 컨트랙트 오너만 외부에서 호출 가능하다.
* 등록할때와 마찬가지로 **hashOfKey** 를 사용하여 해시를 계산한다.
* 등록된 키가 없는 경우(**oracle == address(0)**), 에러를 발생 시킨다. (**revert NoSuchProvingKey(kh)**)
* **s_provingKey** 에 매핑되어 있는 값을 삭제하고, s_provingKeyHashes 배열에 있는 값을 삭제한다. 이 때, 마지막 값을 가져와 삭제할 값에 덮어쓰고 pop을 사용해 마지막 값을 제거한다.
* 위 작업이 완료되면, 이벤트를 발생시켜 키가 성공적으로 deregister 되었음을 알린다.(**emit ProvingKeyDeregistered(kh, oracle)**)


##### **5. setConfig**

```javascript
function setConfig(
    uint16 minimumRequestConfirmations,
    uint32 maxGasLimit,
    uint32 stalenessSeconds,
    uint32 gasAfterPaymentCalculation,
    int256 fallbackWeiPerUnitLink,
    FeeConfig memory feeConfig
  ) external onlyOwner {
    // 요청 확인 수가 최대 허용값을 초과하는 경우 오류 발생
    if (minimumRequestConfirmations > MAX_REQUEST_CONFIRMATIONS) {
      revert InvalidRequestConfirmations(
        minimumRequestConfirmations,
        minimumRequestConfirmations,
        MAX_REQUEST_CONFIRMATIONS
      );
    }
    // LINK가격이 0 이하인 경우 오류 발생
    if (fallbackWeiPerUnitLink <= 0) {
      revert InvalidLinkWeiPrice(fallbackWeiPerUnitLink);
    }
    // 설정 값을 업데이트
    s_config = Config({
      minimumRequestConfirmations: minimumRequestConfirmations,
      maxGasLimit: maxGasLimit,
      stalenessSeconds: stalenessSeconds,
      gasAfterPaymentCalculation: gasAfterPaymentCalculation,
      reentrancyLock: false
    });
    s_feeConfig = feeConfig;
    s_fallbackWeiPerUnitLink = fallbackWeiPerUnitLink;
    emit ConfigSet(
      minimumRequestConfirmations,
      maxGasLimit,
      stalenessSeconds,
      gasAfterPaymentCalculation,
      fallbackWeiPerUnitLink,
      s_feeConfig
    );
}
```
이 함수는 컨트랙트의 설정 값을 언데이트하는 함수이다. 다양한 설정 값들 (minimumRequestConfirmations, maxGasLimit, stalenessSeconds, gasAfterPaymentCalculation, reentrancyLock) 을 입력으로 받아 내부 상태 변수를 업데이트 한다.

**input 매개변수**
* **minimumRequestConfirmations:** 요청이 완료되기 위해 블록체인 네트워크 내에서 필요한 최소한의 블록 확인 수를 설정한다.
* **maxGasLimit:** 요청을 처리하는 데 사용할 수 있는 최대 가스 한도를 설정한다.
* **stalenessSeconds:** 데이터가 유효한다고 간주할 수 있는 최대 시간을 설정한다.
* **gasAfterPaymentCalculation:** 가스비 계산이 완료된 후 추가로 필요한 가스 양을 설정한다.
* **fallbackWeiPerUnitLink:** LINK 토큰의 기본 가격을 Wei 단위로 나타낸다.
* **feeConfig**: 요청을 처리할 때 적용되는 다양한 수수료 설정을 담고 있다.

**함수 설명**
* **setConfig** 함수도 external, onlyOwner로 지정되어 있어 컨트랙트 오너만 외부에서 호출 가능하다.
* 요청 확인 수가 최대 허용값을 초과하는 경우(**(minimumRequestConfirmations > MAX_REQUEST_CONFIRMATIONS)**), 오류를 발생 시킨다. (**revert InvalidRequestConfirmations**)
* LINK 가격이 0 이하인 경우에도 오류를 발생시킨다. (**fallbackWeiPerUnitLink <= 0**)
* 위 과정을 완료하면, **s_config, s_feeConfig, s_fallbackWeiPerUnitLink** 값을 업데이트한다.
* 마지막으로 설정값이 변경되었다는 **ConfigSet** 이벤트를 발생킨다.