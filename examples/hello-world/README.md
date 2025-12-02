# Hello World Example

This is the simplest Midnight smart contract example, demonstrating basic contract structure.

## What You'll Learn

- Basic Compact contract syntax
- State variables
- Entry points (public functions)
- How to interact with contract state

## Contract Overview

The `HelloWorld` contract maintains a single state variable `message` that can be:
- Set using `setMessage()`
- Retrieved using `getMessage()`
- Initialized using `initialize()`

## Code Structure

```typescript
contract HelloWorld {
  state message: string;        // Persistent storage
  
  entry setMessage(msg: string) // Public function
  entry getMessage(): string     // Public query function
  entry initialize(...)          // Initialization function
}
```

## How to Use

### Compile the Contract

```bash
compactc hello_world.compact
```

This generates:
- Circuit files for zero-knowledge proofs
- Deployment artifacts
- Type definitions

### Deploy to Testnet

```bash
midnight-cli deploy hello_world
```

### Interact with the Contract

```typescript
// Initialize the contract
await contract.initialize("Hello, Midnight!");

// Set a new message
await contract.setMessage("Hello, World!");

// Get the current message
const msg = await contract.getMessage();
console.log(msg); // "Hello, World!"
```

## Key Concepts

### State Variables
- Persistent storage that survives across transactions
- Can be any valid Compact type
- Automatically maintained by the blockchain

### Entry Points
- Functions that can be called from outside the contract
- Declared with the `entry` keyword
- Can modify state or return values

### Transactions
- Every entry point call is a transaction
- Transactions are atomic (all-or-nothing)
- Gas fees apply (paid in DUST tokens)

## Next Steps

1. Compile and deploy this contract
2. Interact with it using the Midnight CLI or SDK
3. Modify the contract to add more functionality
4. Move on to the Counter example for more complex state management

## Related Examples

- **Counter**: State management with increment/decrement
- **Token**: More complex state with mappings
- **Private Transfer**: Zero-knowledge proof integration
