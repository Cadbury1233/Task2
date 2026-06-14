<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>FAQ Chatbot</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=Space+Grotesk:wght@500;700&display=swap');

    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    :root {
      --bg:       #0f1117;
      --surface:  #1a1d27;
      --surface2: #22263a;
      --border:   #2e3250;
      --accent:   #6c63ff;
      --accent2:  #a78bfa;
      --bot-bg:   #1e2235;
      --user-bg:  #6c63ff;
      --text:     #e2e8f0;
      --muted:    #7c85a2;
      --success:  #34d399;
      --radius:   16px;
    }

    body {
      font-family: 'Inter', sans-serif;
      background: var(--bg);
      color: var(--text);
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 24px 16px 32px;
    }

    /* ── Header ── */
    header {
      width: 100%; max-width: 760px;
      display: flex; align-items: center; gap: 14px;
      margin-bottom: 20px;
    }
    .bot-avatar {
      width: 52px; height: 52px; border-radius: 14px;
      background: linear-gradient(135deg, var(--accent), var(--accent2));
      display: flex; align-items: center; justify-content: center;
      font-size: 1.5rem; flex-shrink: 0;
      box-shadow: 0 0 20px rgba(108,99,255,0.4);
    }
    .header-text h1 {
      font-family: 'Space Grotesk', sans-serif;
      font-size: 1.4rem; font-weight: 700;
      color: var(--text);
    }
    .header-text p { color: var(--muted); font-size: 0.82rem; margin-top: 2px; }

    .status-dot {
      display: inline-block; width: 8px; height: 8px;
      background: var(--success); border-radius: 50%;
      margin-right: 6px; animation: pulse 2s infinite;
    }
    @keyframes pulse {
      0%,100% { opacity: 1; } 50% { opacity: 0.4; }
    }

    /* ── Main layout ── */
    .main {
      width: 100%; max-width: 760px;
      display: flex; flex-direction: column; gap: 12px;
    }

    /* ── Suggestion chips ── */
    .chips {
      display: flex; flex-wrap: wrap; gap: 8px;
      margin-bottom: 4px;
    }
    .chip {
      background: var(--surface2); border: 1px solid var(--border);
      color: var(--muted); font-size: 0.78rem; padding: 6px 14px;
      border-radius: 999px; cursor: pointer;
      transition: all 0.18s; white-space: nowrap;
    }
    .chip:hover { background: var(--accent); color: #fff; border-color: var(--accent); }

    /* ── Chat window ── */
    .chat-window {
      background: var(--surface);
      border: 1px solid var(--border);
      border-radius: var(--radius);
      height: 420px;
      overflow-y: auto;
      padding: 20px;
      display: flex; flex-direction: column; gap: 14px;
      scroll-behavior: smooth;
    }
    .chat-window::-webkit-scrollbar { width: 4px; }
    .chat-window::-webkit-scrollbar-track { background: transparent; }
    .chat-window::-webkit-scrollbar-thumb { background: var(--border); border-radius: 2px; }

    /* ── Messages ── */
    .msg { display: flex; gap: 10px; align-items: flex-end; max-width: 88%; }
    .msg.user { align-self: flex-end; flex-direction: row-reverse; }
    .msg.bot  { align-self: flex-start; }

    .msg-avatar {
      width: 30px; height: 30px; border-radius: 10px; flex-shrink: 0;
      display: flex; align-items: center; justify-content: center; font-size: 0.85rem;
    }
    .msg.bot  .msg-avatar { background: linear-gradient(135deg, var(--accent), var(--accent2)); }
    .msg.user .msg-avatar { background: var(--surface2); border: 1px solid var(--border); }

    .msg-bubble {
      padding: 11px 16px; border-radius: 14px;
      font-size: 0.9rem; line-height: 1.6;
    }
    .msg.bot  .msg-bubble {
      background: var(--bot-bg); border: 1px solid var(--border);
      border-bottom-left-radius: 4px; color: var(--text);
    }
    .msg.user .msg-bubble {
      background: var(--user-bg);
      border-bottom-right-radius: 4px; color: #fff;
    }

    .msg-meta {
      font-size: 0.68rem; color: var(--muted);
      margin-top: 4px; padding: 0 4px;
    }
    .msg.user .msg-meta { text-align: right; }

    .confidence-bar {
      margin-top: 8px; padding-top: 8px;
      border-top: 1px solid var(--border);
      font-size: 0.72rem; color: var(--muted);
    }
    .bar-track {
      height: 4px; background: var(--border); border-radius: 2px; margin-top: 4px;
    }
    .bar-fill {
      height: 100%; border-radius: 2px;
      background: linear-gradient(90deg, var(--accent), var(--accent2));
      transition: width 0.5s ease;
    }

    .typing {
      display: flex; gap: 5px; padding: 14px 16px;
      background: var(--bot-bg); border: 1px solid var(--border);
      border-radius: 14px; border-bottom-left-radius: 4px;
      width: fit-content;
    }
    .typing span {
      width: 7px; height: 7px; background: var(--muted);
      border-radius: 50%; animation: bounce 1.2s infinite;
    }
    .typing span:nth-child(2) { animation-delay: 0.2s; }
    .typing span:nth-child(3) { animation-delay: 0.4s; }
    @keyframes bounce {
      0%,60%,100% { transform: translateY(0); }
      30% { transform: translateY(-6px); }
    }

    /* ── Input area ── */
    .input-area {
      display: flex; gap: 10px; align-items: flex-end;
    }
    .input-wrap { flex: 1; position: relative; }
    textarea#userInput {
      width: 100%;
      background: var(--surface); border: 1px solid var(--border);
      border-radius: 12px; color: var(--text);
      font-family: 'Inter', sans-serif; font-size: 0.92rem;
      padding: 13px 16px; resize: none; height: 50px;
      max-height: 120px; line-height: 1.5;
      transition: border-color 0.2s, height 0.1s;
      overflow-y: hidden;
    }
    textarea#userInput:focus { outline: none; border-color: var(--accent); }
    textarea#userInput::placeholder { color: var(--muted); }

    .send-btn {
      width: 50px; height: 50px; border-radius: 12px; border: none; flex-shrink: 0;
      background: linear-gradient(135deg, var(--accent), var(--accent2));
      color: #fff; cursor: pointer; display: flex; align-items: center; justify-content: center;
      transition: all 0.18s; box-shadow: 0 4px 14px rgba(108,99,255,0.35);
    }
    .send-btn:hover { transform: translateY(-1px); box-shadow: 0 6px 18px rgba(108,99,255,0.5); }
    .send-btn:disabled { opacity: 0.4; cursor: not-allowed; transform: none; }

    /* ── NLP Info panel ── */
    .nlp-panel {
      background: var(--surface); border: 1px solid var(--border);
      border-radius: var(--radius); padding: 16px 20px;
    }
    .nlp-panel h3 {
      font-size: 0.78rem; font-weight: 600; text-transform: uppercase;
      letter-spacing: 0.08em; color: var(--muted); margin-bottom: 10px;
    }
    .nlp-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
    @media (max-width: 520px) { .nlp-grid { grid-template-columns: 1fr; } }
    .nlp-item label { font-size: 0.7rem; color: var(--muted); display: block; margin-bottom: 3px; }
    .nlp-item .value {
      font-size: 0.82rem; color: var(--text);
      background: var(--surface2); padding: 6px 10px;
      border-radius: 8px; border: 1px solid var(--border);
      min-height: 32px; word-break: break-word;
      font-family: 'Inter', sans-serif;
    }

    footer {
      margin-top: 20px; font-size: 0.75rem; color: var(--muted); text-align: center;
    }
  </style>
