# WETH
TH本身并不符合ERC20标准, 使用智能合约包装， 成为了WETH，符合ERC20同质化代币标准，脱下包装为，1:1兑换ETH。 WETH的开发是为了提高区块链之间的互操作性

WETH符合ERC20标准，它比普通的ERC20多了两个功能：
1. 存款：包装，用户将ETH存入WETH合约，并获得等量的WETH。 
2. 取款：拆包装，用户销毁WETH，并获得等量的ETH




```solidity
     event  Deposit(address indexed dst, uint wad);

     event  Withdrawal(address indexed src, uint wad);


 function deposit() public payable {
        _mint(msg.sender, msg.value);
        emit Deposit(msg.sender, msg.value);
    }

    // 提款函数，用户销毁WETH，取回等量的ETH
function withdraw(uint amount) public {
        require(balanceOf(msg.sender) >= amount);
        _burn(msg.sender, amount);
        payable(msg.sender).transfer(amount);
        emit Withdrawal(msg.sender, amount);
  }

```