# Simple Token Example

This example demonstrates how to create a basic fungible token on Midnight using the Compact language.

## What You'll Learn

- Working with mappings (`Map<K, V>`)
- Managing balances by address
- Transfer logic with validation
- Token metadata (name, symbol, supply)
- The `address` type

## Contract Overview

The `SimpleToken` contract implements:
- **Initialization**: Set token name, symbol, and initial supply
- **Transfer**: Send tokens between addresses
- **BalanceOf**: Query account balances
- **Mint**: Create new tokens (simplified, no access control)
- **Burn**: Destroy tokens from sender's balance
- **Getters**: Query token metadata

## Code Structure

```typescript
contract SimpleToken {
  state name: string;
  state symbol: string;
  state totalSupply: uint;
  state balances: Map<address, uint>;  // Address -> Balance mapping
  
  entry transfer(recipient: address, amount: uint): bool
  entry balanceOf(account: address): uint
  entry mint(account: address, amount: uint)
  entry burn(amount: uint): bool
}
```

## How to Use

### Compile

```bash
compactc simple_token.compact
```

### Deploy

```bash
midnight-cli deploy simple_token \
  --name "MyToken" \
  --symbol "MTK" \
  --supply 1000000 \
  --owner <your-address>
```

### Interact

```typescript
// Initialize the token
await token.initialize(
  "MyToken",
  "MTK",
  1000000,
  ownerAddress
);

// Check initial balance
const balance = await token.balanceOf(ownerAddress);
console.log(`Balance: ${balance}`); // 1000000

// Transfer tokens
const success = await token.transfer(recipientAddress, 100);
console.log(`Transfer successful: ${success}`); // true

// Check new balances
const ownerBal = await token.balanceOf(ownerAddress);
const recipientBal = await token.balanceOf(recipientAddress);
console.log(`Owner: ${ownerBal}, Recipient: ${recipientBal}`);
// Owner: 999900, Recipient: 100

// Mint new tokens
await token.mint(ownerAddress, 500);
const newSupply = await token.getTotalSupply();
console.log(`New supply: ${newSupply}`); // 1000500

// Burn tokens
await token.burn(100);
const burnedSupply = await token.getTotalSupply();
console.log(`Supply after burn: ${burnedSupply}`); // 1000400
```

## Key Concepts

### Mappings
Compact's `Map<K, V>` type stores key-value pairs:
```typescript
state balances: Map<address, uint>;

// Access/set values
balances[account] = 1000;
let balance = balances[account] || 0;  // Default to 0 if not set
```

### Address Type
The `address` type represents blockchain addresses:
- Used for account identification
- `msg.sender` provides the transaction sender's address

### Transfer Logic
Safe transfer requires validation:
```typescript
entry transfer(recipient: address, amount: uint): bool {
  let senderBalance = balances[msg.sender] || 0;
  
  // Validate sufficient balance
  if (senderBalance < amount) {
    return false;
  }
  
  // Perform transfer
  balances[msg.sender] = senderBalance - amount;
  balances[recipient] = balances[recipient] + amount;
  
  return true;
}
```

## Important Notes

### Limitations of This Example
This is a **simplified** token for learning purposes. Production tokens should include:

1. **Access Control**: Only authorized addresses can mint
2. **Events**: Emit events for transfers, mints, burns
3. **Allowances**: Support for approve/transferFrom pattern
4. **Overflow Protection**: Check for arithmetic overflows
5. **Pausability**: Ability to pause transfers
6. **Blacklisting**: Optional address restrictions

### msg.sender
The `msg.sender` variable provides the address of the transaction sender:
```typescript
let senderBalance = balances[msg.sender] || 0;
```

### Default Values
When accessing a mapping key that doesn't exist:
```typescript
balances[account] || 0  // Returns 0 if account not in map
```

## Best Practices

1. **Always Validate**: Check balances before transfers
2. **Use Safe Math**: Prevent overflows and underflows
3. **Return Status**: Indicate success/failure with return values
4. **Document Edge Cases**: What happens with 0 amounts, non-existent accounts, etc.

## Exercises

Extend this token contract:

1. **Add Allowances**: Implement `approve()` and `transferFrom()`
2. **Add Events**: Emit events on transfer, mint, burn
3. **Access Control**: Add owner-only mint function
4. **Pausable**: Add pause/unpause functionality
5. **Max Supply**: Enforce a maximum total supply cap
6. **Batch Transfer**: Transfer to multiple recipients at once

## Next Steps

- **Private Transfer Example**: Add zero-knowledge proofs for private transfers
- Study ERC-20 standard for more complete token functionality
- Learn about token economics and tokenomics

## Related Concepts

- **Fungible Tokens**: Each token is identical and interchangeable
- **Token Standards**: ERC-20 (Ethereum), similar patterns in Midnight
- **Gas Costs**: More complex operations cost more DUST
- **State Storage**: Mappings are efficient for large numbers of accounts

## Security Considerations

⚠️ **Important**: This example lacks several security features:

- No access control on mint
- No reentrancy protection
- No overflow checks
- No pause mechanism

For production use, implement proper security measures or use audited token templates.
