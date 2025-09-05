***

# DocPact Technical Architecture: End-to-End Breakdown

***

## 1. 🌐 **User & Info Gathering Flow**

### 1.1 Entry, Information Collection & Agreement Draft

**Who can start?**  
- *Either party*: client, creator, legal, business.
- No login—**wallet-based onboarding** (MetaMask, WalletConnect, etc).

**Info Collected:**
- What is being exchanged?  
  _e.g. codebase, document, design, access key_
- What are the milestone(s) and payment splits?
- Are there **special privacy/confidentiality needs**? (e.g. legal doc vs. portfolio image)
- What verification agent is required?
- Are there jurisdiction-specific compliance requirements?

***

### 🖼️ **Visual: Data Collection & Setup (Markdown Diagram)**

```mermaid
graph TD
    IN["Initiator (A or B)"] --> FORM["Guided Form or Chat (LangGraph)"]
    FORM --> S1["Describe Deliverable(s)"]
    S1 --> S2["Choose Milestones/Payments"]
    S2 --> S3["Pick Verification Method"]
    S3 --> S4["Declare Confidentiality/Privacy"]
    S4 --> S5["Add Compliance Info (if any)"]
    S5 --> PREV["Preview Contract Summary"]
```

- All input flows through LangGraph nodes, which interactively adapt follow-up questions based on previous answers.
- _Minimal code example:_
```typescript
const { input, deliverable, milestones, verification, privacy, compliance } = userInput();
```

***

## 2. 📝 **Contract Finalization & Digital Execution**

**Contract Drafting:**
- DocPact agent merges user input into a multi-section, human-readable contract (NDA if toggled; milestones itemized).

**Review Mode:**
- Both parties view in “plain English” and can toggle to technical view (on Next.js).

**Digital Signatures:**
- EIP-712 typed data signing.
- Parties sign sequentially or in parallel—every signature is onchain and timestamped.
- Each signature is directly linked to wallet and contract version for audit.

**Multi-party Support:**
- Every required party, or admin agent, may sign in round-robin or in parallel.

##### 🔑 **Signature Status (“At a Glance” Table)**

| Party           | Wallet Address   | Signed?   | Timestamp           |  
|-----------------|-----------------|-----------|---------------------|  
| Client          | 0x123...abc      | ✔         | 2025-09-05 19:45    |  
| Creator         | 0x789...xyz      | ⏳        |                     |  

***

## 3. 💸 **Escrow Creation, Fee Lock, and Storage Estimation**

