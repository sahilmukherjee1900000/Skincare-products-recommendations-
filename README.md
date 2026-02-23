# Skincare-products-recommendations-
It will provide accurate skin product recommendations 
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>LLM-style Skincare AI Agent • Sahil's Recommendation Tool</title>
  <style>
    :root {
      --primary: #6a4c93;
      --primary-light: #9b7fd9;
      --bg: #f9f7fd;
      --card: white;
      --text: #333;
      --gray: #666;
    }

    body {
      font-family: system-ui, -apple-system, sans-serif;
      background: linear-gradient(135deg, #f5f0ff, #e8e0ff);
      color: var(--text);
      margin: 0;
      padding: 20px;
      min-height: 100vh;
      line-height: 1.6;
    }

    .container {
      max-width: 720px;
      margin: 0 auto;
      background: var(--card);
      border-radius: 16px;
      box-shadow: 0 10px 40px rgba(0,0,0,0.08);
      overflow: hidden;
    }

    header {
      background: var(--primary);
      color: white;
      padding: 2rem 1.5rem;
      text-align: center;
    }

    h1 {
      margin: 0;
      font-size: 2.1rem;
    }

    .subtitle {
      opacity: 0.9;
      margin-top: 0.5rem;
    }

    .content {
      padding: 2rem 1.5rem;
    }

    .question {
      margin-bottom: 1.8rem;
      font-size: 1.1rem;
    }

    .question strong {
      color: var(--primary);
    }

    .options {
      display: flex;
      flex-direction: column;
      gap: 0.9rem;
      margin-top: 1rem;
    }

    label {
      display: flex;
      align-items: center;
      gap: 12px;
      padding: 14px 18px;
      background: #f8f6fc;
      border-radius: 12px;
      cursor: pointer;
      transition: all 0.2s;
      font-size: 1.05rem;
      border: 2px solid transparent;
    }

    label:hover {
      background: #f0eaff;
      border-color: var(--primary-light);
    }

    input[type="radio"] {
      accent-color: var(--primary);
      width: 20px;
      height: 20px;
    }

    button {
      background: var(--primary);
      color: white;
      border: none;
      padding: 14px 32px;
      font-size: 1.1rem;
      border-radius: 12px;
      cursor: pointer;
      margin-top: 1.5rem;
      transition: all 0.25s;
    }

    button:hover {
      background: var(--primary-light);
      transform: translateY(-2px);
    }

    button:disabled {
      opacity: 0.6;
      cursor: not-allowed;
    }

    #reasoning {
      margin: 2rem 0;
      padding: 1.5rem;
      background: #f8f6ff;
      border-left: 5px solid var(--primary);
      border-radius: 8px;
      white-space: pre-wrap;
      font-family: 'Segoe UI', monospace;
      line-height: 1.7;
    }

    .products {
      margin-top: 2rem;
    }

    .product-card {
      background: #fff;
      border: 1px solid #eee;
      border-radius: 12px;
      padding: 1.4rem;
      margin-bottom: 1.2rem;
      box-shadow: 0 4px 12px rgba(0,0,0,0.05);
    }

    .product-card h3 {
      margin: 0 0 0.6rem;
      color: var(--primary);
    }

    .product-card .type {
      font-weight: bold;
      color: #555;
    }

    #progress {
      height: 6px;
      background: #e0d4ff;
      margin-bottom: 1.5rem;
      border-radius: 3px;
      overflow: hidden;
    }

    #progress-bar {
      height: 100%;
      width: 0%;
      background: var(--primary);
      transition: width 0.4s ease;
    }

    footer {
      text-align: center;
      padding: 1.5rem;
      color: var(--gray);
      font-size: 0.95rem;
      border-top: 1px solid #eee;
    }
  </style>
</head>
<body>

<div class="container">
  <header>
    <h1>Skincare AI Agent</h1>
    <div class="subtitle">Powered by LLM-style reasoning • Personalized product suggestions</div>
  </header>

  <div class="content">

    <div id="progress"><div id="progress-bar"></div></div>

    <div id="quiz">
      <div class="question" id="q-text"></div>
      <div class="options" id="q-options"></div>
      <button id="nextBtn" disabled>Next →</button>
    </div>

    <div id="result" style="display:none;">
      <h2>Your Personalized Skincare Recommendations</h2>
      <div id="reasoning"></div>
      <div class="products" id="products"></div>
      <button onclick="restart()">Start Over</button>
    </div>

  </div>

  <footer>
    Simple LLM-style Skincare Advisor • Built with HTML + CSS + JavaScript
  </footer>
