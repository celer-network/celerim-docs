# Hello World with Token Transfer

Here is a basic hello-world example that sends and receives cross-chain messages with associated token transfers:

* Users call `sendTokenWithNote` on the source chain to send some tokens with an arbitrary bytes note to the destination chain
* The receiver side implements `executeMessageWithTransfer`, which records received token balances for each sender, and emits events with transfer info including the note.

[Source code at GitHub](https://github.com/celer-network/sgn-v2-contracts/blob/main/contracts/message/apps/examples/MsgExampleBasicTransfer.sol).&#x20;


```solidity
contract MsgExampleBasicTransfer is MessageApp {
    using SafeERC20 for IERC20;

    event MessageWithTransferReceived(
        address sender,
        address token,
        uint256 amount,
        uint64 srcChainId,
        bytes note
    );
    event MessageWithTransferRefunded(
        address sender,
        address token,
        uint256 amount,
        bytes note
    );

    // acccount, token -> balance
    mapping(address => mapping(address => uint256)) public balances;

    constructor(address _messageBus) MessageApp(_messageBus) {}

    // called by user on source chain to send token with note to destination chain
    function sendTokenWithNote(
        address _dstContract,
        address _token,
        uint256 _amount,
        uint64 _dstChainId,
        uint64 _nonce,
        uint32 _maxSlippage,
        bytes calldata _note,
        MsgDataTypes.BridgeSendType _bridgeSendType
    ) external payable {
        IERC20(_token).safeTransferFrom(msg.sender, address(this), _amount);
        bytes memory message = abi.encode(msg.sender, _note);
        sendMessageWithTransfer(
            _dstContract,
            _token,
            _amount,
            _dstChainId,
            _nonce,
            _maxSlippage,
            message,
            _bridgeSendType,
            msg.value
        );
    }

    // called by MessageBus on the destination chain to receive message with token 
    // transfer, record and emit info.
    // the associated token transfer is guaranteed to have already been received
    function executeMessageWithTransfer(
        address, // srcContract
        address _token,
        uint256 _amount,
        uint64 _srcChainId,
        bytes memory _message,
        address // executor
    ) external payable override onlyMessageBus returns (ExecutionStatus) {
        (address sender, bytes memory note) = abi.decode(
            (_message),
            (address, bytes)
        );
        balances[sender][_token] += _amount;
        emit MessageWithTransferReceived(
            sender,
            _token,
            _amount,
            _srcChainId,
            note
        );
        return ExecutionStatus.Success;
    }

    // called by MessageBus on the source chain to handle message with 
    // failed associated token transfer.
    // the failed token transfer is guaranteed to have already been refunded
    function executeMessageWithTransferRefund(
        address _token,
        uint256 _amount,
        bytes calldata _message,
        address // executor
    ) external payable override onlyMessageBus returns (ExecutionStatus) {
        (address sender, bytes memory note) = abi.decode(
            (_message),
            (address, bytes)
        );
        IERC20(_token).safeTransfer(sender, _amount);
        emit MessageWithTransferRefunded(sender, _token, _amount, note);
        return ExecutionStatus.Success;
    }

    // called by user on destination chain to withdraw tokens
    function withdraw(address _token, uint256 _amount) external {
        balances[msg.sender][_token] -= _amount;
        IERC20(_token).safeTransfer(msg.sender, _amount);
    }
}
```

