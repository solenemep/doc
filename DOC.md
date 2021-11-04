# Solidity Contract Interaction

### Get list of whitelisted policies

##### First import interfaces

- IContractRegistry.sol

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.7.4;
pragma experimental ABIEncoderV2;

interface IContractsRegistry {
  function getPolicyBookRegistryContract() external view returns (address);
}

```

- IPolicyBookRegistry.sol

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.7.4;
pragma experimental ABIEncoderV2;

interface IPolicyBookRegistry {
  function count() external view returns (uint256);
  function listWhitelisted(uint256 offset, uint256 limit) external view returns (address[] memory _policyBooksArr);
}

```

##### Then insert this code

```js
function getWhiteListedPolicies() public returns (address[] memory _policyBooksArr) {
  // SET UP
  address contractRegistryAddress = "0x8050c5a46FC224E3BCfa5D7B7cBacB1e4010118d";
  IContractRegistry contractRegistry = IContractsRegistry(contractRegistryAddress);
  IPolicyBookRegistry policyBookRegistry = IPolicyBookRegistry(contractRegistry.getPolicyBookRegistryContract());

  // FUNCTION CALL
  uint256 countWhiteListed = policyBookRegistry.countWhitelisted();
  return policyBookRegistry.listWhitelisted(0, countWhiteListed);
}

```

**Returns**

| Name             | Type      | Description                  |
| ---------------- | --------- | ---------------------------- |
| \_policyBooksArr | address[] | List of whitelisted policies |

**Arguments**

None

### Get list of purchased policies

##### First import interfaces

- IContractRegistry.sol

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.7.4;
pragma experimental ABIEncoderV2;

interface IContractsRegistry {
  function getPolicyRegistryContract() external view returns (address);
}

```

- IClaimingRegistry.sol

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.7.4;
pragma experimental ABIEncoderV2;

interface IClaimingRegistry {
  enum ClaimStatus {
    CAN_CLAIM,
    UNCLAIMABLE,
    PENDING,
    AWAITING_CALCULATION,
    REJECTED_CAN_APPEAL,
    REJECTED,
    ACCEPTED
  }
}

```

- IPolicyRegitry.sol

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.7.4;
pragma experimental ABIEncoderV2;

import "./IClaimingRegistry.sol";

interface IPolicyRegistry {
  struct PolicyInfo {
    uint256 coverAmount;
    uint256 premium;
    uint256 startTime;
    uint256 endTime;
  }
  function getPoliciesLength(address _userAddr) external view returns (uint256);
  function getPoliciesInfo(
    address _userAddr,
    bool _isActive,
    uint256 _offset,
    uint256 _limit
  )
    external
    view
    returns (
      uint256 _policiesCount,
      address[] memory _policyBooksArr,
      PolicyInfo[] memory _policies,
      IClaimingRegistry.ClaimStatus[] memory _policyStatuses
    );
}

```

##### Then insert this code

```js
function getPurchasedPolicies(
  bool _isActive
)
  public
  returns (
    uint256 _policiesCount,
    address[] memory _policyBooksArr,
    PolicyInfo[] memory _policies,
    IClaimingRegistry.ClaimStatus[] memory _policyStatuses
  )
{
  // SET UP
  address contractRegistryAddress = "0x8050c5a46FC224E3BCfa5D7B7cBacB1e4010118d";
  IContractRegistry contractRegistry = IContractsRegistry(contractRegistryAddress);
  IPolicyRegistry policyRegistry = IPolicyRegistry(contractRegistry.getPolicyRegistryContract());

  // FUNCTION CALL
  uint256 count = policyRegistry.getPoliciesLength();
  return policyRegistry.getPoliciesInfo(msg.sender, _isActive, 0, count);
}

```

**Returns**

| Name             | Type                            | Description                                    |
| ---------------- | ------------------------------- | ---------------------------------------------- |
| \_policiesCount  | uint256                         | The number of police in the array              |
| \_policyBooksArr | address[]                       | The array of policy books addresses            |
| \_policies       | PolicyInfo[]                    | The array of policies info (struct PolicyInfo) |
| \_policyStatuses | IClaimingRegistry.ClaimStatus[] | The array of status of claim                   |

**Arguments**

| Name       | Type | Description                                                                                |
| ---------- | ---- | ------------------------------------------------------------------------------------------ |
| \_isActive | bool | If true, returns an array with information about active policies, if false, about inactive |

### Purchase policy

It is supposed that before calling this function : the msg.sender has approved the policybook address to spend the totalPrice get by getPolicyPrice.</br>
Also, the distributor must be whitelisted by BridgeMutual admin to get a fee.

##### First import interfaces

- IPolicyBookFacade.sol

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.7.4;
pragma experimental ABIEncoderV2;

interface IPolicyBookFacade {
    function addLiquidityFor(address _liquidityHolderAddr, uint256 _liquidityAmount) external;
}

```

- IPolicyBook.sol

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.7.4;
pragma experimental ABIEncoderV2;

import "./IPolicyBookFacade.sol";

interface IPolicyBook {
    function policyBookFacade() external view returns (IPolicyBookFacade);
}

```

##### Then insert this code

```js
    function purchasePolicy(
        address policyBookAddress,
        uint256 _epochsNumber,
        uint256 _coverTokens
    ) public {
        // SET UP
        address policyBookFacadeAddress = IPolicyBook(policyBookAddress).policyBookFacade();
        IPolicyBookFacade policyBookFacade = IPolicyBookFacade(policyBookFacadeAddress);

        // FUNCTION CALL
        policyBookFacade.buyPolicyFromDistributorFor(msg.sender, _epochsNumber, _coverTokens, address(this));
    }
```

**Returns**

None

**Arguments**

| Name              | Type    | Description                      |
| ----------------- | ------- | -------------------------------- |
| policyBookAddress | address | The address of the chosen policy |
| \_epochsNumber    | uint256 | The period policy will cover     |
| \_coverTokens     | uint256 | The amount paid for the coverage |

### Earn interest

It is supposed that before calling this function : the msg.sender has approved the policybook address to spend the liquidityAmount converted in stblAmount.

##### First import interfaces

- IPolicyBookFacade.sol

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.7.4;
pragma experimental ABIEncoderV2;

interface IPolicyBookFacade {
    function addLiquidityFor(address _liquidityHolderAddr, uint256 _liquidityAmount) external;
}

```

- IPolicyBook.sol

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.7.4;
pragma experimental ABIEncoderV2;

import "./IPolicyBookFacade.sol";

interface IPolicyBook {
    function policyBookFacade() external view returns (IPolicyBookFacade);
}

```

##### Then insert this code

```js
    function earnInterest(address policyBookAddress, uint256 _liquidityAmount) public {
        // SET UP
        address policyBookFacadeAddress = IPolicyBook(policyBookAddress).policyBookFacade();
        IPolicyBookFacade policyBookFacade = IPolicyBookFacade(policyBookFacadeAddress);

        // FUNCTION CALL
        policyBookFacade.addLiquidityFor(msg.sender, _liquidityAmount);
    }
```

**Returns**

None

**Arguments**

| Name              | Type    | Description                      |
| ----------------- | ------- | -------------------------------- |
| policyBookAddress | address | The address of the chosen policy |
| \_liquidityAmount | uint256 | The amount of coverage provision |