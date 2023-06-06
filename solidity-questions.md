## Easy

1. What is the difference between private, internal, public, and external functions?
External: can only be accessed from an external call. Cheaper than public.
Internal: can only be accessed from within the contract. Cheaper than public and more secure.
Public: can be access both from ouside and inside the contract.
Private: similar to internal but it can't be accessed from derived contracts.

2. Approximately, how large can a smart contract be?
The Spurious Dragon hard-fork introduced EIP-170 which added a smart contract size limit of 24.576 kb. This limit was introduced to prevent  denial-of-service (DOS) attacks. The impact of a contract call for Ethereum nodes increases disproportionately depending on the called contract code's size. Originally, one natural contract size limit was the block gas limit. Obviously a contract needs to be deployed within a transaction that holds all of the contract's bytecode. The issue in that case though is that the block gas limit changes over time and is in theory unbounded. Now, after Ethereum’s London hardfork the block gas limit is 30 million.

3. What is the difference between create and create2?
Create: use the deployer address and the nonce to calculate the new contract's address
new_address = hash(sender, nonce)
Create2: Use a constant that prevents collisions with CREATE (0xFF), the deployer address, a salt and the deployment code to calculate the contract's address.
new_address = hash(0xFF, sender, salt, bytecode)

4. What major change with arithmetic happened with Solidity 0.8.0?
Arithmetic operations are now checked by default, which means that overflow and underflow will cause a revert.

5. What special CALL is required for proxies to work?
Delegate call, this call allows a contract to execute another contract's functionality within the context of the first one. Meaning that any change to the storage will have effects on the first contract storage, and the msg.sender will be the corresponding to the first contract context.

6. Prior to EIP-1559, how do you calculate the dollar cost of an Ethereum transaction?
Prior to EIP-1559, the amount a user pays to get their transaction included in a new block is the product of two values: gas * gas_price
Gas is a measure of how many resources a transaction uses, and is determined automatically by Ethereum nodes. The more intensive the transaction, the more gas it consumes and the more expensive it will be.

Gas price represents the amount a user is willing to pay, per gas, to have their transaction mined and is chosen explicitly by the user. The ‘correct’ gas price is determined by a blind auction - each user chooses the amount they are willing to pay, a miner selects the transactions with the highest gas price, and each user included in the block pays the amount they bid.

This model for picking gas price has two key problems.
The first is that the gas price needed to be included in the next block can be extremely volatile, making it very difficult for a user to predict what price they’ll need to choose. 
The second key problem is that blind auctions are widely known to be inefficient, and frequently lead to users overpaying.

Ethereum’s London hardfork introduced EIP-1559 which replaced the the old, blind auction mechanism behind Ethereum gas price with a new fixed fee model.

Instead of guessing at a volatile gas price, users will be given a set ‘base fee’ needed to be included in the next block. Base fee is the minimum gas price that every transaction must pay, to be included in a block.

With each new block, the network will automatically adjust the base fee based on demand. For example, if a block is 100% full, the fee will increase by 12.5% for the next block. If a block is completely empty, the fee will decrease by 12.5% for the next block. If a block is 50% filled, the fee will remain the same.

Notice that the most the gas price can change from block to block is 12.5%. One significant difference between the new base fee and the previous Ethereum gas price is that the base fee is burned, rather than given to the miner. This is to prevent miners from manipulating the base fee by artificially filling blocks to drive up their own profits. 

In addition to the required base fee, users can optionally bid two parameters in their transactions, max priority fee per gas (“tip”) and max fee per gas (“cap”). Priority fees per gas are the tips with which users incentivize miners to prioritize their transactions. Max fees are the fee caps users will pay including both base fees and priority fees. 


7. What are the challenges of creating a random number on the blockchain?
Randomness is tricky on the blockchain because the blockchain is deterministic, but randomness requires non-determinism (otherwise it becomes predictable). We will briefly discuss some of these solution:

In the Block-hash approach, the hash of blocks or transactions is used as the source of randomness. Although blockchain transactions are perfectly deterministic at the time of execution, they cannot predict the future. Specifically, the future blockhash cannot be predicted. However, this hash is subject to manipulation by the miners guarding the blockchain. 

In the Oracle approach, Oracles collects, as an intermediary, data from external sources and then delivers the data to the applications in need. Regarding random numbers, Oraclize fetches data from random.org into blockchains. Obviously, in this model, one has to trust not only Oraclize but also the data provider and relying on trusted parties contradicts the philosophy of blockchain technology as a centralized service can be corrupted at any time


With the Paris network upgrade, EIP-4399 renamed the opcode difficulty to 'prevrandao' (To make it backwards compatible). At the point of transition block to Proof of Stake, the difficulty block field MUST be 0.
So how is the randomness in Prevrandao created? In simple terms you have every validator in a given epoch pre-commit to a random number computed just locally. The final random number is every validator’s local random number combined via XOR (explicit or). The combination with XOR ensures that even just one single honestly computed random number of one of the validators in an epoch will lead to a new unmanipulated random number.

An epoch consists of 32 slots, so 32 potential validators and assuming all validators are online and proposing a new block, also a total of 32 blocks.

Technically validators are not even computing a local random number anymore actually. Instead they sign the current epoch number with their private key. The signature is then hashed and used as random number. This simplifies the protocol while also allowing for multi-party validators, something that wouldn’t be possible with a commitment scheme.

Once an epoch is finished, the newly computed randomness will be used to determine the validators in the second next epoch (to give validators time to be learn and prepare themselves of their new roles).

The randao number is include in the block header, this provides a straightforward method of accessing it from inside of the EVM as block header data is already available in the EVM context.


8. What is the difference between a Dutch Auction and an English Auction?

In a English auction the auctioneer starts with an initial offering price (the reserve price,
just like with a Dutch auction) and asks for an opening bid. Once some individual has placed a bid for
this price, other are free to offer a higher bid within a short time interval.