</head>
<body>

<header>
  <div class="bot-avatar">🤖</div>
  <div class="header-text">
    <h1>FAQ Assistant</h1>
    <p><span class="status-dot"></span>Online · NLP-powered · Cosine Similarity Matching</p>
  </div>
</header>

<div class="main">

  <!-- Quick question chips -->
  <div class="chips" id="chips"></div>

  <!-- Chat window -->
  <div class="chat-window" id="chatWindow"></div>

  <!-- Input -->
  <div class="input-area">
    <div class="input-wrap">
      <textarea id="userInput" placeholder="Ask me anything… (Enter to send)" rows="1"></textarea>
    </div>
    <button class="send-btn" id="sendBtn">
      <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2">
        <line x1="22" y1="2" x2="11" y2="13"/>
        <polygon points="22 2 15 22 11 13 2 9 22 2"/>
      </svg>
    </button>
  </div>

  <!-- NLP Debug Panel -->
  <div class="nlp-panel">
    <h3>🔬 NLP Processing Info</h3>
    <div class="nlp-grid">
      <div class="nlp-item">
        <label>Tokens (after cleaning)</label>
        <div class="value" id="nlpTokens">—</div>
      </div>
      <div class="nlp-item">
        <label>After Stopword Removal</label>
        <div class="value" id="nlpFiltered">—</div>
      </div>
      <div class="nlp-item">
        <label>Best Match Question</label>
        <div class="value" id="nlpMatch">—</div>
      </div>
      <div class="nlp-item">
        <label>Cosine Similarity Score</label>
        <div class="value" id="nlpScore">—</div>
      </div>
    </div>
  </div>

