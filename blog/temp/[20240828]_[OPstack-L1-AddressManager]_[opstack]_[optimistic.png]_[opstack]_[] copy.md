# AddressManager 컨트랙트

### 컨트랙트 코드

```javascript
// SPDX-License-Identifier: MIT
pragma solidity 0.8.15;

import { Ownable } from "@openzeppelin/contracts/access/Ownable.sol";

/// @custom:legacy
/// @title AddressManager
/// @notice AddressManager is a legacy contract that was used in the old version of the Optimism
///         system to manage a registry of string names to addresses. We now use a more standard
///         proxy system instead, but this contract is still necessary for backwards compatibility
///         with several older contracts.
contract AddressManager is Ownable {
    /// @notice Mapping of the hashes of string names to addresses.
    mapping(bytes32 => address) private addresses;

    /// @notice Emitted when an address is modified in the registry.
    /// @param name       String name being set in the registry.
    /// @param newAddress Address set for the given name.
    /// @param oldAddress Address that was previously set for the given name.
    event AddressSet(string indexed name, address newAddress, address oldAddress);

    /// @notice Changes the address associated with a particular name.
    /// @param _name    String name to associate an address with.
    /// @param _address Address to associate with the name.
    function setAddress(string memory _name, address _address) external onlyOwner {
        bytes32 nameHash = _getNameHash(_name);
        address oldAddress = addresses[nameHash];
        addresses[nameHash] = _address;

        emit AddressSet(_name, _address, oldAddress);
    }

    /// @notice Retrieves the address associated with a given name.
    /// @param _name Name to retrieve an address for.
    /// @return Address associated with the given name.
    function getAddress(string memory _name) external view returns (address) {
        return addresses[_getNameHash(_name)];
    }

    /// @notice Computes the hash of a name.
    /// @param _name Name to compute a hash for.
    /// @return Hash of the given name.
    function _getNameHash(string memory _name) internal pure returns (bytes32) {
        return keccak256(abi.encodePacked(_name));
    }
}
```

`AddressManager` 는 문자열 이름을 주소와 매핑하는 레지스트리를 관리한다. 이는 Optimism 시스템의 이전 버전과의 하위 호환성을 위해 사용된다. 현재는 더 표준적인 프록시 시스템을 사용하지만, 과거의 여러 계약과의 호환성을 유지하기 위해 이 계약이 여전히 필요하다.

1. **상속**
    - `AddressManager`는 OpenZeppelin의 `Ownable` 컨트랙트를 상속받아, 소유자만 특정 기능을 수행할 수 있도록 제한한다.
2. **매핑** 
    - `addresses`라는 `mapping(bytes32 => address)`를 통해 문자열 이름의 해시를 주소와 매핑한다.
3. **이벤트**
    - `AddressSet` 이벤트가 정의되어 있으며, 레지스트리에 있는 주소가 변경될 때 이를 기록한다. 이 이벤트는 이전 주소와 새 주소를 모두 포함한다.
4. **함수**
    - `setAddress`: 이 함수는 특정 이름에 대한 주소를 설정한다. 오직 소유자만 호출할 수 있으며, 이름의 해시값을 키로 사용해 주소를 매핑에 저장 후 주소가 변경되면 `AddressSet` 이벤트를 발생시킨다.
    - `getAddress`: 이 함수는 특정 이름에 해당하는 주소를 조회하고 반환힌디/
    - `_getNameHash`: 내부 함수로, `keccak256` 해시 함수를 사용해 이름의 해시값을 계산해 반환한다.
5. **기타**
    - 주소를 설정하는 setter / getter 함수만 있는 컨트랙트
    - Ownable 보다는 Ownable2Step 을 사용하면 더 좋지 않을까?