In a Dutch Auction the price of the good is initially set to a very high value and then gradually lowered.
The first bidder to make a bid instantly wins the offering at the current price. There may be a non-zero
reserve price at which the auction will end even if no bids have been placed. The offering can never be
sold for less than the reserve price.

9. What is the difference between transfer and transferFrom in ERC20?
In transfer, the owner calls the function to transfer to another address an amount of ERC20.
In transferFrom, an spender transfers on behalf of the owner an amount of ERC20 to another address, and it need to have a previous allowance approval that is high enoght to transfer the specified amount

10. Which is better to use for an address allowlist: a mapping or an array? Why?
It's more gas efficient to use a mapping for an allowlist. In an allowlist you only need to know if an address is allowed to make an action, so in a mapping you don't need to iterate over an array until you reach the address you're looking for. If the address is not allowed in that mapping, it's going to have an empty value.

11. Why shouldn’t tx.origin be used for authentication?
Because tx.origin is the EOA that initially starts the transaction, but the EOA could have call a contract that then makes an external call to your contract, and if your contract uses tx.origin for authentication it's going to allow this contract to use the users funds.

12. What hash function does Ethereum primarily use?
Keccak256. It's NOT the same as the NIST Standard SHA3-256.

13. How much is 1 gwei of Ether?
There are 1 billion wei in one gwei and there are 10^9 or 1,000,000,000 gwei in one ether.
1 gwei ^ 2 is 1 ether

14. How much is 1 wei of Ether?
1 ether is 10^18 in wei

15. What is the difference between assert and require?
assert is commonly used to validate the code is working as expected. On the other hand, require is used to ensure that the function’s input parameters are valid, to check response from external contract or to check a condition before state update.  Use require as early as possible of the function because in case of failure, require does return only unused gas. In assert when the condition is false it will consume all gas remaining gas and reverts all the changes made.

16. What is a flash loan?
A flash loan is a type of uncollateralized loan that lets a user borrow assets with no upfront collateral as long as the borrowed assets are paid back within the same blockchain transaction. If the loan is not paid back inside the same transaction, the transaction will revert and it will be as if the loan never existed.

17. What is the check-effects pattern?
This pattern consist in ordering function components in such a way that prevents reentrancy. Reentrancy can occur when a malicious contract is reentering the initial contract, before the first instance of the function containing the call is finished.

The checks in the beginning assure that the calling entity is in the position to call this particular function (e.g. has enough funds). Afterwards all specified effects are applied and the state variables are updated. Only after the internal state is fully up to date, external interactions should be carried out. In case this order is followed, a re-entrancy attack should not be able to surpass the checks in the beginning of the function, as the state variables used to check for entrance permission have already been updated.

18. What is the minimum amount of Ether required to run a solo staking node?
For solo staking, the minimum deposit required is 32 ETH, which is the amount required to run a validator node on the Ethereum network.

19. What is the difference between fallback and receive?
A fallback function is default function that is called if no other function in the contract matches the specified function in the call. It can be payable if it is marked as payable.
The receive() method is used as a fallback function if Ether are sent to the contract and no calldata are provided (no function is specified). If the receive() function doesn’t exist, the fallback() method is used.

20. What is reentrancy?
A reentrancy attack is a type of smart contract vulnerability where an exploiter contract leverage a bad design pattern to call multiple times to the victim contract before the first external call ends, in such a way that can drain the funds of the contract.
The bad design consists in that the state is updated only after the external call, but before the external call is finished, the same (or other) function is called, allowing to make multiple times an action and only updating the state at the end.

21. As of the Shanghai upgrade, what is the gas limit per block?
The current block gas limit is 30 million gas.

22. What prevents infinite loops from running forever?
The way Ethereum solves infinite loops is with gas, gas is the cost to pay for computation. If an infinite loop is executed, it will stop once the transaction spends all the funds of the user or once it reaches the block gas limit.

23. What is the difference between tx.origin and msg.sender?
tx.origin is the Externally Owned Account that sent the transaction in the first place, msg.sender is the last account (EOA or contract) that calls the current contract.

24. How do you send Ether to a contract that does not have payable functions, or a receive or fallback?
You have 3 ways:
-Create and contract and call selfdestruct with the target address as a parameter will send your contract's balance even if the contract can't receive ether.
-If you calculate the address of the contract before it is created and send ether to that address before the contract creation.
-If a validator uses the address to receive the rewards for validating blocks.

25. What is the difference between view and pure?
With view you can read from storage and with pure you can't

26. What is the difference between transferFrom and safeTransferFrom in ERC721?
The safeTransferFrom functions are almost identical to transferFrom except that they check whether the recipient is a valid ERC721 receiver contract, and if it is, let you pass some data to that contract. safeTransferFrom checks if the receiver is valid because the receiver needs to have implemented a function called onERC721Received() that returns the selector for that function.

## Medium

1. What is the difference between transfer and send? Why should they not be used?
transfer: If the transfer fail it will revert. There is a gas limit of 2300 gas, which was hardcoded to prevent reentrancy attacks.
send: If the transfer fail it wont revert, it only returns a boolean specifying if the call was successful or not. It has a gas limit of 2300 gas as well.
call: It is the recommended way of sending ETH to a smart contract. The empty argument triggers the fallback function of the receiving address. (bool sent,memory data) = _to.call{value: msg.value}("");
It can also trigger other functions defined in the contract and send a fixed amount of gas to execute the function. The transaction status is sent as a boolean and the return value is sent in the data variable.
You should use call because EIP1884 introduce an increase to the gas cost of the SLOAD opcode, causing a contract's fallback function to cost more than 2300 gas.

