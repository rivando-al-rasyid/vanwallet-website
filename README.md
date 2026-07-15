# vanwallet-website

VanWallet is a modern, mobile-responsive, full-stack digital e-wallet application. It allows users to securely manage their digital balance, perform instant peer-to-peer transfers, top up their accounts, and monitor their financial habits with intuitive visual analytics.

This monorepo (or repository structure) contains both the React frontend and the Node.js backend.

---

## 🏗️ Critical Architectural Decisions (FinTech Standards)

To build a secure, production-ready e-wallet, VanWallet bypasses common beginner pitfalls in favor of robust engineering practices:

### 1. Ledger-Based vs. Balance-Column Architecture
* **The Pitfall:** Storing balance as a single mutable `balance` column in a `users` table (e.g., `UPDATE users SET balance = balance - 50`). This is highly susceptible to race conditions and makes financial audits impossible.
* **Our Solution:** **Double-Entry Bookkeeping**. The user's balance is a *derived value* calculated by summing up immutable transaction records. Every transaction has a source wallet (debit) and a destination wallet (credit). 

### 2. Concurrency Control (Race Condition Prevention)
* To prevent a user from rapidly clicking "Transfer" twice and successfully executing a double-spend (withdrawing money they don't have), the backend utilizes **PostgreSQL database transactions** with strict locking (`SELECT ... FOR UPDATE`) or optimistic concurrency control to isolate and serialize wallet updates.

### 3. Secure Token Storage
* **No `localStorage` for session tokens.** Authentication is strictly handled via HttpOnly, Secure, SameSite cookies to mitigate Cross-Site Scripting (XSS) attacks.

---

## Tech Stack

### Frontend
- **Framework:** React `19` (Vite)
- **Styling:** Tailwind CSS `v4` (fully optimized for mobile-first views)
- **State Management:** Redux Toolkit (for predictable transactional state)
- **Icons:** Lucide React

### Backend
- **Runtime:** Node.js (Express)
- **Database:** PostgreSQL (selected over MongoDB for ACID compliance and transactional safety)
- **Authentication:** JWT (JSON Web Tokens) inside HttpOnly cookies
- **Validation:** Zod (runtime request schema validation)

---

## Database Schema (Simplified Ledger)

Below is the relational layout designed to ensure transaction integrity.

```mermaid
erDiagram
    USERS ||--|| PROFILES : has
    USERS ||--|| WALLETS : owns
    WALLETS ||--o{ TRANSACTIONS : initiates
    WALLETS ||--o{ TRANSACTIONS : receives

    USERS {
        uuid id PK
        string email UK
        string password_hash
        timestamp created_at
    }
    WALLETS {
        uuid id PK
        uuid user_id FK
        numeric balance "Cached derived balance for fast reads"
        string currency
        timestamp updated_at
    }
    TRANSACTIONS {
        uuid id PK
        uuid sender_wallet_id FK
        uuid receiver_wallet_id FK
        numeric amount
        string status "pending | completed | failed"
        string type "transfer | topup | withdrawal"
        string description
        timestamp created_at
    }

Features

    Responsive Mobile-First Dashboard: Fully optimized for mobile screens using Tailwind v4 fluid utilities.

    Secure P2P Transfers: Instantly send money to other users via their email or wallet ID.

    Transaction History & Ledger: Search, filter, and paginate past transactions with clear visual debit/credit indicators.

    Dynamic Analytics: Monthly income vs. expense tracking.

    Interactive Top-up Sandbox: Simulates credit card or bank transfer top-ups using clean mock-payment flows.

Getting Started
Prerequisites

    Node.js v18+ or v20+

    PostgreSQL 15+ (locally running or Docker instance)

Local Installation
1. Clone the repository
Bash

git clone [https://github.com/yourusername/vanwallet.git](https://github.com/yourusername/vanwallet.git)
cd vanwallet

2. Backend Setup
Bash

cd backend
npm install
cp .env.example .env

Fill out your .env variables:
Code snippet

PORT=5000
DATABASE_URL=postgresql://postgres:password@localhost:5432/vanwallet
JWT_SECRET=super_secret_high_entropy_key_here
COOKIE_SECURE=false # Set to true in production with HTTPS
ALLOWED_ORIGIN=http://localhost:5173

Run migrations and start the server:
Bash

npm run db:migrate
npm run dev

3. Frontend Setup
Bash

cd ../frontend
npm install
cp .env.example .env

Configure .env:
Code snippet

VITE_API_URL=http://localhost:5000

Start the frontend dev server:
Bash

npm run dev

Core API Endpoints
Auth
Method	Endpoint	Description
POST	/api/v1/auth/register	Register new user + auto-create default Wallet
POST	/api/v1/auth/login	Authenticate user & issue HttpOnly JWT
POST	/api/v1/auth/logout	Revoke session and clear cookies
Wallets & Transactions
Method	Endpoint	Description
GET	/api/v1/wallet/balance	Get current balance and metadata
POST	/api/v1/wallet/transfer	Execute secure P2P transfer (ACID Compliant)
GET	/api/v1/wallet/transactions	Paginated and filtered transaction history
POST	/api/v1/wallet/topup	Initiate simulated fund deposit
Security Checklist (Production Readiness)

Before deploying this to production, ensure these protocols are in place:

    [ ] SSL/TLS (HTTPS): Absolutely mandatory. Cookies must have Secure=true enabled.

    [ ] Rate Limiting: Set up limits on /api/v1/wallet/transfer and auth routes to prevent brute-force attacks and transaction spam.

    [ ] CORS Protection: Limit backend origins specifically to your production frontend domain.

    [ ] Data Encryption: Encrypt sensitive database columns (like profile details or bank credentials) at rest.

License

This project is licensed under the MIT License.