# Spec 009: Product Strategy

> Core belief: **Your thinking compounds — but only if you never lose it.**
>
> Products are the system that makes compounding happen.
> Capture = deposits. AI connections = interest rate. Time = compound growth.

## Product Entry Points

```
         Deposit channels              Withdraw / View returns
    ┌──────────────────┐           ┌──────────────────────┐
    │ Chrome Extension │ auto      │ iOS App              │ see growth
    │                  │──────→   │ (Feed + Search)       │
    │ iOS Share Sheet  │ manual    │                      │
    │                  │──────→   │ Web App              │ deep explore
    └──────────────────┘           │ (Knowledge Map + RAG)│
                    ↓              └──────────────────────┘
              Google Drive
              (user's account)
```

| Entry Point | Role | Compound Interest Analogy |
|-------------|------|--------------------------|
| **Chrome Extension** | Auto-capture AI conversations | Automatic payroll deposit |
| **iOS Share Sheet** | 1-click save links/screenshots | Manual bank transfer |
| **iOS Feed** | See knowledge growing | Check account balance |
| **Web Knowledge Map** | Explore full knowledge graph | View investment portfolio |
| **RAG Chat** | Ask your knowledge base | Withdraw interest |

### Design principle

Deposit channels = always free, always zero-friction.
Value channels (search, RAG, AI connections) = paid.

## Chrome Extension

Primary deposit channel. Fully automatic. User installs and forgets.

```
What it does:
  - Auto-detect AI platforms (Claude, ChatGPT, Gemini)
  - Auto-save conversations to Drive on page close
  - "Save This Page" button for any webpage
  - Zero configuration after install

CWS listing:
  Title: "CrossMem — Auto-save your AI conversations to Google Drive"
  Keywords: save ChatGPT conversations, backup Claude, AI conversation history
```

Already built. See crossmem-chrome repo.

## iOS App

### Screens

```
Tab 1: Feed (free)
  ┌──────────────────────────────────────┐
  │ Today                                │
  │ ┌─ 💡 New connection found ─────────┐│
  │ │ Your March tweet about RAG        ││
  │ │ connects to yesterday's Claude    ││
  │ │ conversation about vector search  ││
  │ └──────────────────────────────────┘│
  │                                      │
  │ This Week: 12 saves                  │
  │ Total: 147 sources                   │
  │ Connections found: 23                │
  │                                      │
  │ Recent Saves                         │
  │  📄 Claude: RAG architecture...      │
  │  🐦 Tweet: @karpathy on embeddings  │
  │  🌐 Blog: Why vector DBs matter     │
  └──────────────────────────────────────┘

Tab 2: Search (Pro)
  Semantic search across all sources.
  Shows results with source type, platform, date, topic.

Tab 3: Chat (Pro)
  RAG-powered conversation with your knowledge base.
  "What did I save about graph RAG?"
  → Answer grounded in your sources with citations.

Tab 4: Topics
  Topic list (Slack-like, grouped by most recent activity).
  Tap into topic → sources + AI summary.
```

### Share Sheet Extension

```
User taps Share → CrossMem
  → Preview: title, URL, extracted text
  → Topic picker (AI suggests 3 existing topics + "New topic")
  → Save → Drive + (if Pro) Supabase ingest
  → Done in <3 seconds
```

### Key Design Decisions

- Feed is FREE — users must see their knowledge growing to stay engaged
- 3 free AI connection previews per month — taste of compound interest
- Search and Chat are Pro — the "withdrawal" features drive conversion
- No manual organization ever — AI handles topics, clustering, connections

## Web App (crossmem.dev)

```
Phase 1 (current): Landing page + waitlist
Phase 2: /search — semantic search (Pro)
Phase 3: /explore — point cloud knowledge map (Pro)
Phase 4: /chat — RAG conversation (Pro)
```

Web is for deep exploration sessions. Mobile is for daily engagement.

## In-App Purchase

### Tiers