2. How do you write a gas-efficient for loop in Solidity?
If you are using any storage variable inside the loop, you should copy to memory before starting the loop.
Need to break the loop as soon as possible
Increment the variable in an unchecked block
Don’t use >= and <= in the condition
Just write it in assembly
“Cache” the array’s length for the loop condition (every time before the condition was checked, the length of ns had to be fetched anew)

3. What is a storage collision in a proxy contract?
Because a proxy contract makes a delegate call to the implementation contract, you need to make sure that there no storage slot that can be overlapped. The main issue is that because you need to store the implementation address, you need to be sure that you are saving that value inside a slot that is not overlapped by the implementation.
One solution for this is the unstructured storage, in this case you choose a pseudo random slot to store the variables you need. This slot is sufficiently random, that the probability of a logic contract declaring a variable at the same slot is infeasible.
For example, to store the implementation address the EIP-1967 uses the storage slot resulting from the following code:
bytes32(uint256(keccak256('eip1967.proxy.implementation')) - 1)
It gets the keccak hash of the string "eip1967.proxy.implementation", parse it to uint, sub 1 and parse to bytes32.

4. What is the difference between abi.encode and abi.encodePacked?
abi.encode: encodes its parameters using the ABI specs. The ABI was designed to make calls to contracts. Parameters are padded to 32 bytes. If you are making calls to a contract you need to use abi.encode.

abi.encodePacked encodes its parameters using the minimal space required by the type. Encoding an uint8 it will use 1 byte. This is not how the EVM works when receiving functions and we cannot send arguments from a function encoded this way, but this ’packaged’ encoding is useful in some scenarios.

5. uint8, uint32, uint64, uint128, uint256 are all valid uint sizes. Are there others?
They are valid.
There are others. You can add a uint8 to the uint8 base until you reach uint256.
uin8, uint16, uint24, uint32, uint40, uint48, uint56, uint64, uint72, uint80, uint88, uint96, uint104, uint112, uint120, uint128, uint136, uint144, uint152, uint160, uint168, uint176, uint184, uint192, uint200, uint208, uint216, uint224, uint232, uint240, uint248, uint256

6. How many arguments can a solidity event have?
A log can have up to 4 topics, but a non-anonymous solidity event can have up to 3 indexed arguments. That is because the first topic is used to store the event signature.
It is possible to search for indexed parameters, in addition to monitoring the emission of events by the value of such parameters. Non-indexed parameters, on the other hand, cannot be filtered or monitored

7. What changed with block.timestamp before and after proof of stake?
In proof of work, miners could modify the block timestamp within 15 seconds, as long as the modified value was greater than the parent timestamp, without the block getting rejected.

Under proof of stake, blocks have pre-determined timestamps that are not modifiable, and a block without that exact timestamp is not valid. The Ethereum block interval is fixed at 12 seconds.

8. What is frontrunning?
Front-running is an attack where a malicious node leverages that all transactions in the mempool are public, and observes a transaction after it is broadcast but before it is finalized, and attempts to have its own transaction confirmed before or instead of the observed transaction.


9. What is a commit-reveal scheme and when would you use it?
A commitment scheme is a cryptographic algorithm used to allow someone to commit to a value while keeping it hidden from others with the ability to reveal it later. The values in a commitment scheme are binding, meaning that no one can change them once committed. The scheme has two phases: a commit phase in which a value is chosen and specified, and a reveal phase in which the value is revealed and checked. You can add a salt if the possible choises are few.
I would use it in cases where data within transactions needs to be kept secret momentarily to prevent frontrunning.
For example, the game rock, paper, scissors.

10. Under what circumstances could abi.encodePacked create a vulnerability?

abi.encodePacked can result in hash collisions when passing more than one dynamic data type.
Since abi.encodePacked() packs all elements in order (using the minimal space required) regardless of whether they're part of an array, you can move elements between arrays and, so long as all elements are in the same order, it will return the same encoding. In a signature verification situation, an attacker could exploit this by modifying the position of elements in a previous function call to effectively bypass authorization.
If you are using more than one dynamic data type, abi.encode should be used instead.

11. How does Ethereum determine the BASEFEE in EIP-1559?
Every block has a base fee which acts as a reserve price. To be eligible for inclusion in a block the offered price per gas must at least equal the base fee. The base fee is determined by the blocks before it - making transaction fees more predictable for users. When the block is mined this base fee is "burned", removing it from circulation.

The base fee is calculated by a formula that compares the size of the previous block (the amount of gas used for all the transactions) with the target size. Each block's BASEFEE can increase or decrease by 12.5% depending on how full the block is compared to the previous block. For example, if 100% full then baseFeePerGas goes +12.5%, if 50% full then it remains same, if 0% full then -12.5%.

12. What is the difference between a cold read and a warm read?
Cold access: access a variable for the first time. Cost 2100 gas.
Warm access: access the variable a second time an further. Cost 100 gas.
The first one is specifying how much it costs to access a variable for the first time (or cold access) while the second one specifies how much it costs to access the variable a second time and further within the same transaction (warm access).
Modifying a storage slot from a zero value to a non-zero one costs 20,000. While storing the same non-zero value or setting a non-zero value to zero costs 5,000. However, in the last scenario when a non-zero value is set to zero, a refund of 15,000 will be given.

13. How does an AMM price assets?
The prices of assets on an AMM automatically change depending on the demand. For example, a liquidity pool could hold ten million dollars of ETH and ten million dollars of USDC. A trader could then swap 500k dollars worth of their own USDC for ETH, which would raise the price of ETH on the AMM.
Automated market makers (AMMs) allow digital assets to be traded without permission and automatically by using liquidity pools instead of a traditional market of buyers and sellers.

14. What is the effect on gas of making a function payable?
In the payable functions, the EVM doesn’t have to validate that msg.value is zero, as it is totally possible to send zero ether or more to such a function, thus the validation is irrelevant. Because of that, payable functions result in less computations, which decreases gas usage.

