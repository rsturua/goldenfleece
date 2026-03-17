# GoldenFleece Platform

A complete, multi-page web application for the GoldenFleece/AurumChain platform - a blockchain-powered crowdfunding platform for small-scale and junior gold mines.

## Features

- **Modern Next.js 16** with App Router and TypeScript
- **Multi-Page Application** with client-side routing
- **User Authentication UI** with login/signup forms
- **Responsive Design** that works on all devices
- **Dark Navy & Gold Theme** matching your brand
- **Tailwind CSS v4** for modern styling
- **Component-Based Architecture** for maintainability

## Pages

### 1. Homepage (Landing Page)
**Route:** `/`

Comprehensive landing page featuring:
- Hero section with value proposition
- Value proposition for investors and miners
- "How It Works" section with timeline
- Problem & opportunity explanation
- Statistics section
- Multiple CTAs throughout
- Fully responsive design

### 2. Mining Projects
**Route:** `/projects`

Browse and invest in mining opportunities:
- Project cards with funding progress
- Key metrics (returns, duration, reserves, risk level)
- Project status indicators (Funding, Active, Coming Soon)
- Filtering options
- Investment CTAs
- Information about due diligence process

### 3. About Us
**Route:** `/about`

Learn about the platform and team:
- Mission and vision statements
- Problem/solution explanation
- Our solution features
- Team member profiles
- Company story and goals

### 4. Support
**Route:** `/support`

Get help and answers:
- Comprehensive FAQ section
- Contact form for support inquiries
- Quick access to resources
- Additional learning materials
- Community links

### 5. Account Page
**Route:** `/account`

User authentication interface:
- **Login Form:** Email and password authentication
- **Signup Form:** New user registration
- Tab-based interface switching between login/signup
- Social login options (Google, GitHub)
- Password confirmation and validation
- Terms acceptance checkbox
- Responsive two-column layout with branding

## Navigation

### Header Component
- Fixed navigation bar with logo (links to home)
- Navigation links: Home, Mining Projects, About Us, Support
- Account button in top right corner
- Mobile-responsive hamburger menu
- Active page highlighting

### Footer Component
- Brand information with logo
- Quick links to all pages
- Resource links
- Social media connections
- Copyright notice
- AURUMCHAIN tagline

## Getting Started

### Prerequisites

- Node.js 18+
- Supabase account
- WalletConnect Project ID (optional)

### Installation

1. **Clone the repository**
```bash
git clone <your-repo-url>
cd GoldenFleece
```

2. **Install dependencies**
```bash
npm install
```

3. **Set up environment variables**

Create a `.env.local` file in the root directory:

```env
# Supabase Configuration (Required)
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key

# WalletConnect Configuration (Optional)
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_walletconnect_project_id
```

