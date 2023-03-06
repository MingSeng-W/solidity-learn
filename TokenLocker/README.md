# Token 流动性锁 (Token Liquidity Locker)

## 介绍
流动性提供者LP代币是一种在AMM协议下，为交易对提供流动性的用户所获得的代币，它代表了用户在流动性池中的份额和收益权。锁定流动性是指用户把LP代币存入一个时间锁合约中，以获取更多的奖励或保证项目方的信任。

 ### AMM 协议
AMM协议是指自动做市商协议，它是一种去中心化的交易方式，不需要传统的买卖订单簿，而是通过一个算法来确定两种资产之间的价格。AMM协议可以提高流动性和效率，降低交易成本和滑点，同时也给流动性提供者带来收益。AMM协议的典型代表有Uniswap、Balancer、Curve等

## 使用
ERC20代币锁合约是一种简单的智能合约，它可以把ERC20代币锁仓一段时间，受益人在锁仓期满后可以取走代币, 一个简单的ERC20代币锁合约的代码示例如下：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

contract TokenLocker {
    // 要锁定的ERC20代币
    IERC20 public token;

    // 代币释放后的受益人
    address public beneficiary;

    // 代币释放的时间戳
    uint256 public releaseTime;

    constructor(
        IERC20 _token,
        address _beneficiary,
        uint256 _releaseTime
    ) {
        // 代币释放时间必须在未来
        require(_releaseTime > block.timestamp, "Release time must be in the future");

        token = _token;
        beneficiary = _beneficiary;
        releaseTime = _releaseTime;
    }

    // 把合约中的代币转给受益人
    function release() public {
        // 当前时间必须在代币释放时间之后
        require(block.timestamp >= releaseTime, "Current time is before release time");

        // 获取合约中的代币数量
        uint256 amount = token.balanceOf(address(this));

        // 代币数量必须大于0
        require(amount > 0, "No tokens to be released");

        // 把代币转给受益人
        token.transfer(beneficiary, amount);
    }
}
```
