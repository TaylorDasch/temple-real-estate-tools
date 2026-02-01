# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Automated real estate investment deal analyzer for Temple, Belton, Harker Heights, and Killeen TX markets. Fetches active listings from the RentCast API, applies investment heuristics, enriches top candidates with rent estimates, and outputs ranked deals as JSON files. Runs weekly via GitHub Actions.

## Commands

```bash
npm install                                    # Install dependencies (only axios)
RENTCAST_API_KEY=<key> npm run analyze         # Run full analysis pipeline
RENTCAST_API_KEY=<key> node src/index.js       # Same as above
```

No test framework or linter is configured. The `npm test` script just runs the analyzer.

## Architecture

**4-stage pipeline** executed per market (3 markets total):

1. **Fetch** (`src/rentcast.js`) — Pull active sale listings from RentCast API with rate limiting (250ms between requests)
2. **Heuristic filter** (`src/analyze.js`) — Estimate rent at $1.00/sqft, discard properties below 6% gross yield threshold
3. **Deep analysis** (`src/analyze.js`) — Enrich top 50 candidates with actual RentCast AVM rent estimates, calculate gross yield, cash flow, GRM, 1% rule
4. **Output** (`src/index.js`) — Rank by gross yield, write top 10 deals per market to `data/<market>.json`

**Key files:**
- `src/index.js` — Entry point, orchestrates pipeline across markets
- `src/config.js` — All tunable parameters (price range, yield thresholds, tax rate 2.4%, vacancy 8%, management 10%, market zip codes)
- `src/analyze.js` — Heuristic filtering, metrics calculation, ranking
- `src/rentcast.js` — RentCast API client with rate limiting and error handling

**Output:** `data/*.json` files consumed by a frontend via jsDelivr CDN.

## Environment

- **Node.js 20+** required
- **`RENTCAST_API_KEY`** env var required (RentCast Scale plan, 5000 requests/month)
- GitHub Actions runs weekly (Sunday 3 AM Central), commits results back to repo
