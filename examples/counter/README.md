# Counter Example

This example demonstrates state management with numeric values in Midnight smart contracts.

## What You'll Learn

- Working with numeric types (`uint`)
- Multiple entry points in a single contract
- State validation and conditional logic
- Query functions that return values

## Contract Overview

The `Counter` contract maintains a numeric counter that supports:
- **Initialization**: Set starting value
- **Increment**: Add 1 to counter
- **Decrement**: Subtract 1 (with underflow protection)
- **IncrementBy**: Add custom amount
- **Reset**: Set to 0
- **GetCount**: Query current value
- **IsZero**: Check if counter is at 0
- **SetCount**: Set to specific value

## Code Structure

```typescript
contract Counter {
  state count: uint;              // Unsigned integer state
  
  entry increment()               // Increase by 1
  entry decrement()               // Decrease by 1 (safe)
  entry incrementBy(amount: uint) // Increase by amount
  entry reset()                   // Set to 0
  entry getCount(): uint          // Query function
}
```

## How to Use

### Compile

```bash
compactc counter.compact
```

### Deploy

```bash
midnight-cli deploy counter --init-value 0
```

### Interact

```typescript
// Initialize counter at 0
await counter.initialize(0);

// Increment operations
await counter.increment();        // count = 1
await counter.increment();        // count = 2
await counter.incrementBy(5);     // count = 7

// Query the count
const value = await counter.getCount();
console.log(value); // 7

// Decrement operations
await counter.decrement();        // count = 6

// Reset
await counter.reset();            // count = 0

// Check if zero
const isZero = await counter.isZero();
console.log(isZero); // true

// Set to specific value
await counter.setCount(100);      // count = 100
```

## Key Concepts

### Numeric Types
- `uint`: Unsigned integer (no negative values)
- Operations: `+`, `-`, `*`, `/`, `%`
- Comparison: `==`, `!=`, `<`, `>`, `<=`, `>=`

### State Validation
The `decrement()` function demonstrates safety checks:
```typescript
entry decrement() {
  if (count > 0) {
    count = count - 1;
  }
}
```
This prevents underflow (going below 0).

### Query Functions
Functions that return values without modifying state:
- Marked with return type (e.g., `: uint`, `: bool`)
- Can be called without creating transactions
- Useful for reading contract state

## Best Practices

1. **Validate State Changes**: Always check bounds before modifying numeric state
2. **Use Appropriate Types**: Choose `uint` for counters, `int` for values that can be negative
3. **Document Behavior**: Clearly document what each function does
4. **Consider Edge Cases**: Handle zero, maximum values, etc.

## Exercises

Try extending this contract:

1. Add a `decrementBy(amount: uint)` function
2. Implement `multiplyBy(factor: uint)`
3. Add maximum value limit (e.g., counter can't exceed 1000)
4. Track total number of increments separately from count value
5. Add an event emission when counter reaches certain milestones

## Next Steps

- **Token Example**: Learn about mappings and more complex state
- **Private Transfer**: Integrate zero-knowledge proofs
- Study the [Compact Language Reference](https://docs.midnight.network/develop/reference/compact/)

## Related Concepts

- **Gas Optimization**: Minimize state changes to reduce costs
- **Atomic Operations**: All state changes in a transaction succeed or fail together
- **State Transitions**: How Midnight ensures consistency across transactions
