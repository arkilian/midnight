# Troubleshooting Guide

Common issues and solutions when developing on Midnight.

## Table of Contents

- [Installation Issues](#installation-issues)
- [Compilation Errors](#compilation-errors)
- [Deployment Problems](#deployment-problems)
- [Runtime Errors](#runtime-errors)
- [Zero-Knowledge Proof Issues](#zero-knowledge-proof-issues)
- [Network and RPC Issues](#network-and-rpc-issues)
- [Gas and Transaction Issues](#gas-and-transaction-issues)
- [Development Tools](#development-tools)

## Installation Issues

### Node.js Version Mismatch

**Problem**: Errors during npm install
```
Error: The engine "node" is incompatible with this module
```

**Solution**:
```bash
# Check your Node.js version
node --version

# Install Node.js 18 or higher
# Using nvm (recommended)
nvm install 18
nvm use 18

# Verify
node --version  # Should show v18.x.x or higher
```

### Compact Compiler Not Found

**Problem**: `compactc: command not found`

**Solution**:
```bash
# Install globally
npm install -g @midnight-ntwrk/compact-compiler

# Or use npx
npx @midnight-ntwrk/compact-compiler contract.compact

# Or add to package.json scripts
{
  "scripts": {
    "compile": "compactc contracts/*.compact"
  }
}
```

### SDK Installation Fails

**Problem**: npm install fails for Midnight SDK

**Solution**:
```bash
# Clear npm cache
npm cache clean --force

# Remove node_modules and package-lock.json
rm -rf node_modules package-lock.json

# Reinstall
npm install

# If still failing, try with --legacy-peer-deps
npm install --legacy-peer-deps
```

## Compilation Errors

### Syntax Errors

**Problem**: "Unexpected token" or syntax errors

**Example Error**:
```
Error: Unexpected token ';' at line 10, column 5
```

**Solution**:
```typescript
// ❌ Wrong: Missing semicolon
state balance: uint

// ✅ Correct: Proper syntax
state balance: uint;

// ❌ Wrong: Invalid entry point syntax
entry function transfer() { }

// ✅ Correct: Proper entry point
entry transfer(to: address, amount: uint) {
  // implementation
}
```

### Type Errors

**Problem**: Type mismatch errors

**Example Error**:
```
Error: Cannot assign type 'string' to 'uint'
```

**Solution**:
```typescript
// ❌ Wrong: Type mismatch
state count: uint;
count = "123";

// ✅ Correct: Matching types
state count: uint;
count = 123;

// Type conversion when needed
let stringValue: string = "123";
let numValue: uint = uint(stringValue);
```

### Undefined State Variable

**Problem**: "State variable not defined"

**Solution**:
```typescript
// ❌ Wrong: Using before declaring
contract MyContract {
  entry initialize() {
    balance = 100;  // balance not declared
  }
}

// ✅ Correct: Declare state first
contract MyContract {
  state balance: uint;
  
  entry initialize() {
    balance = 100;
  }
}
```

### Import/Module Errors

**Problem**: Cannot import modules

**Solution**:
```typescript
// Ensure proper contract structure
// Compact doesn't support imports like TypeScript
// Each .compact file is self-contained

// If you need shared code, use a different approach
// or wait for library support in future versions
```

## Deployment Problems

### Insufficient Gas

**Problem**: "Transaction ran out of gas"

**Solution**:
```bash
# Increase gas limit in deployment
midnight-cli deploy contract.json \
  --gas-limit 5000000

# Or in code
const contract = await sdk.deploy({
  artifact: contractArtifact,
  gasLimit: 5000000
});
```

### Insufficient DUST Tokens

**Problem**: "Insufficient funds for gas"

**Solution**:
```bash
# Check balance
midnight-cli balance <your-address>

# Request testnet tokens
midnight-cli faucet request --address <your-address>

# Or visit the faucet website
# https://testnet-faucet.midnight.network
```

### Contract Already Deployed

**Problem**: "Contract already exists at this address"

**Solution**:
```bash
# Deploy to a new address (default behavior)
midnight-cli deploy contract.json

# Or check existing deployment
midnight-cli inspect <contract-address>

# Force re-deployment (if supported)
midnight-cli deploy contract.json --force
```

### Initialization Fails

**Problem**: Contract deploys but initialization fails

**Solution**:
```typescript
// ✅ Ensure initialization is idempotent
contract SafeInit {
  state initialized: bool;
  
  entry initialize(owner: address) {
    if (initialized) {
      return;  // Already initialized
    }
    
    // Initialize state
    this.owner = owner;
    initialized = true;
  }
}
```

### Network Configuration Issues

**Problem**: "Cannot connect to network"

**Solution**:
```bash
# Check network configuration
midnight-cli config show

# Set correct RPC URL
midnight-cli config set rpc_url https://testnet-rpc.midnight.network

# Test connection
midnight-cli network info
```

## Runtime Errors

### Transaction Reverted

**Problem**: "Transaction reverted without a reason"

**Solution**:
```typescript
// Add error messages to assertions
witness validateAmount(amount: uint): proof {
  assert(amount > 0, "Amount must be positive");
  assert(amount <= maxAmount, "Amount exceeds maximum");
  return proof;
}

// Return status codes
entry transfer(to: address, amount: uint): bool {
  if (balances[msg.sender] < amount) {
    return false;  // Explicit failure
  }
  // ... transfer logic
  return true;
}
```

### State Not Updating

**Problem**: State changes don't persist

**Solution**:
```typescript
// ❌ Wrong: Modifying local variable
entry wrongUpdate() {
  let balance = balances[msg.sender];
  balance = balance + 100;  // Only updates local variable
}

// ✅ Correct: Update state directly
entry correctUpdate() {
  let balance = balances[msg.sender];
  balances[msg.sender] = balance + 100;  // Updates state
}
```

### Reentrancy Attack

**Problem**: Contract vulnerable to reentrancy

**Solution**:
```typescript
contract ReentrancyProtected {
  state locked: bool;
  
  entry withdraw(amount: uint) {
    // Check-Effects-Interactions pattern
    
    // Checks
    if (locked) return;
    let balance = balances[msg.sender];
    if (balance < amount) return;
    
    // Effects
    locked = true;
    balances[msg.sender] = balance - amount;
    
    // Interactions (external calls)
    // ... transfer logic
    
    locked = false;
  }
}
```

### Integer Overflow/Underflow

**Problem**: Unexpected values due to overflow

**Solution**:
```typescript
// Use safe math functions
function safeAdd(a: uint, b: uint): uint {
  let c = a + b;
  assert(c >= a, "Overflow detected");
  return c;
}

entry safeTransfer(to: address, amount: uint) {
  let senderBalance = balances[msg.sender];
  assert(senderBalance >= amount, "Insufficient balance");
  
  let recipientBalance = balances[to];
  
  // Safe operations
  balances[msg.sender] = senderBalance - amount;
  balances[to] = safeAdd(recipientBalance, amount);
}
```

## Zero-Knowledge Proof Issues

### Proof Generation Fails

**Problem**: "Failed to generate proof"

**Solution**:
```typescript
// Simplify witness function
// ❌ Complex: Too many operations
witness complexProof(data: uint[]): proof {
  let result = 0;
  for (let i = 0; i < data.length; i++) {
    result += data[i] * data[i] * data[i];  // Expensive
  }
  assert(result > threshold, "Below threshold");
  return proof;
}

// ✅ Simple: Pre-compute when possible
witness simpleProof(precomputed: uint): proof {
  assert(precomputed > threshold, "Below threshold");
  return proof;
}
```

### Proof Verification Fails

**Problem**: Valid proof rejected on-chain

**Solution**:
```typescript
// Ensure proof parameters match
// Both witness and entry must use same data

// ❌ Mismatch
witness proof1(value: uint): proof {
  assert(value > 100, "Too low");
  return proof;
}

entry verify1(value: uint, zkProof: proof): bool {
  // Verifying different value!
  return true;
}

// ✅ Matching
witness proof2(value: uint): proof {
  assert(value > 100, "Too low");
  return proof;
}

entry verify2(zkProof: proof): bool {
  // Proof already contains validated value
  return true;
}
```

### Circuit Complexity Error

**Problem**: "Circuit too complex"

**Solution**:
```bash
# Optimize witness functions
# - Reduce loop iterations
# - Simplify computations
# - Use lookup tables instead of computation

# Increase circuit size limit (if available)
compactc contract.compact --max-constraints 1000000
```

### Proof Takes Too Long

**Problem**: Proof generation is very slow

**Solution**:
```typescript
// Optimize:
// 1. Reduce witness complexity
// 2. Pre-compute values off-chain
// 3. Batch multiple proofs
// 4. Use more efficient algorithms

// Example: Batch verification
witness batchProof(
  values: uint[],
  threshold: uint
): proof {
  // Verify all at once instead of individually
  let sum = 0;
  for (let i = 0; i < values.length; i++) {
    sum += values[i];
  }
  assert(sum > threshold, "Below threshold");
  return proof;
}
```

## Network and RPC Issues

### RPC Connection Timeout

**Problem**: "Connection timeout" when calling RPC

**Solution**:
```bash
# Check network status
curl https://testnet-rpc.midnight.network/health

# Try alternative RPC endpoint
midnight-cli config set rpc_url https://testnet-rpc2.midnight.network

# Increase timeout
const sdk = await MidnightSDK.connect({
  network: 'testnet',
  timeout: 30000  // 30 seconds
});
```

### Rate Limiting

**Problem**: "Too many requests"

**Solution**:
```javascript
// Implement rate limiting
const delay = ms => new Promise(resolve => setTimeout(resolve, ms));

async function rateLimitedCall() {
  try {
    return await contract.method();
  } catch (error) {
    if (error.code === 429) {  // Too Many Requests
      await delay(1000);  // Wait 1 second
      return await contract.method();  // Retry
    }
    throw error;
  }
}

// Use exponential backoff
async function exponentialBackoff(fn, maxRetries = 5) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await delay(Math.pow(2, i) * 1000);
    }
  }
}
```

### Wrong Network

**Problem**: Deploying to wrong network

**Solution**:
```bash
# Check current network
midnight-cli config show

# Switch to testnet
midnight-cli config set network testnet

# Verify before deployment
midnight-cli network info

# Always specify network explicitly
midnight-cli deploy contract.json --network testnet
```

## Gas and Transaction Issues

### Transaction Too Expensive

**Problem**: Gas cost too high

**Solution**:
```typescript
// Optimize state operations
// ❌ Expensive: Multiple state writes in loop
entry expensive(values: uint[]) {
  for (let i = 0; i < values.length; i++) {
    state[i] = values[i];  // Multiple writes
  }
}

// ✅ Cheaper: Batch operations
entry cheaper(sum: uint) {
  totalSum = sum;  // Single write
}

// Use events instead of state when possible
// Events are cheaper than state storage
```

### Transaction Stuck

**Problem**: Transaction pending for too long

**Solution**:
```bash
# Check transaction status
midnight-cli tx status <tx-hash>

# Check mempool
midnight-cli mempool

# Speed up transaction (if supported)
midnight-cli tx speedup <tx-hash> --gas-price 150%

# Cancel transaction (if supported)
midnight-cli tx cancel <tx-hash>
```

### Nonce Issues

**Problem**: "Invalid nonce"

**Solution**:
```bash
# Check current nonce
midnight-cli account nonce <address>

# Reset nonce (if stuck)
midnight-cli account reset-nonce <address>

# Manually set nonce
const tx = await contract.method({
  nonce: await sdk.getTransactionCount(address)
});
```

## Development Tools

### Debugging Contracts

```typescript
// Add debug entry points
entry debug_getState(): (uint, address, bool) {
  return (balance, owner, isActive);
}

// Use events for debugging
event Debug(message: string, value: uint);

entry debugOperation() {
  emit Debug("Before operation", balance);
  // ... operation
  emit Debug("After operation", balance);
}
```

### Testing Issues

```javascript
// Use proper test setup
describe('Contract Tests', () => {
  let contract;
  
  beforeEach(async () => {
    // Fresh deployment for each test
    contract = await deployContract();
  });
  
  it('should work correctly', async () => {
    // Test implementation
    const result = await contract.method();
    expect(result).to.equal(expected);
  });
  
  afterEach(async () => {
    // Cleanup if needed
  });
});
```

### IDE Configuration

**VS Code settings for Midnight development**:

Create `.vscode/settings.json`:
```json
{
  "files.associations": {
    "*.compact": "typescript"
  },
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "[typescript]": {
    "editor.tabSize": 2
  }
}
```

## Getting Help

When asking for help, provide:

1. **Compact version**: `compactc --version`
2. **Node version**: `node --version`
3. **Error message**: Full error with stack trace
4. **Code snippet**: Minimal reproducible example
5. **What you tried**: Steps taken to solve the issue

### Where to Get Help

- 🔍 **Search Documentation**: [docs.midnight.network](https://docs.midnight.network)
- 💬 **Discord**: [Join Midnight Discord](https://discord.com/invite/midnightntwrk)
- 📋 **Forum**: [forum.midnight.network](https://forum.midnight.network)
- 🐛 **GitHub Issues**: For SDK/tooling bugs
- 📚 **Stack Overflow**: Tag `midnight-blockchain`

## Common Error Messages

### "Transaction reverted"
- Check require/assert conditions
- Verify state before operations
- Add descriptive error messages

### "Out of gas"
- Increase gas limit
- Optimize contract code
- Avoid unbounded loops

### "Invalid proof"
- Check witness function logic
- Verify proof parameters match
- Simplify circuit complexity

### "Insufficient funds"
- Get testnet tokens from faucet
- Check account balance
- Verify gas price settings

### "Contract not found"
- Verify contract address
- Check network selection
- Confirm deployment succeeded

### "Nonce too low"
- Check pending transactions
- Wait for previous transactions
- Reset nonce if stuck

## Prevention Tips

1. **Test Thoroughly**: Write comprehensive tests
2. **Use Linters**: Catch errors early
3. **Code Reviews**: Have others review your code
4. **Start Simple**: Begin with basic contracts
5. **Read Docs**: Official documentation is your friend
6. **Stay Updated**: Follow Midnight announcements
7. **Backup Keys**: Never lose private keys
8. **Use Testnet**: Test everything before mainnet

## Emergency Procedures

If your deployed contract has issues:

1. **Pause Operations**: If emergency pause is implemented
2. **Notify Users**: Communicate the issue
3. **Assess Damage**: Understand the scope
4. **Plan Fix**: Determine solution (upgrade, migrate, etc.)
5. **Test Fix**: Thoroughly test on testnet
6. **Deploy Fix**: Execute carefully
7. **Post-Mortem**: Document what happened

---

**Remember**: Most issues have been encountered by others. Don't hesitate to ask for help in the community! 🌙
