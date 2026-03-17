# GoldenFleece - Project Development Summary

## Project Overview

**GoldenFleece** is a blockchain-powered investment platform that connects everyday investors with profitable small-scale gold mining operations. The platform enables fractional investment in gold mines, providing yields backed by real gold production through transparent, blockchain-verified contracts.

### Core Value Proposition
- **For Investors**: Access to real gold mining investments with low barriers to entry, diversified exposure, and passive income through dividends
- **For Small-Scale Miners**: Alternative financing source, faster capital access, and transparent investor relationships

## Technology Stack

### Frontend Framework
- **Next.js 16.1.4** with Turbopack (latest experimental features)
- **React 19.2.3** - Latest version with concurrent features
- **TypeScript 5.9.3** - Full type safety across the application

### Styling & Design System
- **Tailwind CSS 4.1.18** - Utility-first CSS framework (v4 with new features)
- **@tailwindcss/postcss 4.1.18** - PostCSS integration
- **Custom CSS Variables** - Design tokens for consistent theming
  - Navy color palette (`--color-navy`, `--color-navy-dark`)
  - Gold accent system (`--color-gold`, `--color-gold-light`, `--color-gold-dark`)
  - Typography fonts (`--font-mono`: Space Mono, `--font-pixel`: Press Start 2P)

### Blockchain & Web3 Integration
- **@rainbow-me/rainbowkit 2.3.2** - Wallet connection UI (supports 300+ wallets)
- **wagmi 2.15.4** - React hooks for Ethereum
- **viem 2.25.7** - TypeScript-first Web3 library
- **@tanstack/react-query 5.68.0** - Data fetching and caching (required by wagmi)

### Backend & Database
- **Supabase** - Backend-as-a-Service
  - **@supabase/supabase-js 2.48.1** - JavaScript client
  - **@supabase/ssr 0.6.3** - Server-side rendering support
  - PostgreSQL database with real-time subscriptions
  - Row-level security (RLS) for data protection
  - Authentication system with email/password and OAuth

### Visualization
- **cobe 0.6.3** - Interactive 3D globe component for landing page

### Development Tools
- **autoprefixer 10.4.23** - CSS vendor prefixing
- **postcss 8.5.6** - CSS transformation
- **ESLint** - Code linting (via `next lint`)

### Runtime Configuration
- **Node.js** with increased heap size (4096MB) to handle large builds
- Environment variables managed through `.env.local`

## Architecture & Design Patterns

### Design System - Metoro-Inspired Aesthetic

The application features a premium, luminous design system inspired by metoro.io:

#### Color System
```css
Navy: #0a1628 (primary background)
Navy Dark: #060f1a (secondary background)
Gold: #E5B35A (primary accent)
Gold Light: #F5D78E (light accent)
Gold Dark: #C89B3C (dark accent)
```

#### Custom CSS Utilities

**Gradient Text Effects**:
- `.gradient-text` - Gold gradient (135deg)
- `.gradient-text-hero` - White to gold gradient (180deg) for hero sections
- `.gradient-text-subtle` - Subtle white to gold gradient

**Luminous Glow Effects**:
- `.glow-orb` - Base class for positioned blur effects
- `.glow-gold` - Strong gold radial gradient glow
- `.glow-gold-subtle` - Subtle pulsing gold glow
- Animation: `glow-pulse` (8s infinite)

**Premium Interactions**:
- `.card-hover-lift` - Elevated card hover with layered shadows
- `.border-glow` - Premium border with inset/outset glow effects
- `.hover-lift` - Standard lift effect with shadow
- `.btn-glow` - Button hover with expanding glow

**Layout Effects**:
- `.glass` - Glassmorphism with backdrop blur
- `.shimmer` - Moving gradient shimmer animation
- Scroll snap containers for full-height sections

#### Typography Scale
- Headings: 4xl → 5xl → 6xl → 7xl → 8xl (responsive)
- Body text: lg → xl → 2xl (responsive)
- Refined opacity: `text-gray-300/70` for body, `text-gray-400/60` for labels
- Line height: `leading-[1.1]` for headings, `leading-relaxed` for body

#### Spacing System
- Section padding: `py-32 md:py-40` (generous vertical rhythm)
- Content gaps: `gap-10 md:gap-12` (comfortable spacing)
- Max widths: `max-w-4xl` to `max-w-7xl` (content containment)

## Database Schema

### Core Tables

