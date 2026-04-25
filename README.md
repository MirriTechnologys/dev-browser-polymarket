BTC Up or Down 5m – Polymarket BotAdvanced Automated Trading & Monitoring Bot using dev-browser + Claude Code



![TypeScript](https://img.shields.io/badge/TypeScript-5.8-blue.svg)
![Python](https://img.shields.io/badge/Python-3.12-green.svg)
![dev-browser](https://img.shields.io/badge/dev--browser-Claude%20Code-purple.svg)


BTC Up or Down every 5 minutes only • Dynamic URL that changes every 300 seconds • Full browser control via dev-browser
 Key Features100% Automatic URL Generation: btc-updown-5m-{window_ts} where window_ts = floor(now / 300) * 300 (no manual scraping or searching required).
Persistent Sessions: Log in once and maintain the session all day.
Hybrid Architecture:TypeScript → dev-browser scripts (executed inside QuickJS sandbox).
Python → Orchestrator, Scheduler, Technical Analysis, Logging & Persistence.

Real-time extraction of Up/Down odds directly from the live DOM.
Full multi-step workflows: login → navigate → extract prices → trade → screenshot → structured JSON export.
Built-in error recovery, retry logic, and rate-limit handling.
Advanced logging + JSON output + easy integration with any strategy (TA, ML, etc.).
Fully sandboxed and secure (no filesystem or host access).

 PrerequisitesClaude Code (Anthropic) with dev-browser skill installed
Python 3.12+
Node.js (optional – for TypeScript compilation)
Polymarket account + connected wallet (MetaMask or WalletConnect)
USDC on Polygon for trading

Install dev-browser (one-time)bash

# Inside Claude Code
/plugin marketplace add sawyerhood/dev-browser
/plugin install dev-browser@sawyerhood/dev-browser

 Installationbash

git clone <your-repo-url>
cd polymarket-btc-5m-bot

# Python dependencies
pip install -r requirements.txt

# TypeScript (optional)
cd ts && npm install && npm run build

 Usage1. Run the Python Orchestratorbash

python python/main.py --mode live --strategy ta

2. Core dev-browser Scripts (TypeScript)All scripts are executed inside Claude Code:ts

// ts/browser.ts
import { Browser } from "dev-browser";

const browser = new Browser({ persistent: true });

await browser.goto(`https://polymarket.com/event/btc-updown-5m-${getCurrentWindowTs()}`);

Critical function (optimized for the 5m market):ts

// ts/market.ts
export function getCurrentWindowTs(): number {
  const now = Math.floor(Date.now() / 1000);
  return Math.floor(now / 300) * 300; // divisible by 300 seconds (5 minutes)
}

 Example dev-browser Scripts (TypeScript)Extract Current Oddsts

// ts/extract-odds.ts
const upPrice = await browser.evaluate(() => 
  document.querySelector('[data-testid="yes-price"]')?.textContent?.trim()
);
const downPrice = await browser.evaluate(() => 
  document.querySelector('[data-testid="no-price"]')?.textContent?.trim()
);

return {
  up: parseFloat(upPrice || "0"),
  down: parseFloat(downPrice || "0"),
  timestamp: Date.now(),
  windowTs: getCurrentWindowTs()
};

Execute a Trade (Buy Up or Down)ts

// ts/trade.ts
await browser.click('[data-testid="buy-yes-button"]'); // or buy-no-button
await browser.fill('input[placeholder*="Amount"]', amount.toString());
await browser.click('button:has-text("Confirm trade")');

// Wallet signature is handled manually or via connected extension

 Python Orchestrator (main.py example)python

import asyncio
from datetime import datetime
from ts.market import get_current_window_ts  # or pure Python equivalent

async def main():
    while True:
        window_ts = get_current_window_ts()
        url = f"https://polymarket.com/event/btc-updown-5m-{window_ts}"
        
        # Call Claude Agent with dev-browser
        result = await run_claude_agent(
            f"Use dev-browser → open {url} → extract current Up/Down odds → apply strategy"
        )
        
        print(f"[{datetime.now()}] BTC 5m Market {window_ts} → {result}")
        await asyncio.sleep(300)  # Wait exactly 5 minutes

 Important Security & Legal NotesPolymarket ToS: Automation is allowed for personal use. Avoid spam or excessive requests.
Wallet Security: Never store private keys in code. Use MetaMask + manual signing or WalletConnect.
Risk Disclaimer: This bot is for educational and research purposes only. Trading involves significant risk.
dev-browser is fully sandboxed → completely safe (no filesystem or host access).

 RoadmapAdd Technical Analysis (RSI, EMA, volume via Gamma API)
Full auto-trading with configurable thresholds
Telegram / Discord alerts
Backtesting engine
Multi-asset support (ETH, SOL, etc.)
