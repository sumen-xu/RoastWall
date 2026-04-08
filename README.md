# RoastWall
BaseRoastWall.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.34;

contract BaseRoastWall {
    address public immutable owner;
    uint256 public highlightFee = 0.0005 ether;

    struct Roast {
        address roaster;
        string message;
        uint256 timestamp;
        uint256 likes;
        bool isHighlighted;
    }

    Roast[] public roasts;

    event NewRoast(uint256 indexed id, address roaster, string message, bool highlighted);
    event Liked(uint256 indexed roastId, address liker);

    mapping(uint256 => mapping(address => bool)) public hasLiked;

    constructor() {
        owner = msg.sender;
    }

    function postRoast(string memory _message, bool _highlight) external payable {
        require(bytes(_message).length > 0 && bytes(_message).length <= 280, "Message must be 1-280 chars");

        bool highlighted = _highlight && msg.value == highlightFee;

        roasts.push(Roast({
            roaster: msg.sender,
            message: _message,
            timestamp: block.timestamp,
            likes: 0,
            isHighlighted: highlighted
        }));

        emit NewRoast(roasts.length - 1, msg.sender, _message, highlighted);
    }

    function likeRoast(uint256 _roastId) external {
        require(_roastId < roasts.length, "Roast does not exist");
        require(!hasLiked[_roastId][msg.sender], "Already liked");

        hasLiked[_roastId][msg.sender] = true;
        roasts[_roastId].likes++;

        emit Liked(_roastId, msg.sender);
    }

    function getTotalRoasts() external view returns (uint256) {
        return roasts.length;
    }

    function getLatestRoasts(uint256 _count) external view returns (Roast[] memory) {
        uint256 start = roasts.length > _count ? roasts.length - _count : 0;
        Roast[] memory latest = new Roast[](_count);
        for (uint i = 0; i < _count && start + i < roasts.length; i++) {
            latest[i] = roasts[start + i];
        }
        return latest;
    }

    function withdraw() external {
        require(msg.sender == owner, "Only owner");
        (bool sent, ) = payable(owner).call{value: address(this).balance}("");
        require(sent, "Failed to send Ether");
    }
}