15. What is a signature replay attack?
A signature replay attack is an attack whereby a previously executed valid transaction is fraudulently or maliciously repeated on the same blockchain or a different blockchain.
You can prevent a replay attack in the same blockchain with a nonce, and in a different blockchain with the chainId.
The chainId is used to calculate the v component of the ECDSA digital signature.

16. What is gas griefing?
A Gas Griefing attack takes place when a user provides just enough gas to run the specified smart contract but not enough to run its sub-calls. This flaw can be found in smart contract methods that either fail to check the amount of gas needed to perform a sub-call or fail to check the call's actual returned value.

17. How would you design a game of rock-paper-scissors in a smart contract such that players cannot cheat?
I would implement the commit-reveal scheme. This scheme is a cryptographic algorithm used to allow someone to commit to a value while keeping it hidden from others with the ability to reveal it later. As there is only 3 choices, I would add a salt so nobody can guess other people choices.

18. What is the free memory pointer and where is it stored?
The free memory pointer is a pointer to the next available slot of memory. It allows the compiler to keep track of which memory slots are already written and prevent them from being overwritten.
It is located in memory at position 0x40.

19. What function modifiers are valid for interfaces?
None custom modifier.

20. What is the difference between memory and calldata in a function argument?
Memory is reserved for variables that are defined within the scope of a function and cannot be accessed outside this scope. Calldata is an immutable, temporary location where function arguments are stored.
-Calldata can only be used in function arguments and not in function logic.
-One key difference is that memory can be modified by the function, while calldata cannot. This means that if you want to modify a function argument, you must first copy it into memory.
-It is generally cheaper to load variables directly from calldata, rather than copying them to memory. Only use memory if the variable needs to be modified.

21. Describe the three types of storage gas costs.
If the value of the slot changes from 0 to 1 (or any non-zero value), the cost is:
22100 if the storage key wasn’t accessed
20000 if it was
If the value of the slot changes from 1 to 2 (or any other non-zero value), the cost is:
5000 if the storage key wasn’t accessed
2900 if it was
If the value of the slot changes from 1 (or any non-zero value) to 0, the cost is the same as the previous item, plus the refund.
If the value was previously modified during the same transaction, all the subsequent SSTOREs cost 100.

22. Why shouldn’t upgradeable contracts use the constructor?
The code that is inside a constructor or part of a global variable declaration is not part of a deployed contract’s runtime bytecode. This code is executed only once, when the contract instance is deployed. As a consequence of this, the code within a logic contract’s constructor will never be executed in the context of the proxy’s state.
The problem is easily solved though. Logic contracts should move the code within the constructor to a regular 'initializer' function, and have this function be called whenever the proxy links to this logic contract.

23. What is the difference between UUPS and the Transparent Upgradeable Proxy pattern?
The original proxies included in OpenZeppelin followed the Transparent Proxy Pattern. Our recommendation is now shifting towards UUPS proxies, which are both lightweight and versatile.

While both of these share the same interface for upgrades, in UUPS proxies the upgrade is handled by the implementation, and can eventually be removed. Transparent proxies, on the other hand, include the upgrade and admin logic in the proxy itself. This means TransparentUpgradeableProxy is more expensive to deploy than what is possible with UUPS proxies.

24. What danger do ERC777 tokens pose?
ERC777 introduces a call to the contract that it's being transferred the tokens before actually transfer them.
In 2020 imBTC Uniswap Pool was drained for ~$300k in ETH. The attacker leverages this call to the contract that receives the tokens to make a reentrancy to the pool, in such a way that this person can use a small amount of ETH to swap out most of the tokens. This is achiavable because the “token_reserve” value will not be updated in consecutive token exchanges, leading to the violation of the “xy=k” setting of Uniswap. The attacker could sell tokens at a much better rate to drain the liquidation pool.

25. What is a bonding curve?
A bonding curve is a mathematically defined relationship between price and supply. In short, as the supply increases, so does the price. This is somewhat counter-intuitive to traditional models, but the method is for each subsequent buyer of a token to pay a slightly higher price than the previous buyer and a lower price than any subsequent buyer. BCOs support sustained organic growth. New tokens are created by the contract when demand is there, escalating in price each time. By sidestepping the requirement of setting the initial price, it stops vast swathes of a token being picked up at a particular price point and then subsequently dumped. It also removes the need for exchanges: As a fully automated market maker (AMM), bonding curves not only allow for the calculation of a token’s price, but also enable the investor to simply buy or sell their tokens right there.

26. How does safeMint differ from mint in the OpenZeppelin ERC721 implementation?
When using safeMint, the token contract checks to see that the receiver is an IERC721Receiver, which implies that it knows how to handle ERC721 tokens. When using mint the contract just assumes that it can handle ERC721 tokens.

27. What keywords are provided in Solidity to measure time?
The keyword years is now disallowed due to complications and confusions about leap years.

1 minutes == 60 (seconds)
1 hours == 60 minutes
1 days == 24 hours
1 weeks == 7 days

28. What is a sandwich attack?
It's a combination of frontrunning and backrunning. You choose a transaction from the mempool and you submit one transaction that it's going to be validated first than the target transaction (because it has higher gas fee) and then you submit another transaction that is going to be validated slightly after the target transaction (because it has a slightly lower fee).
You can use this mechanism with a transaction that makes a swap in a liquidation pool and swap the same asset before the target transaction to lower the price and then swapping back after the target transaction the counter asset at that lower price to earn a difference. 

29. What is a gas efficient alternative to multiplying and dividing by a multiple of two?
A division/multiplication by any number x being a power of 2 can be calculated by shifting log2(x) to the right/left. While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.
x * 2 is equivalent to x << 1
x / 2 is equivalent to x >> 1.

