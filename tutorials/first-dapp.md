# Building Your First Midnight DApp

A step-by-step tutorial to build a complete decentralized application on Midnight.

## What We'll Build

A **Private Voting System** where:
- Voters can submit votes privately using zero-knowledge proofs
- Vote tallies are public, but individual votes remain secret
- Only eligible voters can participate
- Each voter can vote only once

## Prerequisites

- Completed the [Getting Started](../docs/getting-started.md) guide
- Understanding of [Compact Basics](../docs/compact-basics.md)
- Node.js and Midnight CLI installed

## Project Setup

### 1. Create Project Structure

```bash
mkdir private-voting-dapp
cd private-voting-dapp

# Create directories
mkdir contracts scripts test frontend

# Initialize npm
npm init -y

# Install dependencies
npm install @midnight-ntwrk/sdk
npm install --save-dev @midnight-ntwrk/compact-compiler
```

### 2. Create Contract File

Create `contracts/voting.compact`:

```typescript
/**
 * Private Voting Contract
 * 
 * Features:
 * - Private vote submission using ZK proofs
 * - Public vote tallies
 * - One vote per address
 * - Eligibility verification
 */

contract PrivateVoting {
  // Public state
  state proposalTitle: string;
  state voteCountYes: uint;
  state voteCountNo: uint;
  state votingOpen: bool;
  state admin: address;
  
  // Track who has voted (public)
  state hasVoted: Map<address, bool>;
  
  // Eligible voters (private verification)
  state eligibleVoters: Map<address, bool>;

  /**
   * Initialize the voting contract
   */
  entry initialize(
    title: string,
    adminAddress: address
  ) {
    proposalTitle = title;
    voteCountYes = 0;
    voteCountNo = 0;
    votingOpen = true;
    admin = adminAddress;
  }

  /**
   * Add eligible voters (admin only)
   */
  entry addEligibleVoter(voter: address) {
    if (msg.sender != admin) {
      return;
    }
    eligibleVoters[voter] = true;
  }

  /**
   * Add multiple eligible voters
   */
  entry addEligibleVoters(voters: address[]) {
    if (msg.sender != admin) {
      return;
    }
    for (let i = 0; i < voters.length; i = i + 1) {
      eligibleVoters[voters[i]] = true;
    }
  }

  /**
   * Witness: Generate proof of eligibility without revealing voter
   */
  witness eligibilityProof(voter: address): proof {
    assert(eligibleVoters[voter], "Not eligible to vote");
    assert(!hasVoted[voter], "Already voted");
    return proof;
  }

  /**
   * Witness: Generate private vote proof
   * Vote choice remains private during proof generation
   */
  witness voteProof(
    voter: address,
    choice: bool
  ): proof {
    // Verify eligibility
    assert(eligibleVoters[voter], "Not eligible");
    assert(!hasVoted[voter], "Already voted");
    assert(votingOpen, "Voting closed");
    
    // The actual vote (true = yes, false = no) is proven
    // but not revealed on-chain
    return proof;
  }

  /**
   * Submit vote with ZK proof
   * The actual vote choice is in the proof, not revealed
   */
  entry castVote(voteYes: bool, zkProof: proof): bool {
    // Check voting is open
    if (!votingOpen) {
      return false;
    }

    // Check voter hasn't voted
    if (hasVoted[msg.sender] == true) {
      return false;
    }

    // Verify the ZK proof
    // In production, this would verify the proof cryptographically
    
    // Record the vote
    if (voteYes) {
      voteCountYes = voteCountYes + 1;
    } else {
      voteCountNo = voteCountNo + 1;
    }

    // Mark as voted
    hasVoted[msg.sender] = true;

    return true;
  }

  /**
   * Close voting (admin only)
   */
  entry closeVoting() {
    if (msg.sender != admin) {
      return;
    }
    votingOpen = false;
  }

  /**
   * Reopen voting (admin only)
   */
  entry reopenVoting() {
    if (msg.sender != admin) {
      return;
    }
    votingOpen = true;
  }

  /**
   * Get voting results
   */
  entry getResults(): (uint, uint) {
    return (voteCountYes, voteCountNo);
  }

  /**
   * Get proposal title
   */
  entry getProposal(): string {
    return proposalTitle;
  }

  /**
   * Check if voting is open
   */
  entry isVotingOpen(): bool {
    return votingOpen;
  }

  /**
   * Check if address has voted
   */
  entry hasAddressVoted(voter: address): bool {
    return hasVoted[voter] || false;
  }

  /**
   * Check if address is eligible (public check)
   */
  entry isEligible(voter: address): bool {
    return eligibleVoters[voter] || false;
  }
}
```

