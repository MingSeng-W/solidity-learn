# ERC721简介
ERC-721（Ethereum Request for Comments 721），由 William Entriken、Dieter Shirley、Jacob Evans、Nastassia Sachs 在 2018 年 1 月提出，是一个在智能合约中实现代币 API 的非同质化代币标准。这种类型的代币是独一无二的，并且可能与来自同一智能合约的另一代币有不同的价值，也许是因为它的年份、稀有性、甚至是它的观感. 我们常见的NFT可以由该协议实现。

# ERC721 与 ERC 20 的区别
ERC 20 是一种同质化代币代币标准，每个同种类的代币都是等价的， 例如BTC和ETH， 每枚的价格都是等值的， 而ERC 721 是一种非同质化代币， 每个NFT都是独一无二的， 例如NFT中最火的无聊猿BAYC， 每枚BAYC都是独一无二的， 有不同的价格。


# ERC721 接口
ERC721的主合约一共引用了4个接口合约：IERC721.sol, IERC721Receiver.sol, IERC721Metadata.sol

IERC165 是一个标准接口，用于检查合约是否支持某个接口，如果支持，返回true，否则返回false。

```solidity
interface IERC165 {
    function supportsInterface(bytes4 interfaceId) external view returns (bool);
}
```

ERC721主合约对supportsInterface()的实现如下

```solidity
function supportsInterface(bytes4 interfaceId) public view virtual override(ERC165, IERC165) returns (bool) {
    return
        interfaceId == type(IERC721).interfaceId ||
        interfaceId == type(IERC721Metadata).interfaceId ||
        super.supportsInterface(interfaceId);
}
```


# IERC721接口介绍

IERC721是ERC721的接口合约，里面包括3个event和9个function

```solidity
  interface IERC721 is IERC165 {
    event Transfer(address indexed from, address indexed to, uint256 indexed tokenId); // 转移代币事件，包括发送方、接收方和代币ID
    event Approval(address indexed owner, address indexed approved, uint256 indexed tokenId); // 授权事件，包括所有者、被授权者和代币ID
    event ApprovalForAll(address indexed owner, address indexed operator, bool approved); // 授权所有事件，包括所有者、操作者和是否授权

    function balanceOf(address owner) external view returns (uint256 balance); // 返回指定地址的代币余额

    function ownerOf(uint256 tokenId) external view returns (address owner); // 返回指定代币ID的所有者地址

    function safeTransferFrom(address from, address to, uint256 tokenId) external; // 安全转移代币，包括发送方、接收方和代币ID

    function transferFrom(address from, address to, uint256 tokenId) external; // 转移代币，包括发送方、接收方和代币ID

    function approve(address to, uint256 tokenId) external; // 授权代币给指定地址，包括被授权者和代币ID

    function getApproved(uint256 tokenId) external view returns (address operator); // 返回指定代币ID的被授权者地址

    function setApprovalForAll(address operator, bool _approved) external; // 授权所有代币给指定地址，包括操作者和是否授权

    function isApprovedForAll(address owner, address operator) external view returns (bool); // 返回指定地址是否被授权操作所有代币

    function safeTransferFrom(address from, address to, uint256 tokenId, bytes calldata data) external; // 安全转移代币，包括发送方、接收方、代币ID和数据
  }
```


# IERC721Receiver介绍
ERC721Receiver是一个接口合约，用于接收ERC721代币，它包含了一个onERC721Received()函数，用于接收代币，该函数的参数有4个，分别是发送方、接收方、代币ID和数据，返回值是一个bytes4类型的值，该值是一个固定的值0x150b7a02，如果接收方不支持接收代币，返回值为0x0。

```solidity
  interface IERC721Receiver {
    function onERC721Received(address operator, address from, uint256 tokenId, bytes calldata data) external returns (bytes4);
  }
```

# IERC721Metadata介绍
ERC721Metadata是一个接口合约，用于获取代币的元数据，它包含了3个函数，分别是name()、symbol()和tokenURI()，分别用于获取代币的名称、代币的符号和代币的URI。

