# SecureX

Blockchain-based, time-locked custody for sensitive digital assets. Demonstrated on **exam paper leak prevention** (NEET), built for Problem Statement 26125.

A paper is hashed, signed and encrypted. The encryption key is split with Shamir's Secret Sharing, so no single person ever holds it. Each share is wrapped for its custodian, and the encrypted file is stored on IPFS. A smart contract enforces the time-lock and share threshold, mints an ERC-721 NFT for custody, and logs every action on-chain.

## How it works

1. **Admin (NTA)** uploads a paper. The backend hashes (SHA-256), signs (ECDSA), encrypts (AES-256-GCM) and splits the key (Shamir, k-of-n).
2. The encrypted file goes to **IPFS**. Its hash, signature, CID, release time and threshold are written to the **smart contract**, and an NFT is minted.
3. **Custodians** each submit their share on-chain from their own wallet.
4. Once the **time-lock has passed and the threshold is met**, anyone can call `attemptRelease()`. The NFT then moves to the **Exam Center**.
5. The paper is decrypted, and its hash and signature are re-verified.
6. **Anyone** can verify a copy of the file against the on-chain hash, and every step appears in the audit log.

## Tech stack

- **Contract:** Solidity, OpenZeppelin `AccessControl` and `ERC-721`, Ethereum Sepolia
- **Backend:** Node.js, Express, `shamirs-secret-sharing`, `eccrypto`, Pinata (IPFS)
- **Frontend:** React (Vite), Tailwind CSS, ethers.js v6, MetaMask

## Prerequisites

- Node.js 18 or newer
- MetaMask, set to the **Sepolia** test network and funded with free test ETH from a faucet
- A free [Pinata](https://pinata.cloud) account and API JWT
- A deployed `SecureAsset` contract on Sepolia (see step 1 below)

## Run it locally

### 1. Deploy the smart contract

Deploy your `SecureAsset` contract (OpenZeppelin `AccessControl` + `ERC-721`) to Sepolia, for example with [Remix](https://remix.ethereum.org). Note the **contract address** and the **block number** it was deployed at. Then grant `CUSTODIAN_ROLE` to each custodian wallet.

### 2. Start the backend

```bash
cd backend
npm install
cp .env.example .env
```

Edit `.env`:

```
PORT=3001
NUM_CUSTODIANS=3
PINATA_JWT=your_pinata_jwt
CONTRACT_ADDRESS=0xYourDeployedContractAddress
SEPOLIA_RPC_URL=https://ethereum-sepolia-rpc.publicnode.com
```

Then run:

```bash
npm start
```

The API runs at `http://localhost:3001`. Check it at `/api/health`.

### 3. Start the frontend

In a new terminal:

```bash
cd frontend
npm install
cp .env.example .env
```

Edit `.env`:

```
VITE_CONTRACT_ADDRESS=0xYourDeployedContractAddress
VITE_EXAM_CENTER_ADDRESS=0xYourExamCenterWalletAddress
VITE_DEPLOY_BLOCK=your_deployment_block_number
VITE_API_BASE_URL=/api
```

Then run:

```bash
npm run dev
```

Open `http://localhost:5173` with MetaMask connected. In development, Vite proxies `/api` requests to the backend on port 3001.

## Roles

Your wallet decides which dashboard you see.

| Wallet | Dashboard |
|---|---|
| Admin (`ADMIN_ROLE`) | Create assets, view all phases and alerts |
| Custodian (`CUSTODIAN_ROLE`) | Submit key shares |
| Exam Center (`VITE_EXAM_CENTER_ADDRESS`) | Release, decrypt, verify, check file integrity |
| Any other wallet | Read-only Public Verify |

## Verify a file from the command line

```bash
cd backend
node verify-tamper-demo.js <tokenId> <filePath>
# example: node verify-tamper-demo.js 4 sample.pdf
```

This fetches the on-chain hash for that token and compares it to the SHA-256 of your file.

## Demo-only notice

This is a prototype. The backend generates and stores the custodian and signer private keys in `data/`, CORS is open, and decrypted files are served without authentication. In a real deployment, each custodian would hold their own key on their own device. Do not use this setup for production.

## Security

- Never commit `.env` files or anything in `backend/data/`. Both are gitignored.
- If a Pinata JWT is ever exposed, revoke it and create a new one.

## Future scope

AI-configured deployments for new use cases, a full W3C DID implementation, a time-lock puzzle as a second protection layer, a decentralized time oracle, Merkle-tree batching, and a formal security audit.