Get Supabase credentials:
- Go to [Supabase Dashboard](https://app.supabase.com)
- Create new project or select existing
- **Settings** → **API**
- Copy **Project URL** and **anon public** key

Get WalletConnect ID:
- Go to [WalletConnect Cloud](https://cloud.walletconnect.com)
- Create a new project
- Copy your Project ID

4. **Set up database**

Run the migration script in Supabase SQL Editor:
```bash
# Copy contents of supabase-migration.sql
# Go to Supabase → SQL Editor → Paste and Run
```

5. **Start development server**
```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) to see your application.

### Build for Production

```bash
npm run build
npm run start
```

## Project Structure

```
/app
  /components
    /Header.tsx       # Navigation header component
    /Footer.tsx       # Footer component
  /projects
    /page.tsx         # Mining Projects page
  /about
    /page.tsx         # About Us page
  /support
    /page.tsx         # Support & FAQ page
  /account
    /page.tsx         # Login/Signup page
  /globals.css        # Global styles with Tailwind v4
  /layout.tsx         # Root layout with Header & Footer
  /page.tsx           # Homepage (landing)
/public
  /logo.png           # Golden ram logo (transparent background)
/Uploads
  # Original brand assets
tailwind.config.ts    # Tailwind configuration
tsconfig.json         # TypeScript configuration
postcss.config.js     # PostCSS configuration
next.config.js        # Next.js configuration
```

## Design System

### Colors
Defined in [app/globals.css](app/globals.css):
- **Navy**: `#0a1628` (primary background)
- **Navy Dark**: `#060f1a` (secondary background)
- **Gold**: `#E5B35A` (primary accent)
- **Gold Light**: `#F5D78E` (hover states)
- **Gold Dark**: `#C89B3C` (borders)

### Typography
- **Body**: System font stack with fallbacks
- **Headings**: Bold, large sizes for hierarchy
- **Optional Pixel Font**: Press Start 2P for retro aesthetic

### Components
All components follow the brand's navy & gold color scheme with:
- Border radius for modern feel
- Hover states with gold accent
- Smooth transitions
- Consistent spacing
- Responsive breakpoints

## Content Strategy

Based on the Gold coin document, the platform emphasizes:

### For Investors:
- Fixed returns tied to real gold production
- Inflation hedge with gold-backed assets
- Blockchain transparency
- Accessible investment minimums
- Vetted mining operations

### For Miners:
- Access to capital without traditional banking barriers
- Fair financing terms
- Global investor reach
- Operational control maintained
- Scale opportunities

## Key Statistics
- 20% of global gold supply from small-scale mining
- 80% of mining employment
- 3-5 initial mining partners
- 100% blockchain transparency

## Implemented Features

### ✅ Authentication System
- Supabase-powered authentication
- Email/password signup and login
- Secure session management
- Protected dashboard routes
- Automatic profile creation

### ✅ Crypto Wallet Integration
- RainbowKit + Wagmi integration
- MetaMask and other Web3 wallets
- Wallet linking to user profiles
- Signature verification
- Investor tier system (Browser, Investor, VIP)

### ✅ Dashboard
- Protected user dashboard
- Investment tracking interface
- Portfolio overview
- Transaction history
- Wallet management
- Settings page

### 🔄 Next Steps for Development

1. **Payment Integration**
   - Connect payment processor
   - Implement investment transactions
   - KYC/AML compliance

2. **Smart Contract Integration**
   - Token minting and distribution
   - Investment tracking on-chain
   - Automated payouts
   - Blockchain verification

3. **Enhanced Features**
   - Real-time chat support
   - Email notifications
   - Investment calculator
   - Advanced project filtering

## Technologies Used

- **Framework:** Next.js 16.1.4
- **UI Library:** React 19.2.3
- **Language:** TypeScript 5.9.3
- **Styling:** Tailwind CSS 4.1.18
- **Image Optimization:** Next.js Image component
- **Routing:** Next.js App Router

## Deployment

### Deploying to Vercel (Recommended)

Vercel is the best platform for Next.js apps with zero-config deployment:

1. **Push to GitHub**
   ```bash
   git add .
   git commit -m "Initial commit"
   git push origin main
   ```

2. **Import to Vercel**
   - Go to [vercel.com](https://vercel.com)
   - Click "New Project"
   - Import your GitHub repository
   - Vercel auto-detects Next.js settings

3. **Add Environment Variables**
   In Vercel dashboard → Settings → Environment Variables:
   ```
   NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
   NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
   NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_walletconnect_id
   ```

4. **Deploy**
   - Click "Deploy"
   - Vercel will build and deploy automatically
   - Get your production URL!

### Other Deployment Options

- **Netlify**: Good alternative with drag-and-drop
- **AWS Amplify**: For AWS ecosystem integration
- **Railway**: Easy deployment with free tier
- **Render**: Simple hosting with auto-deploy

## Brand Assets

All brand assets preserved in [Uploads](Uploads):
- Logo with transparent background
- Presentation PDF with brand guidelines
- Gold coin document with platform details

---

**GoldenFleece** by AurumChain - Own Your Own Gold Mine

© 2025 AurumChain. All rights reserved.