```solidity
  interface IERC721Metadata is IERC721 {
    function name() external view returns (string memory); // 返回代币的名称

    function symbol() external view returns (string memory); // 返回代币的符号

    function tokenURI(uint256 tokenId) external view returns (string memory); // 返回代币的URI
  }
```



# ERC721主合约
ERC721主合约包含6个状态变量和28个函数。

状态变量

```solidity
    string private _name; // 代币名称
    string private _symbol; // 代币符号
    mapping (uint256 => address) private _owners; // 代币ID和所有者地址的映射
    mapping (address => uint256) private _balances; // 地址和代币余额的映射
    mapping (uint256 => address) private _tokenApprovals; // 代币ID和被授权者地址的映射
    mapping (address => mapping (address => bool)) private _operatorApprovals; // 地址和操作者地址和是否授权的映射
```

函数

```solidity
        constructor (string memory name_, string memory symbol_) {
            _name = name_;
            _symbol = symbol_;
        }
        // ERC165标准接口，用于检查合约是否支持某个接口，如果支持，返回true，否则返回false
        function supportsInterface(bytes4 interfaceId) public view virtual override(ERC165, IERC165) returns (bool) {
            return
                interfaceId == type(IERC721).interfaceId ||
                interfaceId == type(IERC721Metadata).interfaceId ||
                super.supportsInterface(interfaceId);
        }

        // 余额查询，返回指定地址的代币余额
        function balanceOf(address owner) public view virtual override returns (uint256) {
            require(owner != address(0), "ERC721: balance query for the zero address");
            return _balances[owner];
        }

        // 所有者查询，返回指定代币ID的所有者地址
        function ownerOf(uint256 tokenId) public view virtual override returns (address) {
            address owner = _owners[tokenId];
            require(owner != address(0), "ERC721: owner query for nonexistent token");
            return owner;
        }

        // 根据Metadata查询代币ID，返回指定代币ID的URI
        function name() public view virtual override returns (string memory) {
            return _name;
        }

        // 根据Metadata查询代币ID，查询代币代号
        function symbol() public view virtual override returns (string memory) {
            return _symbol;
        }

        // 根据Metadata查询代币ID，查询代币URI
        function tokenURI(uint256 tokenId) public view virtual override returns (string memory) {
            require(_exists(tokenId), "ERC721Metadata: URI query for nonexistent token");

            string memory baseURI = _baseURI();
            return bytes(baseURI).length > 0
                ? string(abi.encodePacked(baseURI, tokenId.toString()))
                : '';
        }

        // 基URI，会被tokenURI()调用，跟tokenId拼成tokenURI
        function _baseURI() internal view virtual returns (string memory) {
            return "";
        }

        // 授权
        function approve(address to, uint256 tokenId) public virtual override {
            address owner = ERC721.ownerOf(tokenId);
            require(to != owner, "ERC721: approval to current owner");

            require(_msgSender() == owner || isApprovedForAll(owner, _msgSender()),
                "ERC721: approve caller is not owner nor approved for all"
            );

            _approve(to, tokenId);
        }

        //
        function getApproved(uint256 tokenId) public view virtual override returns (address) {
            require(_exists(tokenId), "ERC721: approved query for nonexistent token");

            return _tokenApprovals[tokenId];
        }

        // 设置操作者
        function setApprovalForAll(address operator, bool approved) public virtual override {
        _setApprovalForAll(_msgSender(), operator, approved);
        }

        // 转账
    function transferFrom(
        address from,
        address to,
        uint256 tokenId
    ) public virtual override {
        //solhint-disable-next-line max-line-length
        require(_isApprovedOrOwner(_msgSender(), tokenId), "ERC721: transfer caller is not owner nor approved");

        _transfer(from, to, tokenId);
    }

    // 安全转账
     function safeTransferFrom(
        address from,
        address to,
        uint256 tokenId,
        bytes memory _data
    ) public virtual override {
        require(_isApprovedOrOwner(_msgSender(), tokenId), "ERC721: transfer caller is not owner nor approved");
        _safeTransfer(from, to, tokenId, _data);
    }
    // 查询 tokenId是否存在
    function _exists(uint256 tokenId) internal view virtual returns (bool) {
        return _owners[tokenId] != address(0);
    }

    //查询 spender地址是否被可以使用tokenId（他是owner或被授权地址）。
    function _isApprovedOrOwner(address spender, uint256 tokenId) internal view virtual returns (bool) {
        require(_exists(tokenId), "ERC721: operator query for nonexistent token");
        address owner = ERC721.ownerOf(tokenId);
        return (spender == owner || getApproved(tokenId) == spender || isApprovedForAll(owner, spender));
    }

    //_safeMint：安全mint函数，铸造tokenId并转账给 to地址
    function _safeMint(
        address to,
        uint256 tokenId,
        bytes memory _data
    ) internal virtual {
        _mint(to, tokenId);
        require(
            _checkOnERC721Received(address(0), to, tokenId, _data),
            "ERC721: transfer to non ERC721Receiver implementer"
        );
    }

    //铸造tokenId并转账给 to地址
      function _mint(address to, uint256 tokenId) internal virtual {
        require(to != address(0), "ERC721: mint to the zero address");
        require(!_exists(tokenId), "ERC721: token already minted");

        _beforeTokenTransfer(address(0), to, tokenId);

        _balances[to] += 1;
        _owners[tokenId] = to;

        emit Transfer(address(0), to, tokenId);

        _afterTokenTransfer(address(0), to, tokenId);
    }

     // 销毁
     function _burn(uint256 tokenId) internal virtual {
        address owner = ERC721.ownerOf(tokenId);

        _beforeTokenTransfer(owner, address(0), tokenId);

        // 清空授权
        _approve(address(0), tokenId);

        _balances[owner] -= 1;
        delete _owners[tokenId];

        emit Transfer(owner, address(0), tokenId);

        _afterTokenTransfer(owner, address(0), tokenId);
    }

    // 转账
    function _transfer(
        address from,
        address to,
        uint256 tokenId
    ) internal virtual {
        require(ERC721.ownerOf(tokenId) == from, "ERC721: transfer from incorrect owner");
        require(to != address(0), "ERC721: transfer to the zero address");

        _beforeTokenTransfer(from, to, tokenId);

        // 清空授权
        _approve(address(0), tokenId);

        _balances[from] -= 1;
        _balances[to] += 1;
        _owners[tokenId] = to;

        emit Transfer(from, to, tokenId);

        _afterTokenTransfer(from, to, tokenId);
    }

    // 授权
    function _approve(address to, uint256 tokenId) internal virtual {
        _tokenApprovals[tokenId] = to;
        emit Approval(ERC721.ownerOf(tokenId), to, tokenId);
    }

    // 设置授权
    function _setApprovalForAll(
        address owner,
        address operator,
        bool approved
    ) internal virtual {
        require(owner != operator, "ERC721: approve to caller");
        _operatorApprovals[owner][operator] = approved;
        emit ApprovalForAll(owner, operator, approved);
    }

    // 检查接受者是否实现了onERC721Received接口
    function _checkOnERC721Received(
        address from,
        address to,
        uint256 tokenId,
        bytes memory _data
    ) private returns (bool) {
        if (to.isContract()) {
            try IERC721Receiver(to).onERC721Received(_msgSender(), from, tokenId, _data) returns (bytes4 retval) {
                return retval == IERC721Receiver.onERC721Received.selector;
            } catch (bytes memory reason) {
                if (reason.length == 0) {
                    revert("ERC721: transfer to non ERC721Receiver implementer");
                } else {
                    assembly {
                        revert(add(32, reason), mload(reason))
                    }
                }
            }
        } else {
            return true;
        }
    }

    // 在转账前调用
    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 tokenId
    ) internal virtual {}


    // 在转账后调用
    function _afterTokenTransfer(
        address from,
        address to,
        uint256 tokenId
    ) internal virtual {}

```
