# GoldenFleece - Revised User Flow

## Overview

The user journey has been redesigned to make wallet connection more prominent and intuitive. Users are now guided through a clear onboarding process and have multiple touchpoints to connect their wallet.

---

## 🎯 New User Journey

### 1. **Sign Up**
**Location**: `/signup`

**What happens:**
- User creates account with email and password
- Account is created in Supabase
- **Redirect**: Automatically goes to `/onboarding` (not dashboard!)

**Initial tier**: `browser` (can only browse, cannot invest)

---

### 2. **Onboarding Wizard** ✨ NEW
**Location**: `/onboarding`

**Step 1: Welcome**
- Welcomes user by name
- Shows 3 key benefits:
  1. Browse Projects
  2. Connect Wallet
  3. Earn Dividends
- Big "Get Started" button → Step 2

**Step 2: Connect Wallet**
- Prominent wallet connection UI
- Explains WHY wallet connection matters
- Shows 2-step process:
  1. **Connect Wallet** - RainbowKit button (supports 300+ wallets)
  2. **Verify Ownership** - Sign message (free, no gas)

**User choices:**
- ✅ **Connect & Verify** → Tier upgrades to `investor` → Dashboard
- 🔄 **Skip for Now** → Stays as `browser` tier → Dashboard with banner

---

### 3. **Dashboard with Banner** ✨ NEW
**Location**: `/dashboard`

**If wallet NOT connected:**
- **Prominent banner** at top of dashboard
- Shows current tier badge: "Browser Tier"
- Clear call-to-action: "Connect Wallet" button
- Lists benefits: Make investments, Receive dividends, Own shares
- Can be dismissed (but reappears on refresh)

**If wallet IS connected but NOT verified:**
- Banner shows "Almost There!"
- "Sign Message to Verify" button
- One click to complete setup

**If wallet IS verified:**
- ✅ Banner hidden
- Full dashboard access
- Shows `investor` tier

---

### 4. **Restricted Features - Interactive Modal** ✨ NEW
**Trigger**: When user clicks on investor-only features (invest buttons, etc.)

**What happens:**
1. **Modal pops up** (not buried in settings!)
2. Shows clear 2-step process:
   - Step 1: Connect Wallet ✓
   - Step 2: Verify Ownership ✓
3. User can complete entire flow in modal
4. After verification → Modal closes, feature unlocks

**Features using this:**
- Investment buttons on project pages
- Dividend withdrawal
- Portfolio management tools
- Any feature wrapped in `<RestrictedFeature>` or `<RestrictedButton>`

---

## 📱 User Touchpoints for Wallet Connection

Users can connect their wallet from **4 different places**:

1. **Onboarding Wizard** (immediate after signup)
2. **Dashboard Banner** (persistent until connected)
3. **Restricted Feature Modal** (when clicking invest buttons)
4. **Settings Page** (traditional location, still available)

---

## 🔄 Complete User Flows

### Flow A: Power User (Connects Immediately)
```
Sign Up → Onboarding Step 1 → Step 2: Connect Wallet
→ Sign Message → Tier: Investor → Dashboard (no banner)
→ Click "Invest" → Works immediately ✓
```

### Flow B: Cautious User (Skips Initially)
```
Sign Up → Onboarding Step 1 → Step 2: Skip
→ Dashboard (shows banner) → Browse projects
→ Click "Invest" on project → Modal pops up
→ Connect wallet in modal → Sign message
→ Tier: Investor → Investment proceeds ✓
```

### Flow C: Gradual Adoption
```
Sign Up → Onboarding: Skip → Dashboard (banner shown)
→ Dismiss banner → Browse site → Dashboard again
→ See banner again → Click "Connect Wallet"
→ Sign message → Tier: Investor → Banner disappears ✓
```

---

## 🎨 UI Components

### 1. **OnboardingPage** (`/app/onboarding/page.tsx`)
- 2-step wizard with progress bar
- Step 1: Welcome with benefits
- Step 2: Wallet connection
- Options: Complete setup OR Skip for now

### 2. **WalletBanner** (`/app/components/WalletBanner.tsx`)
- Shows on dashboard when wallet not connected
- Large, colorful banner with gold accents
- Can be dismissed (local state)
- Different messages for:
  - Not connected: "Connect Wallet"
  - Connected but not verified: "Sign Message"