</div>

<script>
// ──────────────────────────────────────────────
//   Skincare AI Agent – Product Database
// ──────────────────────────────────────────────

const products = [
  { name: "CeraVe Hydrating Facial Cleanser", type: "Cleanser", goodFor: ["dry", "sensitive"], keyIngredients: "ceramides, hyaluronic acid", scoreBoost: { dry: 30, sensitive: 25 } },
  { name: "La Roche-Posay Effaclar Gel", type: "Cleanser", goodFor: ["oily", "acne"], keyIngredients: "thermal water, gentle surfactants", scoreBoost: { oily: 35, acne: 40 } },
  { name: "The Ordinary Niacinamide 10% + Zinc 1%", type: "Serum", goodFor: ["oily", "acne", "combination"], keyIngredients: "niacinamide, zinc", scoreBoost: { oily: 30, acne: 45, pores: 25 } },
  { name: "Paula's Choice 2% BHA Liquid Exfoliant", type: "Exfoliant", goodFor: ["oily", "acne", "blackheads"], keyIngredients: "salicylic acid", scoreBoost: { acne: 40, blackheads: 50, oily: 20 } },
  { name: "CeraVe PM Facial Moisturizing Lotion", type: "Moisturizer", goodFor: ["dry", "sensitive", "normal"], keyIngredients: "ceramides, niacinamide, hyaluronic acid", scoreBoost: { dry: 40, sensitive: 35, normal: 20 } },
  { name: "Neutrogena Hydro Boost Water Gel", type: "Moisturizer", goodFor: ["oily", "combination", "normal"], keyIngredients: "hyaluronic acid", scoreBoost: { oily: 30, combination: 25 } },
  { name: "La Roche-Posay Cicaplast Baume B5", type: "Treatment", goodFor: ["sensitive", "damaged", "dry"], keyIngredients: "panthenol, madecassoside", scoreBoost: { sensitive: 40, damaged: 45, dry: 25 } },
  { name: "The Ordinary Hyaluronic Acid 2% + B5", type: "Serum", goodFor: ["dry", "dehydrated"], keyIngredients: "multiple HA forms, vitamin B5", scoreBoost: { dry: 45, dehydrated: 50 } },
  { name: "Cosrx Advanced Snail 96 Mucin Power Essence", type: "Essence", goodFor: ["dry", "sensitive", "dehydrated"], keyIngredients: "snail secretion filtrate", scoreBoost: { dry: 35, sensitive: 30, dehydrated: 40 } },
];

// ──────────────────────────────────────────────
//   Questions (LLM-style chain-of-thought simulation)
// ──────────────────────────────────────────────

const questions = [
  {
    id: "skinType",
    text: "What is your main **skin type**?",
    options: [
      { value: "oily", label: "Oily / Shiny / Prone to breakouts" },
      { value: "dry", label: "Dry / Tight / Flaky" },
      { value: "combination", label: "Combination (oily T-zone, dry cheeks)" },
      { value: "normal", label: "Normal / Balanced" },
      { value: "sensitive", label: "Sensitive / Easily irritated" }
    ]
  },
  {
    id: "concern",
    text: "What is your **biggest skin concern** right now? (choose one)",
    options: [
      { value: "acne", label: "Active acne / pimples" },
      { value: "blackheads", label: "Blackheads / clogged pores" },
      { value: "dryness", label: "Dryness / dehydration" },
      { value: "redness", label: "Redness / irritation" },
      { value: "aging", label: "Fine lines / aging signs" },
      { value: "none", label: "No major concern — just maintenance" }
    ]
  },
  {
    id: "sensitivity",
    text: "How sensitive is your skin to new products?",
    options: [
      { value: "high", label: "Very sensitive — I react to almost everything" },
      { value: "medium", label: "Somewhat sensitive" },
      { value: "low", label: "Not sensitive — I can try most things" }
    ]
  }
];

let currentQuestion = 0;
let answers = {};

// ──────────────────────────────────────────────
//   DOM Elements
// ──────────────────────────────────────────────

const qText = document.getElementById("q-text");
const qOptions = document.getElementById("q-options");
const nextBtn = document.getElementById("nextBtn");
const quizDiv = document.getElementById("quiz");
const resultDiv = document.getElementById("result");
const reasoningDiv = document.getElementById("reasoning");
const productsDiv = document.getElementById("products");
const progressBar = document.getElementById("progress-bar");

