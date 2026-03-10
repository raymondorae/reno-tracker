# Renovation Tracker

A full-featured renovation budget tracker with tender comparison, milestone payments, scenario modelling, and owner-supplied fixtures tracking.

## Deploy to Vercel (2 minutes)

### Option A — GitHub (recommended)
1. Create a new GitHub repo and push these files to it
2. Go to [vercel.com](https://vercel.com) and sign in with GitHub
3. Click **"Add New Project"** → import your repo
4. Click **Deploy** — no config needed, it just works

### Option B — Vercel CLI
```bash
npm i -g vercel
cd reno-tracker
vercel
```
Follow the prompts. Done.

### Option C — Drag & drop
1. Go to [vercel.com/new](https://vercel.com/new)
2. Drag the entire `reno-tracker` folder onto the page
3. Done.

## How it works
- Single `index.html` file — React + Tailwind loaded from CDN
- Data persists in your browser's localStorage
- No server, no database, no build step
- Works on desktop and mobile

## Features
- **Dashboard** — Budget overview, utilisation bar, selected tender summary
- **Tender Comparison** — Spec-to-tender matrix with deviation/omission flags
- **Milestones** — Payment schedule tied to contract percentage
- **Owner Supplied** — Track fixtures & fittings you source yourself
- **Scenarios** — What-if modelling with toggle-able line items

## Customising with your real data
Upload your spec document and builder tenders to Claude — it will cross-reference them and give you the data to replace the sample values.
