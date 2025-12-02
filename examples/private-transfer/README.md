# Private Token Transfer Example

This example demonstrates Midnight's core innovation: **privacy-preserving transactions using zero-knowledge proofs**.

## What You'll Learn

- **Witness functions**: Off-chain private computation
- **Zero-knowledge proofs**: Prove validity without revealing data
- **Private vs public state**: When to use each
- **Proof verification**: How Midnight validates proofs on-chain

## Contract Overview

The `PrivateToken` contract shows how to:
- Generate zero-knowledge proofs off-chain
- Verify proofs on-chain
- Keep transaction details private
- Prove sufficient balance without revealing it

## Key Concepts

### Witness Functions

Witness functions run **off-chain** to generate zero-knowledge proofs:

```typescript
witness transferProof(
  sender: address,
  recipient: address,
  amount: uint
): proof {
  let senderBalance = getPrivateBalance(sender);
  assert(senderBalance >= amount, "Insufficient balance");
  return proof;
}
```

- **Off-chain execution**: Runs on user's device, not on blockchain
- **Private data access**: Can access sensitive information
- **Proof generation**: Creates cryptographic proof of validity
- **No data leakage**: Proof reveals nothing about the actual balance

### Entry Points with Proofs

Entry points **verify** proofs on-chain:

```typescript
entry privateTransfer(
  recipient: address,
  amount: uint,
  zkProof: proof
): bool {
  // Verify proof without knowing sender's balance
  // Update state if proof is valid
}
```

- **On-chain execution**: Runs on blockchain
- **Proof verification**: Validates the zero-knowledge proof
- **State updates**: Modifies contract state if proof is valid
- **Privacy preserved**: Never sees actual private data

## How It Works

### Private Transfer Flow

1. **User initiates transfer** (off-chain)
   ```typescript
   // User wants to send 100 tokens
   const amount = 100;
   const recipient = "0x742d35...";
   ```

2. **Generate witness proof** (off-chain)
   ```typescript
   // Midnight runtime calls witness function
   const zkProof = await contract.transferProof(
     sender,
     recipient,
     amount
   );
   // Proof: "I have ≥100 tokens" (without revealing actual balance)
   ```

3. **Submit transaction with proof** (on-chain)
   ```typescript
   // Send transaction to blockchain
   await contract.privateTransfer(
     recipient,
     amount,
     zkProof
   );
   ```

4. **Verify and execute** (on-chain)
   ```typescript
   // Contract verifies proof and updates state
   // No one can see sender's original balance
   ```

## Privacy Guarantees

### What's Hidden
- ✅ Sender's total balance
- ✅ Detailed transaction amounts (in advanced implementations)
- ✅ Transaction history patterns
- ✅ Account relationships

### What's Visible
- 🔍 Transaction occurred
- 🔍 Recipient address (in this example)
- 🔍 Proof verification result

### Advanced Privacy
In production implementations, you can hide even more:
- Recipient addresses (using shielded addresses)
- Transaction amounts (using range proofs)
- Transaction linkability (using nullifiers)

## Comparison: Public vs Private

### Public Transfer
```typescript
entry publicTransfer(recipient: address, amount: uint): bool {
  // Everyone can see:
  // - Sender's balance before/after
  // - Recipient's balance before/after
  // - Exact transfer amount
}
```

### Private Transfer
```typescript
entry privateTransfer(recipient: address, amount: uint, zkProof: proof): bool {
  // Hidden:
  // - Sender's actual balance (proved with ZK)
  // - Balance after transfer (only proof verified)
  // Visible:
  // - Transfer occurred (but not details)
}
```

## Example Usage

### Compile

```bash
compactc private_token.compact
```

This generates:
- ZK circuits for witness functions
- Proof verification code
- Contract deployment artifacts

### Deploy

```bash
midnight-cli deploy private_token \
  --name "PrivateToken" \
  --symbol "PRVT" \
  --supply 1000000
```

### Interact