</div>

<footer>NLP Pipeline: Tokenize → Lowercase → Remove Stopwords → TF-IDF → Cosine Similarity</footer>

<script>
// ══════════════════════════════════════════════
//  FAQ DATABASE
// ══════════════════════════════════════════════
const FAQ_DATA = [
  // General
  { q: "What is your return policy?", a: "You can return any product within 30 days of purchase for a full refund. Items must be unused and in original packaging. Contact support@shop.com to initiate a return." },
  { q: "How long does shipping take?", a: "Standard shipping takes 5–7 business days. Express shipping (2–3 days) is available at checkout. International orders may take 10–15 business days." },
  { q: "Do you offer free shipping?", a: "Yes! We offer free standard shipping on all orders above ₹999. Orders below that amount have a flat ₹99 shipping fee." },
  { q: "How can I track my order?", a: "Once your order ships, you'll receive a tracking email with a link. You can also visit our website and enter your order ID under 'Track Order'." },
  { q: "What payment methods do you accept?", a: "We accept all major credit/debit cards, UPI, net banking, PayPal, and EMI options via select banks." },
  { q: "Can I cancel my order?", a: "Orders can be cancelled within 24 hours of placing them. After that, the order may already be dispatched. Contact us immediately at support@shop.com." },
  { q: "Is my personal information safe?", a: "Absolutely. We use SSL encryption and never share your data with third parties. See our Privacy Policy for full details." },
  { q: "How do I create an account?", a: "Click 'Sign Up' on the top right of our website, enter your email and a password, and verify your email address. It takes less than a minute!" },
  { q: "I forgot my password. What do I do?", a: "Click 'Forgot Password' on the login page. Enter your registered email and we'll send a reset link within 2 minutes." },
  { q: "Do you have a mobile app?", a: "Yes! Our app is available on both iOS (App Store) and Android (Google Play). Search for 'ShopEasy' to download it free." },
  // Product
  { q: "Are your products genuine?", a: "100% genuine. All products are sourced directly from authorized manufacturers and distributors. We provide authenticity certificates for branded items." },
  { q: "How do I find the right size?", a: "Each product page has a detailed size guide with measurements. If you're between sizes, we recommend sizing up. Contact support for personal sizing help." },
  { q: "Can I change my order after placing it?", a: "Order modifications (size, color, quantity) are possible within 2 hours of placing the order. Contact us via chat or email immediately." },
  { q: "What if I receive a damaged product?", a: "We're sorry to hear that! Please photograph the damage and email us within 48 hours. We'll send a replacement or issue a full refund — no return required." },
  { q: "Do you offer gift wrapping?", a: "Yes! Select 'Gift Wrap' at checkout for ₹49. You can add a personalized message that we'll include with the package." },
  // Account & Support
  { q: "How do I contact customer support?", a: "You can reach us via: Email: support@shop.com | Phone: 1800-123-4567 (Mon–Sat, 9am–6pm) | Live Chat on our website | WhatsApp: +91-98765-43210" },
  { q: "What are your working hours?", a: "Our customer support team is available Monday to Saturday, 9:00 AM to 6:00 PM IST. For urgent issues, email us and we'll respond within 4 hours." },
  { q: "How do I apply a coupon code?", a: "At checkout, look for the 'Apply Coupon' field and enter your code. The discount will be applied automatically before payment." },
  { q: "Do you have a loyalty rewards program?", a: "Yes! Our 'ShopCoins' program gives you 1 coin per ₹10 spent. Coins can be redeemed on future purchases. Check your account dashboard for your balance." },
  { q: "How do I delete my account?", a: "To delete your account, go to Settings → Privacy → Delete Account. Note: this action is irreversible and all order history will be lost." },
];

// ══════════════════════════════════════════════
//  NLP ENGINE
// ══════════════════════════════════════════════