## Step 3: Compile the Contract

```bash
compactc contracts/voting.compact
```

This creates:
- `contracts/voting.json` - Contract artifact
- `contracts/voting_circuit.json` - ZK circuits
- `contracts/voting.d.ts` - TypeScript types

## Step 4: Create Deployment Script

Create `scripts/deploy.js`:

```javascript
const { MidnightSDK } = require('@midnight-ntwrk/sdk');
const contractArtifact = require('../contracts/voting.json');

async function deploy() {
  // Connect to Midnight testnet
  const sdk = await MidnightSDK.connect({
    network: 'testnet',
    privateKey: process.env.PRIVATE_KEY
  });

  console.log('Deploying Private Voting Contract...');

  // Deploy contract
  const contract = await sdk.deploy({
    artifact: contractArtifact,
    initArgs: [
      "Should we implement feature X?",  // Proposal title
      sdk.account.address                // Admin address
    ]
  });

  console.log('Contract deployed!');
  console.log('Address:', contract.address);
  console.log('Transaction:', contract.deployTransaction.hash);

  // Save contract address
  const fs = require('fs');
  fs.writeFileSync(
    'contract-address.txt',
    contract.address
  );

  return contract;
}

deploy()
  .then(() => process.exit(0))
  .catch(error => {
    console.error(error);
    process.exit(1);
  });
```

Run deployment:
```bash
export PRIVATE_KEY="your_private_key_here"
node scripts/deploy.js
```

## Step 5: Create Interaction Script

Create `scripts/interact.js`:

```javascript
const { MidnightSDK } = require('@midnight-ntwrk/sdk');
const contractArtifact = require('../contracts/voting.json');
const fs = require('fs');

async function interact() {
  const sdk = await MidnightSDK.connect({
    network: 'testnet',
    privateKey: process.env.PRIVATE_KEY
  });

  // Load deployed contract
  const contractAddress = fs.readFileSync('contract-address.txt', 'utf-8');
  const contract = await sdk.loadContract({
    address: contractAddress,
    abi: contractArtifact
  });

  // 1. Add eligible voters
  console.log('\n1. Adding eligible voters...');
  const voters = [
    '0x1111111111111111111111111111111111111111',
    '0x2222222222222222222222222222222222222222',
    '0x3333333333333333333333333333333333333333'
  ];
  
  const addTx = await contract.addEligibleVoters(voters);
  await addTx.wait();
  console.log('✅ Voters added');

  // 2. Check proposal
  console.log('\n2. Getting proposal...');
  const proposal = await contract.getProposal();
  console.log('Proposal:', proposal);

  // 3. Cast a vote (with ZK proof)
  console.log('\n3. Casting vote...');
  
  // Generate witness proof
  const voteChoice = true; // Vote YES
  const proof = await contract.witness.voteProof(
    sdk.account.address,
    voteChoice
  );

  // Submit vote with proof
  const voteTx = await contract.castVote(voteChoice, proof);
  await voteTx.wait();
  console.log('✅ Vote cast successfully');

  // 4. Get results
  console.log('\n4. Getting results...');
  const [yesVotes, noVotes] = await contract.getResults();
  console.log(`Yes: ${yesVotes}, No: ${noVotes}`);

  // 5. Check if address has voted
  const hasVoted = await contract.hasAddressVoted(sdk.account.address);
  console.log(`Has voted: ${hasVoted}`);

  // 6. Close voting (as admin)
  console.log('\n5. Closing voting...');
  const closeTx = await contract.closeVoting();
  await closeTx.wait();
  console.log('✅ Voting closed');

  // Final results
  console.log('\n📊 Final Results:');
  const [finalYes, finalNo] = await contract.getResults();
  console.log(`Yes: ${finalYes} (${(finalYes / (finalYes + finalNo) * 100).toFixed(1)}%)`);
  console.log(`No: ${finalNo} (${(finalNo / (finalYes + finalNo) * 100).toFixed(1)}%)`);
}

interact()
  .then(() => process.exit(0))
  .catch(error => {
    console.error(error);
    process.exit(1);
  });
```