```typescript
// Initialize
await token.initialize("PrivateToken", "PRVT", 1000000, ownerAddress);

// Public transfer (everyone sees details)
await token.publicTransfer(recipient, 100);

// Private transfer (only proves validity)
// Step 1: Generate proof (automatic)
const proof = await token.witness.transferProof(
  sender,
  recipient,
  100
);

// Step 2: Submit transaction with proof
const success = await token.privateTransfer(
  recipient,
  100,
  proof
);

// Prove minimum balance without revealing exact amount
const minBalanceProof = await token.witness.balanceProof(
  myAddress,
  1000  // Prove I have at least 1000 tokens
);

const hasMinBalance = await token.hasMinimumBalance(
  myAddress,
  1000,
  minBalanceProof
);
console.log(`Has ≥1000 tokens: ${hasMinBalance}`); // true/false
```

## Zero-Knowledge Proofs Explained

### What is a Zero-Knowledge Proof?

A ZK proof allows you to prove a statement is true without revealing why it's true.

**Example**: Prove you're over 18 without revealing your exact age.

### In Midnight Context

```typescript
witness transferProof(...) {
  assert(balance >= amount, "Insufficient balance");
  return proof;
}
```

This generates a proof that says:
- **Claim**: "I have at least `amount` tokens"
- **Proof**: Mathematical proof the claim is true
- **Hidden**: Actual balance value

The blockchain verifies the proof without ever seeing your balance!

## Best Practices

### When to Use Private Transfers

✅ **Use private transfers for:**
- Financial transactions (salaries, payments)
- Sensitive business operations
- Personal transactions
- Compliance-requiring scenarios

❌ **Use public transfers for:**
- Public fundraising
- Transparent governance
- When privacy isn't needed
- Lower gas costs preferred

### Security Considerations

1. **Proof Generation Security**
   - Witness functions run on user's device
   - Keep private keys secure
   - Validate proof parameters

2. **On-Chain Verification**
   - Always verify proofs before state changes
   - Handle invalid proofs gracefully
   - Prevent replay attacks

3. **Privacy Leakage**
   - Be careful with timing analysis
   - Consider metadata privacy
   - Use shielded addresses when possible

## Advanced Topics

### Range Proofs
Prove a value is within a range without revealing it:
```typescript
witness rangeProof(value: uint, min: uint, max: uint): proof {
  assert(value >= min && value <= max, "Out of range");
  return proof;
}
```

### Nullifiers
Prevent double-spending in private transactions:
```typescript
witness spendProof(note: Note, nullifier: bytes32): proof {
  // Prove you can spend this note without revealing which note
}
```

### Shielded Addresses
Hide recipient addresses:
```typescript
entry shieldedTransfer(
  shieldedRecipient: bytes32,
  amount: uint,
  zkProof: proof
): bool {
  // Recipient address encrypted and hidden
}
```

## Performance Considerations

### Proof Generation
- Happens off-chain (no gas cost)
- Takes computational time
- Depends on circuit complexity

### Proof Verification
- Happens on-chain (gas cost)
- Usually constant time
- Much faster than proof generation

### Optimization Tips
1. Minimize witness function complexity
2. Batch multiple operations
3. Cache proof parameters
4. Use efficient ZK circuits

## Next Steps

1. **Study ZK Proofs**: Understand the cryptography
2. **Experiment**: Try different privacy patterns
3. **Advanced Examples**: Explore voting, auctions, DeFi
4. **Production Ready**: Add comprehensive error handling

## Resources

- [Zero-Knowledge Proofs Explained](https://docs.midnight.network/academy/cryptography/zk-proofs)
- [Witness Functions Guide](https://docs.midnight.network/compact/witnesses)
- [Privacy Patterns](https://docs.midnight.network/guides/privacy-patterns)
- [ZK Circuit Optimization](https://docs.midnight.network/advanced/circuit-optimization)

## Related Examples

- **Private Voting**: Vote without revealing choice
- **Confidential Auction**: Bid without showing amount
- **Private DeFi**: Trading with privacy
- **Compliance Tokens**: Regulated privacy

---

This example demonstrates the **core innovation** of Midnight: combining blockchain transparency with cryptographic privacy. Master this pattern and you unlock powerful privacy-preserving applications! 🌙