```
Free:
  ✅ Unlimited saves (Extension auto-capture + iOS Share Sheet)
  ✅ Feed (see what you saved, watch it grow)
  ✅ Topic browsing
  ✅ 3x/month AI connection previews
  ❌ Semantic search
  ❌ RAG chat
  ❌ Daily digest push notifications
  ❌ Auto-clustering
  ❌ Export (Markdown, JSON)

Pro: $4.99/month or $39.99/year (save 33%)
  ✅ Everything in Free
  ✅ Unlimited semantic search
  ✅ RAG chat (ask your knowledge base)
  ✅ Daily digest (AI-curated knowledge resurfacing)
  ✅ Auto-clustering
  ✅ Export
```

### Pricing Rationale

- $4.99/month < a coffee, << any AI subscription ($20/mo for ChatGPT/Claude)
- Infrastructure cost: $0.18/user/month → 96% gross margin
- Annual $39.99 = one dinner for a year of knowledge compounding
- Position as add-on to existing AI subscriptions, not replacement

### Conversion Trigger

The natural upgrade moment:

```
User has 50+ sources (accumulated passively over ~3 weeks)
  → Searches for something they vaguely remember saving
  → Free tier: Drive full-text search returns poor results
  → Upgrade prompt: "Pro semantic search finds what you're looking for"
  → Or: monthly AI connection preview shows a powerful connection
  → "Unlock unlimited connections for $4.99/month"
```

### What Must Stay Free Forever

Deposit channels must never be gated. If we charge for saving,
we break the compound interest loop at the source. This is non-negotiable.

```
Free forever:
  - Extension auto-capture
  - iOS Share Sheet save
  - Google Drive storage (it's their Drive)
  - Feed / topic browsing
  - Basic import
```

## 0→1 Quantitative Metrics

### North Star Metric

**Weekly Compound Moments (WCM)**

= Number of users per week who experience:
  "AI surfaced content I had forgotten I saved, and I found it useful."

This directly measures whether thinking is compounding.

### Funnel Metrics

```
Level   Metric                              Target
─────────────────────────────────────────────────────
L1      Extension install                   —
L2      First save (auto or manual)         D0, >90%
L3      10 sources saved                    D7
L4      30 sources saved                    D21
        (compound interest threshold)
L5      First compound moment               D30
        (AI surfaces a forgotten connection)
L6      Free → Pro conversion               >5%
L7      Pro D90 retention                   >70%
```

### Why L4 (30 sources) Is the Critical Threshold

Below 30 sources, AI connections are sparse and often obvious.
Above 30, cross-source connections start appearing that surprise the user.
All growth tactics should aim to accelerate users to L4.

### Acceleration to L4

```
Day 0:  Install Extension
        → Prompt: "Import your ChatGPT / Claude conversation history"
        → One-click import → instantly +20-50 sources
        → User is already at or near L4 on Day 0

Day 1-7: Extension auto-captures AI conversations passively
         → +2-3 sources/day with zero effort

Day 3:   Push notification: "You have 28 sources — 2 more to unlock AI connections"
         → CTA: Install iOS app → share 2 links → reach 30

Day 7:   First compound moment triggered
         → Push: "We found a connection in your knowledge base"
         → User opens → "I forgot I saved this" → Hook complete
```

### Metrics to Track Weekly

| Metric | What It Tells Us |
|--------|-----------------|
| New installs (Extension + iOS) | Top of funnel |
| Sources saved / user / week | Deposit velocity |
| % users at L4 (30+ sources) | Compound readiness |
| Compound moments / user / week | Core value delivery |
| Search queries / user / week | Engagement depth |
| RAG conversations / user / week | Engagement depth |
| Free→Pro conversion rate | Monetization |
| Pro churn rate (monthly) | Product-market fit |
| Revenue per user (ARPU) | Business health |

## Cold Start Growth Strategy

### Phase 0: Pre-launch (now → launch)

Waitlist with qualification question:

```
crossmem.dev waitlist form:
  Email: ____
  "How many AI tools do you use regularly?"
  □ 1    □ 2    □ 3+

→ Prioritize 3+ for beta invites
→ These users have the strongest pain point
→ Highest likelihood to retain and pay
```

### Phase 1: Extension-Led Growth

