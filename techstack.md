# Technology Stack for Izzy Yum

## Overview
Full-stack recipe system with web app (planning/cooking) and iOS app (shopping), deployed on free tier cloud platforms under the domain `izzy-yum.com`.

---

## Web Application

### Framework: Next.js 14+ (App Router)
- React-based, excellent free deployment options
- Server-side rendering for fast initial loads
- API routes for backend logic (serving size calculations)
- TypeScript for type safety

### Styling: Tailwind CSS
- Rapid UI development
- Responsive grid layouts for protein/cuisine/recipe cards
- Easy to create beautiful, consistent designs

### Animations: Framer Motion
- Smooth transitions between protein → cuisine → recipe screens
- Fade-out filtering effects for ingredient wheel

---

## Backend & Database

### Platform: Supabase (Free Tier)
- PostgreSQL database (500MB free)
- **Real-time subscriptions** (critical for iOS ↔ Web sync)
- Built-in authentication
- Row-level security for user data
- File storage for recipe images (1GB free)
- REST + GraphQL APIs auto-generated

### Why Supabase:
- Real-time updates when iOS app saves substitutions
- Web app can subscribe to changes and show "Shopping list updated" notifications
- Free tier is generous enough for this use case

---

## iOS Application

### Framework: SwiftUI + Swift
- Native iOS experience
- SwiftData or Core Data for offline shopping lists
- Supabase Swift client for real-time sync

---

## Deployment

### Web Hosting: Vercel (Free Tier)
- Automatic deployments from Git
- Custom domain (izzy-yum.com) supported on free tier
- 100GB bandwidth/month free
- Global CDN for fast loading
- Zero configuration for Next.js

### Database: Supabase Cloud (Free Tier)
- Hosted PostgreSQL
- Real-time server included
- No DevOps required

---

## Architecture Overview

```text
┌─────────────────┐
│   Web App       │
│   (Next.js)     │ ← Deployed on Vercel (izzy-yum.com)
│   - Browse      │
│   - Plan        │
│   - Cook        │
└────────┬────────┘
         │
         ├─────── Supabase ─────┐
         │       (Real-time DB) │
         │                      │
┌────────┴────────┐             │
│   iOS App       │             │
│   (SwiftUI)     │             │
│   - Shop        │             │
│   - Substitute  │             │
└─────────────────┘             │
         │                      │
         └──────────────────────┘
              (Real-time sync)
```

---

## Key Features Enabled

- ✅ Free deployment (both Vercel + Supabase free tiers)
- ✅ Custom domain support
- ✅ Real-time sync for substitutions
- ✅ Image hosting for recipe photos
- ✅ User authentication
- ✅ Offline-capable iOS app
- ✅ Fast, beautiful UI with smooth animations
- ✅ TypeScript throughout for reliability

---

## Cost Estimate

- **Month 1-∞:** $0 (stays on free tier unless you exceed limits)
- **Domain:** ~$12/year for izzy-yum.com (only real cost)

---

## Technology Rationale

### Why Next.js?
- Best-in-class React framework with excellent DX
- Built-in API routes eliminate need for separate backend
- Vercel deployment is seamless
- Image optimization built-in (important for recipe photos)

### Why Supabase?
- Real-time subscriptions are critical for iOS ↔ Web sync
- PostgreSQL is robust and SQL is powerful for recipe queries
- Free tier is very generous
- No backend code needed - focus on features

### Why SwiftUI?
- Modern, declarative iOS development
- Native performance and UX
- Built-in support for lists, offline storage, and animations

### Why Vercel?
- Zero-config Next.js deployment
- Free custom domains
- Automatic HTTPS
- Global CDN included
- Preview deployments for every commit

---

## Local Development Strategy

### Overview

Develop locally on your laptop using Supabase CLI, then seamlessly migrate to Supabase Cloud for alpha testing. This approach ensures identical environments between development and production with zero costs during development.

### Local Development Stack

```text
┌─────────────────┐
│   Next.js       │ ← localhost:3000
│   (dev server)  │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│   Supabase CLI  │ ← Runs in Docker
│   - PostgreSQL  │   localhost:54322
│   - Auth        │
│   - Real-time   │
│   - Storage     │
└─────────────────┘
```

### Phase 1: Local Development Setup

#### Initial Setup (One-time)

```bash
# Install Supabase CLI
brew install supabase/tap/supabase

# Install Docker Desktop (if not already installed)
# Download from docker.com

# Initialize Next.js project
npx create-next-app@latest izzy-yum-web --typescript --tailwind --app
cd izzy-yum-web

# Initialize Supabase in project
supabase init

# Start local Supabase (runs in Docker)
supabase start
```

#### Daily Development

```bash
# Start Supabase local instance
supabase start

# In another terminal, start Next.js dev server
npm run dev

# Access points:
# - Web app: http://localhost:3000
# - Supabase Studio: http://localhost:54323 (database GUI)
# - PostgreSQL: localhost:54322
```

#### Database Schema Management

```bash
# Create migration files as you develop
supabase migration new create_recipes_table

# Edit migration file in supabase/migrations/
# Apply migrations locally
supabase db reset  # Resets and applies all migrations

# View local database changes
supabase db diff   # See what changed
```

#### Environment Configuration

**Local Development (.env.local)**
```env
NEXT_PUBLIC_SUPABASE_URL=http://localhost:54321
NEXT_PUBLIC_SUPABASE_ANON_KEY=<your-local-anon-key>
```

**Production (.env.production - add when ready for alpha)**
```env
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=<your-cloud-anon-key>
```

### Phase 2: Migration to Alpha (Supabase Cloud)

When ready for alpha testing:

#### 1. Create Supabase Cloud Project

```bash
# Link to cloud project
supabase link --project-ref your-project-ref

# Push database schema to cloud
supabase db push

# Migrations are automatically applied to cloud
```

#### 2. Deploy to Vercel

```bash
# Update .env.production with cloud credentials
# Deploy to Vercel
vercel --prod
```

#### 3. Configure Custom Domain

```bash
# In Vercel dashboard:
# Settings → Domains → Add izzy-yum.com

# Point DNS to Vercel (in your domain registrar):
# A record: 76.76.21.21
# CNAME www: cname.vercel-dns.com
```

### Recommended Project Structure

```
izzy-yum/
├── izzy-yum-web/              # Next.js web app
│   ├── app/                   # Next.js app directory
│   ├── components/            # React components
│   ├── lib/
│   │   └── supabase.ts        # Supabase client
│   ├── supabase/              # Supabase local config
│   │   ├── config.toml
│   │   └── migrations/        # Database migrations (version controlled)
│   ├── .env.local             # Local environment
│   └── .env.production        # Cloud environment
│
├── izzy-yum-ios/              # iOS app (separate Xcode project)
│   └── IzzyYum.xcodeproj
│
├── ISABELLE_PERSONA.md
└── techstack.md
```

### Key Benefits of This Approach

- ✅ **Identical environments** - Local and cloud are the same
- ✅ **Fast development** - No network latency during dev
- ✅ **Version controlled schema** - Migrations in git
- ✅ **Easy rollback** - `supabase db reset` restarts fresh
- ✅ **Offline development** - Works without internet
- ✅ **Smooth migration** - One command to push to cloud
- ✅ **No costs during development** - All local, all free
- ✅ **Database GUI included** - Supabase Studio for visual editing
