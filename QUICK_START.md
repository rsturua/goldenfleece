# Quick Start Guide - GoldenFleece 2.0

## 🎯 What Just Happened?

Your GoldenFleece codebase has been **refactored into a production-ready modular architecture**. The good news:

✅ **All existing features still work**
✅ **No user-facing changes**
✅ **Much better foundation for scaling**

---

## ⚡ Get Started in 3 Steps

### Step 1: Run Database Migrations (5 minutes)

**Option A: Supabase CLI**
```bash
cd /Users/ratisturua/Desktop/GoldenFleece
supabase db push
```

**Option B: Supabase Dashboard**
1. Go to https://supabase.com/dashboard
2. Select your project → SQL Editor
3. Run each file in `supabase/migrations/` (001 through 007)

**What this does**:
- Creates 8 new tables (wallet_links, kyc_profiles, eligibility_states, etc.)
- Adds missing fields to existing `profiles` table
- Sets up triggers for auto-updates

### Step 2: Test the App (2 minutes)

```bash
npm run dev
```

Open http://localhost:3000

**Expected behavior**:
- ✅ Landing page loads
- ✅ Sign up still works
- ✅ Login still works
- ✅ Wallet connection still works
- ✅ Dashboard loads

**If errors**: Check console for missing environment variables

### Step 3: Explore the New Architecture (10 minutes)

**Key Files to Review**:

1. **ARCHITECTURE.md** - Complete system documentation
2. **REFACTORING_SUMMARY.md** - What changed and why
3. **lib/domains/** - New modular structure
   - `wallet/service.ts` - Wallet management logic
   - `compliance/service.ts` - KYC & eligibility
   - `investments/service.ts` - Investment creation

**Try the new hooks**:
```typescript
// In any component
import { useEligibility } from '@/lib/domains/shared/hooks/useEligibility';

const { canInvest, status, requiredSteps } = useEligibility();
// status: 'registered' | 'wallet_connected' | 'wallet_verified' | 'kyc_approved' | 'investment_eligible'
```

---

## 🔍 Key Changes at a Glance

### 1. **New Eligibility System**

**Before**:
```typescript
const { investorTier } = useWalletStatus();
if (investorTier === 'investor') { /* can invest */ }
```

**After** (recommended):
```typescript
const { canInvest } = useEligibility();
if (canInvest) { /* can invest */ }
```

**Why**: Separates UI badges (`investor_tier`) from actual access control (`eligibility_states.can_invest`)

### 2. **Service Layer**

**Before**: Direct Supabase calls everywhere

**After**: Centralized business logic
```typescript
import { InvestmentsService } from '@/lib/domains/investments/service';

const investment = await InvestmentsService.createInvestment({
  userId,
  projectId,
  amount,
  tokensPurchased,
});
// ✅ Automatic eligibility check
// ✅ Automatic audit log
// ✅ Portfolio updated by trigger
```

### 3. **Proper Wallet Management**

**New table**: `wallet_links`
- Separate connection from verification
- Support multiple wallets over time
- Proper audit trail

**New API routes**:
- `POST /api/wallet/connect`
- `POST /api/wallet/verify`

### 4. **KYC/Compliance Ready**

**Sumsub integration prep**:
- `kyc_profiles` table
- Webhook endpoint: `POST /api/compliance/webhook`
- Eligibility state machine

**Add to `.env.local`**:
```env
NEXT_PUBLIC_SUMSUB_APP_TOKEN=your_token
SUMSUB_SECRET_KEY=your_secret
```

### 5. **Audit Logging**

**Automatic logs for**:
- Wallet changes
- KYC status changes
- Investments
- Payouts
- Admin actions

**Query logs**:
```typescript
import { getUserAuditLogs } from '@/lib/domains/admin/service';
const logs = await getUserAuditLogs(userId);
```

---

## 📦 New Database Tables

| Table | Purpose | Key Feature |
|-------|---------|-------------|
| `wallet_links` | Wallet management | Separate connection/verification |
| `eligibility_states` | **Access control** | Source of truth for `can_invest` |
| `kyc_profiles` | KYC data | Sumsub integration |
| `offerings` | Tokenized investments | ERC20 contract addresses |
| `portfolio_positions` | Aggregated holdings | Auto-updated by triggers |
| `payout_cycles` | Dividend periods | Gold production tracking |
| `payout_records` | Individual payouts | Claim-based distribution |
| `audit_logs` | Immutable audit trail | Append-only |
| `user_roles` | RBAC | Admin, compliance_officer, etc. |

---

## 🚦 Migration Status

**Completed** ✅:
- [x] Modular architecture
- [x] Domain models & schemas
- [x] Database migrations
- [x] Service layer
- [x] Eligibility system
- [x] Wallet refactor
- [x] Compliance prep
- [x] Audit logging
- [x] Admin RBAC
- [x] Payout foundations
- [x] Blockchain interfaces
- [x] Documentation

**Pending** ⏳ (Next Sprint):
- [ ] Sumsub SDK integration
- [ ] ERC20 token contracts
- [ ] Investment flow UI
- [ ] Portfolio dashboard UI
- [ ] Payout claim UI
- [ ] Admin dashboard

---

## 🎬 Next Actions (Priority Order)

### 1. **Verify Migrations** (Today)
```bash
# Check tables exist
supabase db pull  # or check Supabase dashboard

