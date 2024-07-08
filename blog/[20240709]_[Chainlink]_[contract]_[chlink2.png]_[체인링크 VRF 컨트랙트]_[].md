# ⛓️ 체인링크 VRF 컨트랙트: 심층 분석 (VRF Consumer, Coordinator)

체인링크 VRF(Verifiable Random Function)는 블록체인 상에서 검증 가능한 무작위성을 제공하는 혁신적인 서비스이다. DeFi, 게임, NFT 등 다양한 분야에서 안전하고 투명한 무작위성을 필요로 하는 응용 프로그램에 필수적인 도구라고 볼 수 있다. 

체인링크 VRF는 두 가지 주요 컨트랙트로 구성되어 있는데, 이 컨트랙트를 한 번 살펴보고자 한다.

#### 1. VRF COORDINATOR

VRFCoordinatorV2.sol 컨트랙트는은 VRF(Verifiable Random Function) 요청과 이행을 관리하는 체인링크의 스마트 계약이다. 이 스마트 컨트랙트는 VRF 요청을 처리하고, 결과를 검증하며, LINK 토큰을 통해 비용을 관리하며, 주요 기능은 아래와 같다.

* **난수 생성:** 난수 생성 알고리즘을 사용하여 난수를 생성한다.
* **요청 관리:** 여러 VRF Consumer 컨트랙트로부터 난수 요청을 관리하고 처리한다.
* **응답 전송:** 생성된 난수와 난수 생성 과정을 증명하는 데이터를 VRF Consumer 컨트랙트에 전송한다.
* **오라클 네트워크 관리:** 탈중앙화 오라클 네트워크를 관리하고, 난수 생성 과정의 투명성을 보장한다.
* **보안 관리:** 난수 생성 과정의 보안을 유지하고, 악의적인 행위를 방지한다.

그럼 코드를 하나하나 뜯어 보자. 코드는 아래 깃헙에서 확인 가능하다.
[VRF Coorinator Github Link](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.8/vrf/VRFCoordinatorV2.sol)


```javascript
console.log('hi');
console.log('hi2');
```
