//SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/utils/math/SafeMath.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";

contract AmtToken is ERC20, ReentrancyGuard, ERC20Burnable {
    using SafeMath for uint;
    uint public constant maxEth = 10 ether;
    uint public constant maxBuy = 0.5 ether;
    uint public constant tokenToSend = 100 ether;
    uint public constant burnFee = 2;
    uint public constant forEach = 0.1 ether;
    address private constant deadAddress = 0x000000000000000000000000000000000000dEaD;
    address payable public immutable crowdsaleOwner;

    mapping(address => uint) public purchase;
    uint public ethDeposit;

    constructor() public ERC20("Amrut", "AMT") {
        _mint(address(this), 10000 ether);
        crowdsaleOwner = payable(msg.sender);
    }

    function transfer(address to, uint256 amount) public virtual override returns (bool) {
        address owner = _msgSender();
        uint amountToTransfer = amount.mul(burnFee).div(100);
        _transfer(owner, deadAddress, amountToTransfer);
        _transfer(owner, to, amount.sub(amountToTransfer));
        return true;
    }

    function transferFrom(address from, address to, uint256 amount) public virtual override returns (bool) {
        address spender = _msgSender();
        uint amountToTransfer = amount.mul(burnFee).div(100);
        _spendAllowance(from, spender, amount);
        _transfer(from, deadAddress, amountToTransfer);
        _transfer(from, to, amount.sub(amountToTransfer));
        return true;
    }

    function buy() public payable nonReentrant {
        require(purchase[msg.sender].add(msg.value) <= maxBuy, "you can spend max 0.5 eth");
        require(ethDeposit.add(msg.value) <= maxEth, "maximum eth limit reached!");
        uint ethers = msg.value;
        uint amountToSend = ethers.mul(tokenToSend).div(forEach);
        _transfer(address(this), msg.sender, amountToSend);
        purchase[msg.sender] = purchase[msg.sender].add(msg.value);
        ethDeposit = ethDeposit.add(msg.value);
    }

    function removeEth() public {
        require(msg.sender == crowdsaleOwner, "You are not owner");
        require(ethDeposit == maxEth , "You are not owner");
        bool sent = crowdsaleOwner.send(ethDeposit);
        require(sent, "Failed to send Ether");
    }
}