# 📜 Blockchain-Based Student Certificate Verification System
### INSTRUCTIONS.md — Full Setup & Developer Guide

---

## 📌 Table of Contents
1. [Project Overview](#1-project-overview)
2. [System Architecture](#2-system-architecture)
3. [Tech Stack](#3-tech-stack)
4. [Prerequisites](#4-prerequisites)
5. [Folder Structure](#5-folder-structure)
6. [Phase 1 — Smart Contract (Solidity + Hardhat)](#6-phase-1--smart-contract-solidity--hardhat)
7. [Phase 2 — Deploy to Polygon Testnet](#7-phase-2--deploy-to-polygon-testnet)
8. [Phase 3 — MetaMask Web Interface (React + TypeScript)](#8-phase-3--metamask-web-interface-react--typescript)
9. [Phase 4 — IPFS Decentralized File Storage](#9-phase-4--ipfs-decentralized-file-storage)
10. [Phase 5 — DAO Governance Framework](#10-phase-5--dao-governance-framework)
11. [End-to-End Working Flow](#11-end-to-end-working-flow)
12. [Environment Variables](#12-environment-variables)
13. [Smart Contract Function Reference](#13-smart-contract-function-reference)
14. [Troubleshooting](#14-troubleshooting)

---

## 1. Project Overview

### 🔴 Problem
- Fake certificates and mark sheets are increasingly common
- Companies cannot easily verify the authenticity of documents
- Centralized college records are vulnerable to modification or hacking
- Manual verification via phone/email is slow and unreliable

### 🟢 Solution
Instead of storing the full certificate on the blockchain, we store its **SHA-256 hash** — a unique digital fingerprint.

- If even **1 character** changes in the certificate → the hash changes → fake is detected instantly
- Verification becomes **automatic, instant, and tamper-proof**
- No central authority can modify records

### 👥 Stakeholders
| Stakeholder | Role | Benefit |
|---|---|---|
| College / University | Issues certificates, uploads to system | Certificates cannot be faked |
| Student | Receives certificate, shares with companies | Easy verification, no need to contact college |
| Company / Recruiter | Uploads certificate to verify | Instant, automated, safe hiring |
| Blockchain Network | Stores hash permanently | Tamper-proof, decentralized trust |

---

## 2. System Architecture

### Architecture Diagram
```
┌─────────────────────────────────────────────────────────────────┐
│                        COLLEGE SIDE                             │
│   [Upload Certificate PDF]                                      │
│          │                                                      │
│          ▼                                                      │
│   [FastAPI Backend]                                             │
│    ├── Generate SHA-256 Hash                                    │
│    ├── Upload PDF → IPFS (via Pinata)                           │
│    └── Store Hash → Ethereum Smart Contract                     │
└─────────────────────────────────────────────────────────────────┘
                          │
              ┌───────────┴───────────┐
              ▼                       ▼
     [Ethereum Blockchain]        [IPFS Network]
      Stores: certId,              Stores: PDF File
      studentName, course,         Returns: IPFS CID
      hash, timestamp
              │
┌─────────────┴───────────────────────────────────────────────────┐
│                       COMPANY SIDE                              │
│   [Upload Certificate PDF]                                      │
│          │                                                      │
│          ▼                                                      │
│   [React Frontend + MetaMask]                                   │
│    ├── Generate SHA-256 Hash                                    │
│    ├── Call verifyCertificate() on blockchain                   │
│    └── Show Result: ✅ GENUINE / ❌ FAKE                        │
└─────────────────────────────────────────────────────────────────┘
```

### Workflow Diagram
```
COLLEGE FLOW:
Certificate PDF
      │
      ▼
SHA-256 Hash Generated
      │
      ├──────────────────────► IPFS Storage (PDF file)
      │                              │
      ▼                              ▼
Smart Contract ◄─── FastAPI ◄─── IPFS CID returned
addCertificate()
(hash stored on blockchain)


COMPANY FLOW:
Upload Same Certificate PDF
      │
      ▼
SHA-256 Hash Generated
      │
      ▼
verifyCertificate(hash) called
      │
      ├── Hash matches blockchain record → ✅ GENUINE (shows timestamp, name, course)
      └── Hash does NOT match           → ❌ FAKE / NOT FOUND
```

>  **Screenshot Placeholder — Architecture Diagram**
> ![Architecture Diagram](./docs/screenshots/architecture-diagram.png)
> *(Add your architecture diagram image to `docs/screenshots/` folder)*

---

## 3. Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Layer 1 — Blockchain | Ethereum Sepolia Testnet | Record and verify certificate hashes |
| Layer 2 — Scaling | Polygon Mumbai Testnet | Faster transactions, lower gas fees |
| Layer 3 — Application | React + TypeScript + Ethers.js + MetaMask | User interface for issuing & verifying |
| Layer 4 — Storage | IPFS via Pinata | Decentralized PDF storage |
| Layer 5 — Governance | DAO Smart Contract (Solidity) | Manage approvals and disputes |
| Backend | FastAPI (Python) | Hash generation, IPFS upload, contract calls |
| Smart Contract | Solidity + Hardhat | Certificate logic on-chain |
| Cryptography | SHA-256 + ECDSA (MetaMask) | Hashing + transaction signing |

---

## 4. Prerequisites

### Software to Install

| Tool | Version | Download |
|---|---|---|
| Node.js | v18+ | https://nodejs.org |
| Python | v3.10+ | https://python.org |
| Git | Latest | https://git-scm.com |
| MetaMask | Browser Extension | https://metamask.io |
| VS Code | Latest | https://code.visualstudio.com |

### Accounts to Create

| Service | Purpose | Link |
|---|---|---|
| Alchemy or Infura | Ethereum/Polygon RPC provider | https://alchemy.com |
| Pinata | IPFS file pinning service | https://pinata.cloud |
| Etherscan | View transactions on Sepolia | https://sepolia.etherscan.io |
| PolygonScan | View transactions on Polygon | https://mumbai.polygonscan.com |

### Install Global Tools
```bash
# Install Hardhat globally
npm install --global hardhat

# Install Python dependencies manager
pip install virtualenv
```

### Get Testnet Tokens (Free)
- **Sepolia ETH**: https://sepoliafaucet.com
- **Polygon MATIC (Mumbai)**: https://faucet.polygon.technology

---

## 5. Folder Structure

```
certificate-verification/
│
├── contracts/                  # Solidity smart contracts
│   ├── CertificateRegistry.sol     # Main certificate contract
│   └── CertificateDAO.sol          # DAO governance contract
│
├── scripts/                    # Hardhat deployment scripts
│   ├── deploy.js                   # Deploy to Sepolia
│   └── deployPolygon.js            # Deploy to Polygon
│
├── test/                       # Smart contract unit tests
│   └── certificate.test.js
│
├── backend/                    # FastAPI Python backend
│   ├── main.py                     # API entry point
│   ├── hash_utils.py               # SHA-256 hashing logic
│   ├── ipfs_utils.py               # Pinata IPFS upload
│   ├── contract_utils.py           # Ethers/web3 contract calls
│   └── requirements.txt
│
├── frontend/                   # React + TypeScript UI
│   ├── src/
│   │   ├── components/
│   │   │   ├── IssueCertificate.tsx    # College upload form
│   │   │   ├── VerifyCertificate.tsx   # Company verify form
│   │   │   ├── ConnectWallet.tsx       # MetaMask connect button
│   │   │   └── ResultDisplay.tsx       # Show genuine/fake result
│   │   ├── utils/
│   │   │   ├── ethers.ts               # Ethers.js setup
│   │   │   └── hash.ts                 # Browser-side SHA-256
│   │   └── App.tsx
│   ├── package.json
│   └── tsconfig.json
│
├── ipfs/                       # IPFS standalone scripts
│   └── upload.py                   # Direct Pinata upload script
│
├── dao/                        # DAO documentation and extras
│   └── README.md
│
├── docs/                       # Documentation assets
│   └── screenshots/                # Add your screenshots here
│       ├── architecture-diagram.png
│       ├── metamask-connect.png
│       ├── issue-certificate.png
│       ├── verify-genuine.png
│       ├── verify-fake.png
│       └── ipfs-upload.png
│
├── hardhat.config.js           # Hardhat network configuration
├── .env                        # Environment variables (never commit!)
├── .env.example                # Template for .env
├── .gitignore
└── INSTRUCTIONS.md             # This file
```

---

## 6. Phase 1 — Smart Contract (Solidity + Hardhat)

> **Assignment 1:** Design a smart contract deployed on a blockchain network.

### Step 1: Initialize Project
```bash
mkdir certificate-verification
cd certificate-verification
npm init -y
npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox
npx hardhat init
# Choose: Create a JavaScript project
```

### Step 2: Write the Smart Contract
Create file: `contracts/CertificateRegistry.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract CertificateRegistry {

    struct Certificate {
        string studentName;
        string course;
        string docHash;       // SHA-256 hash of the PDF
        string ipfsCID;       // IPFS content identifier
        uint256 timestamp;
        address issuedBy;     // College wallet address
        bool isValid;
    }

    mapping(uint256 => Certificate) public certificates;
    uint256 public certificateCount;

    event CertificateAdded(uint256 certId, string studentName, uint256 timestamp);
    event CertificateRevoked(uint256 certId);

    // College calls this to issue a certificate
    function addCertificate(
        string memory _name,
        string memory _course,
        string memory _docHash,
        string memory _ipfsCID
    ) public returns (uint256) {
        certificateCount++;
        certificates[certificateCount] = Certificate(
            _name, _course, _docHash, _ipfsCID,
            block.timestamp, msg.sender, true
        );
        emit CertificateAdded(certificateCount, _name, block.timestamp);
        return certificateCount;
    }

    // Anyone can call this to verify
    function verifyCertificate(string memory _docHash)
        public view returns (bool, string memory, string memory, uint256) {
        for (uint256 i = 1; i <= certificateCount; i++) {
            if (keccak256(bytes(certificates[i].docHash)) == keccak256(bytes(_docHash))
                && certificates[i].isValid) {
                return (true,
                    certificates[i].studentName,
                    certificates[i].course,
                    certificates[i].timestamp);
            }
        }
        return (false, "", "", 0);
    }

    // College can revoke a certificate
    function revokeCertificate(uint256 _certId) public {
        require(certificates[_certId].issuedBy == msg.sender, "Not authorized");
        certificates[_certId].isValid = false;
        emit CertificateRevoked(_certId);
    }
}
```

### Step 3: Compile & Test Locally
```bash
npx hardhat compile
npx hardhat test
npx hardhat node          # Start local blockchain
npx hardhat run scripts/deploy.js --network localhost
```

---

## 7. Phase 2 — Deploy to Polygon Testnet

> **Assignment 2:** Set up Polygon test network and deploy smart contract.

### Step 1: Configure hardhat.config.js
```javascript
require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config();

module.exports = {
  solidity: "0.8.19",
  networks: {
    sepolia: {
      url: process.env.ALCHEMY_SEPOLIA_URL,
      accounts: [process.env.PRIVATE_KEY]
    },
    polygon_mumbai: {
      url: process.env.ALCHEMY_POLYGON_URL,
      accounts: [process.env.PRIVATE_KEY]
    }
  }
};
```

### Step 2: Deploy
```bash
# Deploy to Sepolia
npx hardhat run scripts/deploy.js --network sepolia

# Deploy to Polygon Mumbai
npx hardhat run scripts/deploy.js --network polygon_mumbai
```

### Step 3: Save the Contract Address
After deployment, the terminal will print:
```
CertificateRegistry deployed to: 0xYourContractAddress
```
Copy this address into your `.env` file.

> 📸 **Screenshot Placeholder — Successful Deployment on PolygonScan**
> ![Polygon Deploy](./docs/screenshots/polygon-deploy.png)

---

## 8. Phase 3 — MetaMask Web Interface (React + TypeScript)

> **Assignment 3:** Web interface where users can sign and send transactions using MetaMask.

### Overview
The frontend has two main flows:
- **College Side** — Connect wallet → Upload PDF → Issue certificate on blockchain
- **Company Side** — Connect wallet → Upload PDF → Verify against blockchain

### Step 1: Setup React Project
```bash
cd frontend
npx create-react-app . --template typescript
npm install ethers axios react-dropzone
```

### Step 2: Connect MetaMask Wallet

MetaMask injects `window.ethereum` into the browser. Ethers.js wraps it to make contract calls easy.

```typescript
// src/utils/ethers.ts
import { ethers } from "ethers";
import CertificateRegistryABI from "../abi/CertificateRegistry.json";

const CONTRACT_ADDRESS = process.env.REACT_APP_CONTRACT_ADDRESS!;

// Request wallet connection from MetaMask
export async function connectWallet(): Promise<string> {
  if (!window.ethereum) throw new Error("MetaMask not found. Please install it.");
  const accounts = await window.ethereum.request({ method: "eth_requestAccounts" });
  return accounts[0];  // Returns connected wallet address
}

// Get a contract instance connected to the user's wallet (for write operations)
export async function getSignedContract() {
  const provider = new ethers.BrowserProvider(window.ethereum);
  const signer = await provider.getSigner();   // MetaMask signs transactions
  return new ethers.Contract(CONTRACT_ADDRESS, CertificateRegistryABI, signer);
}

// Get a contract instance for read-only operations (no MetaMask needed)
export async function getReadContract() {
  const provider = new ethers.BrowserProvider(window.ethereum);
  return new ethers.Contract(CONTRACT_ADDRESS, CertificateRegistryABI, provider);
}
```

**Why MetaMask Signs Transactions:**
- When a college calls `addCertificate()`, it writes data to the blockchain
- This costs gas (small fee in ETH/MATIC)
- MetaMask shows a popup asking the user to **confirm and sign** the transaction
- This is the ECDSA (Elliptic Curve Digital Signature Algorithm) step
- Read operations like `verifyCertificate()` are **free** — no signature needed

### Step 3: Browser-Side SHA-256 Hashing

The hash is generated in the browser **before** sending to the backend, so the PDF never needs to be sent anywhere to get a hash.

```typescript
// src/utils/hash.ts
export async function generateFileHash(file: File): Promise<string> {
  const buffer = await file.arrayBuffer();
  const hashBuffer = await crypto.subtle.digest("SHA-256", buffer);
  const hashArray = Array.from(new Uint8Array(hashBuffer));
  return hashArray.map(b => b.toString(16).padStart(2, "0")).join("");
}
```

### Step 4: Issue Certificate Component (College Side)
```typescript
// src/components/IssueCertificate.tsx
import { connectWallet, getSignedContract } from "../utils/ethers";
import { generateFileHash } from "../utils/hash";

async function handleIssue(file: File, name: string, course: string) {
  // 1. Connect MetaMask
  await connectWallet();

  // 2. Generate hash in browser
  const hash = await generateFileHash(file);

  // 3. Upload PDF to IPFS via backend
  const formData = new FormData();
  formData.append("file", file);
  const ipfsRes = await axios.post("http://localhost:8000/upload-ipfs", formData);
  const ipfsCID = ipfsRes.data.cid;

  // 4. Call smart contract — MetaMask popup appears here
  const contract = await getSignedContract();
  const tx = await contract.addCertificate(name, course, hash, ipfsCID);
  await tx.wait();  // Wait for blockchain confirmation

  alert(`✅ Certificate issued! Transaction: ${tx.hash}`);
}
```

### Step 5: Verify Certificate Component (Company Side)
```typescript
// src/components/VerifyCertificate.tsx
async function handleVerify(file: File) {
  // 1. Generate hash from uploaded file
  const hash = await generateFileHash(file);

  // 2. Call read-only contract function (free, no MetaMask popup)
  const contract = await getReadContract();
  const [isValid, studentName, course, timestamp] = await contract.verifyCertificate(hash);

  if (isValid) {
    const date = new Date(Number(timestamp) * 1000).toLocaleDateString();
    alert(`✅ GENUINE\nStudent: ${studentName}\nCourse: ${course}\nIssued: ${date}`);
  } else {
    alert("❌ FAKE — Certificate not found on blockchain");
  }
}
```

> 📸 **Screenshot Placeholder — MetaMask Connect Button**
> ![MetaMask Connect](./docs/screenshots/metamask-connect.png)

> 📸 **Screenshot Placeholder — Issue Certificate Form**
> ![Issue Certificate](./docs/screenshots/issue-certificate.png)

> 📸 **Screenshot Placeholder — Verification Result (Genuine)**
> ![Verify Genuine](./docs/screenshots/verify-genuine.png)

> 📸 **Screenshot Placeholder — Verification Result (Fake)**
> ![Verify Fake](./docs/screenshots/verify-fake.png)

---

## 9. Phase 4 — IPFS Decentralized File Storage

> **Assignment 4:** Implement IPFS for decentralized file storage.

### What is IPFS and Why Use It?

**IPFS (InterPlanetary File System)** is a decentralized storage network.

| Traditional Storage | IPFS Storage |
|---|---|
| File stored at a URL (can go down) | File stored by its content hash (permanent) |
| Server can delete the file | No single server controls it |
| URL can change | CID (Content ID) never changes |
| Centralized — single point of failure | Distributed across many nodes |

In our system:
- The **PDF certificate** is stored on IPFS
- The **IPFS CID** (a hash like `QmXoypiz...`) is stored on the blockchain
- Anyone with the CID can retrieve the original certificate

### Step 1: Create Pinata Account
1. Go to https://pinata.cloud and create a free account
2. Go to **API Keys** → Create new key
3. Enable `pinFileToIPFS` and `pinJSONToIPFS`
4. Copy your **API Key** and **Secret Key** to `.env`

### Step 2: Install Backend Dependencies
```bash
cd backend
python -m venv venv
source venv/bin/activate      # On Windows: venv\Scripts\activate
pip install fastapi uvicorn python-multipart requests python-dotenv web3
pip freeze > requirements.txt
```

### Step 3: IPFS Upload Utility
```python
# backend/ipfs_utils.py
import requests
import os
from dotenv import load_dotenv

load_dotenv()

PINATA_API_KEY = os.getenv("PINATA_API_KEY")
PINATA_SECRET = os.getenv("PINATA_SECRET_KEY")
PINATA_URL = "https://api.pinata.cloud/pinning/pinFileToIPFS"

def upload_to_ipfs(file_bytes: bytes, filename: str) -> str:
    """
    Upload a file to IPFS via Pinata.
    Returns the IPFS CID (Content Identifier).
    """
    headers = {
        "pinata_api_key": PINATA_API_KEY,
        "pinata_secret_api_key": PINATA_SECRET,
    }
    files = {
        "file": (filename, file_bytes, "application/pdf")
    }
    response = requests.post(PINATA_URL, files=files, headers=headers)
    response.raise_for_status()

    cid = response.json()["IpfsHash"]
    print(f"[IPFS] File uploaded. CID: {cid}")
    print(f"[IPFS] View at: https://gateway.pinata.cloud/ipfs/{cid}")
    return cid
```

### Step 4: SHA-256 Hash Utility
```python
# backend/hash_utils.py
import hashlib

def generate_sha256(file_bytes: bytes) -> str:
    """
    Generate SHA-256 hash of file bytes.
    This is the unique fingerprint stored on the blockchain.
    """
    sha256 = hashlib.sha256()
    sha256.update(file_bytes)
    return sha256.hexdigest()
```

### Step 5: FastAPI Main Application
```python
# backend/main.py
from fastapi import FastAPI, UploadFile, File
from fastapi.middleware.cors import CORSMiddleware
from hash_utils import generate_sha256
from ipfs_utils import upload_to_ipfs

app = FastAPI(title="Certificate Verification API")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.post("/upload-ipfs")
async def upload_certificate(file: UploadFile = File(...)):
    """Upload certificate PDF to IPFS and return CID + hash"""
    file_bytes = await file.read()
    sha256_hash = generate_sha256(file_bytes)
    ipfs_cid = upload_to_ipfs(file_bytes, file.filename)
    return {
        "cid": ipfs_cid,
        "hash": sha256_hash,
        "ipfs_url": f"https://gateway.pinata.cloud/ipfs/{ipfs_cid}"
    }

@app.post("/generate-hash")
async def get_hash_only(file: UploadFile = File(...)):
    """Generate SHA-256 hash without uploading to IPFS"""
    file_bytes = await file.read()
    return {"hash": generate_sha256(file_bytes)}
```

### Step 6: Run the Backend
```bash
uvicorn main:app --reload --port 8000
# API docs available at: http://localhost:8000/docs
```

### How IPFS CID Links to Blockchain

```
Certificate PDF
      │
      ▼ (Pinata upload)
IPFS CID: QmXoypiz7ATpPbuEJvgKfhbhkLGpnEequ9D...
      │
      ▼ (stored in smart contract)
certificates[1].ipfsCID = "QmXoypiz7ATpPbuEJvgKfhbhkLGpnEequ9D..."
      │
      ▼ (anyone can retrieve)
https://gateway.pinata.cloud/ipfs/QmXoypiz7ATpPbuEJvgKfhbhkLGpnEequ9D...
```

> 📸 **Screenshot Placeholder — Pinata Dashboard showing uploaded file**
> ![IPFS Upload](./docs/screenshots/ipfs-upload.png)

---

## 10. Phase 5 — DAO Governance Framework

> **Assignment 5:** Set up a basic DAO framework using a smart contract.

### What is a DAO and Why Use It Here?

A **DAO (Decentralized Autonomous Organization)** is an organization governed by smart contract rules instead of a central authority.

In our certificate system, the DAO governs:
- **Who can issue certificates** (college addresses must be approved)
- **Certificate revocation requests** (requires voting)
- **Dispute resolution** (company disputes a certificate's authenticity)
- **Adding new authorized colleges** (requires majority vote)

### How DAO Voting Works
```
Member proposes action (e.g., "Revoke certificate #12")
        │
        ▼
Voting period opens (e.g., 7 days)
        │
        ├── Members vote YES or NO
        │
        ▼
If YES votes > 50% of total members → Proposal PASSES
        │
        ▼
Anyone can execute the proposal → Action happens on blockchain
```

### DAO Smart Contract
Create file: `contracts/CertificateDAO.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

interface ICertificateRegistry {
    function revokeCertificate(uint256 certId) external;
}

contract CertificateDAO {

    // DAO Members
    address[] public members;
    mapping(address => bool) public isMember;

    // Proposal Types
    enum ProposalType { AddCollege, RevokeCollege, RevokeCertificate }

    struct Proposal {
        uint256 id;
        ProposalType proposalType;
        address targetAddress;       // For college add/remove
        uint256 targetCertId;        // For certificate revoke
        string description;
        uint256 voteCount;
        uint256 deadline;            // Voting deadline (timestamp)
        bool executed;
        mapping(address => bool) hasVoted;
    }

    mapping(uint256 => Proposal) public proposals;
    uint256 public proposalCount;

    address public certificateContract;
    uint256 public votingPeriod = 7 days;

    event ProposalCreated(uint256 id, string description);
    event Voted(uint256 proposalId, address voter);
    event ProposalExecuted(uint256 id);

    constructor(address _certContract, address[] memory _initialMembers) {
        certificateContract = _certContract;
        for (uint i = 0; i < _initialMembers.length; i++) {
            members.push(_initialMembers[i]);
            isMember[_initialMembers[i]] = true;
        }
    }

    modifier onlyMember() {
        require(isMember[msg.sender], "Not a DAO member");
        _;
    }

    // Create a new proposal
    function createProposal(
        ProposalType _type,
        address _target,
        uint256 _certId,
        string memory _description
    ) public onlyMember returns (uint256) {
        proposalCount++;
        Proposal storage p = proposals[proposalCount];
        p.id = proposalCount;
        p.proposalType = _type;
        p.targetAddress = _target;
        p.targetCertId = _certId;
        p.description = _description;
        p.deadline = block.timestamp + votingPeriod;
        emit ProposalCreated(proposalCount, _description);
        return proposalCount;
    }

    // Vote on a proposal
    function vote(uint256 _proposalId) public onlyMember {
        Proposal storage p = proposals[_proposalId];
        require(block.timestamp < p.deadline, "Voting period ended");
        require(!p.hasVoted[msg.sender], "Already voted");
        p.hasVoted[msg.sender] = true;
        p.voteCount++;
        emit Voted(_proposalId, msg.sender);
    }

    // Execute a passed proposal
    function executeProposal(uint256 _proposalId) public {
        Proposal storage p = proposals[_proposalId];
        require(block.timestamp >= p.deadline, "Voting still active");
        require(!p.executed, "Already executed");
        require(p.voteCount * 2 > members.length, "Not enough votes (need >50%)");

        p.executed = true;

        if (p.proposalType == ProposalType.AddCollege) {
            members.push(p.targetAddress);
            isMember[p.targetAddress] = true;
        } else if (p.proposalType == ProposalType.RevokeCollege) {
            isMember[p.targetAddress] = false;
        } else if (p.proposalType == ProposalType.RevokeCertificate) {
            ICertificateRegistry(certificateContract).revokeCertificate(p.targetCertId);
        }

        emit ProposalExecuted(_proposalId);
    }

    function getMemberCount() public view returns (uint256) {
        return members.length;
    }
}
```

### Deploy DAO Contract
```bash
npx hardhat run scripts/deployDAO.js --network sepolia
```

> 📸 **Screenshot Placeholder — DAO Proposal and Voting UI**
> ![DAO Framework](./docs/screenshots/dao-voting.png)

---

## 11. End-to-End Working Flow

### College Side — Issuing a Certificate
```
Step 1: College opens the web app
Step 2: Clicks "Connect Wallet" → MetaMask popup → Approve connection
Step 3: Fills in: Student Name, Course Name
Step 4: Uploads Certificate PDF
Step 5: Clicks "Issue Certificate"
        → Browser generates SHA-256 hash of PDF
        → Backend uploads PDF to IPFS → Gets CID
        → MetaMask popup: "Confirm transaction" (costs small gas fee)
        → College signs → Transaction sent to Ethereum
        → Smart contract stores: name, course, hash, CID, timestamp
Step 6: Success message + Transaction hash shown
```

### Company Side — Verifying a Certificate
```
Step 1: Company opens the web app
Step 2: Uploads the certificate PDF received from student
Step 3: Clicks "Verify Certificate"
        → Browser generates SHA-256 hash of uploaded PDF
        → Calls verifyCertificate(hash) on blockchain (free, no gas)
        → Blockchain searches all certificates for matching hash
Step 4a: MATCH FOUND → Shows ✅ GENUINE
         - Student Name, Course, Issue Date, College Wallet
Step 4b: NO MATCH    → Shows ❌ FAKE / TAMPERED
```

---

## 12. Environment Variables

Create a `.env` file in the root folder:

```env
# Blockchain
PRIVATE_KEY=your_metamask_wallet_private_key_here
ALCHEMY_SEPOLIA_URL=https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY
ALCHEMY_POLYGON_URL=https://polygon-mumbai.g.alchemy.com/v2/YOUR_KEY

# Deployed Contract Addresses (fill after deployment)
CONTRACT_ADDRESS_SEPOLIA=0x...
CONTRACT_ADDRESS_POLYGON=0x...
DAO_CONTRACT_ADDRESS=0x...

# IPFS - Pinata
PINATA_API_KEY=your_pinata_api_key
PINATA_SECRET_KEY=your_pinata_secret_key

# Frontend (React uses REACT_APP_ prefix)
REACT_APP_CONTRACT_ADDRESS=0x...
REACT_APP_BACKEND_URL=http://localhost:8000
```

**⚠️ IMPORTANT:** Never commit `.env` to GitHub! Add it to `.gitignore`:
```
.env
node_modules/
__pycache__/
venv/
```

---

## 13. Smart Contract Function Reference

| Function | Who Calls It | Costs Gas? | Description |
|---|---|---|---|
| `addCertificate(name, course, hash, cid)` | College (via MetaMask) | ✅ Yes | Issues a new certificate on-chain |
| `verifyCertificate(hash)` | Anyone | ❌ No | Returns certificate details if hash matches |
| `revokeCertificate(certId)` | Issuing college | ✅ Yes | Marks certificate as invalid |
| `createProposal(...)` | DAO member | ✅ Yes | Creates a governance proposal |
| `vote(proposalId)` | DAO member | ✅ Yes | Casts a vote on a proposal |
| `executeProposal(id)` | Anyone | ✅ Yes | Executes a passed proposal |

---

## 14. Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| `MetaMask not found` | Extension not installed | Install MetaMask from https://metamask.io |
| `Transaction failed: insufficient funds` | No testnet ETH | Get free ETH from https://sepoliafaucet.com |
| `Wrong network` | MetaMask on wrong chain | Switch to Sepolia or Mumbai in MetaMask |
| `CORS error` | Backend not allowing frontend origin | Check CORSMiddleware in `main.py` |
| `Hash mismatch` | File was altered | Original file was tampered — certificate is FAKE |
| `Pinata upload failed` | Wrong API keys | Double-check `.env` Pinata credentials |
| `Contract not deployed` | Missing contract address | Run deploy script and update `.env` |
| `nonce too low` | Transaction ordering issue | Reset MetaMask account in Settings → Advanced |

---

*INSTRUCTIONS.md — Blockchain-Based Student Certificate Verification System*
*Created for T.Y. B.Tech Computer Engineering — Blockchain Lab*
