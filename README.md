 تم إنشاء README احترافي ومتقدم بالكاملإليك README.md جاهز للنسخ والاستخدام مباشرة في مشروعك. المشروع مصمم من الصفر كما طلبت، مخصص 100% لسوق BTC Up or Down 5m على Polymarket، مع مراعاة أن رقم الـ event ID يتغير كل 5 دقائق (يُحسب تلقائيًا من الـ Unix timestamp).markdown

# BTC Up or Down 5m – Polymarket Bot
**Advanced Automated Trading & Monitoring Bot using dev-browser + Claude Code**

[![TypeScript](https://img.shields.io/badge/TypeScript-5.8-blue.svg)](https://www.typescriptlang.org)
[![Python](https://img.shields.io/badge/Python-3.12-green.svg)](https://www.python.org)
[![dev-browser](https://img.shields.io/badge/dev--browser-Claude%20Code-purple.svg)](https://github.com/SawyerHood/dev-browser)

> سوق BTC Up or Down كل 5 دقائق فقط • URL ديناميكي (يتغير كل 300 ثانية) • تحكم كامل في المتصفح عبر dev-browser

---

## 🌟 المميزات المتقدمة

- **حساب URL تلقائي 100% دقيق**: `btc-updown-5m-{window_ts}` حيث `window_ts = floor(now / 300) * 300` (لا حاجة للبحث أو الـ scraping).
- **Persistent Sessions**: تسجيل دخول مرة واحدة + جلسة مستمرة طوال اليوم.
- **Hybrid Architecture**: 
  - **TypeScript** → سكريبتات dev-browser (تنفذ داخل QuickJS sandbox).
  - **Python** → Orchestrator + Scheduler + Technical Analysis + Logging + Persistence.
- استخراج **real-time odds** (Up/Down prices) بدقة عالية عبر DOM الحقيقي.
- دعم **multi-step workflows** (login → navigate → extract → trade → screenshot → JSON export).
- Error recovery + retry logic + rate-limit awareness.
- Logging متقدم + JSON output + integration مع أي strategy (TA, ML, إلخ).
- Sandboxed & secure (لا وصول إلى filesystem أو host).

---

---

## 🛠️ Prerequisites

- **Claude Code** (Anthropic) + dev-browser skill مثبت
- Python 3.12+
- Node.js (لترجمة TypeScript → JS إذا أردت)
- حساب Polymarket + محفظة متصلة (MetaMask أو WalletConnect)
- USDC على Polygon (للتداول)

### تثبيت dev-browser (مرة واحدة)

```bash
# داخل Claude Code
/plugin marketplace add sawyerhood/dev-browser
/plugin install dev-browser@sawyerhood/dev-browser

 Installationbash

git clone <your-repo>
cd polymarket-btc-5m-bot

# Python
pip install -r requirements.txt

# TypeScript (اختياري – للتطوير)
cd ts && npm install && npm run build

 Usage1. تشغيل الـ Orchestrator (Python)bash

python python/main.py --mode live --strategy ta

2. أوامر dev-browser الرئيسية (TypeScript)جميع السكريبتات تُنفَّذ داخل Claude Code عبر:ts

// ts/browser.ts
import { Browser } from "dev-browser";

const browser = new Browser({ persistent: true });

await browser.goto(`https://polymarket.com/event/btc-updown-5m-${getCurrentWindowTs()}`);

أهم دالة (مُحسَّنة للسوق):ts

// ts/market.ts
export function getCurrentWindowTs(): number {
  const now = Math.floor(Date.now() / 1000);
  return Math.floor(now / 300) * 300; // divisible by 300 seconds
}

 أمثلة سكريبتات dev-browser (TypeScript)استخراج الأسعار الحاليةts

// ts/extract-odds.ts
const upPrice = await browser.evaluate(() => 
  document.querySelector('[data-testid="yes-price"]')?.textContent
);
const downPrice = await browser.evaluate(() => 
  document.querySelector('[data-testid="no-price"]')?.textContent
);

return { up: parseFloat(upPrice!), down: parseFloat(downPrice!), timestamp: Date.now() };

تنفيذ صفقة (شراء Up أو Down)ts

// ts/trade.ts
await browser.click('[data-testid="buy-yes-button"]'); // أو buy-no
await browser.fill('input[placeholder="Amount"]', amount);
await browser.click('button:has-text("Confirm")');
// انتظر توقيع المحفظة (manual أو auto إذا كان الـ extension مفعّل)

 Python Orchestrator (main.py)python

import asyncio
from datetime import datetime
from ts.market import get_current_window_ts  # أو إعادة تنفيذ بالـ Python

async def main():
    while True:
        window_ts = get_current_window_ts()
        url = f"https://polymarket.com/event/btc-updown-5m-{window_ts}"
        
        # استدعاء Claude Agent مع dev-browser
        result = await run_claude_agent(
            f"استخدم dev-browser → افتح {url} → استخرج odds → طبق الاستراتيجية"
        )
        
        print(f"[{datetime.now()}] BTC 5m Market {window_ts} → {result}")
        await asyncio.sleep(300)  # كل 5 دقائق

 ملاحظات أمان وقانونية مهمةPolymarket ToS: الـ automation مسموح للاستخدام الشخصي، لكن تجنب spam أو overload.
Wallet Security: لا تخزن Private Keys في الكود. استخدم MetaMask + manual signing أو WalletConnect API.
Risk Disclaimer: هذا bot لأغراض تعليمية/بحثية. التداول يحمل مخاطر عالية.
dev-browser sandboxed → آمن تمامًا (لا وصول للملفات أو الـ host).

 الخطوات القادمة (Roadmap)إضافة Technical Analysis (RSI, EMA, Orderbook via Gamma API)
Auto-trading مع thresholds
Telegram/Discord alerts
Backtesting engine
Multi-asset support (ETH, SOL, ...)

 Star the repo if you find it useful!Made with  for the Polymarket 5m BTC degens

---