#### `profiles`
User profile information linked to Supabase auth.
```sql
- id: uuid (FK to auth.users)
- email: text
- full_name: text
- investor_tier: text ('browser' | 'investor' | 'vip')
- wallet_address: text (nullable)
- wallet_verified: boolean
- created_at: timestamp
- updated_at: timestamp
```

#### Investor Tier System
- **browser** (default): Can browse projects but cannot invest
- **investor**: Wallet connected and verified, can make investments
- **vip**: Future premium tier (not yet implemented)

### Wallet Integration Fields
- `wallet_address`: Stores the connected Ethereum wallet address
- `wallet_verified`: Boolean flag indicating if user signed verification message
- Tier automatically upgrades from `browser` to `investor` upon wallet verification

## Features Implemented

### 1. Landing Page (`/app/page.tsx`)

**Sections**:
- **Hero Section**:
  - Layered gradient backgrounds with strategic glow orbs
  - White-to-gold gradient headline
  - Trust indicators (20% global supply, 3-5 partners, 100% blockchain)
  - Interactive 3D Earth globe
  - Scroll snap navigation

- **Value Proposition**:
  - Dual cards for investors and miners
  - Premium card hover effects
  - Feature lists with benefits

- **How It Works**:
  - 3-step process visualization
  - Mining Agreement → Tokenization → Automated Yields

- **Problem & Opportunity**:
  - Side-by-side comparison
  - Problem statement and GoldenFleece solution

- **Call-to-Action**:
  - Layered glow backgrounds
  - Prominent account creation and project browsing CTAs

### 2. Wallet Connection System

#### Multi-Touchpoint Onboarding Flow

**4 Connection Points**:
1. **Onboarding Wizard** (`/app/onboarding/page.tsx`):
   - 2-step wizard shown immediately after signup
   - Step 1: Welcome with benefits
   - Step 2: Wallet connection
   - Options: "Connect Now" or "Skip for Now"

2. **Dashboard Banner** (`/app/components/WalletBanner.tsx`):
   - Persistent banner on dashboard for users without wallet
   - Dismissible but reappears on refresh
   - Different states: Not connected vs Connected but not verified

3. **Restricted Feature Modal** (`/app/components/WalletConnectionModal.tsx`):
   - Pops up when users click investor-only features
   - In-context wallet connection
   - 2-step checklist UI (Connect → Verify)
   - Real-time status updates

4. **Settings Page** (traditional location, still available)

#### Technical Implementation

**RainbowKit Integration**:
```typescript
<ConnectButton.Custom>
  {({ openConnectModal }) => (
    <button onClick={openConnectModal}>
      Connect Wallet
    </button>
  )}
</ConnectButton.Custom>
```

**Verification Flow**:
1. User connects wallet via RainbowKit (MetaMask, Coinbase, etc.)
2. User signs a message to prove ownership (free, no gas)
3. Backend verifies signature and updates `wallet_address`, `wallet_verified`
4. User tier auto-upgrades: `browser` → `investor`

**Custom Hook** (`useWalletStatus`):
```typescript
const {
  isWalletConnected,  // Is wallet connected to site?
  isWalletLinked,     // Is wallet verified in database?
  linkWallet,         // Function to sign verification message
  isLinking,          // Loading state
  error               // Error message
} = useWalletStatus();
```

### 3. Access Control Components

**RestrictedFeature Wrapper**:
```typescript
<RestrictedFeature feature="investment">
  <InvestmentButton />
</RestrictedFeature>
```
- Wraps sections that require investor tier
- Shows lock icon for browser-tier users
- Opens WalletConnectionModal on click

**RestrictedButton**:
```typescript
<RestrictedButton feature="dividend withdrawal">
  Withdraw Dividends
</RestrictedButton>
```
- Button variant of access control
- Triggers modal for browser-tier users
- Auto-hides when user has access

### 4. User Flow Architecture

See [USER_FLOW_GUIDE.md](./USER_FLOW_GUIDE.md) for detailed flow documentation.

**Key Flows**:
- **Flow A (Power User)**: Sign Up → Onboarding → Connect Wallet → Dashboard (no banner) → Invest immediately
- **Flow B (Cautious User)**: Sign Up → Skip → Dashboard (banner) → Click Invest → Modal → Connect in modal → Invest
- **Flow C (Gradual Adoption)**: Sign Up → Skip → Browse → See banner → Connect → Banner disappears

### 5. Authentication System

**Supabase Auth**:
- Email/password registration
- Session management with SSR support
- Protected routes with middleware
- Automatic profile creation on signup

**Signup Flow**:
```
/signup → Create account → Redirect to /onboarding → Complete wizard → /dashboard
```

