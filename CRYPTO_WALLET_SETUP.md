# Crypto Wallet Authentication Setup Guide

## Overview

GoldenFleece now implements a **hybrid authentication model** with tiered access:

- **Browser Tier**: Email/password login - can browse projects and view information
- **Investor Tier**: Email + crypto wallet connected - can make investments and receive dividends
- **VIP Tier**: Premium features (future expansion)

## Quick Start

### 1. Install Dependencies

Dependencies are already installed:
- `@rainbow-me/rainbowkit` - Beautiful wallet connection UI
- `wagmi` - React hooks for Ethereum
- `viem` - Modern Web3 library
- `@tanstack/react-query` - Required peer dependency

### 2. Get WalletConnect Project ID

1. Visit [https://cloud.walletconnect.com](https://cloud.walletconnect.com)
2. Create a free account
3. Create a new project
4. Copy your Project ID

### 3. Configure Environment Variables

Copy `.env.local.example` to `.env.local`:

```bash
cp .env.local.example .env.local
```

Add your WalletConnect Project ID:

```env
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_actual_project_id_here
```

### 4. Run Database Migration

Execute the migration in your Supabase SQL Editor:

```sql
-- Run this in Supabase Dashboard > SQL Editor
-- Location: /supabase/migrations/001_add_crypto_wallet_fields.sql
```

Or run:
```bash
supabase db push
```

### 5. Start Development Server

```bash
npm run dev
```

## Architecture

### Components

1. **Web3Provider** (`app/components/Web3Provider.tsx`)
   - Wraps the entire app with RainbowKit + Wagmi providers
   - Configured for Polygon mainnet (low fees)
   - Custom gold theme matching GoldenFleece design

2. **WalletConnection** (`app/components/WalletConnection.tsx`)
   - Full wallet connection UI for settings page
   - Shows investor tier, connection status
   - Link/unlink wallet functionality

3. **RestrictedFeature** (`app/components/RestrictedFeature.tsx`)
   - Wrapper component for investor-only features
   - Shows watermark overlay for non-investors
   - Two variants: full feature block and button lock

### Hooks

**useWalletStatus** (`hooks/useWalletStatus.ts`)
- Central hook for wallet and auth state
- Provides `investorTier`, `isWalletLinked`, `linkWallet()`, etc.
- Automatically syncs wallet address with Supabase

### Configuration

**wagmi.ts** (`lib/wagmi.ts`)
- Wagmi/RainbowKit configuration
- Chains: Polygon mainnet (production)
- Optional testnets for development

## Usage Examples

### Restrict an Investment Button

```tsx
import { RestrictedButton } from '@/app/components/RestrictedFeature';

<RestrictedButton
  requiredTier="investor"
  onClick={() => handleInvest()}
  className="bg-gold text-navy px-6 py-3 rounded-lg"
>
  Invest $500
</RestrictedButton>
```

### Restrict Entire Feature Section

```tsx
import { RestrictedFeature } from '@/app/components/RestrictedFeature';

<RestrictedFeature
  requiredTier="investor"
  feature="investment analytics"
>
  <div>
    {/* Premium investment analytics content */}
    <InvestmentChart />
    <DividendCalculator />
  </div>
</RestrictedFeature>
```

### Check Access Programmatically

```tsx
import { useWalletStatus } from '@/hooks/useWalletStatus';

function MyComponent() {
  const { investorTier, isWalletLinked, walletAddress } = useWalletStatus();

  if (investorTier === 'browser') {
    return <ConnectWalletPrompt />;
  }

  return <InvestmentForm />;
}
```

## User Flow

### New User Journey

1. **Sign Up** → Email/password → Browser tier
2. **Browse Projects** → View opportunities, calculate returns
3. **Connect Wallet** → Dashboard → Settings → Crypto Wallet
4. **Sign Message** → Prove wallet ownership
5. **Upgrade to Investor** → Automatic tier upgrade
6. **Make Investment** → Investment features unlocked

### Investment Flow

1. User clicks "Invest" on a project
2. System checks `investorTier`
3. If `browser`: Show wallet connection prompt
4. If `investor`: Proceed with investment
5. Transaction sent to Polygon network
6. Investment recorded in Supabase + blockchain

## Database Schema

### New Fields in `profiles`

```sql
crypto_wallet_address TEXT UNIQUE       -- Ethereum/EVM wallet address
crypto_wallet_type TEXT                 -- 'metamask', 'coinbase', etc.
crypto_wallet_connected_at TIMESTAMPTZ  -- When wallet was linked
wallet_verification_nonce TEXT          -- For signature verification
investor_tier TEXT                      -- 'browser', 'investor', 'vip'
```

## Supported Wallets

Via RainbowKit + WalletConnect, supports 300+ wallets:

**Popular wallets:**
- MetaMask
- Coinbase Wallet
- Trust Wallet
- Rainbow Wallet
- Ledger (hardware wallet)
- And many more...

## Blockchain Configuration

**Production**: Polygon (MATIC)
- Low fees (~$0.01 per transaction)
- Fast confirmations (~2 seconds)
- Ethereum-compatible
- Native MATIC token for gas

**Development**: Polygon Amoy (testnet)
- Free test MATIC from faucets
- Same as mainnet, zero risk

## Security

### Wallet Verification

1. User connects wallet via RainbowKit
2. System generates random nonce
3. User signs message with wallet (proves ownership)
4. Signature + nonce stored in database
5. Wallet address saved to profile

### Non-Custodial

- Private keys never leave user's wallet
- No access to user funds
- Users always in control

## Next Steps

### Phase 1: Foundation (✅ Complete)
- [x] Install RainbowKit/Wagmi/Viem
- [x] Database schema updates
- [x] Provider configuration
- [x] Wallet connection UI
- [x] Access control hooks
- [x] Watermark components

### Phase 2: Smart Contracts (Todo)
- [ ] Deploy ERC-20 token for mining shares
- [ ] Investment smart contract
- [ ] Dividend distribution contract
- [ ] Deploy to Polygon mainnet

### Phase 3: Integration (Todo)
- [ ] Connect investment flow to smart contracts
- [ ] Blockchain transaction tracking
- [ ] Automated dividend payments
- [ ] NFT-based ownership certificates

### Phase 4: Advanced Features (Todo)
- [ ] Secondary market for share trading
- [ ] Governance voting with tokens
- [ ] VIP tier with exclusive projects
- [ ] Multi-signature for large investments

## Troubleshooting

### "Module not found" errors
Run: `npm install`

### Wallet connection fails
1. Check `NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID` is set
2. Verify Project ID is valid at cloud.walletconnect.com
3. Check browser console for errors

### Database errors
1. Ensure migration is run in Supabase
2. Check RLS policies allow profile updates
3. Verify user is authenticated

### Signature verification fails
1. User must sign the exact message
2. Check nonce is generated correctly
3. Verify wallet address format (lowercase)

## Resources

- [RainbowKit Docs](https://rainbowkit.com)
- [Wagmi Docs](https://wagmi.sh)
- [Polygon Docs](https://docs.polygon.technology)
- [WalletConnect](https://walletconnect.com)

## Support

For issues or questions, check:
1. Browser console for errors
2. Supabase logs for database issues
3. Network tab for API failures
4. This documentation for configuration

---

**Built with ❤️ for GoldenFleece - Making gold mining investment accessible to everyone**
