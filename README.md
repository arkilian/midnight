# Midnight Programming Learning Guide 🌙

Welcome to your comprehensive guide to learning Midnight blockchain development! This repository provides tutorials, examples, and resources to help you master the Compact programming language and build privacy-preserving decentralized applications.

## 📚 Table of Contents

- [What is Midnight?](#what-is-midnight)
- [What is Compact?](#what-is-compact)
- [Getting Started](#getting-started)
- [Learning Path](#learning-path)
- [Examples](#examples)
- [Resources](#resources)
- [Community](#community)

## 🌟 What is Midnight?

Midnight is a privacy-first blockchain developed as a sidechain to Cardano, designed for compliant and private decentralized applications. Key features include:

- **Zero-Knowledge Proofs**: Validate transactions without exposing sensitive data
- **Selective Disclosure**: Users control what information is shared
- **Hybrid Public/Private Data**: Public data is auditable; private data stays on user devices
- **Native Tokens**: NIGHT (network security) and DUST (contract execution)

Midnight enables developers to create applications that maintain transparency on a public blockchain while keeping transactional and business logic confidential.

## 💻 What is Compact?

**Compact** is the TypeScript-based smart contract language for Midnight. Unlike traditional blockchain languages where all data goes on-chain, Compact leverages zero-knowledge proofs to keep private information off-chain while still proving its validity.

### Key Features:
- **TypeScript-based syntax**: Familiar to web developers
- **Privacy-first**: Private data never touches the blockchain
- **Zero-Knowledge Proofs**: Cryptographic proofs validate logic without revealing data
- **Modular design**: Separation between public and private contract logic

## 🚀 Getting Started

### Prerequisites

Before you begin, ensure you have:
- **Node.js v18+** installed
- Basic understanding of TypeScript/JavaScript
- Familiarity with blockchain concepts (helpful but not required)

### Installation

1. **Clone this repository**:
   ```bash
   git clone https://github.com/arkilian/midnight.git
   cd midnight
   ```

2. **Install the Midnight development tools**:
   Visit the [Midnight Developer Hub](https://midnight.network/developer-hub) to download:
   - Compact compiler
   - Midnight CLI tools
   - Development SDK

3. **Set up your environment**:
   ```bash
   npm install
   ```

### Your First Compact Contract

Create a file named `hello_world.compact`:

```typescript
contract HelloWorld {
  state message: string;

  entry setMessage(msg: string) {
    message = msg;
  }

  entry getMessage(): string {
    return message;
  }
}
```

Compile it:
```bash
compactc hello_world.compact
```

## 📖 Learning Path

Follow this structured path to master Midnight development:

### 1. **Fundamentals** (Start Here)
   - [ ] Understand blockchain basics
   - [ ] Learn about zero-knowledge proofs
   - [ ] Study the Compact language syntax
   - [ ] Set up your development environment

### 2. **Basic Contracts**
   - [ ] Write a Hello World contract
   - [ ] Create a simple counter contract
   - [ ] Build a basic storage contract
   - [ ] Test your contracts locally

### 3. **Intermediate Concepts**
   - [ ] Work with state management
   - [ ] Implement witnesses for private data
   - [ ] Handle events and transactions
   - [ ] Deploy to testnet

### 4. **Advanced Topics**
   - [ ] Build privacy-preserving tokens
   - [ ] Create complex DApps
   - [ ] Implement selective disclosure
   - [ ] Optimize zero-knowledge circuits

### 5. **Real-World Applications**
   - [ ] Private voting systems
   - [ ] Confidential DeFi protocols
   - [ ] Privacy-preserving DAOs
   - [ ] Compliance-friendly smart contracts

## 💡 Examples

Check the `examples/` directory for practical code samples:

- **Hello World**: Basic contract structure
- **Counter**: State management
- **Token**: Simple token implementation
- **Private Transfer**: Zero-knowledge proof integration
- **Voting**: Privacy-preserving voting system

## 📚 Resources

### Official Documentation
- [Midnight Developer Academy](https://docs.midnight.network/academy/) - Comprehensive learning modules
- [Compact Language Reference](https://docs.midnight.network/develop/reference/compact/) - Language specification
- [Writing Contracts Guide](https://docs.midnight.network/compact/writing) - How to write and deploy
- [Developer Hub](https://midnight.network/developer-hub) - Tools, guides, and quick-starts

### Community Resources
- [Compact by Example](https://github.com/Olanetsoft/compact-by-example) - Community examples
- [Midnight GitHub Examples](https://github.com/MeshJS/midnight) - Integration tools
- [Developer Blog](https://midnight.network/blog/compact-the-smart-contract-language-of-midnight) - Latest updates

### Video Tutorials
- [Getting Started with Midnight](https://www.youtube.com/watch?v=dWkYH8CfTko) - Workshop
- [Intro to Compact](https://www.youtube.com/watch?v=QCuO0--CO14) - Language overview

### Articles & Guides
- [Midnight Smart Contract Development Guide](https://webisoft.com/articles/midnight-smart-contract-development/)
- [Learning Compact from Ground Up](https://dev.to/devsofmidnight/learning-web3-from-the-ground-up-smart-contracts-and-the-compact-language-1meb)

## 👥 Community

Join the Midnight developer community:

- **Discord**: [Join the Midnight Network Discord](https://discord.com/invite/midnightntwrk)
- **Forums**: Engage in technical discussions
- **Twitter**: Follow [@MidnightNtwrk](https://twitter.com/MidnightNtwrk)
- **Developer Diaries**: Share your journey
- **Hackathons**: Participate in events for rewards

## 🛠️ Project Structure

```
midnight/
├── examples/           # Code examples and tutorials
│   ├── hello-world/
│   ├── counter/
│   ├── token/
│   └── private-transfer/
├── docs/              # Additional documentation
│   ├── getting-started.md
│   ├── compact-basics.md
│   └── advanced-topics.md
├── tutorials/         # Step-by-step tutorials
└── README.md          # This file
```

## 🔐 Security & Privacy

When working with Midnight:
- Never commit private keys or sensitive data
- Test thoroughly on testnet before mainnet
- Follow security best practices for smart contracts
- Understand the privacy implications of your contracts

## 🤝 Contributing

Contributions are welcome! If you have examples, tutorials, or improvements:
1. Fork this repository
2. Create a feature branch
3. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🎯 Next Steps

1. **Complete the fundamentals** section of the learning path
2. **Run the Hello World example** from the examples directory
3. **Join the Discord community** to connect with other developers
4. **Build your first DApp** using Compact
5. **Share your experience** with the community

Happy coding! 🚀

---

*For questions or support, join our [Discord community](https://discord.com/invite/midnightntwrk) or check the [official documentation](https://docs.midnight.network/academy/).*
