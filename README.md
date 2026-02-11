# Medio

> **“We stop Solana scams before they happen — not after funds are gone.”**

Every month, tens of thousands of users lose funds on Solana. Not because they’re stupid, but because:

- Approvals live forever
- Transactions are unreadable  
- Wallets don’t explain intent  
- “Sign” is just a green button

**Medio** sits between **humans and blockchains**. It thinks *before* you sign.

---

## 1. Problem

Normal users cannot safely understand Solana transactions.

- A Jupiter “swap” transaction can:
  - Set infinite token approvals
  - Transfer SOL to a drainer
  - Create hidden accounts
- Wallets show partial summaries, but:
  - They don’t know your history
  - They don’t track your approvals as “risk debt”
  - They don’t detect when a dApp’s UI lies about what a tx does

Result: **wallet drainers and malicious tx builders win by default**.

---

## 2. Solution – Medio as an Autonomous Wallet Mediator

> Medio is an autonomous wallet advisor that inspects, simulates, summarizes, and scores Solana transactions before a user signs.

It is:

- **Agentic** – runs simulations and checks automatically
- **Autonomous** – tracks your wallet over time, not just single txs
- **Preventive** – stops bad txs before they execute
- **Advisory & protective** – it recommends, warns, and blocks *by advice*, but never signs for you

Medio:

1. Receives an **unsigned transaction** from a dApp or wallet.
2. **Simulates** it on Solana (via Helius).
3. Compares:
   - What the UI claims (“Swap 1 SOL to USDC on Jupiter”)
   - What actually happens on-chain (accounts, balances, approvals).
4. Produces:
   - Human‑readable summary  
   - Risk analysis  
   - **Wallet hygiene score** update  
   - Recommendation: `allow`, `warn`, or `block`.

---

## 3. Core Features

### 🔴 1. Approval Risk Debt Cleaner

**Real problem:** Infinite approvals and forgotten permissions are stealth rug pulls waiting to happen.

**What Medio does:**

- Tracks historical approvals via Helius / RPC.
- Flags:
  - Infinite approvals
  - Approvals to unknown or flagged programs
  - Approvals unused for N days
- Ranks them by **blast radius**.

Example message:

> “This token approval allows unlimited transfers by Program X.  
> You last used it 94 days ago. If compromised, you could lose your entire balance.”

Medio doesn’t just say “revoke” — it explains **why**.

---

### 🧪 2. Transaction Simulation & Intent Mismatch Detection

**Non‑negotiable core.**

For every transaction, Medio:

- Simulates with a fresh blockhash and actual accounts.
- Extracts:
  - Expected vs actual balance changes
  - New accounts created
  - Programs invoked
  - Hidden approvals or drains

Then it compares:

- **UI intent**: `“Swap 1 SOL to USDC on Jupiter”`
- **On‑chain reality** from simulation.

If they don’t match → 🚨

Example:

> “This transaction claims to mint an NFT, but simulation shows a 1.2 SOL transfer to an unrelated address and a new token approval.”

This is how we stop **drainers** at the point of signing.

---

### 🧠 3. Wallet Memory & Account Summarization

Medio builds a **memory** of your wallet:

- Frequent programs you safely use (Jupiter, Kamino, Raydium, Solana Pay).
- Past interactions that looked scammy.
- Patterns of blind signing.

It categorizes accounts as:

- ✅ Trusted  
- ⚪ Unknown  
- 🔴 High Risk

Examples:

> “You’ve interacted with this program 12 times via Jupiter without issues. It is commonly used for swaps.”

> “This account has no prior history in your wallet and was created recently. Similar patterns appear in known drainer contracts.”

This makes the experience feel like a **living mediator**, not a static explorer.

---

### ⭐ 4. Wallet Hygiene Score

We expose a simple **wallet hygiene score** (0–100):

- Starts at 100.
- Subtract points for:
  - Infinite approvals
  - Signing transactions with high‑risk flags
  - Blind signing patterns
- Add points for:
  - Revoking approvals
  - Simulating before signing
  - Avoiding recommended‑blocked txs

Framed as:

> “Your wallet hygiene score – how safely you’re using Solana.”