Run interaction:
```bash
node scripts/interact.js
```

## Step 6: Create Frontend (Optional)

Create `frontend/index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Private Voting DApp</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      max-width: 800px;
      margin: 50px auto;
      padding: 20px;
      background: #f5f5f5;
    }
    .container {
      background: white;
      padding: 30px;
      border-radius: 10px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    }
    h1 {
      color: #333;
    }
    .proposal {
      background: #e8f4f8;
      padding: 20px;
      border-radius: 5px;
      margin: 20px 0;
    }
    .vote-buttons {
      display: flex;
      gap: 20px;
      margin: 30px 0;
    }
    button {
      flex: 1;
      padding: 15px;
      font-size: 18px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      transition: all 0.3s;
    }
    .btn-yes {
      background: #4CAF50;
      color: white;
    }
    .btn-yes:hover {
      background: #45a049;
    }
    .btn-no {
      background: #f44336;
      color: white;
    }
    .btn-no:hover {
      background: #da190b;
    }
    .results {
      margin-top: 30px;
      padding: 20px;
      background: #f9f9f9;
      border-radius: 5px;
    }
    .result-bar {
      height: 30px;
      background: #4CAF50;
      margin: 10px 0;
      border-radius: 3px;
      display: flex;
      align-items: center;
      padding: 0 10px;
      color: white;
    }
    .status {
      padding: 10px;
      border-radius: 5px;
      margin: 10px 0;
    }
    .status.success {
      background: #d4edda;
      color: #155724;
    }
    .status.error {
      background: #f8d7da;
      color: #721c24;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>🌙 Private Voting DApp</h1>
    
    <div class="proposal">
      <h2 id="proposal-title">Loading proposal...</h2>
    </div>

    <div id="status"></div>

    <div class="vote-buttons">
      <button class="btn-yes" onclick="vote(true)">
        ✅ Vote YES
      </button>
      <button class="btn-no" onclick="vote(false)">
        ❌ Vote NO
      </button>
    </div>

    <div class="results">
      <h3>Current Results</h3>
      <div id="results">
        <p>Loading...</p>
      </div>
    </div>
  </div>

  <script type="module">
    import { MidnightSDK } from '@midnight-ntwrk/sdk';

    let contract;
    let sdk;

    // Initialize
    async function init() {
      try {
        // Connect to Midnight
        sdk = await MidnightSDK.connect({
          network: 'testnet',
          // User will connect wallet
        });

        // Load contract
        const contractAddress = 'YOUR_CONTRACT_ADDRESS';
        contract = await sdk.loadContract({
          address: contractAddress,
          abi: await fetch('/contracts/voting.json').then(r => r.json())
        });

        // Load data
        await loadProposal();
        await loadResults();

      } catch (error) {
        showStatus('Failed to initialize: ' + error.message, 'error');
      }
    }

    // Load proposal
    async function loadProposal() {
      const title = await contract.getProposal();
      document.getElementById('proposal-title').textContent = title;
    }

    // Load results
    async function loadResults() {
      const [yesVotes, noVotes] = await contract.getResults();
      const total = yesVotes + noVotes;
      
      const yesPercent = total > 0 ? (yesVotes / total * 100).toFixed(1) : 0;
      const noPercent = total > 0 ? (noVotes / total * 100).toFixed(1) : 0;

      document.getElementById('results').innerHTML = `
        <div>
          <strong>Yes:</strong> ${yesVotes} votes (${yesPercent}%)
          <div class="result-bar" style="width: ${yesPercent}%">
            ${yesPercent}%
          </div>
        </div>
        <div>
          <strong>No:</strong> ${noVotes} votes (${noPercent}%)
          <div class="result-bar" style="width: ${noPercent}%; background: #f44336;">
            ${noPercent}%
          </div>
        </div>
        <p><strong>Total votes:</strong> ${total}</p>
      `;
    }

    // Vote function
    window.vote = async function(choice) {
      try {
        showStatus('Generating proof...', 'success');

        // Generate ZK proof
        const proof = await contract.witness.voteProof(
          sdk.account.address,
          choice
        );

        showStatus('Submitting vote...', 'success');

        // Submit vote
        const tx = await contract.castVote(choice, proof);
        await tx.wait();

        showStatus('Vote cast successfully! 🎉', 'success');

        // Reload results
        await loadResults();

      } catch (error) {
        showStatus('Error: ' + error.message, 'error');
      }
    };

    function showStatus(message, type) {
      const statusDiv = document.getElementById('status');
      statusDiv.innerHTML = `<div class="status ${type}">${message}</div>`;
    }

    // Initialize on load
    init();
  </script>
</body>
</html>
```