**SDKs/Smart Contracts Used:**
- [@filoz/synapse-sdk/PaymentsService](https://www.npmjs.com/package/@filoz/synapse-sdk)
- [FilecoinPay](https://github.com/FilOzone/filecoin-pay) escrow contract suite.

**Steps:**
1. Calculate/estimate storage (automatically suggested, but user editable).
2. Calculate platform/agent fees (shown on UI).
3. User(s) sign and initiate payment to escrow (USDFC or other FEVM-supported tokens).

***

## 4. 🔐 **Asset Submission and Cryptographic Storage**

**Deliverables Submission:**
- Providers never upload raw files—**AES/ChaCha-encrypted** (using [crypto](https://developer.mozilla.org/en-US/docs/Web/API/Crypto), [libsodium](https://github.com/jedisct1/libsodium.js)) in browser.
- Chunked uploads for large repos/media.
- Immediate **warm storage upload** via `createStorage({ withCDN: true })`. When completed, **auto-archival to Filecoin cold storage** (permanent).

**All uploads are referenced with CIDs onchain** via the EscrowDealMaker contract.

##### 🗃️ **Assets in This Deal**

| Phase   | CID                    | Storage Tier | Encryption | Verified? |
|---------|------------------------|--------------|------------|-----------|
| Phase 1 | bafybeib...            | Warm + Cold  | Yes        | ⏳        |

***

## 5. 🤖 **Agent Verification: Review, Privacy Checks, and TEE Escalation**

**Verification agent (AI or plugin) gets auto-notified.**
- Fetches encrypted CID from Filecoin.
- Decryption requires ephemeral, isolated agent context.

**Normal Checks:**
- Runs standard tasks (code test, hash check, document scan) in memory and releases pass/fail/proof to escrow contract.

**Confidential/TEE/Zero-Knowledge-TEE**
- If flagged confidential, agent runs in **TEE enclave** (SGX/Nitro/Conclave).
- ZK proof support: For asset claims (e.g., _"file passes test suite X"_) **without content leak**.

-----  
#### 🖼️ **Diagram: Verification Privacy Route**

```mermaid
flowchart TD
    CID["Encrypted Deliverable (CID)"] --> Agent
    Agent --ephemeral-decrypt--> Check["Verifies in-memory / in-enclave"]
    Check --If confidential--> TEE(["TEE / ZK-TEE"])
    Check --If standard--> Logic["Run asset checks"]
    TEE --attestation+result/proof--> Escrow
    Logic --pass/fail--> Escrow
```
-----

##### **Agent Result Example (JSON)**

```json
{
  "phase": 1,
  "outcome": "pass",
  "proof": "ZK-snark-proof-hash",
  "agent_attestation": "0xagentSignature",
  "verifier": "AI:CodeCheck-v1 (in TEE)",
  "timestamp": "2025-09-05T22:19:12Z"
}
```

***

## 6. 👛 **Milestone Approval, Payment, and Key Handover Workflow**

**On successful verification:**
- Milestone payment released from escrow (via FilecoinPay/SDK).
- If all verified, final phase decrypt key is sent to client (who can now decrypt their files).
- If any phase fails, parties are auto-notified for mediation or dispute; possible fallback to a human/agent arbiter.

***

## 7. 🔒 **Audit Logging, Cold Storage, and Onchain Proof**

- **Every event**—signature, upload, AI check, TEE attestation, payment, handover—is logged as an onchain contract event and optionally Merkle-proved offchain.
- On full agreement completion, assets and logs **archived to Filecoin cold storage.**
- Transparency: Anyone with deal/cid can verify—prevents "he said, she said".

***

## 8. 🧩 **All Technology Stack Locations**

- **Frontend:** Next.js + Tailwind, Ethers.js for wallet ops, all encryption in-browser.
- **Agent Middleware:** LangGraph for orchestration, pluggable custom nodes (TS/JS).
- **Backend/Serverless:** Next.js API routes, calls SynapseSDK (Payments, Storage).
- **Smart Contracts:** FEVM, FilecoinPay suite, custom EscrowDealMaker (handles all funds, CIDs, agent attestations, state).
- **Storage:** SynapseSDK (warm/cold), Filecoin direct.
- **Confidential Verification:** TEE runtime (SGX/AWS Nitro/Oasis/Conclave), optionally ZK circuits.
- **Compliance/Timestamps:** All actions logged on FEVM and, if required, synchronized with compliance APIs (future).

***

## 9. **Example: Realistic Escrow Agreement Lifecycle (with major Signposts and Actor View)**

```mermaid
flowchart TB
    Start(["User Initiation"])
    InfoGather(["Interactive Info Gathering"])
    Contract("Draft Contract Preview")
    Sign("Both Parties Sign Digitally")
    EscrowLock("USDFC Lock-in (via FilecoinPay)")
    Upload("Provider Encrypts & Uploads (Warm)")
    AgentReview("Agent Verifies (AI/TEE/Plugin)")
    ApproveOrDispute{"Agent Result?"}
    Payment("Payment Released")
    Mediate("Dispute Mediation Process")
    ColdStore("Archival to Cold Storage")
    End(["Deal Complete/Archived"])

    Start --> InfoGather --> Contract
    Contract --> Sign --> EscrowLock --> Upload --> AgentReview --> ApproveOrDispute
    ApproveOrDispute -- Pass --> Payment --> ColdStore --> End
    ApproveOrDispute -- Fail --> Mediate --> ColdStore
```

***

## 10. **UX "Snapshots" for Every Core Step**

**A. Contract Preview/Signature:**
```
-----------------------------------------------------
 Digital Agreement Summary
-----------------------------------------------------
- Client: 0x123...
- Creator: 0x456...
- Asset: Project Repo (encrypted, 42MB)
- Milestones: [Phase 1: $1,000][Phase 2: $1,000][Phase 3: $1,000]
- Verification: AI-Agent (in TEE)
- Confidential: Yes, phases 2/3 only
---> E-SIGN / VIEW IN TECHNICAL MODE <---
-----------------------------------------------------
```

**B. Milestone Progress:**
```
-----------------------------------------------------
 [✓] Contract Signed
 [✓] Funds Locked
 [✓] Phase 1 Submission (CID: baf...)
 [✓] Agent Verified (Proof: 0x...)
 [✓] Phase 1 Payment Released
 [ ] Phase 2...
-----------------------------------------------------
```

***

This full architecture.md is ready for both teams and developers—**richly annotating every technical, UX, privacy, and contract detail.**

***

## 11. **Detailed System Flow Diagram**

The following diagram provides a comprehensive, step-by-step view of the DocPact system architecture, showing the complete flow from contract creation through payment settlement:

```mermaid 
flowchart TD
    A["Party A<br>Client / Commissioner"] --> S1["Step 1: Create Escrow Deal<br>Pays USDFC plus Commission Upfront"] & S2["Step 2: Set Terms and Agent Logic"]
    S1 --> Escrow["Escrow Smart Contract<br>DocPact FEVM / FilecoinPay"]
    S2 --> Agent["AI Agent<br>LangGraph + TEE AI ZK"]
    S2 --> S2a["Step 2a: Natural Language Contract Review<br>Both Parties Review Terms"]
    S2a --> S2b["Step 2b: Digital Signature Process<br>Cryptographic Signing by Both Parties"]
    S2b --> Escrow
    Escrow --> S3["Step 3: Holds USDFC and Fees"] & S4["Step 4: Sends Link to B"] & S13["Step 13: Party A Checks or Approves<br>manual or automatic"] & S14["Step 14: On Success<br>Release Payment to B, Archive File to Cold Storage"] & S15["Step 15: Move File to Cold Storage"] & S17["Step 17: Collect Commission and Gas Fees"] & S18["Step 18: On Fail or Dispute<br>Refund Party A minus Fees"] & S19["Step 19: All Steps and Proofs Auditable<br>via Filecoin Onchain Logs"]
    S3 --> Bank["USDFC Vault<br>Wallet Payments"]
    S4 --> B["Party B<br>Developer / Seller"]
    B --> S5["Step 5: Accesses Upload Portal"]
    S5 --> Portal["Next.js Upload Portal"]
    Portal --> S6["Step 6: Encrypts ZK Contract Locally"]
    S6 --> Encrypt["Client-side Encryption<br>AES ChaCha"]
    Encrypt --> S7["Step 7: Upload Encrypted File"]
    S7 --> Warm["Filecoin Warm Storage<br>SynapseSDK"]
    Warm --> S8["Step 8: Store CID and Metadata to Escrow Contract"] & S9["Step 9: Notify Agent for Review"]
    S8 --> Escrow
    S9 --> Agent
    Agent --> S10["Step 10: Download and Decrypt File<br>in TEE Enclave if needed"] & S11["Step 11: Verify ZK Contract<br>Run Tests and Proofs"]
    S10 --> Warm
    S11 --> S12["Step 12: Submit Pass or Fail with Proof"]
    S12 --> Escrow
    S13 --> A
    S14 --> Bank
    S15 --> Cold["Filecoin Cold Storage<br>SynapseSDK"]
    Cold --> S16["Step 16: B Accesses File After Payment"]
    S16 --> B
    S17 --> FeePool["Commission and Fees"]
    S18 --> Bank
    S19 --> A

     Escrow:::escrow
     Agent:::agent
     Warm:::storage
     Cold:::storage
    classDef storage fill:#fff6db,stroke:#eab308
```

This detailed flow diagram illustrates:
- **Contract Creation & Signing:** Natural language contract review and digital signature processes (Steps 1-2b)
- **Escrow Management:** USDFC payment handling and smart contract orchestration (Steps 3-4)
- **Secure File Handling:** Client-side encryption, warm storage upload, and metadata management (Steps 5-8)
- **Agent Verification:** TEE-based decryption, ZK contract verification, and proof submission (Steps 9-12)
- **Settlement & Archival:** Payment release, cold storage archival, and audit trail creation (Steps 13-19)

Each step is cryptographically verifiable and permanently logged on Filecoin for complete transparency and auditability.
