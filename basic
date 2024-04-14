// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "0x1/Signer.sol";

contract MyToken {
    using Signer for Signer.SignerType;

    mapping(address => uint256) public balanceOf;
    mapping(address => VestingSchedule) public vestingSchedules;

    uint256 public constant totalSupply = 10000000;
    uint256 public constant issuanceDate = 1646064000; // Example issuance date (2022-02-28)
    uint256 public constant cliffDuration = 36 * 30 days; // 36 months in seconds
    uint256 public constant vestingDuration = 24 * 30 days; // 24 months in seconds
    uint256 public constant tgePercentage = 5;

    struct VestingSchedule {
        uint256 totalBalance;
        uint256 startTime;
    }

    event TokensReleased(address indexed beneficiary, uint256 amount);

    constructor() {
        balanceOf[msg.sender] = totalSupply;
        vestingSchedules[msg.sender] = VestingSchedule(totalSupply, block.timestamp);
    }

    modifier onlySigner {
        require(msg.sender.is_signer(), "Caller is not a signer");
        _;
    }

    function releaseTokens() external onlySigner {
        address beneficiary = msg.sender;
        VestingSchedule storage vesting = vestingSchedules[beneficiary];
        require(vesting.startTime != 0, "Vesting schedule not initialized");

        uint256 available = calculateAvailableTokens(beneficiary);
        require(available > 0, "No tokens available for withdrawal yet");

        vesting.totalBalance -= available;
        balanceOf[beneficiary] += available;

        emit TokensReleased(beneficiary, available);
    }

    function calculateAvailableTokens(address beneficiary) internal view returns (uint256) {
        VestingSchedule storage vesting = vestingSchedules[beneficiary];
        uint256 elapsedTime = block.timestamp - vesting.startTime;

        if (elapsedTime < cliffDuration) {
            return 0;
        } else {
            uint256 vestedTokens = (vesting.totalBalance * tgePercentage) / 100;
            uint256 vestedMonths = (elapsedTime - cliffDuration) / vestingDuration;
            uint256 linearVestingTokens = ((vesting.totalBalance - vestedTokens) * vestedMonths * 10) / 100;

            return vestedTokens + linearVestingTokens;
        }
    }
}