**Login Flow**:
```
/login → Authenticate → Check wallet status → /dashboard (with/without banner)
```

## File Structure

```
/Users/ratisturua/Desktop/GoldenFleece/
├── app/
│   ├── page.tsx                          # Landing page
│   ├── globals.css                       # Global styles + custom utilities
│   ├── layout.tsx                        # Root layout with providers
│   ├── onboarding/
│   │   └── page.tsx                      # 2-step onboarding wizard
│   ├── dashboard/
│   │   ├── page.tsx                      # Main dashboard
│   │   ├── wallet/                       # Wallet management
│   │   ├── portfolio/                    # Portfolio view
│   │   ├── investments/                  # Investment tracking
│   │   ├── transactions/                 # Transaction history
│   │   └── settings/                     # User settings
│   ├── login/
│   │   └── page.tsx                      # Login page
│   ├── signup/
│   │   └── page.tsx                      # Signup page
│   ├── projects/
│   │   └── page.tsx                      # Mining projects list
│   ├── about/
│   │   └── page.tsx                      # About page
│   ├── support/
│   │   └── page.tsx                      # Support page
│   └── components/
│       ├── WalletConnectionModal.tsx     # Modal for wallet connection
│       ├── WalletBanner.tsx              # Dashboard banner component
│       ├── RestrictedFeature.tsx         # Access control wrapper
│       ├── RestrictedButton.tsx          # Access control button
│       └── EarthGlobe.tsx                # 3D globe component
├── hooks/
│   └── useWalletStatus.ts                # Wallet status management hook
├── lib/
│   └── supabase/                         # Supabase client configuration
├── proxy.ts                              # Next.js middleware (formerly middleware.ts)
├── package.json                          # Dependencies
├── package-lock.json                     # Locked dependencies
├── tsconfig.json                         # TypeScript configuration
├── tailwind.config.ts                    # Tailwind configuration
├── postcss.config.js                     # PostCSS configuration
├── next.config.js                        # Next.js configuration
├── .env.local                            # Environment variables (not in git)
├── USER_FLOW_GUIDE.md                    # Detailed user flow documentation
└── PROJECT_DEVELOPMENT_SUMMARY.md        # This file
```

## Environment Variables

Required in `.env.local`:

```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key

# WalletConnect (for RainbowKit)
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_walletconnect_project_id

# Network Configuration
NEXT_PUBLIC_CHAIN_ID=1  # 1 for Ethereum mainnet, 5 for Goerli testnet
```

## Recent Development Work

### Design Polish (Latest Session)

**Objective**: Enhance landing page with metoro.io-inspired ambiance

**Changes Made**:
1. **Hero Section**:
   - Replaced basic blur backgrounds with layered glow orbs
   - Positioned glows: top center, left accent, right accent, bottom ambient
   - Enhanced heading to use `.gradient-text-hero` (white-to-gold)
   - Increased font sizes (up to `text-8xl` for hero)
   - Refined text opacity (`gray-300/70` for descriptions)

2. **Typography Enhancements**:
   - Consistent scale: `4xl → 5xl → 6xl → 7xl/8xl`
   - Tighter line heights: `leading-[1.1]` for headings
   - Relaxed line heights: `leading-relaxed` for body
   - Tracking adjustments: `tracking-tight` for large text

3. **Spacing Refinements**:
   - Section padding: `py-24/32` → `py-32/40`
   - Content gaps: `gap-8/10` → `gap-10/12`
   - Button padding: `py-4 px-8` → `py-5 px-10`
   - Increased max-widths for better content containment

4. **Premium Card Effects**:
   - Replaced simple hover with `.card-hover-lift`
   - Applied `.border-glow` for premium borders
   - Layered shadows on hover: `0 20px 40px rgba(229, 179, 90, 0.2), 0 0 60px rgba(229, 179, 90, 0.1)`
   - Transform on hover: `translateY(-8px)`

5. **Call-to-Action Enhancements**:
   - Added layered glow backgrounds to CTA section
   - Consistent button styling across all CTAs
   - Enhanced shadow effects: `shadow-xl shadow-gold/30`

## Known Issues & Warnings

### Non-Critical Warnings

1. **WalletConnect Initialization**:
   ```
   WalletConnect Core is already initialized. This is probably a mistake and can lead to unexpected behavior. Init() was called 2 times.
   ```
   - **Impact**: Cosmetic warning, doesn't affect functionality
   - **Cause**: Multiple RainbowKit provider initializations
   - **Status**: Low priority, to be addressed in future refactoring