30. How large a uint can be packed with an address in one slot?
256 (slot) - 160 (address) = 96
96uint is the max uint that can be packed with an address in one slot.

31. Which operations give a partial refund of gas?
SELFDESTRUCT: destroys the current executed contract. It refunds 24000 gas. This opcode is going to be deprecated.
SSTORE: it refunds 15000 gas when it replaces a non-zero value by a zero.

32. What is ERC165 used for?
ERC165 is an interface to check if a contract supports an interface, it's a meta-interface so to say. It standardizes how interfaces are identified. This is useful if we want to interact with a contract but we don't know if it supports an interface such as ERC721, which has a supportsInterface function.

interface IERC165 {
    function supportsInterface(bytes4 interfaceId) external view returns (bool);
}

This method takes the interfaceId as the param and checks if the contract supports that interface. Solidity has in built method called type which returns the unique id of the interface.
type(MyInterface).interfaceId


33. What is a slippage parameter useful for?
Slippage means the difference between the price that you see on the screen when initialing a transaction and the actual price the swap is executed at. This difference appears because there’s a short delay between when you send a transaction and when it gets validated.

One important problem that slippage protection fixes is sandwich attacks. During sandwiching, attackers “wrap” your swap transactions in their two transactions: one goes before your transaction and the other goes after it. In the first transaction, an attacker modifier the state of a pool so that your swap becomes very unprofitable for you and somewhat profitable for the attacker. This is achieved by adjusting pool liquidity so that your trade happens at a lower price. In the second transaction, the attacker reestablishes pool liquidity and the price. As a result, you get much less tokens than expected due to manipulated prices, and the attacker get some profit.

In UniswapV3 the parameter that allow a user to set a slippage is sqrtPriceLimitX96.


## Hard

1. How does fixed point arithmetic represent numbers?
Fixed-point is a method of representing fractional (non-integer) numbers by storing a fixed number of digits of their fractional part.
A fixed-point representation of a fractional number is essentially an integer that is to be implicitly multiplied by a fixed scaling factor. For example, the value 1.23 can be stored in a variable as the integer value 1230 with implicit scaling factor of 1/1000 (meaning that the last 3 decimal digits are implicitly assumed to be a decimal fraction).
The main difference between floating point and fixed point numbers is that the number of bits used for the integer and the fractional part (the part after the decimal dot) is flexible in the former, while it is strictly defined in the latter.

2. What is an ERC20 approval frontrunning attack?
The approve() function allows a another person to be able to spend N amount of tokens from the owner's balance.
If the malicious party already has an approve from the owner, and the owner wants to reduce the amount of token the malicious party can spend, the owner can send a new approve for a lower amount of token. Here is where the malicious party can take advantage of this. Using the insertion attack, the attacker could front-run the new approve transaction and spend the old value before the new value is set, and then additionally spend the new amount.

3. What opcode accomplishes address(this).balance?
You need to call first opcode ADDRESS (30) and then call opcode BALANCE (31).

4. What is an anonymous Solidity event?
Events can be marked as anonymous, in which case they will not have a selector. Because of this they cannot be easily searched for, or decoded with certainty unless you have the specific contract ABI or the source code of the contract.

Because the event signature is used as one of the indexes, an anonymous function can have four indexed topics, since the function signature is “freed up” as one of the topics.

Anonymous events are rarely used in practice. To make an event anonymous, include the anonymous keyword after declaring the variables:

-event Start(uint start, uint middle, uint end) anonymous;

5. Under what circumstances can a function receive a mapping as an argument?
Mappings can only have a data location of storage and thus are allowed for state variables, as storage reference types in functions, or as parameters for library functions.
Mappings cannot be used as parameters or return parameters of contract functions that are publicly visible. These restrictions are also true for arrays and structs that contain mappings.
But can they can be used as a parameter for internal or private functions since they are not exposed to the outside through the contract’s ABI, they can take parameters of internal types like mappings or storage references.

6. What is an inflation attack in ERC4626?
ERC4626 is an extension of ERC20 that proposes a standard interface for token vaults. In exchange for the assets deposited into an ERC4626 vault, a user receives shares. These shares can later be burned to redeem the corresponding underlying assets. The number of shares a user gets depends on the amount of assets they put in and on the exchange rate of the vault. This exchange rate is defined by the current liquidity held by the vault. When depositing tokens, the number of shares a user gets is rounded down. 
Attack scenario:
A hacker back-runs the transaction of an ERC4626 pool creation.
The hacker mints for themself one share: deposit(1). Thus, totalAsset()==1, totalSupply()==1.
The hacker front-runs the deposit of the victim who wants to deposit 20,000 USDT (20,000.000000).
The hacker inflates the denominator right in front of the victim: asset.transfer(20_000e6). Now totalAsset()==20_000e6 + 1, totalSupply()==1.
Next, the victim's tx takes place. The victim gets 1 * 20_000e6 / (20_000e6 + 1) == 0 shares. The victim gets zero shares.
The hacker burns their share and gets all the money.

7. How many arguments can a solidity function have?
Function can take 16 parameters, otherwise if you exceeds the 16 parameter number else you run into a Stack too deep error. This limit is because the number of stack slots you can directly access is 16.


8. How many storage slots does this use? uint64[] x = [1,2,3,4,5]? Does it differ from memory?
Storage: It will occupy 3 slots.
    -First slot: the size of the array.
    -Second slot: the numbers 1, 2, 3 and 4 because four uint64 equals to one uint256, which is the size of a slot.
    -Third slot: the number 5.

Values of dynamic arrays are stored in slots that are computed using a Keccak-256 hash. So in this case, first slot will be slot zero, and second and third slots will be at the corresponding hashing location.

In memory you can't have a dynamic size array, but if you create a fixed size array this way:
    uint64[] memory x = new uint64[](5);
    x[0] = 1;
    x[1] = 2;
    x[2] = 3;
    x[3] = 4;
    x[4] = 5;