Not a credit score. A **safety meter**.

---

## 4. How Medio Uses Solana’s Strengths

Medio is built specifically for Solana’s **speed, low fees, and composability**.

- **Protocols integrated / demoed:**
  - **Jupiter** – swaps and routing
  - (Extendable to Raydium, Kamino, Meteora, Solana Pay, etc.)
- **Infrastructure:**
  - **Helius** – RPC and simulation
  - **AgentWallet** – wallet management, signing, devnet funding
  - **@solana/web3.js** and **@solana/kit** – transaction parsing, account discovery
- **Why Solana?**
  - Cheap, fast simulations → feasible to simulate *every* tx before signing.
  - Rich DeFi ecosystem → many protocols to protect users on.

Medio is **protocol‑agnostic**: Jupiter today; any Solana dApp that builds transactions tomorrow.

---

## 5. Architecture

### High‑Level

- **Frontend:** Next.js (App Router) + Tailwind
  - Connects to wallet (AgentWallet / standard adapters)
  - Intercepts transactions before `signAndSend`
  - Shows Medio’s analysis in a clean modal.

- **Backend:** Node.js + Express (TypeScript)
  - `POST /analyze-tx` – simulate, analyze, explain, recommend
  - `GET /wallet-approvals` – approval risk debt
  - `GET /wallet-score` – hygiene score
  - Integrations:
    - Helius RPC for simulation & history
    - OpenRouter (LLM) for natural language explanations

- **Chain:** Solana devnet + mainnet-compatible flows.

### Transaction Flow (Jupiter Example)

1. dApp builds a Jupiter swap transaction using `@jup-ag` + `@solana/kit`.
2. Instead of calling `signAndSend` directly, it:
   - Serializes the unsigned tx.
   - Sends it to `POST /analyze-tx` with context:
     - User wallet
     - UI label: “Swap 1 SOL to USDC on Jupiter”.
3. Medio:
   - Simulates tx via Helius.
   - Runs risk checks (unexpected SOL out, new unknown accounts, approvals).
   - Compares UI intent vs simulation results.
   - Updates wallet hygiene score.
4. Frontend shows Medio’s modal:
   - Summary, risks, hygiene impact, recommendation.
5. User chooses:
   - **Proceed** → then wallet signs.
   - **Reject** → drain avoided.

---

## 6. Demo Flows

We show two live flows:

1. **Safe Jupiter Swap**
   - Medio: “This looks like a standard Jupiter swap from 1 SOL to ~100 USDC.”
   - Risks: low.
   - Decision: ✅ Allow.
   - Hygiene score: small positive bump.

2. **Drainer / Malicious Swap**
   - UI claims “mint NFT” or “cheap swap”.
   - Simulation reveals:
     - Large SOL outflow to unknown account.
     - New unlimited approvals.
   - Medio:
     - Flags intent mismatch.
     - Explains risks in plain language.
     - Decision: 🚫 Block (strong warning).
   - Hygiene score: avoids a severe penalty by not signing.

---

## 7. Why This Isn’t Just Another Explorer or Wallet

- **Pre‑sign, not post‑mortem** – we mediate before the transaction hits the chain.
- **User‑specific memory** – we learn your wallet patterns; explorers don’t.
- **Protocol‑agnostic risk brain** – one agent that understands Jupiter, Raydium, Kamino, Solana Pay, etc.
- **Agentic behavior** – Medio acts; it doesn’t just display raw data.

This is the kind of agent that **only makes sense in an agent‑native hackathon**: it runs 24/7, reads your txs, learns your behavior, and protects you without you having to think about it.

---

## 8. Roadmap (Post‑Hackathon)

- Integrate more Solana DeFi protocols (Kamino, Raydium, Meteora).
- Deeper wallet behavior analytics (pattern‑based scam detection).
- Optional: use hygiene score as a signal in **on‑chain credit** (ClawCredit, etc.).
- Browser extension / direct wallet plugin so Medio can sit in front of any Solana dApp.

---

If you want, next I can:

- Trim this into a **90-second spoken pitch** script, or  
- Start giving you the actual `express` + TypeScript backend scaffolding (`/analyze-tx` + basic risk engine).# medio
