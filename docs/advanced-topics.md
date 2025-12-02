# Advanced Topics in Midnight Development

This guide covers advanced concepts for building production-ready privacy-preserving applications on Midnight.

## Table of Contents

- [Advanced Zero-Knowledge Proofs](#advanced-zero-knowledge-proofs)
- [Performance Optimization](#performance-optimization)
- [Security Best Practices](#security-best-practices)
- [Gas Optimization](#gas-optimization)
- [Privacy Patterns](#privacy-patterns)
- [Production Deployment](#production-deployment)
- [Monitoring and Maintenance](#monitoring-and-maintenance)

## Advanced Zero-Knowledge Proofs

### Range Proofs

Prove a value lies within a range without revealing the exact value:

```typescript
contract AgeVerification {
  /**
   * Prove age is above minimum without revealing exact age
   */
  witness ageRangeProof(
    age: uint8,
    minAge: uint8,
    maxAge: uint8
  ): proof {
    assert(age >= minAge, "Below minimum age");
    assert(age <= maxAge, "Above maximum age");
    return proof;
  }

  entry verifyAgeRange(
    minAge: uint8,
    maxAge: uint8,
    zkProof: proof
  ): bool {
    // Verify proof without seeing actual age
    return true;
  }
}
```

### Merkle Tree Proofs

Prove membership in a set efficiently:

```typescript
contract WhitelistVerification {
  state merkleRoot: bytes32;

  /**
   * Prove address is in whitelist using Merkle proof
   */
  witness membershipProof(
    account: address,
    merkleProof: bytes32[]
  ): proof {
    // Note: In actual Compact implementation, use appropriate hash function
    // This is pseudocode for illustration purposes
    // Compact may provide hash functions like hash() or similar
    
    // Compute leaf hash
    let leaf = hash(account);  // Use Compact's hash function
    let computedHash = leaf;

    // Verify Merkle path
    for (let i = 0; i < merkleProof.length; i = i + 1) {
      let proofElement = merkleProof[i];
      
      if (computedHash < proofElement) {
        computedHash = hash(computedHash, proofElement);
      } else {
        computedHash = hash(proofElement, computedHash);
      }
    }

    // Verify against root
    assert(computedHash == merkleRoot, "Not in whitelist");
    return proof;
  }

  entry verifyMembership(zkProof: proof): bool {
    // Verify without revealing which member
    return true;
  }
}
```

### Nullifier Patterns

Prevent double-spending while maintaining privacy:

```typescript
contract PrivateNFT {
  state usedNullifiers: Map<bytes32, bool>;

  /**
   * Generate nullifier to prevent double-spending
   */
  witness spendProof(
    tokenId: uint,
    secret: bytes32
  ): (proof, bytes32) {
    // Nullifier = hash(tokenId, secret)
    // Note: Use Compact's built-in hash function
    let nullifier = hash(tokenId, secret);
    
    // Verify not already used
    assert(!usedNullifiers[nullifier], "Already spent");
    
    return (proof, nullifier);
  }

  entry spendToken(
    zkProof: proof,
    nullifier: bytes32
  ): bool {
    // Check nullifier not used
    if (usedNullifiers[nullifier]) {
      return false;
    }

    // Mark as used
    usedNullifiers[nullifier] = true;
    return true;
  }
}
```

### Commitment Schemes

Hide values with cryptographic commitments:

```typescript
contract SealedBidAuction {
  state commitments: Map<address, bytes32>;
  state reveals: Map<address, uint>;
  state auctionPhase: uint8; // 0=commit, 1=reveal, 2=ended

  /**
   * Commit to a bid without revealing amount
   */
  witness bidCommitment(
    bidAmount: uint,
    salt: bytes32
  ): bytes32 {
    // Commitment = hash(bidAmount, salt)
    // Note: Use Compact's built-in hash function
    let commitment = hash(bidAmount, salt);
    return commitment;
  }

  entry commitBid(commitment: bytes32) {
    if (auctionPhase != 0) {
      return;
    }
    commitments[msg.sender] = commitment;
  }

  /**
   * Reveal bid with proof it matches commitment
   */
  witness revealProof(
    bidAmount: uint,
    salt: bytes32,
    commitment: bytes32
  ): proof {
    let computed = hash(bidAmount, salt);
    assert(computed == commitment, "Invalid reveal");
    return proof;
  }

  entry revealBid(
    bidAmount: uint,
    zkProof: proof
  ) {
    if (auctionPhase != 1) {
      return;
    }
    reveals[msg.sender] = bidAmount;
  }
}
```

## Performance Optimization

### Circuit Complexity

Minimize witness function complexity:

```typescript
// ❌ Bad: Complex loops in witness
witness inefficientProof(values: uint[]): proof {
  let sum = 0;
  for (let i = 0; i < values.length; i++) {
    sum += values[i] * values[i]; // Expensive
  }
  assert(sum > threshold, "Below threshold");
  return proof;
}

// ✅ Good: Pre-computed values
witness efficientProof(precomputedSum: uint): proof {
  assert(precomputedSum > threshold, "Below threshold");
  return proof;
}
```

### Batch Operations

Batch multiple operations together:

```typescript
contract OptimizedToken {
  /**
   * Batch transfer to multiple recipients
   */
  entry batchTransfer(
    recipients: address[],
    amounts: uint[]
  ): bool {
    if (recipients.length != amounts.length) {
      return false;
    }

    let totalAmount = 0;
    for (let i = 0; i < amounts.length; i++) {
      totalAmount += amounts[i];
    }

    // Single balance check
    if (balances[msg.sender] < totalAmount) {
      return false;
    }

    // Update balances
    balances[msg.sender] -= totalAmount;
    for (let i = 0; i < recipients.length; i++) {
      balances[recipients[i]] += amounts[i];
    }

    return true;
  }
}
```

### State Access Patterns

Minimize redundant state reads:

```typescript
// ❌ Bad: Multiple state reads
entry inefficient(account: address) {
  if (balances[account] > 100) {
    balances[account] = balances[account] - 100;
    totalSupply = totalSupply - 100;
  }
}

// ✅ Good: Cache state reads
entry efficient(account: address) {
  let balance = balances[account];
  if (balance > 100) {
    balances[account] = balance - 100;
    totalSupply -= 100;
  }
}
```

## Security Best Practices

### Access Control

Implement robust permission systems:

```typescript
contract SecureContract {
  state owner: address;
  state admins: Map<address, bool>;
  state paused: bool;

  // Modifiers (as functions)
  function onlyOwner(): bool {
    return msg.sender == owner;
  }

  function onlyAdmin(): bool {
    return admins[msg.sender] || msg.sender == owner;
  }

  function whenNotPaused(): bool {
    return !paused;
  }

  // Protected functions
  entry addAdmin(newAdmin: address) {
    if (!onlyOwner()) {
      return;
    }
    admins[newAdmin] = true;
  }

  entry criticalOperation() {
    if (!onlyAdmin() || !whenNotPaused()) {
      return;
    }
    // Critical logic
  }

  entry pause() {
    if (!onlyOwner()) {
      return;
    }
    paused = true;
  }

  entry unpause() {
    if (!onlyOwner()) {
      return;
    }
    paused = false;
  }
}
```

### Reentrancy Protection

Prevent reentrancy attacks:

```typescript
contract ReentrancyGuard {
  state locked: bool;

  function nonReentrant(): bool {
    if (locked) {
      return false;
    }
    locked = true;
    return true;
  }

  function unlock() {
    locked = false;
  }

  entry withdraw(amount: uint) {
    if (!nonReentrant()) {
      return;
    }

    // Critical operations
    let balance = balances[msg.sender];
    if (balance >= amount) {
      balances[msg.sender] = balance - amount;
      // Transfer logic
    }

    unlock();
  }
}
```

### Input Validation

Always validate inputs thoroughly:

```typescript
contract SafeContract {
  entry safeTransfer(
    recipient: address,
    amount: uint
  ): bool {
    // Validate recipient
    if (recipient == address(0)) {
      return false;
    }
    if (recipient == msg.sender) {
      return false;
    }

    // Validate amount
    if (amount == 0) {
      return false;
    }

    // Check balance
    let senderBalance = balances[msg.sender];
    if (senderBalance < amount) {
      return false;
    }

    // Prevent overflow
    let recipientBalance = balances[recipient];
    if (recipientBalance + amount < recipientBalance) {
      return false; // Overflow detected
    }

    // Execute transfer
    balances[msg.sender] = senderBalance - amount;
    balances[recipient] = recipientBalance + amount;

    return true;
  }
}
```

### Integer Overflow Protection

Implement safe math:

```typescript
contract SafeMath {
  function safeAdd(a: uint, b: uint): (bool, uint) {
    let c = a + b;
    if (c < a) {
      return (false, 0); // Overflow
    }
    return (true, c);
  }

  function safeSub(a: uint, b: uint): (bool, uint) {
    if (b > a) {
      return (false, 0); // Underflow
    }
    return (true, a - b);
  }

  function safeMul(a: uint, b: uint): (bool, uint) {
    if (a == 0) {
      return (true, 0);
    }
    let c = a * b;
    if (c / a != b) {
      return (false, 0); // Overflow
    }
    return (true, c);
  }

  function safeDiv(a: uint, b: uint): (bool, uint) {
    if (b == 0) {
      return (false, 0); // Division by zero
    }
    return (true, a / b);
  }
}
```

## Gas Optimization

### Storage Patterns

Optimize storage layout:

```typescript
// ❌ Bad: Many small state variables
state var1: uint8;
state var2: uint8;
state var3: uint8;
state var4: uint8;

// ✅ Good: Pack variables
state packedVars: uint32; // Pack four uint8 into one uint32

function getVar1(): uint8 {
  return uint8(packedVars & 0xFF);
}

function setVar1(value: uint8) {
  packedVars = (packedVars & 0xFFFFFF00) | value;
}
```

### Loop Optimization

Avoid unbounded loops:

```typescript
// ❌ Bad: Unbounded loop
entry processAll() {
  for (let i = 0; i < users.length; i++) {
    // Process user - could run out of gas
  }
}

// ✅ Good: Paginated processing
entry processPage(startIndex: uint, count: uint) {
  let endIndex = startIndex + count;
  if (endIndex > users.length) {
    endIndex = users.length;
  }

  for (let i = startIndex; i < endIndex; i++) {
    // Process user safely
  }
}
```

## Privacy Patterns

### Shielded Addresses

Hide recipient addresses:

```typescript
contract ShieldedTransfer {
  /**
   * Transfer to encrypted address
   */
  witness shieldedTransferProof(
    recipient: address,
    encryptionKey: bytes32,
    amount: uint
  ): (proof, bytes32) {
    // Encrypt recipient address
    // Note: Actual encryption would use Compact's cryptographic primitives
    // This is pseudocode for illustration
    let shieldedAddress = hash(recipient, encryptionKey);  // Simplified
    
    return (proof, shieldedAddress);
  }

  entry shieldedTransfer(
    shieldedRecipient: bytes32,
    amount: uint,
    zkProof: proof
  ): bool {
    // Transfer occurs without revealing recipient
    // Recipient can decrypt to claim funds
    return true;
  }
}
```

### Private Balances

Hide account balances:

```typescript
contract PrivateBalances {
  state commitments: Map<bytes32, bool>;

  witness balanceCommitment(
    account: address,
    balance: uint,
    salt: bytes32
  ): bytes32 {
    // Commit to balance without revealing it
    let commitment = keccak256(account, balance, salt);
    return commitment;
  }

  entry registerCommitment(commitment: bytes32) {
    commitments[commitment] = true;
  }

  witness transferProof(
    senderCommitment: bytes32,
    senderBalance: uint,
    amount: uint
  ): proof {
    assert(senderBalance >= amount, "Insufficient");
    return proof;
  }
}
```

## Production Deployment

### Pre-Deployment Checklist

✅ Security audit completed
✅ Test coverage > 95%
✅ Gas optimization applied
✅ Access controls tested
✅ Emergency pause mechanism
✅ Upgrade strategy defined
✅ Documentation complete
✅ Monitoring setup
✅ Incident response plan

### Deployment Script

```javascript
const deploymentConfig = {
  network: 'mainnet',
  gasLimit: 5000000,
  confirmations: 3,
  verify: true
};

async function deployProduction() {
  console.log('🚀 Starting production deployment...');

  // 1. Pre-flight checks
  await runSecurityChecks();
  await verifyTestCoverage();

  // 2. Deploy contract
  const contract = await deploy({
    artifact: contractArtifact,
    config: deploymentConfig
  });

  console.log('✅ Contract deployed:', contract.address);

  // 3. Verify on explorer
  await verifyContract(contract.address);

  // 4. Initialize
  await contract.initialize(productionConfig);

  // 5. Transfer ownership
  await contract.transferOwnership(multisigAddress);

  // 6. Setup monitoring
  await setupMonitoring(contract.address);

  console.log('🎉 Deployment complete!');
}
```

### Upgrade Strategy

```typescript
contract Upgradeable {
  state implementation: address;
  state admin: address;

  entry upgrade(newImplementation: address) {
    if (msg.sender != admin) {
      return;
    }
    implementation = newImplementation;
  }

  entry delegateCall(data: bytes): bytes {
    // Delegate to implementation
    // Maintain state, upgrade logic
  }
}
```

## Monitoring and Maintenance

### Event Logging

```typescript
contract MonitoredContract {
  // Define events
  event Transfer(from: address, to: address, amount: uint);
  event AdminAdded(admin: address);
  event EmergencyPause(timestamp: uint);

  entry transfer(to: address, amount: uint) {
    // Transfer logic
    emit Transfer(msg.sender, to, amount);
  }

  entry addAdmin(newAdmin: address) {
    // Add admin logic
    emit AdminAdded(newAdmin);
  }
}
```

### Health Checks

```typescript
contract HealthChecks {
  entry healthCheck(): (bool, string) {
    // Check contract health
    if (paused) {
      return (false, "Contract paused");
    }
    
    if (totalSupply > maxSupply) {
      return (false, "Supply exceeded");
    }

    return (true, "All systems operational");
  }

  entry getMetrics(): (uint, uint, uint) {
    return (
      totalUsers,
      totalTransactions,
      totalVolume
    );
  }
}
```

### Emergency Response

```typescript
contract EmergencyControl {
  state emergency: bool;
  state emergencyAdmin: address;

  entry triggerEmergency() {
    if (msg.sender != emergencyAdmin) {
      return;
    }
    emergency = true;
    // Pause all operations
  }

  entry emergencyWithdraw(
    token: address,
    amount: uint
  ) {
    if (!emergency || msg.sender != emergencyAdmin) {
      return;
    }
    // Emergency withdrawal logic
  }
}
```

## Testing Advanced Features

```javascript
describe('Advanced Features', () => {
  it('should handle range proofs correctly', async () => {
    const age = 25;
    const proof = await contract.witness.ageRangeProof(
      age, 18, 100
    );
    
    const result = await contract.verifyAgeRange(18, 100, proof);
    expect(result).to.be.true;
  });

  it('should prevent replay attacks', async () => {
    const proof = await generateProof();
    await contract.useProof(proof);
    
    // Try to reuse proof
    await expect(
      contract.useProof(proof)
    ).to.be.reverted;
  });

  it('should handle high load', async () => {
    const promises = [];
    for (let i = 0; i < 1000; i++) {
      promises.push(contract.operation());
    }
    
    await Promise.all(promises);
    // Verify state consistency
  });
});
```

## Best Practices Summary

1. **Security First**: Audit before mainnet
2. **Test Everything**: Aim for 100% coverage
3. **Optimize Gas**: Users pay for execution
4. **Plan for Upgrades**: Contracts can have bugs
5. **Monitor Always**: Know what's happening
6. **Document Well**: Future you will thank you
7. **Privacy by Design**: Think about data leakage
8. **Fail Gracefully**: Handle errors properly

## Resources

- [Midnight Security Guide](https://docs.midnight.network/security/)
- [Gas Optimization Patterns](https://docs.midnight.network/optimization/)
- [ZK Proof Advanced Topics](https://docs.midnight.network/zk-advanced/)
- [Production Deployment Guide](https://docs.midnight.network/deployment/)

---

Master these advanced topics and you'll be ready to build production-grade privacy-preserving applications on Midnight! 🌙
