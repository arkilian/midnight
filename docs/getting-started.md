# Getting Started with Midnight Development

This guide will help you set up your development environment and create your first Midnight smart contract using the Compact language.

## Prerequisites

Before you begin, ensure you have:

### Required Software
- **Node.js v18 or higher**: [Download Node.js](https://nodejs.org/)
- **npm or yarn**: Package manager (comes with Node.js)
- **Git**: Version control
- **A code editor**: VS Code, Sublime Text, or your preferred editor

### Recommended Knowledge
- Basic TypeScript/JavaScript
- Understanding of blockchain concepts
- Command line familiarity

### Helpful (But Not Required)
- Smart contract development experience
- Cryptography basics
- Zero-knowledge proof concepts

## Installation Steps

### 1. Install Node.js

Check if Node.js is installed:
```bash
node --version
# Should output v18.x.x or higher
```

If not installed, download from [nodejs.org](https://nodejs.org/)

### 2. Get Midnight Development Tools

Visit the [Midnight Developer Hub](https://midnight.network/developer-hub) to download the required tools.

#### Compact Compiler

Download and install the Compact compiler from the official Midnight website. Follow the installation instructions provided in the [official documentation](https://docs.midnight.network/install/).

```bash
# Installation command will vary based on your OS
# Check the official docs for the latest instructions:
# https://docs.midnight.network/install/

# Example (verify with official docs):
npm install -g @midnight-ntwrk/compact-compiler
```

#### Midnight CLI

Command-line tools for deployment and interaction:
```bash
# Check official documentation for installation
npm install -g @midnight-ntwrk/cli
```

#### Midnight SDK

Development libraries for building DApps:
```bash
npm install @midnight-ntwrk/sdk
```

**Note**: Always refer to the [official installation guide](https://docs.midnight.network/install/) for the most up-to-date installation instructions.

### 3. Set Up Your Workspace

Create a new project directory:
```bash
mkdir my-midnight-project
cd my-midnight-project
npm init -y
```

Install dependencies:
```bash
npm install @midnight-ntwrk/sdk
npm install --save-dev @midnight-ntwrk/compact-compiler
```

### 4. Configure Your Environment

Create a `.env` file:
```bash
# Network configuration
MIDNIGHT_NETWORK=testnet
MIDNIGHT_RPC_URL=https://testnet-rpc.midnight.network

# Your wallet configuration
WALLET_MNEMONIC="your twelve word mnemonic phrase here"

# API keys (if required)
MIDNIGHT_API_KEY=your_api_key_here
```

⚠️ **Security Note**: Never commit `.env` files with real credentials!

## Your First Contract

### Step 1: Create a Contract File

Create `hello_world.compact`:
```typescript
contract HelloWorld {
  state message: string;

  entry initialize(msg: string) {
    message = msg;
  }

  entry setMessage(msg: string) {
    message = msg;
  }

  entry getMessage(): string {
    return message;
  }
}
```

### Step 2: Compile the Contract

```bash
compactc hello_world.compact
```

This generates:
- `hello_world.json` - Contract artifact
- `hello_world_circuit.json` - ZK circuit definitions
- `hello_world.d.ts` - TypeScript type definitions

### Step 3: Deploy to Testnet

```bash
midnight-cli deploy hello_world.json \
  --network testnet \
  --init-args '["Hello, Midnight!"]'
```

You'll receive:
- Contract address
- Transaction hash
- Deployment confirmation

### Step 4: Interact with Your Contract

Create `interact.js`:
```javascript
const { MidnightSDK } = require('@midnight-ntwrk/sdk');

async function main() {
  // Initialize SDK
  const sdk = await MidnightSDK.connect({
    network: 'testnet',
    privateKey: process.env.PRIVATE_KEY
  });

  // Load your contract
  const contract = await sdk.loadContract({
    address: 'your_contract_address',
    abi: require('./hello_world.json')
  });

  // Call a function
  const message = await contract.getMessage();
  console.log('Current message:', message);

  // Send a transaction
  const tx = await contract.setMessage('Hello, Web3!');
  await tx.wait();
  console.log('Message updated!');
}

main();
```

Run it:
```bash
node interact.js
```

## Development Workflow

### Typical Development Cycle

1. **Write Contract** (`*.compact`)
   ```bash
   vim my_contract.compact
   ```

2. **Compile**
   ```bash
   compactc my_contract.compact
   ```

3. **Test Locally**
   ```bash
   midnight-test my_contract.json
   ```

4. **Deploy to Testnet**
   ```bash
   midnight-cli deploy my_contract.json --network testnet
   ```

5. **Verify Deployment**
   ```bash
   midnight-cli verify <contract-address>
   ```

6. **Interact & Debug**
   ```bash
   midnight-cli call <contract-address> getState
   ```

### Using a Local Test Network

Start a local Midnight node:
```bash
midnight-node --dev
```

Deploy to local:
```bash
midnight-cli deploy my_contract.json \
  --network local \
  --rpc http://localhost:8545
```

## Project Structure

Recommended directory structure:
```
my-midnight-project/
├── contracts/           # Your .compact files
│   ├── HelloWorld.compact
│   ├── Token.compact
│   └── MyDApp.compact
├── build/              # Compiled artifacts
│   ├── HelloWorld.json
│   └── circuits/
├── scripts/            # Deployment & interaction scripts
│   ├── deploy.js
│   └── interact.js
├── test/               # Test files
│   └── HelloWorld.test.js
├── src/                # Frontend code (if building a DApp)
├── .env               # Environment variables (gitignored)
├── .gitignore
├── package.json
└── README.md
```

## Getting Test Tokens

### For Testnet Development

1. **Visit the Faucet**:
   - Go to [testnet-faucet.midnight.network](https://testnet-faucet.midnight.network)
   - Enter your wallet address
   - Request test DUST tokens

2. **Via Discord**:
   - Join [Midnight Discord](https://discord.com/invite/midnightntwrk)
   - Use the `#faucet` channel
   - Request tokens with your address

3. **Via CLI**:
   ```bash
   midnight-cli faucet request --address <your-address>
   ```

## Common Commands

### Compilation
```bash
# Compile a single contract
compactc contract.compact

# Compile with optimization
compactc contract.compact --optimize

# Compile all contracts in directory
compactc contracts/*.compact
```

### Deployment
```bash
# Deploy to testnet
midnight-cli deploy contract.json --network testnet

# Deploy with initialization args
midnight-cli deploy contract.json --init-args '[arg1, arg2]'

# Deploy and verify
midnight-cli deploy contract.json --verify
```

### Interaction
```bash
# Call a read-only function
midnight-cli call <address> functionName [args]

# Send a transaction
midnight-cli send <address> functionName [args]

# Check contract state
midnight-cli state <address>

# View transaction history
midnight-cli history <address>
```

### Account Management
```bash
# Create new account
midnight-cli account create

# Import account
midnight-cli account import --mnemonic "your words"

# List accounts
midnight-cli account list

# Check balance
midnight-cli balance <address>
```

## Development Tools

### VS Code Extensions
- **Compact Language Support**: Syntax highlighting for `.compact` files
- **Midnight Snippets**: Code snippets for common patterns
- **Solidity**: Helps with similar syntax patterns

### Browser Extensions
- **Midnight Wallet**: Browser wallet for testnet
- **MetaMask**: Can be configured for Midnight (if compatible)

### Testing Tools
- **Midnight Test Framework**: Built-in testing tools
- **Chai/Mocha**: JavaScript testing (for integration tests)

## Debugging Tips

### Enable Verbose Logging
```bash
export MIDNIGHT_LOG_LEVEL=debug
compactc contract.compact
```

### Check Contract Deployment
```bash
midnight-cli inspect <contract-address>
```

### View Transaction Details
```bash
midnight-cli tx <transaction-hash>
```

### Common Issues

**Issue**: "Insufficient DUST for transaction"
```bash
# Solution: Request test tokens from faucet
midnight-cli faucet request
```

**Issue**: "Contract compilation failed"
```bash
# Solution: Check syntax, enable verbose mode
compactc contract.compact --verbose
```

**Issue**: "RPC connection failed"
```bash
# Solution: Check network configuration
midnight-cli config show
midnight-cli config set rpc_url https://testnet-rpc.midnight.network
```

## Next Steps

Now that you have your environment set up:

1. ✅ **Complete the Hello World example**
2. 📚 **Work through the examples directory**
   - Counter contract (state management)
   - Token contract (mappings and transfers)
   - Private transfer (zero-knowledge proofs)
3. 🎓 **Take the Midnight Academy courses**
   - [Midnight Developer Academy](https://docs.midnight.network/academy/)
4. 🛠️ **Build your own DApp**
   - Start with a simple use case
   - Add privacy features incrementally
5. 👥 **Join the community**
   - [Discord](https://discord.com/invite/midnightntwrk)
   - [Forums](https://forum.midnight.network)

## Useful Resources

- [Official Documentation](https://docs.midnight.network/)
- [Compact Language Reference](https://docs.midnight.network/develop/reference/compact/)
- [API Documentation](https://docs.midnight.network/api/)
- [Example Projects](https://github.com/MeshJS/midnight)
- [Developer Blog](https://midnight.network/blog/)

## Getting Help

If you encounter issues:

1. **Check Documentation**: [docs.midnight.network](https://docs.midnight.network)
2. **Search Forums**: Someone might have had the same issue
3. **Ask Discord**: Active community support
4. **GitHub Issues**: For SDK/tooling problems
5. **Stack Overflow**: Tag questions with `midnight-blockchain`

## Community Support

- **Discord**: [discord.com/invite/midnightntwrk](https://discord.com/invite/midnightntwrk)
- **Forum**: [forum.midnight.network](https://forum.midnight.network)
- **Twitter**: [@MidnightNtwrk](https://twitter.com/MidnightNtwrk)
- **GitHub**: [github.com/midnight-network](https://github.com/midnight-network)

---

**Congratulations!** 🎉 You're now ready to start developing on Midnight. Happy coding! 🌙
