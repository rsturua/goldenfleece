# GoldenFleece Architecture Documentation

## Table of Contents
1. [Overview](#overview)
2. [Module Architecture](#module-architecture)
3. [Database Schema](#database-schema)
4. [Eligibility & Compliance System](#eligibility--compliance-system)
5. [Service Layer](#service-layer)
6. [Blockchain Integration](#blockchain-integration)
7. [API Routes](#api-routes)
8. [Security & Audit](#security--audit)
9. [Migration Guide](#migration-guide)

---

## Overview

GoldenFleece has been refactored from a monolithic structure into a production-ready modular architecture for RWA (Real World Assets) gold investment. The system is organized into domain-oriented modules with clear boundaries and responsibilities.

### Core Principles

- **Domain-Driven Design**: Code organized by business domain, not technical layers
- **Separation of Concerns**: Clear boundaries between UI, services, and data
- **Type Safety**: Strongly typed with Zod schemas for runtime validation
- **Auditability**: Immutable audit logs for all sensitive operations
- **Compliance-First**: KYC and eligibility integrated from the ground up
- **Blockchain-Ready**: Clean abstractions for smart contract integration

---

## Module Architecture

### Directory Structure

```
lib/domains/
├── auth/                    # Authentication (existing)
├── wallet/                  # Crypto wallet management
│   ├── models.ts           # WalletLink schema
│   └── service.ts          # Connection & verification logic
├── compliance/              # KYC and eligibility
│   ├── models.ts           # KycProfile, EligibilityState
│   └── service.ts          # Sumsub integration, eligibility checks
├── projects/                # Mining projects & offerings
│   ├── models.ts           # Project, Offering schemas
│   └── service.ts          # Project queries
├── investments/             # Investment creation & tracking
│   ├── models.ts           # Investment, PortfolioPosition
│   └── service.ts          # Investment lifecycle
├── portfolio/               # User portfolio management
│   └── service.ts          # Portfolio aggregation
├── payouts/                 # Dividend distribution
│   ├── models.ts           # PayoutCycle, PayoutRecord
│   └── service.ts          # Payout calculations & claims
├── admin/                   # Admin operations & audit
│   ├── models.ts           # AuditLog, UserRole
│   └── service.ts          # Role management, audit logging
└── shared/                  # Cross-domain utilities
    ├── types.ts            # Shared enums and types
    ├── schemas.ts          # Common Zod schemas
    ├── blockchain-interfaces.ts  # Smart contract abstractions
    ├── hooks/              # Shared React hooks
    └── utils/              # Utility functions
```

### Module Responsibilities

| Module | Purpose | Key Exports |
|--------|---------|-------------|
| **wallet** | Wallet connection & verification | `WalletService`, `WalletLink` |
| **compliance** | KYC verification & eligibility | `ComplianceService`, `EligibilityState` |
| **projects** | Project & offering management | `ProjectsService`, `Project`, `Offering` |
| **investments** | Investment lifecycle | `InvestmentsService`, `Investment` |
| **portfolio** | Portfolio aggregation | `PortfolioService`, `PortfolioPosition` |
| **payouts** | Dividend distribution | `PayoutsService`, `PayoutCycle` |
| **admin** | Audit & role management | `AdminService`, `createAuditLog` |

---

## Database Schema

### New Tables

#### `wallet_links`
**Purpose**: Manages wallet connections with separate verification state

```sql
Key fields:
- wallet_address: Ethereum address
- verified: Boolean (separate from connection)
- verification_nonce, verification_signature
- is_active: Current wallet status
```

**Why separate from profiles?**
- Users can link multiple wallets over time
- Connection state ≠ verification state
- Supports wallet switching and history

#### `kyc_profiles`
**Purpose**: Stores KYC verification data with Sumsub integration

```sql
Key fields:
- provider: 'sumsub' | 'manual' | 'other'
- provider_applicant_id: External KYC provider ID
- status: KYC verification status
- rejection_reason, reviewer_notes
```

#### `eligibility_states`
**Purpose**: **Source of truth** for investment eligibility

```sql
Key fields:
- status: EligibilityStatus (state machine)
- can_invest, can_withdraw, can_receive_dividends: Permission flags
- restriction_reason: Why user is restricted
```

**Status Flow**:
```
registered → wallet_connected → wallet_verified
           ↓
    kyc_pending → kyc_under_review → kyc_approved
                                           ↓
                                  investment_eligible
```

**Critical**: Access control uses `can_invest`, NOT `investor_tier`

#### `offerings`
**Purpose**: Tokenized investment opportunities for projects

```sql
Key fields:
- token_symbol, token_name
- total_tokens, available_tokens
- contract_address: Deployed smart contract (nullable)
- offering_start_date, offering_end_date
```

#### `portfolio_positions`
**Purpose**: Aggregated user holdings per project

```sql
Key fields:
- total_tokens, total_invested, average_token_price
- total_dividends_received, return_percentage
```

Auto-updated by triggers on `investments` and `payout_records`

#### `payout_cycles`
**Purpose**: Dividend distribution periods

```sql
Key fields:
- total_amount, tokens_eligible, amount_per_token
- total_gold_produced (optional)
- distribution_tx_hash: Blockchain transaction
```

#### `payout_records`
**Purpose**: Individual investor payouts

```sql
Key fields:
- tokens_held, amount_due
- is_claimed: Claim-based distribution
- tx_hash: Blockchain payment proof
```

#### `audit_logs`
**Purpose**: **Immutable** audit trail

```sql
Key fields:
- event_type: Sensitive operation type
- user_id, actor_id: Who and whom
- previous_state, new_state: Before/after snapshots
- ip_address, user_agent: Context
```

**Append-only**: No updates or deletes allowed

#### `user_roles`
**Purpose**: Role-based access control

```sql
Roles:
- user: Default
- admin: Project management
- compliance_officer: KYC reviews
- super_admin: Full system access
```

### Reconciliation

#### `profiles` table (existing)
**Added fields**:
- `investor_tier`: UI/segmentation only (deprecated for access control)
- `crypto_wallet_address`: Deprecated - migrating to `wallet_links`

**IMPORTANT**:
- `investor_tier` is for UI badges only
- Access control uses `eligibility_states.can_invest`
- Existing wallet data will migrate to `wallet_links` table

---

## Eligibility & Compliance System

### State Machine

User eligibility follows a strict state machine:

```
┌─────────────┐
│ registered  │ (account created)
└─────┬───────┘
      ↓
┌─────────────────┐
│ wallet_connected│ (wallet linked)
└─────┬───────────┘
      ↓
┌──────────────┐
│wallet_verified│ (signature verified)
└─────┬────────┘
      ↓
┌────────────┐      ┌──────────────┐
│ kyc_pending├─────→│kyc_under_review│
└────────────┘      └──────┬───────┘
                          ↓
                   ┌──────────────┐
                   │ kyc_approved │
                   └──────┬───────┘
                          ↓ (if wallet verified)
                   ┌──────────────────┐
                   │investment_eligible│ ✓ CAN INVEST
                   └───────────────────┘
```

### Permission Flags

Derived from eligibility status:

| Status | can_invest | can_withdraw | can_receive_dividends |
|--------|-----------|-------------|---------------------|
| registered | ❌ | ❌ | ❌ |
| wallet_connected | ❌ | ❌ | ❌ |
| wallet_verified | ❌ | ❌ | ❌ |
| kyc_pending | ❌ | ❌ | ❌ |
| kyc_under_review | ❌ | ❌ | ❌ |
| kyc_approved | ❌ | ✅ | ✅ |
| **investment_eligible** | **✅** | **✅** | **✅** |
| restricted | ❌ | ❌ | ❌ |
| suspended | ❌ | ❌ | ❌ |

### KYC Integration (Sumsub)

**Flow**:
1. User submits KYC → `ComplianceService.createSumsubApplicant()`
2. Sumsub reviews → Webhook: `POST /api/compliance/webhook`
3. Webhook updates `kyc_profiles` and `eligibility_states`
4. Audit log created automatically

**Webhook Payload**:
```typescript
{
  applicantId: string;
  reviewResult: {
    reviewAnswer: 'GREEN' | 'RED' | 'YELLOW';
    rejectLabels?: string[];
  }
}
```

**Implementation Status**:
- ✅ Database schema ready
- ✅ Service layer interfaces
- ✅ Webhook endpoint
- ⏳ Sumsub SDK integration (TODO)

---

## Service Layer

### Design Patterns

**Services are stateless and encapsulate business logic**:
- No direct Supabase calls from UI components
- Input validation with Zod schemas
- Automatic audit logging for sensitive operations
- Type-safe interfaces

### Example: Investment Creation

```typescript
// Before (direct Supabase in component):
const { data } = await supabase.from('investments').insert({ ... });

// After (service layer):
const investment = await InvestmentsService.createInvestment({
  userId,
  projectId,
  amount,
  tokensPurchased,
});
// ✓ Eligibility checked automatically
// ✓ Token availability verified
// ✓ Audit log created
// ✓ Portfolio updated by trigger
```

### Service APIs

#### WalletService
```typescript
connectWallet(input: CreateWalletLinkInput): Promise<WalletLink>
verifyWallet(input: VerifyWalletInput): Promise<WalletLink>
getActiveWallet(userId: string): Promise<WalletLink | null>
hasVerifiedWallet(userId: string): Promise<boolean>
disconnectWallet(userId: string): Promise<void>
```

#### ComplianceService
```typescript
getOrCreateKycProfile(userId: string): Promise<KycProfile>
createSumsubApplicant(userId, userData): Promise<string>
handleSumsubWebhook(payload: SumsubWebhookPayload): Promise<void>
getEligibility(userId: string): Promise<EligibilityState>
updateEligibility(userId, newStatus, reason?): Promise<void>
isEligibleToInvest(userId: string): Promise<boolean>
```

#### InvestmentsService
```typescript
createInvestment(input: CreateInvestmentInput): Promise<Investment>
completeInvestment(investmentId: string): Promise<Investment>
getUserInvestments(userId: string): Promise<Investment[]>
```

#### PortfolioService
```typescript
getUserPortfolio(userId: string): Promise<PortfolioPosition[]>
getPortfolioSummary(userId): Promise<{
  totalInvested,
  totalReturn,
  returnPercentage,
  activePositions
}>
```

#### PayoutsService
```typescript
getUserPayouts(userId: string): Promise<PayoutRecord[]>
getClaimableBalance(userId, projectId?): Promise<ClaimableBalance>
claimPayout(payoutRecordId, userId): Promise<PayoutRecord>
calculatePayoutCycle(cycleId): Promise<void> // Admin only
```

---

## Blockchain Integration

### Architecture Decision: EVM-Compatible

**Chosen**: Ethereum / Polygon / Base
**Token Standard**: ERC20 (upgradeable to ERC1400 for securities)

**Why EVM?**
- Mature ecosystem and tooling
- Wide wallet support (MetaMask, Coinbase, etc.)
- Cost-effective L2 options (Polygon, Base)
- Compatibility with existing RWA standards

### Module Boundaries

Located in `lib/domains/shared/blockchain-interfaces.ts`

**Key Interfaces**:

```typescript
ITokenizationService {
  deployToken(input: TokenizationInput): Promise<TokenizationResult>
  mintTokens(contractAddress, to, amount, chainId): Promise<Hash>
  getBalance(contract, holder, chainId): Promise<bigint>
}

IDividendService {
  distributeDividends(input: DividendDistributionInput): Promise<Result>
  calculateEntitlements(contract, totalAmount, chainId): Promise<Entitlements[]>
}

IEventSyncService {
  syncEvents(contract, fromBlock, toBlock, chainId): Promise<BlockchainEvent[]>
  processEvent(event: BlockchainEvent): Promise<void>
}
```

### Implementation Status

**Current**: Placeholder implementations
**Next Steps**:
1. Deploy ERC20 token contracts (Solidity/Foundry)
2. Implement `TokenizationService` with viem
3. Set up event sync for token transfers
4. Integrate payout distribution contracts

**Supported Chains**:
- Ethereum (chainId: 1)
- Polygon (chainId: 137)
- Base (chainId: 8453)

### Smart Contract TODOs

```solidity
// TODO: Implement in /contracts directory

contract GoldenFleeceToken is ERC20 {
  // Offering-specific token with dividend tracking
}

contract DividendDistributor {
  // Batch dividend distribution
  // Claim-based or push-based
}

contract TokenFactory {
  // Deploy new offering tokens
}
```

---

## API Routes

### Wallet Routes

**POST `/api/wallet/connect`**
- Connect wallet to user account
- Creates `wallet_link` record
- Updates eligibility to `wallet_connected`

**POST `/api/wallet/verify`**
- Verify wallet ownership with signature
- Validates signature server-side
- Updates eligibility to `wallet_verified`

### Compliance Routes

**POST `/api/compliance/webhook`**
- Sumsub webhook handler
- Processes KYC review results
- Updates `kyc_profiles` and `eligibility_states`
- Creates audit logs

### Investment Routes (TODO)

**POST `/api/investments/create`**
- Create new investment
- Checks eligibility
- Validates offering availability

**POST `/api/investments/{id}/complete`**
- Mark investment as completed (after payment)

### Payout Routes (TODO)

**POST `/api/payouts/claim/{id}`**
- Claim pending payout
- Initiates transfer to wallet

---

## Security & Audit

### Audit Logging

**All sensitive operations create audit logs**:
- Wallet linked/unlinked/verified
- KYC status changes
- Eligibility changes
- Investments created/completed
- Payouts claimed
- Admin actions

**Audit Log Structure**:
```typescript
{
  event_type: AuditEventType,
  user_id: UUID,
  actor_id: UUID, // Who performed the action
  description: string,
  previous_state: JSONB,
  new_state: JSONB,
  ip_address, user_agent, request_id,
  timestamp: TIMESTAMPTZ
}
```

**Append-Only**: Updates and deletes blocked by RLS policies

### Role-Based Access Control

**Roles**:
- `user`: Default, basic access
- `admin`: Manage projects, offerings
- `compliance_officer`: Review KYC, manage eligibility
- `super_admin`: Full system access

**Helper Functions**:
```typescript
AdminService.hasRole(userId, role): Promise<boolean>
AdminService.isAdmin(userId): Promise<boolean>
AdminService.grantRole(userId, role, grantedBy): Promise<UserRole>
```

### Row-Level Security

All tables have RLS enabled:
- Users can only see own data
- Admins have elevated access via role checks
- Audit logs are append-only
- Projects visible to all (based on status)

---

## Migration Guide

### For Developers

**Old Pattern** (deprecated):
```typescript
// Checking investor tier
const { investorTier } = useWalletStatus();
if (investorTier === 'investor') { ... }
```

**New Pattern**:
```typescript
// Using eligibility system
const { canInvest } = useEligibility();
if (canInvest) { ... }
```

**Old Pattern**:
```typescript
// Direct Supabase calls
const { data } = await supabase.from('investments').insert({ ... });
```

**New Pattern**:
```typescript
// Service layer
const investment = await InvestmentsService.createInvestment({ ... });
```

### Database Migration

**Run migrations in order**:
1. `001_reconcile_profiles.sql` - Add missing profile fields
2. `002_create_wallet_links.sql` - New wallet management
3. `003_create_kyc_compliance.sql` - KYC & eligibility
4. `004_create_offerings.sql` - Tokenized offerings
5. `005_create_portfolio_positions.sql` - Portfolio aggregation
6. `006_create_payout_tables.sql` - Dividend system
7. `007_create_audit_admin_tables.sql` - Audit & RBAC

**Apply**:
```bash
# Via Supabase CLI
supabase db push

# Or manually in SQL editor
```

### Breaking Changes

**Minimal** - existing features preserved:

| Old | New | Status |
|-----|-----|--------|
| `investor_tier` access control | `eligibility_states.can_invest` | Deprecated |
| `profiles.crypto_wallet_address` | `wallet_links` table | Migrating |
| Direct Supabase calls | Service layer | Recommended |

**No user-facing changes** - existing flows still work

---

## Future Enhancements

### Immediate (MVP)
- [ ] Complete Sumsub SDK integration
- [ ] Deploy ERC20 token contracts
- [ ] Implement tokenization service
- [ ] Build investment flow UI

### Medium-Term
- [ ] Batch dividend distribution
- [ ] Secondary market for tokens
- [ ] Mobile app (React Native)
- [ ] Multi-currency support

### Long-Term
- [ ] DAO governance for platform decisions
- [ ] Cross-chain bridge support
- [ ] Institutional investor features
- [ ] Automated compliance monitoring

---

**Architecture Version**: 2.0
**Last Updated**: March 6, 2026
**Status**: Production-Ready Foundation