The array will occupy 6 memory slot:
0x80: size of the array
0xa0: 1
0xc0: 2
0xe0: 3
0x100: 4
0x120: 5


9. Prior to the Shanghai upgrade, under what circumstances is returndatasize() more efficient than push zero?
Prior to the Shanghai upgrade pushing a zero into the stack would cost 3 gas and would take 2 instructions to do it.
Push1: 0x60
zero: 0x00

The Shanghai upgrade incorporated a new opcode called Push0 which consumes 2 gas and doesn't require any argument.
Push0: 0x5f

Returndatasize spends 2 gas and doesn't need any argument, so sometimes it was more efficient to use this opcode instead of pushing a 0 into the stack.
Returndatasize: 0x30


10. Why does the compiler insert the INVALID op code?
The invalid opcode usually to indicate that something particularly bad or unexpected happened. It's equivalent to any other opcode not currently used by the compiler, but guaranteed to remain an invalid instruction. It's equivalent to REVERT (since Byzantium fork) with 0,0 as stack parameters, except that all the gas given to the current context is consumed.

11. What is the difference between how a custom error and a require with error string is encoded at the EVM level?
Custom errors are encoded the same way functions are, that means that the first 4 bytes are the error selector and the rest of the bytes are the arguments.
In require with string error the string is directly encoded and returned, with no extra arguments. String are expensive and the more length it has the more expensive it becomes. Other difference of require is that it can't have dynamic arguments.

When the length of the string is 31 bytes or less it’s stored in a single slot starting from the left side and the length * 2 is stored in the final byte on the right.
For anything larger than 31 bytes the storage process is similar to an array. Where the slot of the string stores length * 2 + 1 and the data is stored via keccak256(slot) + i

12. What is the kink parameter in the Compound DeFi formula?
https://ianm.com/posts/2020-12-20-understanding-compound-protocols-interest-rates

13. How can the name of a function affect its gas cost, if at all?
In smart contracts, the order of the functions are based on their Method ID and the later the sort will consume more. Each position will have an extra 22 gas. This is why you should prioritize most used functions and reduce the number of functions and variables that are public.

contract Test {
    function b() public {
    }
    function a() public {
    }
}

a: 0x0dbe671f => 125 gas
b: 0x4df7e3d0 => 147 gas

14. What is a common vulnerability with ecrecover?
The vulnerability is called Signature malleability. An attacker can modify the parameters of a signature (v, r and s) and still get a different valid signature, without having access to the private key of the original signer. This happens because elliptic curves are symmetric, and for every value of v,r and s, there will be a different set of v, r and s that have the same relationship.

The EIP-2 hardfork does invalidate signatures with a high s value but the precompiled contract ecrecover points to was left as is, meaning that individuals need to properly account for malleable signatures and sanitize them.
This is why ecrecover() does not follow the recommended best practice for signature verification (It does not check for malleable signatures). s values that are part of accepted signatures should be checked to be in lower ranges.

ecrecover: code that utilizes the ecrecover() mechanism directly should primarily validate that the resulting address is not equal to the 0 address and secondarily validate that the v value is either 27 or 28 and that the s value of the signature is lower-than-or-equal-to the value of (n / 2) of the secp256k1 bonding curve.

Open Zeppeling ECDSA library bug: The vulnerability existed, because it was possible to submit an EIP-2098 compact signature to bypass signature check. This is already fixed.

Instead of ecrecover(), OpenZeppelin’s ECDSA library should be used instead because it solves the signature malleability problems.


15. What is the difference between an optimistic rollup and a zk-rollup?

Property                 | Optimistic | ZK-Rollup

Proof                    | Fraud Proof (Users would have to challenge a transaction bundle to determine the invalidity of transactions)      | Validity proof (the operator is responsible for producing validity proofs to ensure the accuracy of the changes made)

Fixed gas cost           | ~40,000 (a lightweight transaction that mainly just changes the value of the state root)  | 500,000 (verification of a ZK-SNARK is computationally intensive)

Withdraw period          | ~1 week (withdrawals need to be delayed to give time for someone to publish a fraud proof and cancel the withdrawal if it is fraudulent) |   Very fast (just wait for the next batch)

Complexity of technology | Low       |    High (ZK-SNARKs are very new and mathematically complex technology)

Generalizability         | Easier (general-purpose EVM rollups are already close to mainnet) |  Harder (ZK-SNARK proving general-purpose EVM execution is much harder than proving simple computations, though there are efforts (eg. Cairo) working to improve on this)

Per-transaction on-chain gas costs | Higher |   Lower (if data in a transaction is only used to verify, and not to cause state changes, then this data can be left out, whereas in an optimistic rollup it would need to be published in case it needs to be checked in a fraud proof)

Off-chain computation costs        | Lower (though there is more need for many full nodes to redo the computation) |    Higher (ZK-SNARK proving especially for general-purpose computation can be expensive, potentially many thousands of times more expensive than running the computation directly)


16. How does EIP1967 pick the storage slots, how many are there, and what do they represent?
EIP1967 choose a pseudo-random slot of the storage calculated using the keccak hash of a string. There are 3 storage slots that the EIP1967 uses this way:

Logic contract address slot: holds the address of the logic contract that this proxy delegates to:
bytes32(uint256(keccak256('eip1967.proxy.implementation')) - 1).

Beacon contract address slot: holds the address of the beacon contract this proxy relies on (fallback). SHOULD be empty if a logic address is used directly instead:
bytes32(uint256(keccak256('eip1967.proxy.beacon')) - 1).

Admin address slot: holds the address that is allowed to upgrade the logic contract address for this proxy (optional):
bytes32(uint256(keccak256('eip1967.proxy.admin')) - 1).