The extension is the trojan horse. Free. Zero config. Chrome Web Store organic traffic.

```
CWS optimization:
  Title: "CrossMem — Auto-save your AI conversations to Google Drive"
  Target keywords:
    "save ChatGPT conversations"     ~7K monthly searches
    "backup Claude conversations"    growing
    "AI conversation history"        growing
    "ChatGPT export"                 ~12K monthly searches
    "AI memory"                      growing

  These searchers already feel the pain: their conversations are disappearing.
```

Flywheel from extension to iOS to Pro:

```
Extension install (free, CWS organic)
  → Auto-saves conversations for 7 days
  → Push: "You've auto-saved 15 AI conversations"
  → CTA: "Install iOS app to see your knowledge grow"
  → iOS install
  → 30 days: "Upgrade to Pro for AI search + connections"
  → Paid conversion
```

### Phase 2: Content-Led SEO

Core narrative: **"Your AI conversations are disappearing."**

```
Blog posts (crossmem.dev/blog):
  "ChatGPT just lost everyone's memory — again"
    → Reference the Feb 2025 memory wipe crisis (300+ Reddit threads)
    → Position: "Your data should be in YOUR Google Drive"

  "I use 3 AI tools and waste 2 hours/week re-explaining context"
    → Target: multi-AI power users
    → Solution: one memory across all platforms

  "The compound interest of knowledge: why your old AI conversations are worth more than you think"
    → Thought leadership piece
    → Introduce the core belief

Distribution:
  Hacker News (Show HN)
  Reddit: r/ClaudeAI, r/ChatGPT, r/productivity, r/PKMS
  Twitter/X: AI influencer community
  Dev.to / Medium
```

### Phase 3: Community Seeding (Beta)

```
Find 10 power users:
  - 2-3 researchers (heavy Claude + paper reading)
  - 2-3 freelancers/consultants (multiple clients, multiple AI tools)
  - 2-3 developers (Cursor + Claude Code + Copilot)
  - 2-3 PMs / knowledge workers (ChatGPT + Claude + Gemini)

Give them Pro free for 90 days.
Ask them to use normally — no special effort.

At day 30, interview:
  "Did CrossMem surface anything you'd forgotten you saved?"
  "How many times did AI connections surprise you?"
  "Would you pay $5/month to keep this?"

Collect testimonials and use cases for launch.
```

### Phase 4: Product Hunt Launch

```
Timing: When beta has 100+ users AND NPS > 40

Title: "CrossMem — Your thinking compounds"
Tagline: "Every AI conversation you've ever had, saved and connected"

Angle: Consumer-friendly (vs Mem0's developer API angle)
  → "Install in 1 click. Your AI conversations auto-save to Google Drive.
     AI finds the connections you missed."

Maker comment emphasizes:
  1. Your data stays in YOUR Google Drive (privacy angle)
  2. Zero friction — install and forget
  3. The compound interest moment — "3 months in, it knew things about
     my work that I'd forgotten"
```

### What We Don't Do (based on belief)

```
❌ B2B / enterprise sales       Compound interest is personal
❌ Developer API / platform     That's Mem0's fight
❌ Social features (early)      Individual compounding first
❌ AI content generation        We are memory, not creation
❌ Freemium feature walls       Deposit channels must be free and unlimited
   on capture
❌ Paid ads (early)             Organic + content first, ads after PMF
```

## Timeline

```
Month 1: Extension + Web landing page              ✅ Done
Month 2: iOS MVP (Share Sheet + Feed + Topics)
Month 3: Supabase RAG backend + Pro tier IAP
Month 4: Import ChatGPT/Claude history + AI connection push
Month 5: Beta 100 users → measure L4 & L5
Month 6: Product Hunt launch
```

## Success Criteria (Month 6)

```
Extension installs: 1,000+
iOS installs: 500+
Users at L4 (30+ sources): 200+
Users with compound moments: 100+
Pro conversions: 50+ (>5% of L4 users)
MRR: $250+ (50 x $5)
NPS: >40
```

These are modest numbers. The goal at month 6 is not scale —
it's validation that thinking compounds and people will pay for it.
