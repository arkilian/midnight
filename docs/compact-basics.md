# Compact Language Basics

A comprehensive guide to the Compact programming language, the smart contract language for Midnight blockchain.

## Table of Contents

- [Introduction](#introduction)
- [Basic Syntax](#basic-syntax)
- [Data Types](#data-types)
- [Contract Structure](#contract-structure)
- [State Variables](#state-variables)
- [Entry Points](#entry-points)
- [Witness Functions](#witness-functions)
- [Control Flow](#control-flow)
- [Operators](#operators)
- [Best Practices](#best-practices)

## Introduction

Compact is a **TypeScript-based domain-specific language** designed for writing privacy-preserving smart contracts on Midnight. It combines familiar TypeScript syntax with powerful zero-knowledge proof capabilities.

### Key Features

- **TypeScript-like syntax**: Easy to learn for web developers
- **Type safety**: Strong typing prevents common errors
- **Privacy-first**: Built-in zero-knowledge proof support
- **Modular**: Clean separation between public and private logic
- **Efficient**: Optimized for blockchain execution

## Basic Syntax

### Contract Declaration

Every Compact file defines one contract:

```typescript
contract MyContract {
  // Contract body
}
```

### Comments

```typescript
// Single-line comment

/*
  Multi-line
  comment
*/

/**
 * Documentation comment
 * @param value - Description
 */
```

### Semicolons

Statements end with semicolons:
```typescript
let x = 5;
state count = 0;
```

## Data Types

### Primitive Types

#### Numbers

```typescript
// Unsigned integers
uint        // Unsigned integer (default 256-bit)
uint8       // 8-bit unsigned integer (0 to 255)
uint16      // 16-bit unsigned integer
uint32      // 32-bit unsigned integer
uint64      // 64-bit unsigned integer
uint128     // 128-bit unsigned integer
uint256     // 256-bit unsigned integer

// Signed integers
int         // Signed integer (default 256-bit)
int8        // 8-bit signed integer (-128 to 127)
int16       // 16-bit signed integer
int32       // 32-bit signed integer
int64       // 64-bit signed integer
int128      // 128-bit signed integer
int256      // 256-bit signed integer
```

Examples:
```typescript
state counter: uint;
state temperature: int;
state age: uint8;
```

#### Boolean

```typescript
bool        // true or false

state isActive: bool;
state hasPermission: bool;
```

#### String

```typescript
string      // UTF-8 encoded text

state name: string;
state message: string;
```

#### Bytes

```typescript
bytes       // Dynamic byte array
bytes32     // Fixed 32-byte array
bytes64     // Fixed 64-byte array

state hash: bytes32;
state signature: bytes64;
state data: bytes;
```

#### Address

```typescript
address     // Blockchain address (20 bytes)

state owner: address;
state recipient: address;
```

### Complex Types

#### Arrays

Fixed-size arrays:
```typescript
uint[5]          // Array of 5 uints
address[10]      // Array of 10 addresses
string[3]        // Array of 3 strings

state scores: uint[10];
state winners: address[3];
```

Dynamic arrays:
```typescript
uint[]           // Dynamic array of uints
address[]        // Dynamic array of addresses

state participants: address[];
state values: uint[];
```

#### Mappings

Key-value storage:
```typescript
Map<K, V>        // Key type K, Value type V

state balances: Map<address, uint>;
state allowances: Map<address, Map<address, uint>>;
state usernames: Map<address, string>;
```

Accessing mappings:
```typescript
// Set value
balances[account] = 1000;

// Get value (with default)
let balance = balances[account] || 0;

// Check existence
if (balances[account] != null) {
  // Key exists
}
```

#### Structs

Custom data structures:
```typescript
struct Person {
  name: string;
  age: uint8;
  wallet: address;
}

struct Transaction {
  from: address;
  to: address;
  amount: uint;
  timestamp: uint;
}

state owner: Person;
state transactions: Transaction[];
```

#### Enums

Named constants:
```typescript
enum Status {
  Pending,
  Active,
  Completed,
  Cancelled
}

state currentStatus: Status;

entry updateStatus(newStatus: Status) {
  currentStatus = newStatus;
}
```

### Type Conversions

```typescript
// Explicit conversion
let a: uint16 = 100;
let b: uint32 = uint32(a);

// String to bytes
let text: string = "hello";
let data: bytes = bytes(text);

// Address to bytes
let addr: address = msg.sender;
let addrBytes: bytes32 = bytes32(addr);
```

## Contract Structure

A typical contract includes:

```typescript
contract MyContract {
  // 1. State variables
  state owner: address;
  state balance: uint;

  // 2. Entry points (public functions)
  entry initialize(initialOwner: address) {
    owner = initialOwner;
  }

  entry deposit(amount: uint) {
    balance = balance + amount;
  }

  // 3. Witness functions (private/ZK)
  witness proofOfBalance(threshold: uint): proof {
    assert(balance >= threshold, "Insufficient balance");
    return proof;
  }

  // 4. Internal helper functions
  function isOwner(account: address): bool {
    return account == owner;
  }
}
```

## State Variables

State variables persist across transactions:

```typescript
state variableName: Type;
```

### Visibility

All state variables are private by default (not directly accessible externally):

```typescript
state privateData: uint;      // Private to contract
```

Access via entry points:
```typescript
entry getData(): uint {
  return privateData;
}
```

### Initialization

State can be initialized in an entry point:
```typescript
contract MyContract {
  state count: uint;
  state initialized: bool;

  entry initialize() {
    count = 0;
    initialized = true;
  }
}
```

### Multiple State Variables

```typescript
state name: string;
state symbol: string;
state decimals: uint8;
state totalSupply: uint;
state balances: Map<address, uint>;
state allowances: Map<address, Map<address, uint>>;
```

## Entry Points

Entry points are public functions callable from outside:

```typescript
entry functionName(param1: Type1, param2: Type2): ReturnType {
  // Function body
}
```

### Without Return Value

```typescript
entry setName(newName: string) {
  name = newName;
}

entry increment() {
  count = count + 1;
}
```

### With Return Value

```typescript
entry getName(): string {
  return name;
}

entry getBalance(account: address): uint {
  return balances[account] || 0;
}
```

### With Multiple Parameters

```typescript
entry transfer(to: address, amount: uint): bool {
  let senderBalance = balances[msg.sender] || 0;
  
  if (senderBalance < amount) {
    return false;
  }

  balances[msg.sender] = senderBalance - amount;
  balances[to] = (balances[to] || 0) + amount;
  
  return true;
}
```

### Special Variables

```typescript
msg.sender      // Address of transaction sender
msg.value       // Amount sent with transaction (if applicable)
block.timestamp // Current block timestamp
block.number    // Current block number
```

Example usage:
```typescript
entry onlyOwner() {
  if (msg.sender != owner) {
    // Revert or return
    return;
  }
  // Owner-only logic
}
```

## Witness Functions

Witness functions execute **off-chain** to generate zero-knowledge proofs:

```typescript
witness functionName(params): proof {
  // Off-chain computation
  // Access to private data
  assert(condition, "Error message");
  return proof;
}
```

### Basic Witness

```typescript
witness verifyAge(age: uint): proof {
  assert(age >= 18, "Must be 18 or older");
  return proof;
}
```

### Using in Entry Points

```typescript
entry verifyAdult(zkProof: proof): bool {
  // Verify the proof on-chain
  // Proof confirms age >= 18 without revealing exact age
  return true;
}
```

### Complex Witness Example

```typescript
witness transferProof(
  sender: address,
  recipient: address,
  amount: uint
): proof {
  // Off-chain: get private balance
  let senderBalance = privateBalances[sender];
  
  // Generate proof of sufficient balance
  assert(senderBalance >= amount, "Insufficient funds");
  assert(amount > 0, "Amount must be positive");
  
  return proof;
}

entry privateTransfer(
  recipient: address,
  amount: uint,
  zkProof: proof
): bool {
  // On-chain: verify proof and execute
  // Balance details remain private
  return true;
}
```

## Control Flow

### If Statements

```typescript
if (condition) {
  // code
}

if (condition) {
  // code
} else {
  // code
}

if (condition1) {
  // code
} else if (condition2) {
  // code
} else {
  // code
}
```

### For Loops

```typescript
// Fixed iterations
for (let i = 0; i < 10; i = i + 1) {
  // code
}

// Array iteration
let items: uint[] = [1, 2, 3, 4, 5];
for (let i = 0; i < items.length; i = i + 1) {
  let item = items[i];
  // process item
}
```

### While Loops

```typescript
while (condition) {
  // code
}

let count = 0;
while (count < 10) {
  count = count + 1;
}
```

### Return Statements

```typescript
entry checkValue(value: uint): string {
  if (value > 100) {
    return "High";
  } else if (value > 50) {
    return "Medium";
  } else {
    return "Low";
  }
}
```

### Assert Statements

Used primarily in witness functions:
```typescript
assert(condition, "Error message");

witness validateAmount(amount: uint): proof {
  assert(amount > 0, "Amount must be positive");
  assert(amount <= maxAmount, "Amount exceeds maximum");
  return proof;
}
```

## Operators

### Arithmetic Operators

```typescript
+       // Addition
-       // Subtraction
*       // Multiplication
/       // Division
%       // Modulo (remainder)

let a = 10 + 5;     // 15
let b = 10 - 5;     // 5
let c = 10 * 5;     // 50
let d = 10 / 5;     // 2
let e = 10 % 3;     // 1
```

### Comparison Operators

```typescript
==      // Equal
!=      // Not equal
<       // Less than
>       // Greater than
<=      // Less than or equal
>=      // Greater than or equal

if (balance >= 100) {
  // ...
}
```

### Logical Operators

```typescript
&&      // AND
||      // OR
!       // NOT

if (isActive && balance > 0) {
  // ...
}

if (isOwner || isAdmin) {
  // ...
}

if (!isBlacklisted) {
  // ...
}
```

### Assignment Operators

```typescript
=       // Assign
+=      // Add and assign
-=      // Subtract and assign
*=      // Multiply and assign
/=      // Divide and assign

count = count + 1;
// Or shorter:
count += 1;

balance -= amount;
```

### Bitwise Operators

```typescript
&       // AND
|       // OR
^       // XOR
~       // NOT
<<      // Left shift
>>      // Right shift

let flags = 0b1010;
let mask = 0b0011;
let result = flags & mask;  // 0b0010
```

## Best Practices

### 1. Naming Conventions

```typescript
// Contracts: PascalCase
contract MyToken { }

// State variables: camelCase
state totalSupply: uint;
state userBalance: Map<address, uint>;

// Entry points: camelCase
entry transferTokens(to: address, amount: uint) { }

// Constants: UPPER_SNAKE_CASE (if supported)
const MAX_SUPPLY: uint = 1000000;
```

### 2. State Management

```typescript
// ✅ Good: Initialize state properly
entry initialize() {
  totalSupply = 0;
  owner = msg.sender;
}

// ❌ Bad: Uninitialized state
entry useBalance() {
  balance = balance + 1;  // What if balance was never set?
}
```

### 3. Input Validation

```typescript
// ✅ Good: Validate inputs
entry transfer(to: address, amount: uint): bool {
  if (amount == 0) {
    return false;
  }
  if (to == address(0)) {
    return false;
  }
  // ... rest of logic
}

// ❌ Bad: No validation
entry transfer(to: address, amount: uint) {
  balances[to] += amount;  // What if amount is 0? What if to is invalid?
}
```

### 4. Safe Math

```typescript
// ✅ Good: Check for overflows
entry safeAdd(a: uint, b: uint): uint {
  let c = a + b;
  assert(c >= a, "Overflow");
  return c;
}

// ⚠️ Be aware: Unchecked math
entry unsafeAdd(a: uint, b: uint): uint {
  return a + b;  // Could overflow
}
```

### 5. Access Control

```typescript
// ✅ Good: Implement access control
state owner: address;

function onlyOwner(): bool {
  return msg.sender == owner;
}

entry privilegedAction() {
  if (!onlyOwner()) {
    return;  // Or revert
  }
  // Owner-only logic
}

// ❌ Bad: No access control
entry dangerousAction() {
  // Anyone can call this!
}
```

### 6. Gas Efficiency

```typescript
// ✅ Good: Minimize state changes
entry efficientUpdate(values: uint[]) {
  let sum = 0;
  for (let i = 0; i < values.length; i++) {
    sum += values[i];
  }
  total = sum;  // Single state write
}

// ❌ Bad: Multiple state writes
entry inefficientUpdate(values: uint[]) {
  for (let i = 0; i < values.length; i++) {
    total += values[i];  // State write in loop!
  }
}
```

### 7. Error Handling

```typescript
// ✅ Good: Return success status
entry transfer(to: address, amount: uint): bool {
  if (balances[msg.sender] < amount) {
    return false;  // Clear failure indication
  }
  // Transfer logic
  return true;
}

// ❌ Bad: Silent failures
entry transfer(to: address, amount: uint) {
  // What happens if balance is insufficient?
  balances[msg.sender] -= amount;
}
```

### 8. Privacy Considerations

```typescript
// ✅ Good: Use witness for private data
witness balanceProof(threshold: uint): proof {
  assert(balance >= threshold, "Below threshold");
  return proof;
}

entry hasMinBalance(threshold: uint, zkProof: proof): bool {
  // Verify without revealing actual balance
  return true;
}

// ❌ Bad: Exposing private data publicly
entry getPrivateBalance(): uint {
  return secretBalance;  // Now it's not secret!
}
```

## Summary

Compact provides a powerful, type-safe language for building privacy-preserving smart contracts:

- **TypeScript-inspired syntax**: Familiar and easy to learn
- **Strong typing**: Prevents common errors
- **Privacy-first**: Zero-knowledge proofs built in
- **Flexible**: Supports complex data structures
- **Efficient**: Optimized for blockchain execution

## Next Steps

1. **Practice**: Write simple contracts using these basics
2. **Study examples**: Review the examples directory
3. **Advanced topics**: Learn about advanced ZK patterns
4. **Build**: Create your own DApp

## Resources

- [Compact Language Reference](https://docs.midnight.network/develop/reference/compact/)
- [Smart Contract Patterns](https://docs.midnight.network/guides/patterns/)
- [ZK Proof Guide](https://docs.midnight.network/guides/zk-proofs/)
- [Security Best Practices](https://docs.midnight.network/security/)

---

Happy coding in Compact! 🌙