In a Beacon proxy pattern, the proxy contract doesn’t hold the implementation address in storage but the address of a UpgradeableBeacon contract, which is where the implementation address is actually stored and retrieved from. The upgrade operations that change the implementation contract address are then sent to the beacon instead of to the proxy contract, and all proxies that follow that beacon are automatically upgraded.

17. How much is one Sazbo of ether?
1 Sazbo is 0.000001 ETH and 1,000,000 Szabo	is 1 ETH

18. Under what circumstances would a smart contract that works on Ethereum not work on Polygon or Optimism? (Assume no dependencies on external contracts)
For example if one opcode is valid in one blockchain and not valid in another, like a new opcode added recently 0x5f.


19. How can a smart contract change its bytecode without changing its address?
In order to accomplish a metamorphic contract we need:
-The first smart contract to be deployed with create2
-The constructor of this contract needs to make an external call to get the final bytecode (so the actual bytecode is stored in another contract).
-To have the ability to selfdestruct.
-Once the first smart contract is destroyed, we change the bytecode inside the external contract and re-deploy the same first contract again with create2.

Create2 takes as parameters the following arguments:
-a constant (0xff)
-the deployer address
-a salt
-the deployment code

Because the deployment code is identical, we're going to have the same address for this second contract but with another final bytecode.

20. What is the danger of putting msg.value inside of a loop?
Using msg.value inside a loop is dangerous because this might allow the sender to “re-use” the msg.value.

This can show up with payable multicalls. Multicalls enable a user to submit a list of transactions to avoid paying the 21,000 gas transaction fee over and over. However, msg.value gets “re-used” while looping through the functions to execute, potentially enabling the user to double spend.

function batchBuy(address[] memory addr) external payable{
    for (uint i = 0; i < addr.length; i++) {
        buy1NFT(addr[i])
    }
}
function buy1NFT(address to) internal {
    if (msg.value < 1 ether) { revert("not enough ether") }
    nft[numero] = address;
}

In the function batch-buy() we see that we can buy lot of NFTs at once. But the issue is that in every iteration of a for loop, Buy1NFT() is called. It verifies that msg.value is at least 1 ether, but don’t subtract 1 ether from the smart contract.

As a result a malicious party can send only 1 ETH to the smart contract and put x addresses in the “addr” argument (or x times my address) to get x NFTs for only 1 ETH.


21. Describe the calldata of a function that takes a dynamic length array of uint128 when uint128[1,2,3,4] is passed as an argument

0x66454c64                                                          // function selector
0000000000000000000000000000000000000000000000000000000000000020    // offset from here (skip the next 32 bytes)
0000000000000000000000000000000000000000000000000000000000000004    // length
0000000000000000000000000000000000000000000000000000000000000001    // first value
0000000000000000000000000000000000000000000000000000000000000002    // second value
0000000000000000000000000000000000000000000000000000000000000003    // third value
0000000000000000000000000000000000000000000000000000000000000004    // forth value

22. Why is strict inequality comparisons more gas efficient than ≤ or ≥? What extra opcode(s) are added?
Strict inequality is more efficient because you only need to use one opcode:
0x14-EQ (3 gas)

if you want to check ≤ or ≥ you would need to use 3 opcodes:
0x14-EQ (3 gas)
0x10-LT or 0x11-GT (3 gas)
0x17-OR (3 gas)
This happens because there is no opcode that represents the operations LTE or GTE


23. What is the relationship between variable scope and stack depth?


24. What is an access list transaction?
An access list transaction (or EIP-2930 transaction) is transaction that has a list of addresses and storage keys that it plans to access.

The Berlin hard fork raised the “cold” cost of account access opcodes (such as BALANCE, all CALL(s), and EXT*) to 2600 and raised the “cold” cost of state access opcode (SLOAD) from 800 to 2100 while lowering the “warm” cost for both to 100.

An EIP-2930 transaction is carried out the same way as any other transaction, except that the cold storage cost is paid upfront with a discount, rather during the execution of the SLOAD operation. It does not require any modifications to the Solidity code and is purely specified client-side.

The fee prepays the cold access of the storage slot so that during the actual execution, only the warm fee is paid. When the storage keys are known in advance, Ethereum node clients can pre-fetch storage values, effectively lowering the computational resource overhead.

EIP-2930 does not prevent storage access outside the access list; putting an address-storage combination in the access list is not a commitment to use it. However, the result would be prepaying the cold storage load for no purpose.


25. Why is it necessary to take a snapshot of balances before conducting a governance vote?

Because the voting power is retrieved from past snapshots rather than current balance. So if you don't take a snapshop before a governance vote, it will use the last snapshop which will probably be outdated.

The snapshopts solve the problem of double voting. If votes are weighed by the number of tokens someone holds, then a malicious actor can use their tokens to vote, then transfer the tokens to another address, vote with that, and so forth.

26. How can a transaction be executed without a user paying for gas?

Traditionally, a blockchain transaction would require the person sending the transaction to pay for gas. Meta-transactions allow anyone to interact with the blockchain without holding tokens to pay for transaction fees. This is achieved by separating the transaction's sender from the gas payer. This means that one person is allowed to send a transaction and have the transaction finalized and paid for by a totally separate entity.

A smart contract wallet (Account Abstraction - ERC4337) such as Sequence provides an effective means of implementing meta-transactions.
Gasless transactions are the most common type of meta transaction, meaning that users do not have to pay gas at all. This allow your users perform key actions in your app or game without the barriers of having to fund their wallet or pay a fee.

27. In solidity, without assembly, how do you get the function selector of the calldata?
Using msg.sig

28. How is an Ethereum address derived?
We need to separate between EOA and Contracts.

EOA address are created by taking the last 20 bytes of the Keccak-256 hash of the public key and representing it as a hexadecimal number.

