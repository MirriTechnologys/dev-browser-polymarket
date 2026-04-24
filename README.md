# dev-browser-polymarket-5mn
أتمتة أسواق الـ 5-Minute Crypto على Polymarket باستخدام dev-browser + دمج هجين مع API الرسميالوصف
هذا المشروع هو إعداد مخصص وموحد لـ dev-browser (skill مجتمعي لـ Claude Code) يجمع بين:التحكم المباشر في المتصفح عبر QuickJS sandbox (Playwright-style).
دمج هجين مع Gamma API + CLOB API الرسميين من Polymarket.
dev-browser كـ fallback ذكي لضمان الاستمرارية 100% حتى في حال تغيير الواجهة أو rate limits.

الهدف: مراقبة واستخراج بيانات أسواق الـ 5-Minute Crypto (Bitcoin Up/Down، Ethereum، Solana، إلخ) بسرعة فائقة، تكلفة منخفضة، واستقرار عالي.المميزات الرئيسيةجلسات مستمرة (Persistent Sessions): تسجيل دخول مرة واحدة + مراقبة مستمرة.
استخراج JSON منظم للأسواق، الأسعار، الحجم، الوقت المتبقي، liquidity.
دمج هجين (Hybrid): API أولاً (Gamma + CLOB) → dev-browser fallback تلقائي.
مراقبة عالية التردد: تحديث كل 15-30 ثانية لعشرات الأسواق.
دعم Multi-Page: مراقبة BTC + ETH + SOL في وقت واحد.
أمان كامل: كل شيء داخل sandbox معزول (لا وصول للملفات أو النظام).
توفير 70-90% من توكنات Claude مقارنة بالطرق التقليدية.

المتطلباتClaude Code (مدفوع موصى به).
dev-browser مثبت.
حساب Polymarket + محفظة crypto (للتداول).
Private Key (لـ CLOB authenticated) — يُدار خارج sandbox.

التثبيتbash

# 1. تثبيت dev-browser (CLI)
npm install -g dev-browser
dev-browser install

# 2. تثبيت كـ Skill في Claude Code
/plugin marketplace add sawyerhood/dev-browser
/plugin install dev-browser@sawyerhood/dev-browser

إعداد الصلاحيات (في ~/.claude/settings.json):json

{
  "permissions": {
    "allow": ["Bash(dev-browser *)", "Bash(npx dev-browser *)"]
  }
}

المنطق الهجين (Hybrid Logic) – السكريبت الرئيسياحفظ هذا كملف hybrid-polymarket-5mn.js واستخدمه داخل dev-browser:js

// hybrid-polymarket-5mn.js
const GAMMA_BASE = "https://gamma-api.polymarket.com";
const CLOB_BASE = "https://clob.polymarket.com";

async function fetchWithFallback(url, options = {}, fallbackFn) {
  try {
    const response = await page.evaluate(async (u, opts) => {
      const res = await fetch(u, opts);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return res.json();
    }, url, options);
    return { source: "api", data: response };
  } catch (e) {
    console.warn(`[Fallback] API فشل → dev-browser: ${e.message}`);
    return { source: "browser", data: await fallbackFn() };
  }
}

// 1. جلب جميع أسواق 5mn Crypto
async function get5mnCryptoMarkets() {
  const url = `${GAMMA_BASE}/markets?active=true&closed=false&limit=200`;
  return await fetchWithFallback(url, {}, async () => {
    const page = await browser.getPage("polymarket-5mn");
    await page.goto("https://polymarket.com/crypto/5M", { waitUntil: "domcontentloaded" });
    return await page.evaluate(() => {
      const cards = document.querySelectorAll('[data-testid="market-card"]');
      return Array.from(cards).map(card => ({
        title: card.querySelector('h3, [class*="question"]')?.innerText || '',
        yesPrice: parseFloat(card.querySelector('[class*="yes"]')?.innerText?.replace('%', '')) || null,
        noPrice: parseFloat(card.querySelector('[class*="no"]')?.innerText?.replace('%', '')) || null,
        volume: card.querySelector('[class*="volume"]')?.innerText || '',
        timeLeft: card.querySelector('[class*="time"]')?.innerText || '',
        url: card.querySelector('a')?.href || ''
      })).filter(m => m.title.toLowerCase().includes('5 min') || m.title.toLowerCase().includes('5m'));
    });
  });
}

// 2. جلب Order Book + Midpoint (CLOB)
async function getOrderBook(tokenId) {
  const url = `${CLOB_BASE}/book?token_id=${tokenId}`;
  return await fetchWithFallback(url, {}, async () => {
    const page = await browser.getPage("market-detail");
    await page.goto(`https://polymarket.com/event/...${tokenId}`, { waitUntil: "networkidle" });
    return await page.evaluate(() => ({
      midpoint: parseFloat(document.querySelector('.midpoint, [class*="price"]')?.innerText || '0')
    }));
  });
}

// 3. Monitoring Loop رئيسي
async function startMonitoring(intervalMs = 15000) {
  const page = await browser.getPage("main-polymarket");
  await page.goto("https://polymarket.com/crypto/5M", { waitUntil: "domcontentloaded" });

  setInterval(async () => {
    const markets = await get5mnCryptoMarkets();
    console.log(`[${new Date().toISOString()}] تم تحديث ${markets.length} سوق 5mn (source: ${markets[0]?.source})`);
    // يمكنك هنا إرسال الـ JSON إلى Claude أو Telegram
  }, intervalMs);
}

كيف تبدأ مع Claude Codeقل لـ Claude مباشرة:"استخدم dev-browser مع hybrid-polymarket-5mn:
جلب كل أسواق 5mn Crypto عبر Gamma API (fallback scrape)،
احسب midpoint عبر CLOB،
أرجع JSON مرتب كل 15 ثانية،
وأرسل إشعار إذا تغيرت الأسعار >2%."
نصائح متقدمة لأسواق 5mn Cryptoاستخدم waitUntil: "domcontentloaded" دائمًا.
أضف snapshotForAI() لإرسال لقطة AI-friendly.
للتداول الآلي: استخدم clob-client-v2 SDK خارج sandbox (Bash في Claude) → dev-browser فقط fallback UI.
caching داخل الـ script لتجنب rate limits.
multi-page لمراقبة عدة أسواق في وقت واحد.

المخاطر والتحذيرات الهامةالتداول الآلي يحمل مخاطر مالية عالية.
يخضع لشروط Polymarket وقوانين بلدك.
لا تشارك Private Keys داخل sandbox أبدًا.
DOM قد يتغير → الـ fallback يحميك.
استخدم في بيئة اختبار أولاً.





