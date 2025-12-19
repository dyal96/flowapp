# 🌊 Flow Accounting (Personal Edition) | Full Development Stack 

**Flow Accounting** is a premium, portable and privacy focus personal finance application designed for privacy, speed, and ease of use. Built with a "local-first" philosophy, your financial data never leaves your device and is encrypted securely in your browser.

Try Prototype [Here](https://github.com/dyal96/flow)

![Dashboard Preview](https://github.com/dyal96/flow/blob/main/demo/screenshot.jpg)

## ✨ Key Features

### 🔒 Privacy & Security
- **Client-Side Encryption:** All data used in the app is encrypted using industry-standard Web Crypto API before being stored in your browser's LocalStorage.
- **Offline First:** Works completely offline. No server, no cloud, no subscriptions.
- **Password Protection:** Secure your financial data with a login password.
- **Recovery Options:** Set up security keywords to recover your account if you forget your password.

### 💸 Financial Management
- **Dashboard:** Real-time overview of your Net Worth, Income vs Expenses, and Monthly Cash Flow.
- **Comprehensive Accounts:** Manage Bank Accounts, Cash, Credit Cards, Loans, Investments (FDs), and "Person" accounts (for tracking debts/receivables).
- **Transactions:** Quick and easy entry for Income, Expenses, and Transfers.
- **Categorization:** Custom categories for detailed tracking.
- **Budgeting:** Set monthly budgets per category and track your progress with visual progress bars.

### 📊 Reports & Analytics
- **Visual Charts:** Interactive charts for Spending Breakdown, Income Analysis, and Monthly Trends.
- **Filters:** Powerful filtering by date range, account, type, and category.
- **Export Data:** Export your data to **CSV** (Excel) or **PDF** reports.
- **SQL Export:** For advanced users, export your entire database as SQL commands.

### ⚡ Power User Features
- **Keyboard Shortcuts:** Navigate the entire app without a mouse. Press `?` to view the cheat sheet.
  - `Q` / `N`: Quick Add Transaction
  - `D`: Dashboard
  - `A`: Accounts
  - `T`: Transactions
  - `/`: Open Search
- **Dark Mode:** Beautifully designed Dark Mode for comfortable night usage.
- **Custom Themes:** Choose from multiple themes (Default Blue, Sea Blue, Pink, Pastels).
- **Import/Backup:** Easy JSON backup and restore functionality.

---

## 📋 **Planned Architecture**

```
┌─────────────────────────────────────────────────────────────────┐
│                         DOCKER COMPOSE                          │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────────────┐ │
│  │   Frontend   │   │   Backend    │   │      Database        │ │
│  │   (Nginx)    │◄──┤   (Node.js)  │◄──┤   (PostgreSQL /      │ │
│  │   Static     │   │   Express    │   │    SQLite)           │ │
│  │   SPA        │   │   REST API   │   │                      │ │
│  └──────────────┘   └──────────────┘   └──────────────────────┘ │
│         ▲                   ▲                                   │
│         │                   │                                   │
│    Port 80/443        Port 3001                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔧 **Phase 1: Backend Development**

### Recommended Tech Stack:
| Component | Technology | Reason |
|-----------|------------|--------|
| **Runtime** | Node.js + Express.js | JavaScript consistency with frontend |
| **Database** | SQLite (dev) → PostgreSQL (prod) | Easy to start, scales well |
| **ORM** | Prisma or Sequelize | Type-safe, migration support |
| **Auth** | JWT + Bcrypt | Secure, stateless authentication |
| **API** | REST or GraphQL | REST is simpler for this use case |

### Database Schema (Core Tables):
```sql
-- Users
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR UNIQUE NOT NULL,
  password_hash VARCHAR NOT NULL,
  name VARCHAR,
  created_at TIMESTAMP,
  settings JSONB  -- Theme, preferences
);

-- Accounts
CREATE TABLE accounts (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  name VARCHAR NOT NULL,
  type VARCHAR,  -- asset, liability, credit_card, general
  opening_balance DECIMAL,
  notes TEXT,
  created_at TIMESTAMP
);

-- Transactions
CREATE TABLE transactions (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  account_id UUID REFERENCES accounts(id),
  to_account_id UUID,  -- For transfers
  type VARCHAR,  -- income, expense, transfer
  amount DECIMAL NOT NULL,
  description TEXT,
  category_id UUID REFERENCES categories(id),
  date DATE,
  is_recurring BOOLEAN,
  recurring_config JSONB,
  created_at TIMESTAMP
);

-- Categories, Investments, Loans, RDs tables...
```

---

## 🔧 **Phase 2: Refactor Frontend**

### Option A: Keep Vanilla JS (Recommended for simplicity)
- Split modules:
  ```
  /frontend
    /js
      /api.js          # API calls
      /auth.js         # Login/logout
      /store.js        # State management
      /router.js       # SPA routing
      /components/     # UI components
    /css
    index.html
  ```

### Option B: Migrate to Framework (For scale)
- **React/Vue/Svelte** with Vite for a modern toolchain
- Better component architecture and state management

---

## 🔧 **Phase 3: Multi-User Features**

### Essential Features:
1. **User Registration & Login**
   - Email + password signup
   - JWT token-based sessions
   - Password reset via email

2. **Data Isolation**
   - Every query filters by `user_id`
   - Row-level security in database

3. **User Settings**
   - Theme preferences (synced to cloud)
   - Currency/locale settings
   - Export/import user data

4. **Optional: Family/Shared Accounts**
   - Link multiple users to shared accounts
   - Role-based permissions (admin/viewer)

---

## 🐳 **Phase 4: Docker Deployment**

### Project Structure:
```
/FlowApp
├── docker-compose.yml
├── /backend
│   ├── Dockerfile
│   ├── package.json
│   ├── src/
│   └── prisma/
├── /frontend
│   ├── Dockerfile
│   ├── nginx.conf
│   └── src/
└── /database
    └── init.sql
```

### Sample `docker-compose.yml`:
```yaml
version: '3.8'
services:
  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      - backend

  backend:
    build: ./backend
    ports:
      - "3001:3001"
    environment:
      DATABASE_URL: postgres://postgres:password@db:5432/flowapp
      JWT_SECRET: your-secret-key
    depends_on:
      - db

  db:
    image: postgres:15-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql
    environment:
      POSTGRES_DB: flowapp
      POSTGRES_PASSWORD: password

volumes:
  pgdata:
```

---

## 📝 **What Should We Can Choose?**

Before Building this app you should plan, so you need to understand your preferences:

1. **Backend Language Preference?**
   - Node.js/Express (recommended for JS consistency)
   - Python/FastAPI (great for data analytics features)
   - Go (if performance is critical)

2. **Database Preference?**
   - PostgreSQL (recommended for production)
   - SQLite (simpler, good for single-server deployment)
   - MySQL

3. **Frontend Approach?**
   - Keep vanilla JS/HTML and modularize
   - Migrate to React/Vue/Svelte

4. **Authentication Requirements?**
   - Simple email/password only
   - Add OAuth (Google, GitHub login)
   - Two-factor authentication (2FA)

5. **Additional Features?**
   - Email notifications (reminders for recurring transactions)
   - Data backup/restore to cloud
   - Mobile-responsive PWA optimization
   - API for mobile app integration later

6. **Deployment Target?**
   - Self-hosted Docker (home server, VPS)
   - Cloud provider (AWS, DigitalOcean, etc.)
   - Both

---

# Final Architectural Roadmap: Flow Accounting (Open Source) 

This plan outlines the transition from a monolithic prototype to a professional, offline-first, multi-user application suitable for open-source distribution and Docker deployment.

## Core Philosophy: "Low Weight, High Efficiency"

- **Low Dependency**: Minimize NPM packages. Use vanilla JS and Go for speed and simplicity.
- **Offline First**: All actions work instantly locally; sync happens in the background.
- **Privacy Centric**: Local-first storage with optional server sync.

## Proposed Stack

### 1. Frontend (The "App")

- **Language**: Vanilla JavaScript (ES Modules).
- **Styling**: Tailwind CSS (Utility-first, minimal custom CSS).
- **Storage**: **IndexedDB** using a lightweight wrapper like `idb-keyval`.
- **Mobile**: **PWA (Progressive Web App)**. This provides a "native-like" experience on iOS/Android without the weight of React Native or Cordova.
- **Charts**: Chart.js (already in use).

### 2. Backend (The "Sync Server")

- **Language**: **Go (Golang)**.
    - _Why_: Extremely fast, low memory usage, compiles to a single binary (perfect for Docker), and very easy for others to host.
- **API**: RESTful API with JSON over HTTP.
- **Database**: **SQLite** (with WAL mode).
    - _Why_: Incredible performance for personal accounting, easy backups (one file), and scales to millions of users with proper partitioning.

### 3. Sync Logic (The "Magic")

- **Change Log Pattern**:
    1. Every action (Add transaction, edit account) creates a "Change Event" in a local `pending_sync` table.
    2. A background worker picks up these events and pushes them to the server.
    3. The server validates and applies them to the master record.
    4. Server sends back a "Last Seen" timestamp to keep local and remote in sync.

---

## Proposed Changes

### [New Layer] Data Isolation & Reconciliation

- **Feature**: Month Locking.
- **Implementation**: Add a `locked_until` field in User Settings. Backend/Frontend will block edits to transactions before this date.

### [New Feature] Sparklines

- **Implementation**: Integrate micro-charts into the account list rows using `<canvas>` or SVG, pulling the last 30 days of balance history.

### [New Feature] Cash Flow Forecast

- **Implementation**: logic layer that project recurring transactions (salaries, bills) onto a future timeline.