Contract address can be created in two different ways.
Create opcode: It will use the deployer address and the nonce in order to derive the contract address
Create2 opcode: It will use a constant (0xff), the deployer address, a salt, and the deployment code to derive the contract address


## Advanced

1. What addresses to the ethereum precompiles live at?
Ethereum precompiles behave like smart contracts built into the Ethereum protocol. The nine precompiles live in addresses 0x01 to 0x09. Precompiles do not execute inside a smart contract, they are part of the Ethereum client specification. Most precompiles don’t have a solidity wrapper (with ecrecover being the sole exception). You’ll need to call the address directly with addressOfPrecompile.staticcall(…) or use assembly. Although none of the precompiled contracts are state changing, the solidity function that calls them cannot be pure because the solidity compiler has no way of inferring that a staticcall won’t change the state.

Address 0x01: ecRecover
ECRecover is the precompile for recovering an address from a hash and a digital signature for that hash, i.e. determining who signed it if the signature is valid. 

Address 0x02 and 0x03: SHA-256 and RIPEMD-160
Both of these precompiles will hash the bytes supplied in the calldata. Why does Ethereum support SHA-256 and RIPEMD-160? Bitcoin makes heavy use of SHA256 the way Ethereum makes heavy use of keccak256. However, Bitcoin addresses use RIPEMD-160 to hash the public key and make the public address more compact.

Address 0x04: Identity
The identity precompile copies one region of memory to another. Ethereum doesn’t have a “memcopy” opcode (an opcode to copy one region in memory to another). Normally, you’d have to MLOAD a word of memory onto the stack and then MSTORE it to copy it, and you’d have to do the copy word by word. With the identity precompile, you can copy a contiguous set of 32 byte words in one go, rather than one byte at a time.

Address 0x05: Modexp
The address 0x05 implements the formula base**exp % mod. It returns the result from the given data.

Address 0x06 and 0x07 and 0x08: ecAdd, ecMul, and ecPairing (EIP-196 and EIP-197)
These precompiles are used to make zero knowledge proof cryptography more efficient. In fact, you can see all three precompiles being used in the Tornado Cash zero knowledge proof verifier:

Elliptic Curve Addition: staticcall to address(6)
Elliptic Curve Multiplication: staticcall to address(7)
Elliptic Curve Pairing: static call to address(8)

Address 0x09: Blake2 (EIP-152)
The Blake2 hash is the preferred hash of zcash. Similar to SHA256 and RIPEMD-160, Blake2 was added to enable Ethereum to validate claims about transactions on that blockchain. This precompile was added in EIP152.


2. How does Solidity manage the function selectors when there are more than 4 functions?


3. How does ABI encoding vary between calldata and memory, if at all?
The main difference is that dinamic types in calldata also have an offset (where the variable start). Also, in memory you can't have a dynamic size array.

4. What is the difference between how a uint64 and uint256 are abi-encoded in calldata?
There is no difference neither in calldata nor memory, the variables are padded in such a way that they are a 32 bytes number. The only difference will be in storage.


5. What is read-only reentrancy?
The read-only reentrancy is a reentrancy scenario where a view the function is reentered, which in most cases is unguarded as it does not modify the contract’s state. However, if the state is inconsistent, wrong values could be reported. While the reentered contract cannot be affected by its view function, others reading the contract’s state can. As In result, other protocols blindly relying on a return value (e.g. from Oracles or Price Feeds) can be tricked into reading the wrong state to perform unwanted actions.


6. If you deploy an empty Solidity contract, what bytecode will be present on the blockchain, if any?
0x6080604052600080fdfea264697066735822122005587ff223b480866e9d3010eed2b9bec42b505b79181e2348263f4045a036a864736f6c63430008130033

It will set the free memory pointer and will revert. Then, it will set an explicit invalid opcode. After that, it will be same opcodes that will never be reached by the contract's execution.


7. How does the EVM price memory usage?
Memory usage has a static gas cost of 3 and a dynamic cost.
Memory expansion may be triggered when the byte offset (modulo 32) accessed is bigger than previous offsets. If a larger offset trigger of memory expansion occurs, the cost of accessing the higher offset is computed and removed from the total gas available at the current call context. When a memory expansion is triggered, only the additional bytes of memory must be paid for. The memory_byte_size can be obtained with opcode MSIZE.


static_gas = 3

memory_cost = (memory_size_word ** 2) / 512 + (3 * memory_size_word)
memory_size_word = (memory_byte_size or MSIZE + 31) / 32
memory_expansion_cost = new_memory_cost - last_memory_cost
dynamic_gas = memory_expansion_cost

8. What is stored in the metadata section of a smart contract? x
9. What is the uncle-block attack from an MEV perspective? x
10. How do you conduct a signature malleability attack? x
11. Under what circumstances do addresses with leading zeros save gas and why?
12. What is the difference between payable(msg.sender).call{value: value}(””) and msg.sender.call{value: value}(””)? x
13. How many storage slots does a string take up? x
14. How does the --via-ir functionality in the Solidity compiler work? x
15. Are function modifiers called from right to left or left to right, or is it non-deterministic? x
16. If you do a delegatecall to a contract and the opcode CODESIZE executes, which contract size will be returned?
17. Why is it important to ECDSA sign a hash rather than an arbitrary bytes32? x
18. Describe how symbolic manipulation testing works. x
19. What is the most efficient way to copy regions of memory? x
20. How can you validate on-chain that another smart contract emitted an event, without using an oracle? x
21. When selfdestruct is called, at what point is the Ether transferred? At what point is the smart contract's bytecode erased? x
22. Why did Solidity deprecate the "years" keyword? x
23. What does the verbatim keyword do, and where can it be used? x
24. How much gas can be forwarded in a call to another smart contract? x
25. What does an int256 variable that stores -1 look like in hex? x
26. What is the use of the signextend opcode? x