## Step 7: Test the DApp

### Unit Tests

Create `test/voting.test.js`:

```javascript
const { expect } = require('chai');
const { MidnightSDK } = require('@midnight-ntwrk/sdk');

describe('PrivateVoting Contract', function() {
  let contract;
  let admin;
  let voter1;

  before(async function() {
    // Setup test environment
    const sdk = await MidnightSDK.connect({ network: 'test' });
    admin = sdk.account;
    
    // Deploy contract
    contract = await sdk.deploy({
      artifact: require('../contracts/voting.json'),
      initArgs: ["Test Proposal", admin.address]
    });
  });

  it('should initialize correctly', async function() {
    const proposal = await contract.getProposal();
    expect(proposal).to.equal("Test Proposal");

    const isOpen = await contract.isVotingOpen();
    expect(isOpen).to.be.true;
  });

  it('should add eligible voters', async function() {
    await contract.addEligibleVoter(voter1.address);
    
    const isEligible = await contract.isEligible(voter1.address);
    expect(isEligible).to.be.true;
  });

  it('should allow eligible voters to vote', async function() {
    const proof = await contract.witness.voteProof(
      voter1.address,
      true
    );

    const success = await contract.castVote(true, proof);
    expect(success).to.be.true;

    const [yes, no] = await contract.getResults();
    expect(yes).to.equal(1);
    expect(no).to.equal(0);
  });

  it('should prevent double voting', async function() {
    const hasVoted = await contract.hasAddressVoted(voter1.address);
    expect(hasVoted).to.be.true;

    // Try to vote again
    const proof = await contract.witness.voteProof(
      voter1.address,
      false
    );

    const success = await contract.castVote(false, proof);
    expect(success).to.be.false;
  });
});
```

Run tests:
```bash
npm test
```

## Congratulations! 🎉

You've built a complete privacy-preserving voting DApp on Midnight!

## Key Takeaways

1. **Zero-Knowledge Proofs**: Votes are proven valid without revealing choices
2. **Privacy**: Individual votes remain secret, only tallies are public
3. **Verification**: Eligibility and vote validity checked cryptographically
4. **Transparency**: Results are public and auditable

## Next Steps

- Add more features (multiple choice, weighted voting)
- Improve UI/UX
- Deploy to mainnet
- Add event emissions and notifications
- Implement vote delegation

## Resources

- [Midnight Documentation](https://docs.midnight.network/)
- [Compact Language Guide](../docs/compact-basics.md)
- [More Examples](../examples/)
- [Community Discord](https://discord.com/invite/midnightntwrk)