// English stopwords (NLTK-style)
const STOPWORDS = new Set([
  "i","me","my","myself","we","our","ours","ourselves","you","your","yours",
  "yourself","yourselves","he","him","his","himself","she","her","hers",
  "herself","it","its","itself","they","them","their","theirs","themselves",
  "what","which","who","whom","this","that","these","those","am","is","are",
  "was","were","be","been","being","have","has","had","having","do","does",
  "did","doing","a","an","the","and","but","if","or","because","as","until",
  "while","of","at","by","for","with","about","against","between","into",
  "through","during","before","after","above","below","to","from","up","down",
  "in","out","on","off","over","under","again","further","then","once","here",
  "there","when","where","why","how","all","both","each","few","more","most",
  "other","some","such","no","nor","not","only","own","same","so","than","too",
  "very","s","t","can","will","just","don","should","now","d","ll","m","o",
  "re","ve","y","ain","aren","couldn","didn","doesn","hadn","hasn","haven",
  "isn","ma","mightn","mustn","needn","shan","shouldn","wasn","weren","won",
  "wouldn","i'm","you're","he's","she's","it's","we're","they're","i've",
  "please","tell","let","know","give","need","want","get","make","use"
]);

// Tokenize: lowercase, remove punctuation, split
function tokenize(text) {
  return text
    .toLowerCase()
    .replace(/[^\w\s]/g, ' ')
    .split(/\s+/)
    .filter(t => t.length > 1);
}

// Remove stopwords
function removeStopwords(tokens) {
  return tokens.filter(t => !STOPWORDS.has(t));
}

// Simple stemmer (suffix stripping)
function stem(word) {
  return word
    .replace(/ing$/, '').replace(/tion$/, '').replace(/tions$/, '')
    .replace(/ness$/, '').replace(/ment$/, '').replace(/able$/, '')
    .replace(/ible$/, '').replace(/ful$/, '').replace(/less$/, '')
    .replace(/ly$/, '').replace(/ed$/, '').replace(/er$/, '')
    .replace(/est$/, '').replace(/s$/, '');
}

// Full NLP pipeline
function preprocess(text) {
  const tokens = tokenize(text);
  const filtered = removeStopwords(tokens);
  const stemmed = filtered.map(stem).filter(t => t.length > 1);
  return { tokens, filtered, stemmed };
}

// Build vocabulary from all FAQ questions
function buildVocabulary() {
  const vocab = new Set();
  FAQ_DATA.forEach(({ q }) => {
    const { stemmed } = preprocess(q);
    stemmed.forEach(t => vocab.add(t));
  });
  return Array.from(vocab);
}

// TF vector for a document
function tfVector(stemmed, vocab) {
  const freq = {};
  stemmed.forEach(t => freq[t] = (freq[t] || 0) + 1);
  return vocab.map(word => (freq[word] || 0) / (stemmed.length || 1));
}

// Cosine similarity between two vectors
function cosineSimilarity(a, b) {
  let dot = 0, magA = 0, magB = 0;
  for (let i = 0; i < a.length; i++) {
    dot  += a[i] * b[i];
    magA += a[i] * a[i];
    magB += b[i] * b[i];
  }
  const denom = Math.sqrt(magA) * Math.sqrt(magB);
  return denom === 0 ? 0 : dot / denom;
}

// Pre-build vocab and FAQ vectors once
const VOCAB = buildVocabulary();
const FAQ_VECTORS = FAQ_DATA.map(({ q }) => {
  const { stemmed } = preprocess(q);
  return tfVector(stemmed, VOCAB);
});

// Find best matching FAQ
function findBestMatch(userQuery) {
  const nlp = preprocess(userQuery);
  const qVec = tfVector(nlp.stemmed, VOCAB);

  let bestScore = -1, bestIdx = -1;
  FAQ_VECTORS.forEach((fVec, i) => {
    const score = cosineSimilarity(qVec, fVec);
    if (score > bestScore) { bestScore = score; bestIdx = i; }
  });

  return {
    nlp,
    faq: FAQ_DATA[bestIdx],
    score: bestScore,
    idx: bestIdx
  };
}

// ══════════════════════════════════════════════
//  CHAT UI
// ══════════════════════════════════════════════
const chatWindow = document.getElementById('chatWindow');
const userInput  = document.getElementById('userInput');
const sendBtn    = document.getElementById('sendBtn');
const chipsEl    = document.getElementById('chips');

// Suggestion chips — sample questions
const SUGGESTIONS = [
  "What's your return policy?",
  "How long does shipping take?",
  "Do you offer free shipping?",
  "How can I track my order?",
  "How do I contact support?"
];

SUGGESTIONS.forEach(q => {
  const chip = document.createElement('button');
  chip.className = 'chip';
  chip.textContent = q;
  chip.addEventListener('click', () => { userInput.value = q; handleSend(); });
  chipsEl.appendChild(chip);
});