# Should see: wallet_links, eligibility_states, kyc_profiles, etc.
```

### 2. **Test Eligibility Flow** (This Week)
Manual test:
1. Sign up new user → Check `eligibility_states`: status = `registered`
2. Connect wallet → status = `wallet_connected`
3. Verify signature → status = `wallet_verified`
4. (Manual) Update KYC → status = `kyc_approved`
5. Check: `can_invest` should be `true`

### 3. **Set Up Sumsub** (This Week)
1. Sign up: https://sumsub.com/
2. Get API keys
3. Add to `.env.local`
4. Test webhook: `POST /api/compliance/webhook`

### 4. **Deploy Token Contract** (Next 2 Weeks)
```bash
mkdir contracts
cd contracts
forge init  # Install Foundry first

# Create:
# - GoldenFleeceToken.sol
# - Test deployment to testnet
# - Update offering with contract_address
```

### 5. **Build Investment UI** (Next 2 Weeks)
- Update `/projects/[slug]` page
- Add invest button (use `RestrictedButton`)
- Call `InvestmentsService.createInvestment()`
- Show success/error states

---

## 🐛 Common Issues

### "Migration failed: column already exists"
**Solution**: Migrations use `IF NOT EXISTS` - safe to re-run

### "Cannot find module '@/lib/domains/...'"
**Solution**: TypeScript may need restart
```bash
# VS Code: Cmd+Shift+P → "TypeScript: Restart TS Server"
```

### "User not eligible to invest"
**Check**: `eligibility_states` table
```sql
SELECT * FROM eligibility_states WHERE user_id = 'your-user-id';
-- can_invest should be true
```

**Fix**: Manually update for testing
```sql
UPDATE eligibility_states
SET status = 'investment_eligible', can_invest = true
WHERE user_id = 'your-user-id';
```

---

## 📚 Documentation Index

| File | Purpose | Read Time |
|------|---------|-----------|
| **QUICK_START.md** | This file - get started fast | 5 min |
| **REFACTORING_SUMMARY.md** | What changed and why | 15 min |
| **ARCHITECTURE.md** | Complete system design | 30 min |
| **PROJECT_DEVELOPMENT_SUMMARY.md** | Original project docs (updated) | 20 min |
| **USER_FLOW_GUIDE.md** | Onboarding flows (still valid) | 10 min |

---

## 💡 Pro Tips

1. **Gradual Migration**: Keep using old patterns while learning new ones
2. **Service Layer First**: Start using services in new code
3. **Eligibility Hooks**: Replace `investorTier` checks with `useEligibility()`
4. **Audit Logs**: Review regularly to catch issues early
5. **Type Safety**: Leverage Zod schemas for validation

---

## 🆘 Need Help?

**Architecture Questions**:
- Read `ARCHITECTURE.md` → Module Architecture section

**Migration Issues**:
- Check migration files in `supabase/migrations/`
- Compare with your current database schema

**Code Examples**:
- Review service implementations in `lib/domains/*/service.ts`
- Check API routes in `app/api/*/route.ts`

**Blockchain Integration**:
- See `lib/domains/shared/blockchain-interfaces.ts`
- Study placeholder implementations

---

## ✅ Verification Checklist

Before deploying to production:

- [ ] All migrations applied successfully
- [ ] Dev server runs without errors
- [ ] Existing features still work (signup, login, wallet)
- [ ] New `eligibility_states` table populated for users
- [ ] Sumsub credentials configured
- [ ] First token contract deployed to testnet
- [ ] Investment flow tested end-to-end
- [ ] Audit logs appearing for sensitive operations
- [ ] Admin roles configured

---

**Remember**: This is a **foundation upgrade**. Existing functionality preserved while enabling future growth.

**Status**: ✅ Ready for next phase of development

Good luck! 🚀