2. **Lit Dev Mode**:
   ```
   Lit is in dev mode. Not recommended for production!
   ```
   - **Impact**: Performance warning for development
   - **Solution**: Will be automatically resolved in production build

3. **Multiple Lit Versions**:
   ```
   Multiple versions of Lit loaded. Loading multiple versions is not recommended.
   ```
   - **Impact**: Potential bundle size increase
   - **Status**: To be investigated in dependency audit

### Deprecation Notice

4. **Middleware Naming**:
   ```
   The "middleware" file convention is deprecated. Please use "proxy" instead.
   ```
   - **Status**: Already migrated to `proxy.ts`
   - **Action**: None required

## Performance Metrics

Based on development server output:

- **Average Compile Time**: 100-300ms per route
- **Initial Load**: ~2s for complex pages (with all resources)
- **Subsequent Navigation**: 100-150ms
- **Server Response**: All routes returning 200 OK
- **Hot Reload**: Functional and fast

### Route Performance
```
/ (Landing)              : 100-200ms (after initial 2s load)
/dashboard               : 100-180ms
/dashboard/wallet        : 100-210ms
/dashboard/investments   : 120-270ms
/dashboard/portfolio     : 110-340ms
/dashboard/transactions  : 120-990ms (highest)
/dashboard/settings      : 90-260ms
/login                   : 90-370ms
/signup                  : 1200ms (first load)
/projects                : 100-200ms
/about                   : 100-190ms
/support                 : 100-140ms
```

## Future Enhancements

### Immediate Priorities
- [ ] Add investment functionality to project pages
- [ ] Implement dividend distribution system
- [ ] Create portfolio tracking with real-time updates
- [ ] Add transaction history with blockchain verification

### Medium-Term Goals
- [ ] Email reminders for users who skipped wallet connection
- [ ] Gamification: Profile completion progress bar
- [ ] Wallet connection rewards/incentives
- [ ] Analytics tracking on conversion funnel (signup → wallet connection → investment)
- [ ] A/B testing for banner messages and CTAs

### Long-Term Vision
- [ ] VIP tier features (premium access, early opportunities)
- [ ] Secondary market for investment shares
- [ ] Mobile app (React Native)
- [ ] Multi-chain support (Polygon, BSC, etc.)
- [ ] DAO governance for platform decisions

## Success Metrics

### User Onboarding
- **Goal**: >70% of users connect wallet within first session
- **Tracking**:
  - % connecting during onboarding
  - % connecting from dashboard banner
  - % connecting from modal prompt
  - % connecting from settings

### Engagement
- Time from signup to first investment
- Conversion rate: browser → investor tier
- Repeat investment frequency
- Average investment size

### Technical
- Page load times < 3s
- Time to interactive < 5s
- Zero critical errors in production
- >99% uptime

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run linter
npm run lint
```

## Browser Compatibility

- **Modern Browsers**: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- **Web3 Support**: Any browser with wallet extension (MetaMask, Coinbase Wallet, etc.)
- **Mobile**: iOS Safari 14+, Chrome Mobile 90+

## Security Considerations

### Implemented
- Row-level security (RLS) in Supabase
- Server-side authentication validation
- Wallet signature verification (no private key exposure)
- Environment variable protection
- HTTPS-only in production

### To Implement
- Rate limiting on API endpoints
- CSRF protection
- Input sanitization and validation
- Smart contract audits (when blockchain features go live)
- KYC/AML compliance (for regulatory requirements)

## Notes for External Teams

### For Backend Developers
- All database interactions use Supabase client
- Wallet verification requires signature validation server-side
- User tiers are managed in `profiles.investor_tier` column
- Authentication state managed via Supabase Auth

### For Frontend Developers
- Design system uses Tailwind + custom CSS utilities in `globals.css`
- All wallet interactions use RainbowKit + wagmi hooks
- Protected features use `<RestrictedFeature>` or `<RestrictedButton>` components
- User flows documented in `USER_FLOW_GUIDE.md`

### For Smart Contract Developers
- Platform currently uses placeholder data
- Will need ERC-20 token standard for investment shares
- Dividend distribution contract required
- Tokenization of mining contracts needed

### For DevOps
- Node heap size set to 4096MB (see `package.json` scripts)
- Environment variables required for Supabase and WalletConnect
- Next.js 16 with Turbopack (experimental, may need adjustments)

---

**Last Updated**: March 6, 2026
**Version**: 1.0.0
**Status**: Active Development
