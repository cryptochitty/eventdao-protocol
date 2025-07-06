# 🧾 Specification: Modular Governance Protocol

## 📁 Modules Overview

### ✅ Frontend (Next.js)

Responsible for interfacing with users.

- **Proposal UI**
  - List all proposals by stage.
  - View proposal details.
  - Create proposal drafts.
  - Trigger stage transitions (Draft → Review → Voting).

- **Profile Pages**
  - Display builder/sponsor info.
  - Link wallet/socials.
  - Show badges & reputation.

- **Voting Interface**
  - Render voting options.
  - Display real-time tallies.
  - Show voting status and banners.

- **Comment Threads**
  - Nested/threaded discussions per proposal.
  - CRUD comments tied to proposal ID.

---

### ✅ Backend (Node.js + Supabase)

Acts as the middleware between frontend, blockchain, and database.

#### Responsibilities:

- **Central API hub**
  - Abstracts smart contract calls.
  - Fetches and updates proposal states.

- **Data Store (Supabase)**
  - Proposals: `id`, `title`, `description`, `proposer`, `stage`, `timestamps`.
  - Profiles: `wallet`, `socials`, `bio`, `badges`, `reputation`.
  - Comments: `proposal_id`, `author`, `content`, `parent_id`, `timestamp`.
  - Votes: `proposal_id`, `voter`, `vote`, `timestamp`.

---

### ✅ MemberRegistry.sol

Stores member identity and reputation data.

#### Responsibilities:

- Map `wallet` → `display name`.
- Track `reputation score` (settable by authorized contracts).
- Validate proposers/voters.

---

### ✅ ProposalRegistry.sol

Manages proposal metadata and lifecycle.

#### Responsibilities:

- Store proposal state:
  - `Draft → Review → VotingOpen → VotingClosed → Executed/Rejected`
- Metadata:
  - `descriptionHash`, `callData`, `creator`, `timestamps`.
- Control stage transitions with `startVoting()`, `closeVoting()`, `executeProposal()`.

---

### ✅ VotingModule.sol

Handles core governance logic.

#### Responsibilities:

- Voting eligibility checks (reputation or token threshold).
- Record votes (`yes`, `no`) on-chain.
- Enforce quorum (`minQuorum`).
- Pass/fail logic:
  - `yesVotes / totalVotes ≥ passThreshold`.
- On success: trigger `ProposalRegistry.executeProposal()`.

---

### ✅ Reputation Logic

Flexible scoring system for builder contributions.

- Off-chain badge processing (events, GitHub, participation).
- Optional on-chain badge claim/verify.
- Updates `MemberRegistry.reputation`.

---

## 🔁 Data Flow Steps

### 1️⃣ Draft Creation
- Proposer submits draft via frontend.
- Backend stores metadata.
- Optionally stores draft hash in `ProposalRegistry`.

### 2️⃣ Community Discussion
- Comments posted to backend (Supabase).
- Threaded view tied to proposal ID.

### 3️⃣ Start Voting
- Proposer invokes `startVoting()` on-chain.
- Updates stage to `VotingOpen`.
- Backend reflects change in UI.

### 4️⃣ Voting Period
- Members vote via frontend → hits VotingModule.
- On-chain records vote.
- Backend syncs for real-time UI tally.

### 5️⃣ Voting Close
- `closeVoting()` checks:
  - `quorum met?`
  - `yes % ≥ threshold?`
- Status → `Passed` or `Rejected`.

### 6️⃣ Execution
- If passed → `executeProposal()` fires `callData`.
- Proposal stage → `Executed`.

### 7️⃣ Profile Stats
- Backend updates:
  - Proposals created.
  - Votes cast.
  - Badges earned.

### 8️⃣ Reputation Updates
- Events trigger off-chain/on-chain badge systems.
- New scores pushed to `MemberRegistry`.

---

## 🔑 Design Principles

- **Frontend decoupled from contracts** → communicates via secure API.
- **Backend stores off-chain metadata** → improves UX performance.
- **Modular smart contracts** → individually upgradeable.
- **Reputation system cross-domain** → encourages broader builder engagement.