// ──────────────────────────────────────────────
//   Render current question
// ──────────────────────────────────────────────

function renderQuestion() {
  const q = questions[currentQuestion];
  qText.innerHTML = q.text;
  qOptions.innerHTML = "";

  q.options.forEach(opt => {
    const label = document.createElement("label");
    label.innerHTML = `
      <input type="radio" name="\( {q.id}" value=" \){opt.value}">
      ${opt.label}
    `;
    qOptions.appendChild(label);
  });

  nextBtn.disabled = true;
  progressBar.style.width = ((currentQuestion + 1) / questions.length * 100) + "%";
}

// ──────────────────────────────────────────────
//   Handle selection & next
// ──────────────────────────────────────────────

qOptions.addEventListener("change", (e) => {
  if (e.target.name) {
    nextBtn.disabled = false;
  }
});

nextBtn.addEventListener("click", () => {
  const selected = document.querySelector(`input[name="${questions[currentQuestion].id}"]:checked`);
  if (!selected) return;

  answers[questions[currentQuestion].id] = selected.value;

  currentQuestion++;

  if (currentQuestion < questions.length) {
    renderQuestion();
  } else {
    showResults();
  }
});

// ──────────────────────────────────────────────
//   LLM-style reasoning + recommendation scoring
// ──────────────────────────────────────────────

function showResults() {
  quizDiv.style.display = "none";
  resultDiv.style.display = "block";

  let reasoning = "🧠 AI Agent Thinking Step-by-Step:\n\n";

  reasoning += `1. Skin type detected → ${answers.skinType.toUpperCase()}\n`;
  reasoning += `2. Primary concern → ${answers.concern.toUpperCase()}\n`;
  reasoning += `3. Sensitivity level → ${answers.sensitivity.toUpperCase()}\n\n`;

  reasoning += "Analyzing your profile...\n";
  if (answers.sensitivity === "high") {
    reasoning += "→ High sensitivity detected → prioritizing very gentle / barrier-friendly products\n\n";
  }

  // Simple scoring logic (mimics LLM product ranking)
  const scoredProducts = products.map(p => {
    let score = 0;

    // Skin type match
    if (p.goodFor.includes(answers.skinType)) score += 30;
    if (answers.skinType === "combination" && p.goodFor.includes("oily")) score += 15;

    // Concern match
    if (p.goodFor.includes(answers.concern)) score += 45;

    // Sensitivity penalty / bonus
    if (answers.sensitivity === "high" && !p.goodFor.includes("sensitive")) score -= 25;
    if (p.goodFor.includes("sensitive")) score += 20;

    // Extra boosts from product data
    if (p.scoreBoost[answers.concern]) score += p.scoreBoost[answers.concern];
    if (p.scoreBoost[answers.skinType]) score += p.scoreBoost[answers.skinType];

    return { ...p, score: Math.max(0, score) };
  });

  // Sort & take top 4
  scoredProducts.sort((a,b) => b.score - a.score);
  const topProducts = scoredProducts.slice(0, 4).filter(p => p.score > 20);

  reasoning += "Top matches (scored by relevance):\n";
  topProducts.forEach(p => {
    reasoning += `• \( {p.name}  ( \){p.type})  score: ${p.score}\n`;
  });

  reasoningDiv.textContent = reasoning;

  // Render product cards
  productsDiv.innerHTML = "";
  if (topProducts.length === 0) {
    productsDiv.innerHTML = "<p style='color:#777;'>No strong matches found — try different answers or consult a dermatologist.</p>";
    return;
  }

  topProducts.forEach(p => {
    const card = document.createElement("div");
    card.className = "product-card";
    card.innerHTML = `
      <h3>${p.name}</h3>
      <div class="type">${p.type}</div>
      <div><strong>Great for:</strong> ${p.goodFor.join(", ")}</div>
      <div><strong>Key ingredients:</strong> ${p.keyIngredients}</div>
      <div style="margin-top:0.8rem;color:#27ae60;font-weight:600;">
        Match score: ${p.score}/100
      </div>
    `;
    productsDiv.appendChild(card);
  });
}

function restart() {
  currentQuestion = 0;
  answers = {};
  quizDiv.style.display = "block";
  resultDiv.style.display = "none";
  renderQuestion();
}

// Start
renderQuestion();
</script>
</body>
</html>
