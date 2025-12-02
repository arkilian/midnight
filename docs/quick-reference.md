# Midnight Quick Reference

A quick reference guide for Midnight and Compact development.

## Contract Structure

```typescript
contract ContractName {
  // 1. State variables
  state variableName: Type;
  
  // 2. Entry points (public functions)
  entry functionName(param: Type): ReturnType {
    // implementation
  }
  
  // 3. Witness functions (ZK proofs)
  witness proofName(param: Type): proof {
    assert(condition, "error message");
    return proof;
  }
  
  // 4. Internal functions
  function helperName(param: Type): ReturnType {
    // implementation
  }
}
```

## Data Types

### Primitives
```typescript
uint, uint8, uint16, uint32, uint64, uint128, uint256  // Unsigned integers
int, int8, int16, int32, int64, int128, int256         // Signed integers
bool                                                     // Boolean
string                                                   // UTF-8 string
bytes, bytes32, bytes64                                  // Byte arrays
address                                                  // Blockchain address
```

### Complex Types
```typescript
uint[]                          // Dynamic array
uint[10]                        // Fixed array
Map<address, uint>              // Mapping
struct Person { name: string }  // Struct
enum Status { A, B, C }         // Enum
```

## Operators

### Arithmetic
```typescript
+   // Addition
-   // Subtraction
*   // Multiplication
/   // Division
%   // Modulo
```

### Comparison
```typescript
==  // Equal
!=  // Not equal
<   // Less than
>   // Greater than
<=  // Less or equal
>=  // Greater or equal
```

### Logical
```typescript
&&  // AND
||  // OR
!   // NOT
```

### Assignment
```typescript
=   // Assign
+=  // Add and assign
-=  // Subtract and assign
*=  // Multiply and assign
/=  // Divide and assign
```

## Control Flow

### If Statement
```typescript
if (condition) {
  // code
}

if (condition) {
  // code
} else if (otherCondition) {
  // code
} else {
  // code
}
```

### For Loop
```typescript
for (let i = 0; i < 10; i = i + 1) {
  // code
}
```

### While Loop
```typescript
while (condition) {
  // code
}
```

### Return
```typescript
return value;
```

### Assert (in witnesses)
```typescript
assert(condition, "error message");
```

## State Management

### Declaration
```typescript
state balance: uint;
state owner: address;
state users: Map<address, bool>;
```

### Reading
```typescript
let value = stateVariable;
let balance = balances[account] || 0;  // With default
```

### Writing
```typescript
stateVariable = newValue;
balances[account] = 100;
```

## Entry Points

### Basic Entry
```typescript
entry setName(newName: string) {
  name = newName;
}
```

### With Return
```typescript
entry getName(): string {
  return name;
}
```

### With Validation
```typescript
entry transfer(to: address, amount: uint): bool {
  if (balances[msg.sender] < amount) {
    return false;
  }
  balances[msg.sender] -= amount;
  balances[to] += amount;
  return true;
}
```

## Witness Functions

### Basic Witness
```typescript
witness verifyAge(age: uint): proof {
  assert(age >= 18, "Must be 18+");
  return proof;
}
```

### Complex Witness
```typescript
witness transferProof(
  sender: address,
  amount: uint
): proof {
  let balance = balances[sender];
  assert(balance >= amount, "Insufficient balance");
  assert(amount > 0, "Amount must be positive");
  return proof;
}
```

### Using Proofs
```typescript
entry verifyAndExecute(zkProof: proof): bool {
  // Proof is verified automatically
  // Execute if valid
  return true;
}
```

## Special Variables

```typescript
msg.sender      // Transaction sender address
msg.value       // Value sent with transaction
block.timestamp // Current block timestamp
block.number    // Current block number
```

## Common Patterns

### Access Control
```typescript
state owner: address;

function onlyOwner(): bool {
  return msg.sender == owner;
}

entry restricted() {
  if (!onlyOwner()) return;
  // Owner-only logic
}
```

### Initialization
```typescript
state initialized: bool;

entry initialize(owner: address) {
  if (initialized) return;
  this.owner = owner;
  initialized = true;
}
```

### Safe Transfer
```typescript
entry transfer(to: address, amount: uint): bool {
  if (to == address(0)) return false;
  if (amount == 0) return false;
  
  let senderBalance = balances[msg.sender];
  if (senderBalance < amount) return false;
  
  balances[msg.sender] = senderBalance - amount;
  balances[to] = balances[to] + amount;
  return true;
}
```

### Emergency Pause
```typescript
state paused: bool;

entry pause() {
  if (msg.sender != owner) return;
  paused = true;
}

entry unpause() {
  if (msg.sender != owner) return;
  paused = false;
}

entry operation() {
  if (paused) return;
  // Normal operation
}
```

## CLI Commands

### Compilation
```bash
# Compile contract
compactc contract.compact

# Compile with optimization
compactc contract.compact --optimize

# Compile all contracts
compactc contracts/*.compact
```