### 3. **WalletConnectionModal** (`/app/components/WalletConnectionModal.tsx`)
- Pops up when clicking restricted features
- Shows 2-step checklist:
  - Step 1: Connect (with RainbowKit button)
  - Step 2: Verify (with Sign button)
- Real-time status updates
- Success state with "Continue" button

### 4. **RestrictedFeature** & **RestrictedButton**
- Wraps investor-only features
- Shows lock icon + opens modal on click
- Auto-hides when user has access

---

## 💡 Benefits of New Flow

### For Users:
✅ **Clear path** - Know exactly what to do next
✅ **Flexible** - Can skip and connect later
✅ **No hidden features** - Wallet connection always visible
✅ **In-context prompts** - Connect when you need it
✅ **No navigation maze** - Modal appears where you are

### For Business:
✅ **Higher conversion** - More users will connect wallets
✅ **Better onboarding** - Reduced drop-off
✅ **Clear value prop** - Users understand WHY to connect
✅ **Multiple touchpoints** - More opportunities to convert
✅ **Reduced support** - Users can't get lost

---

## 🔧 Technical Implementation

### Investor Tier System

**Database**: `profiles.investor_tier`
- `browser` - Default, can only browse
- `investor` - Wallet connected, can invest
- `vip` - Future premium tier

**Auto-upgrade**:
- When wallet is linked → Tier changes from `browser` to `investor`
- Happens in `useWalletStatus.linkWallet()`
- Updates immediately in UI

### Components Used

**Wallet Connection:**
- `@rainbow-me/rainbowkit` - Wallet UI
- `wagmi` - React hooks for Ethereum
- `viem` - Web3 library
- Custom `useWalletStatus()` hook

**Access Control:**
- `<RestrictedFeature>` - Wraps sections
- `<RestrictedButton>` - Wraps buttons
- Both trigger `WalletConnectionModal`

---

## 📝 Migration Notes

### Changes from Previous Flow:

**Before:**
- Sign up → Dashboard
- Wallet connection hidden in Settings
- Users had to navigate: Dashboard → Sidebar → Settings → Crypto Wallet
- No prompt when clicking restricted features

**After:**
- Sign up → **Onboarding wizard**
- Wallet connection in **4 prominent places**
- **Dashboard banner** always visible (until connected)
- **Modal pops up** for restricted features
- Settings still available as 5th option

### No Breaking Changes:
- Existing wallet connections still work
- Settings page unchanged
- Database schema same
- All existing features compatible

---

## 🧪 Testing the New Flow

### Test Case 1: New User
1. Go to `/signup`
2. Create account
3. Should redirect to `/onboarding`
4. Click "Get Started"
5. See wallet connection step
6. Click "Skip for Now"
7. Should see dashboard with banner

### Test Case 2: Connect from Banner
1. On dashboard with banner visible
2. Click "Connect Wallet"
3. Choose wallet (MetaMask)
4. Click "Sign Message to Verify"
5. Sign in wallet
6. Banner should disappear
7. Tier should show "Investor"

### Test Case 3: Connect from Modal
1. Browse to `/projects`
2. Click "Invest" button
3. Modal should pop up
4. Complete wallet connection in modal
5. After verification, modal closes
6. Investment should proceed

### Test Case 4: Already Connected
1. User with wallet already linked
2. Sign in → Redirected to dashboard
3. No banner shown
4. Can invest immediately
5. No prompts

---

## 🎯 Next Steps

### Immediate:
- [x] Onboarding wizard
- [x] Dashboard banner
- [x] Wallet connection modal
- [x] Update signup redirect
- [x] Update RestrictedFeature components

### Future Enhancements:
- [ ] Add project investment buttons with RestrictedButton
- [ ] Email reminders for users who skipped
- [ ] Gamification: "Complete your profile" progress bar
- [ ] Wallet connection rewards/incentives
- [ ] Analytics tracking on conversion funnel
- [ ] A/B testing different banner messages

---

## 📊 Success Metrics

**Track these:**
- % of users who connect during onboarding
- % who connect from dashboard banner
- % who connect from modal prompt
- % who connect from settings
- Time from signup to wallet connection
- Conversion rate: browser → investor tier

**Goal**: >70% of users connect wallet within first session

---

**Built with 💛 for better UX - Making gold investment accessible to everyone!**