function formatTime() {
  return new Date().toLocaleTimeString([], { hour:'2-digit', minute:'2-digit' });
}

function addMessage(role, html, score = null, matchQ = null) {
  const wrap = document.createElement('div');
  wrap.className = `msg ${role}`;

  const avatar = document.createElement('div');
  avatar.className = 'msg-avatar';
  avatar.textContent = role === 'bot' ? '🤖' : '👤';

  const inner = document.createElement('div');

  const bubble = document.createElement('div');
  bubble.className = 'msg-bubble';
  bubble.innerHTML = html;

  if (role === 'bot' && score !== null && matchQ) {
    const bar = document.createElement('div');
    bar.className = 'confidence-bar';
    const pct = Math.round(score * 100);
    bar.innerHTML = `
      <span>Match confidence: ${pct}%</span>
      <div class="bar-track"><div class="bar-fill" style="width:0%"></div></div>`;
    bubble.appendChild(bar);
    setTimeout(() => {
      const fill = bar.querySelector('.bar-fill');
      if (fill) fill.style.width = pct + '%';
    }, 50);
  }

  const meta = document.createElement('div');
  meta.className = 'msg-meta';
  meta.textContent = formatTime();

  inner.appendChild(bubble);
  inner.appendChild(meta);
  wrap.appendChild(avatar);
  wrap.appendChild(inner);
  chatWindow.appendChild(wrap);
  chatWindow.scrollTop = chatWindow.scrollHeight;
  return wrap;
}

function addTyping() {
  const wrap = document.createElement('div');
  wrap.className = 'msg bot';
  wrap.id = 'typing';
  wrap.innerHTML = `
    <div class="msg-avatar">🤖</div>
    <div class="typing"><span></span><span></span><span></span></div>`;
  chatWindow.appendChild(wrap);
  chatWindow.scrollTop = chatWindow.scrollHeight;
}

function removeTyping() {
  const t = document.getElementById('typing');
  if (t) t.remove();
}

function updateNLPPanel(nlp, matchQ, score) {
  document.getElementById('nlpTokens').textContent =
    nlp.tokens.join(', ') || '—';
  document.getElementById('nlpFiltered').textContent =
    nlp.filtered.join(', ') || '—';
  document.getElementById('nlpMatch').textContent = matchQ || '—';
  document.getElementById('nlpScore').textContent =
    score !== null ? `${(score * 100).toFixed(1)}% similarity` : '—';
}

async function handleSend() {
  const text = userInput.value.trim();
  if (!text) return;

  userInput.value = '';
  userInput.style.height = '50px';
  sendBtn.disabled = true;

  addMessage('user', escapeHtml(text));

  // Simulate NLP processing delay
  addTyping();
  await delay(520 + Math.random() * 300);
  removeTyping();

  const { nlp, faq, score } = findBestMatch(text);

  let responseHtml;
  let displayScore = score;

  if (score < 0.05) {
    // No match
    responseHtml = `I couldn't find a close match for your question. Try rephrasing, or pick one of the suggestions above. You can also contact us at <strong>support@shop.com</strong>.`;
    displayScore = null;
    updateNLPPanel(nlp, 'No confident match found', score);
  } else {
    responseHtml = escapeHtml(faq.a).replace(
      /(support@\S+|www\.\S+|\+\d[\d\-]+)/g,
      '<strong>$1</strong>'
    );
    updateNLPPanel(nlp, faq.q, score);
  }

  addMessage('bot', responseHtml, displayScore, faq?.q);
  sendBtn.disabled = false;
  userInput.focus();
}

function escapeHtml(s) {
  return s.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;');
}
function delay(ms) { return new Promise(r => setTimeout(r, ms)); }

// Event listeners
sendBtn.addEventListener('click', handleSend);
userInput.addEventListener('keydown', e => {
  if (e.key === 'Enter' && !e.shiftKey) { e.preventDefault(); handleSend(); }
});
userInput.addEventListener('input', function() {
  this.style.height = '50px';
  this.style.height = Math.min(this.scrollHeight, 120) + 'px';
});

// Welcome message
window.addEventListener('DOMContentLoaded', async () => {
  await delay(300);
  addMessage('bot',
    `👋 Hi! I'm your <strong>FAQ Assistant</strong>.<br><br>
    Ask me anything about <em>orders, shipping, returns, payments, or your account</em>.<br>
    I use <strong>NLP + Cosine Similarity</strong> to find the best answer from my knowledge base!`
  );
});
</script>
</body>
</html>