### Deployment
```bash
# Deploy to testnet
midnight-cli deploy contract.json --network testnet

# Deploy with init args
midnight-cli deploy contract.json --init-args '[arg1, arg2]'

# Deploy and verify
midnight-cli deploy contract.json --verify
```

### Interaction
```bash
# Call read-only function
midnight-cli call <address> functionName [args]

# Send transaction
midnight-cli send <address> functionName [args]

# Get contract state
midnight-cli state <address>
```

### Account
```bash
# Create account
midnight-cli account create

# Check balance
midnight-cli balance <address>

# Request testnet tokens
midnight-cli faucet request --address <address>
```

### Network
```bash
# Show config
midnight-cli config show

# Set network
midnight-cli config set network testnet

# Network info
midnight-cli network info
```

## SDK Usage

### Connect
```javascript
const { MidnightSDK } = require('@midnight-ntwrk/sdk');

const sdk = await MidnightSDK.connect({
  network: 'testnet',
  privateKey: process.env.PRIVATE_KEY
});
```

### Deploy
```javascript
const contract = await sdk.deploy({
  artifact: require('./contract.json'),
  initArgs: [arg1, arg2]
});
```

### Load Existing
```javascript
const contract = await sdk.loadContract({
  address: '0x...',
  abi: require('./contract.json')
});
```

### Call Functions
```javascript
// Read-only call
const value = await contract.getValue();

// Transaction
const tx = await contract.setValue(newValue);
await tx.wait();  // Wait for confirmation
```

### Generate Proofs
```javascript
const proof = await contract.witness.proofFunction(
  arg1,
  arg2
);

const tx = await contract.verifyFunction(proof);
await tx.wait();
```

## Testing

### Basic Test
```javascript
const { expect } = require('chai');

describe('Contract Tests', () => {
  let contract;
  
  beforeEach(async () => {
    contract = await deployContract();
  });
  
  it('should work', async () => {
    const result = await contract.method();
    expect(result).to.equal(expected);
  });
});
```

### ZK Proof Test
```javascript
it('should verify proof', async () => {
  const proof = await contract.witness.generateProof(value);
  const result = await contract.verify(proof);
  expect(result).to.be.true;
});
```

## Best Practices

### Security
- ✅ Validate all inputs
- ✅ Check for overflow/underflow
- ✅ Implement access control
- ✅ Prevent reentrancy
- ✅ Test thoroughly

### Gas Optimization
- ✅ Minimize state writes
- ✅ Use events for logs
- ✅ Batch operations
- ✅ Cache state reads
- ✅ Avoid unbounded loops

### Privacy
- ✅ Use witnesses for sensitive data
- ✅ Think about metadata leakage
- ✅ Minimize public information
- ✅ Use ZK proofs appropriately
- ✅ Consider timing attacks

### Code Quality
- ✅ Clear naming conventions
- ✅ Document functions
- ✅ Handle errors gracefully
- ✅ Write comprehensive tests
- ✅ Follow style guides

## Error Handling

### Return Status
```typescript
entry operation(): bool {
  if (errorCondition) {
    return false;  // Indicate failure
  }
  // Success logic
  return true;
}
```

### Assertions in Witnesses
```typescript
witness validate(value: uint): proof {
  assert(value > 0, "Must be positive");
  assert(value < MAX, "Exceeds maximum");
  return proof;
}
```

## Common Mistakes

### ❌ Uninitialized State
```typescript
// Wrong
entry use() {
  balance += 1;  // balance never initialized!
}

// Correct
entry initialize() {
  balance = 0;
}
entry use() {
  balance += 1;
}
```

### ❌ Missing Return
```typescript
// Wrong
entry getValue(): uint {
  let value = 100;
  // Missing return!
}

// Correct
entry getValue(): uint {
  let value = 100;
  return value;
}
```

### ❌ State Not Updated
```typescript
// Wrong
entry update() {
  let balance = balances[msg.sender];
  balance += 100;  // Only local variable updated!
}

// Correct
entry update() {
  let balance = balances[msg.sender];
  balances[msg.sender] = balance + 100;
}
```

## Useful Links

- 📖 [Midnight Docs](https://docs.midnight.network/)
- 🎓 [Developer Academy](https://docs.midnight.network/academy/)
- 💬 [Discord](https://discord.com/invite/midnightntwrk)
- 🌐 [Developer Hub](https://midnight.network/developer-hub)
- 📝 [Blog](https://midnight.network/blog/)
- 🐙 [GitHub](https://github.com/midnight-network)

## Keyboard Shortcuts (VS Code)

- `Ctrl/Cmd + Space`: Autocomplete
- `F12`: Go to definition
- `Shift + F12`: Find all references
- `Ctrl/Cmd + .`: Quick fix
- `Ctrl/Cmd + Shift + P`: Command palette

---

Keep this reference handy while developing on Midnight! 🌙
