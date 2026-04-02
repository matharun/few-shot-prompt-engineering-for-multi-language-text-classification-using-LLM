<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Few-Shot Multilingual Text Classification</title>
<link href="https://fonts.googleapis.com/css2?family=Space+Mono:ital,wght@0,400;0,700;1,400&family=Syne:wght@400;600;700;800&display=swap" rel="stylesheet"/>
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --bg: #080c10;
    --surface: #0d1117;
    --surface2: #161b22;
    --border: #21282f;
    --accent: #00e5ff;
    --accent2: #7c3aed;
    --accent3: #10b981;
    --text: #e6edf3;
    --muted: #8b949e;
    --font-display: 'Syne', sans-serif;
    --font-mono: 'Space Mono', monospace;
  }

  html { scroll-behavior: smooth; }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: var(--font-display);
    overflow-x: hidden;
    line-height: 1.6;
  }

  /* Animated background grid */
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background-image:
      linear-gradient(rgba(0,229,255,0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0,229,255,0.03) 1px, transparent 1px);
    background-size: 40px 40px;
    z-index: 0;
    pointer-events: none;
  }

  /* Glowing orbs */
  .orb {
    position: fixed;
    border-radius: 50%;
    filter: blur(80px);
    opacity: 0.12;
    pointer-events: none;
    z-index: 0;
    animation: drift 20s ease-in-out infinite alternate;
  }
  .orb1 { width: 500px; height: 500px; background: var(--accent); top: -100px; left: -100px; animation-delay: 0s; }
  .orb2 { width: 400px; height: 400px; background: var(--accent2); bottom: -100px; right: -100px; animation-delay: -10s; }
  .orb3 { width: 300px; height: 300px; background: var(--accent3); top: 50%; left: 50%; animation-delay: -5s; }

  @keyframes drift {
    from { transform: translate(0, 0) scale(1); }
    to   { transform: translate(40px, 30px) scale(1.1); }
  }

  /* HERO */
  .hero {
    position: relative;
    z-index: 1;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    text-align: center;
    padding: 4rem 2rem;
    overflow: hidden;
  }

  .hero-badge {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    background: rgba(0,229,255,0.08);
    border: 1px solid rgba(0,229,255,0.2);
    border-radius: 100px;
    padding: 6px 16px;
    font-family: var(--font-mono);
    font-size: 0.7rem;
    color: var(--accent);
    letter-spacing: 0.1em;
    text-transform: uppercase;
    margin-bottom: 2rem;
    animation: fadeUp 0.8s ease both;
  }

  .hero-badge .dot {
    width: 6px; height: 6px;
    background: var(--accent);
    border-radius: 50%;
    animation: pulse 2s ease infinite;
  }

  @keyframes pulse {
    0%, 100% { opacity: 1; transform: scale(1); }
    50% { opacity: 0.4; transform: scale(0.8); }
  }

  .hero h1 {
    font-size: clamp(2.4rem, 7vw, 5.5rem);
    font-weight: 800;
    line-height: 1.05;
    letter-spacing: -0.02em;
    animation: fadeUp 0.9s 0.1s ease both;
    max-width: 800px;
  }

  .hero h1 .line1 { display: block; color: var(--text); }
  .hero h1 .line2 {
    display: block;
    background: linear-gradient(135deg, var(--accent), var(--accent2));
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
  }

  .hero-sub {
    margin-top: 1.5rem;
    font-family: var(--font-mono);
    font-size: 0.85rem;
    color: var(--muted);
    max-width: 560px;
    line-height: 1.8;
    animation: fadeUp 1s 0.2s ease both;
  }

  .hero-pills {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    justify-content: center;
    margin-top: 2rem;
    animation: fadeUp 1s 0.3s ease both;
  }

  .pill {
    background: var(--surface2);
    border: 1px solid var(--border);
    border-radius: 8px;
    padding: 6px 14px;
    font-family: var(--font-mono);
    font-size: 0.72rem;
    color: var(--muted);
    transition: all 0.3s;
  }
  .pill:hover { border-color: var(--accent); color: var(--accent); transform: translateY(-2px); }

  .scroll-hint {
    margin-top: 4rem;
    animation: fadeUp 1s 0.5s ease both;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 8px;
    color: var(--muted);
    font-family: var(--font-mono);
    font-size: 0.65rem;
    letter-spacing: 0.15em;
    text-transform: uppercase;
  }
  .scroll-hint .arrow {
    width: 1px;
    height: 40px;
    background: linear-gradient(to bottom, var(--accent), transparent);
    animation: scrollArrow 2s ease infinite;
  }
  @keyframes scrollArrow {
    0% { transform: scaleY(0); transform-origin: top; }
    50% { transform: scaleY(1); transform-origin: top; }
    51% { transform-origin: bottom; }
    100% { transform: scaleY(0); transform-origin: bottom; }
  }

  @keyframes fadeUp {
    from { opacity: 0; transform: translateY(30px); }
    to   { opacity: 1; transform: translateY(0); }
  }

  /* SECTIONS */
  .container {
    position: relative;
    z-index: 1;
    max-width: 960px;
    margin: 0 auto;
    padding: 0 2rem 6rem;
  }

  .section { margin-bottom: 5rem; opacity: 0; transform: translateY(40px); transition: opacity 0.7s ease, transform 0.7s ease; }
  .section.visible { opacity: 1; transform: translateY(0); }

  .section-label {
    font-family: var(--font-mono);
    font-size: 0.65rem;
    letter-spacing: 0.2em;
    text-transform: uppercase;
    color: var(--accent);
    margin-bottom: 1rem;
    display: flex;
    align-items: center;
    gap: 10px;
  }
  .section-label::after {
    content: '';
    flex: 1;
    height: 1px;
    background: linear-gradient(to right, rgba(0,229,255,0.3), transparent);
  }

  .section-title {
    font-size: 2rem;
    font-weight: 800;
    letter-spacing: -0.02em;
    margin-bottom: 1.5rem;
  }

  /* HOW IT WORKS STEPS */
  .steps {
    display: grid;
    gap: 1px;
    background: var(--border);
    border: 1px solid var(--border);
    border-radius: 16px;
    overflow: hidden;
  }

  .step {
    background: var(--surface);
    padding: 1.5rem 2rem;
    display: flex;
    align-items: flex-start;
    gap: 1.5rem;
    transition: background 0.3s;
  }
  .step:hover { background: var(--surface2); }

  .step-num {
    font-family: var(--font-mono);
    font-size: 0.7rem;
    color: var(--accent);
    background: rgba(0,229,255,0.08);
    border: 1px solid rgba(0,229,255,0.2);
    border-radius: 8px;
    width: 36px;
    height: 36px;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
    font-weight: 700;
  }

  .step-body h3 { font-size: 1rem; font-weight: 700; margin-bottom: 0.3rem; }
  .step-body p  { font-size: 0.85rem; color: var(--muted); font-family: var(--font-mono); }

  /* FEATURES GRID */
  .features-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 1rem;
  }

  .feat-card {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 14px;
    padding: 1.5rem;
    transition: all 0.35s;
    position: relative;
    overflow: hidden;
  }
  .feat-card::before {
    content: '';
    position: absolute;
    inset: 0;
    background: linear-gradient(135deg, rgba(0,229,255,0.04), transparent);
    opacity: 0;
    transition: opacity 0.35s;
  }
  .feat-card:hover { border-color: rgba(0,229,255,0.3); transform: translateY(-4px); box-shadow: 0 20px 40px rgba(0,0,0,0.3); }
  .feat-card:hover::before { opacity: 1; }

  .feat-icon { font-size: 1.8rem; margin-bottom: 0.8rem; }
  .feat-card h3 { font-size: 0.95rem; font-weight: 700; margin-bottom: 0.4rem; }
  .feat-card p  { font-size: 0.78rem; color: var(--muted); font-family: var(--font-mono); line-height: 1.7; }

  /* TECH STACK */
  .tech-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
    gap: 1rem;
  }

  .tech-card {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 1.2rem 1.5rem;
    display: flex;
    align-items: center;
    gap: 12px;
    transition: all 0.3s;
    cursor: default;
  }
  .tech-card:hover { border-color: var(--accent2); transform: translateX(4px); }

  .tech-icon {
    width: 36px; height: 36px;
    border-radius: 8px;
    display: flex; align-items: center; justify-content: center;
    font-size: 1.1rem;
    flex-shrink: 0;
  }
  .tc-react .tech-icon  { background: rgba(0,212,255,0.1); }
  .tc-vite .tech-icon   { background: rgba(126,87,194,0.1); }
  .tc-node .tech-icon   { background: rgba(16,185,129,0.1); }
  .tc-openai .tech-icon { background: rgba(255,255,255,0.05); }

  .tech-info { font-size: 0.82rem; }
  .tech-info strong { display: block; font-weight: 700; }
  .tech-info span   { color: var(--muted); font-family: var(--font-mono); font-size: 0.72rem; }

  /* PROMPT BLOCK */
  .prompt-block {
    background: #0d1117;
    border: 1px solid var(--border);
    border-radius: 14px;
    overflow: hidden;
  }

  .prompt-header {
    background: var(--surface2);
    border-bottom: 1px solid var(--border);
    padding: 0.75rem 1.5rem;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }
  .prompt-dots { display: flex; gap: 6px; }
  .prompt-dots span { width: 10px; height: 10px; border-radius: 50%; }
  .d1 { background: #ff5f57; }
  .d2 { background: #febc2e; }
  .d3 { background: #28c840; }

  .prompt-label { font-family: var(--font-mono); font-size: 0.65rem; color: var(--muted); letter-spacing: 0.1em; text-transform: uppercase; }

  .prompt-body {
    padding: 1.5rem;
    font-family: var(--font-mono);
    font-size: 0.8rem;
    line-height: 1.9;
    color: #cdd9e5;
  }

  .p-comment { color: #484f58; }
  .p-key     { color: var(--accent); }
  .p-val     { color: #a5d6ff; }
  .p-cat     { color: var(--accent3); }
  .p-var     { color: #ffa657; }

  /* LANGUAGES */
  .lang-grid {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
  }

  .lang-chip {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 100px;
    padding: 8px 18px;
    font-family: var(--font-mono);
    font-size: 0.78rem;
    color: var(--muted);
    transition: all 0.3s;
    cursor: default;
  }
  .lang-chip:hover { border-color: var(--accent); color: var(--accent); background: rgba(0,229,255,0.05); }

  /* USE CASES */
  .use-cases { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 1rem; }
  .use-card {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 1.2rem 1.5rem;
    transition: all 0.3s;
  }
  .use-card:hover { border-color: var(--accent2); transform: translateY(-3px); }
  .use-card .uc-icon { font-size: 1.4rem; margin-bottom: 0.6rem; }
  .use-card h4 { font-size: 0.9rem; font-weight: 700; margin-bottom: 0.3rem; }
  .use-card p  { font-size: 0.75rem; color: var(--muted); font-family: var(--font-mono); }

  /* SETUP */
  .setup-steps { display: flex; flex-direction: column; gap: 1rem; }
  .setup-item {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 12px;
    overflow: hidden;
    transition: border-color 0.3s;
  }
  .setup-item:hover { border-color: rgba(0,229,255,0.25); }
  .setup-item-head {
    display: flex;
    align-items: center;
    gap: 14px;
    padding: 1rem 1.5rem;
  }
  .setup-num {
    width: 28px; height: 28px;
    border-radius: 50%;
    background: linear-gradient(135deg, var(--accent), var(--accent2));
    color: #000;
    font-family: var(--font-mono);
    font-size: 0.72rem;
    font-weight: 700;
    display: flex; align-items: center; justify-content: center;
    flex-shrink: 0;
  }
  .setup-item-head h3 { font-size: 0.95rem; font-weight: 700; }
  .code-block {
    background: #060a0e;
    border-top: 1px solid var(--border);
    padding: 1rem 1.5rem;
    font-family: var(--font-mono);
    font-size: 0.78rem;
    color: var(--accent3);
    line-height: 1.8;
  }
  .code-block .cmd::before { content: '$ '; color: var(--muted); }
  .code-block .env-key { color: var(--accent); }
  .code-block .env-val { color: var(--muted); }

  /* FUTURE */
  .future-list { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 1rem; }
  .future-item {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 1.2rem 1.5rem;
    display: flex;
    gap: 12px;
    align-items: flex-start;
    transition: all 0.3s;
  }
  .future-item:hover { border-color: rgba(124,58,237,0.4); transform: translateY(-2px); }
  .future-dot {
    width: 8px; height: 8px;
    border-radius: 50%;
    background: var(--accent2);
    margin-top: 6px;
    flex-shrink: 0;
  }
  .future-item p { font-size: 0.82rem; color: var(--muted); font-family: var(--font-mono); line-height: 1.6; }

  /* FOOTER */
  .footer {
    position: relative;
    z-index: 1;
    border-top: 1px solid var(--border);
    padding: 3rem 2rem;
    text-align: center;
  }
  .footer-inner { max-width: 960px; margin: 0 auto; }
  .footer-badge {
    display: inline-flex;
    align-items: center;
    gap: 10px;
    background: var(--surface2);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 1rem 2rem;
    margin-bottom: 1.5rem;
  }
  .footer-badge .av {
    width: 40px; height: 40px;
    border-radius: 50%;
    background: linear-gradient(135deg, var(--accent), var(--accent2));
    display: flex; align-items: center; justify-content: center;
    font-weight: 800; font-size: 1rem; color: #000;
  }
  .footer-badge .info strong { display: block; font-size: 0.95rem; }
  .footer-badge .info span  { font-size: 0.75rem; color: var(--muted); font-family: var(--font-mono); }
  .footer-note { font-family: var(--font-mono); font-size: 0.7rem; color: var(--muted); }
</style>
</head>
<body>

<div class="orb orb1"></div>
<div class="orb orb2"></div>
<div class="orb orb3"></div>

<!-- HERO -->
<section class="hero">
  <div class="hero-badge">
    <span class="dot"></span>
    Final Year Academic Project · 2025
  </div>

  <h1>
    <span class="line1">Few-Shot Prompt Engineering</span>
    <span class="line2">Multilingual Text Classification</span>
  </h1>

  <p class="hero-sub">
    Classify text across any language — zero training, zero datasets.<br/>
    Pure in-context learning powered by Large Language Models.
  </p>

  <div class="hero-pills">
    <span class="pill">🧠 In-Context Learning</span>
    <span class="pill">🌍 Multilingual NLP</span>
    <span class="pill">⚡ No Fine-Tuning</span>
    <span class="pill">React + Node.js + OpenAI</span>
    <span class="pill">📜 Educational Research</span>
  </div>

  <div class="scroll-hint">
    <span>explore</span>
    <div class="arrow"></div>
  </div>
</section>

<!-- MAIN CONTENT -->
<div class="container">

  <!-- OBJECTIVE -->
  <div class="section">
    <div class="section-label">01 &mdash; Objective</div>
    <div class="section-title">Why this approach?</div>
    <div class="steps">
      <div class="step">
        <div class="step-num">❌</div>
        <div class="step-body">
          <h3>Traditional NLP requires...</h3>
          <p>Large labeled datasets · Model training & fine-tuning · Language-specific pipelines</p>
        </div>
      </div>
      <div class="step">
        <div class="step-num">✅</div>
        <div class="step-body">
          <h3>This project uses in-context learning</h3>
          <p>No training data · No fine-tuning · Any language supported out of the box via few-shot prompting</p>
        </div>
      </div>
    </div>
  </div>

  <!-- HOW IT WORKS -->
  <div class="section">
    <div class="section-label">02 &mdash; How It Works</div>
    <div class="section-title">The Pipeline</div>
    <div class="steps">
      <div class="step">
        <div class="step-num">01</div>
        <div class="step-body">
          <h3>Build the Prompt</h3>
          <p>A structured prompt is constructed — task description + few labeled examples</p>
        </div>
      </div>
      <div class="step">
        <div class="step-num">02</div>
        <div class="step-body">
          <h3>Append User Input</h3>
          <p>The user's text is injected at the end of the prompt as the target for classification</p>
        </div>
      </div>
      <div class="step">
        <div class="step-num">03</div>
        <div class="step-body">
          <h3>Send to OpenAI API</h3>
          <p>The full prompt is submitted to the LLM for inference in a single API call</p>
        </div>
      </div>
      <div class="step">
        <div class="step-num">04</div>
        <div class="step-body">
          <h3>Receive Prediction</h3>
          <p>The model returns a category based purely on the in-context examples — no training needed</p>
        </div>
      </div>
    </div>
  </div>

  <!-- FEATURES -->
  <div class="section">
    <div class="section-label">03 &mdash; Features</div>
    <div class="section-title">Key Capabilities</div>
    <div class="features-grid">
      <div class="feat-card">
        <div class="feat-icon">🌍</div>
        <h3>Multilingual Support</h3>
        <p>Classifies text in English, Tamil, Hindi, and any other language supported by OpenAI models.</p>
      </div>
      <div class="feat-card">
        <div class="feat-icon">🧪</div>
        <h3>Few-Shot Learning</h3>
        <p>Learns from just 2–5 examples embedded directly in the prompt — no dataset required.</p>
      </div>
      <div class="feat-card">
        <div class="feat-icon">⚡</div>
        <h3>Zero Training</h3>
        <p>Eliminates model training and fine-tuning entirely. Plug in examples, get predictions.</p>
      </div>
      <div class="feat-card">
        <div class="feat-icon">🤖</div>
        <h3>OpenAI Powered</h3>
        <p>Leverages state-of-the-art LLMs via the OpenAI API for robust, accurate inference.</p>
      </div>
      <div class="feat-card">
        <div class="feat-icon">🖥️</div>
        <h3>Interactive UI</h3>
        <p>Clean React frontend built with Vite for fast, responsive user interaction.</p>
      </div>
      <div class="feat-card">
        <div class="feat-icon">🔁</div>
        <h3>Rapid Prototyping</h3>
        <p>Change examples, change categories, change languages — all without retraining anything.</p>
      </div>
    </div>
  </div>

  <!-- TECH STACK -->
  <div class="section">
    <div class="section-label">04 &mdash; Stack</div>
    <div class="section-title">Technology</div>
    <div class="tech-grid">
      <div class="tech-card tc-react">
        <div class="tech-icon">⚛️</div>
        <div class="tech-info"><strong>React</strong><span>Frontend UI</span></div>
      </div>
      <div class="tech-card tc-vite">
        <div class="tech-icon">⚡</div>
        <div class="tech-info"><strong>Vite</strong><span>Build Tool</span></div>
      </div>
      <div class="tech-card tc-node">
        <div class="tech-icon">🟩</div>
        <div class="tech-info"><strong>Node.js</strong><span>Runtime</span></div>
      </div>
      <div class="tech-card tc-openai">
        <div class="tech-icon">🤖</div>
        <div class="tech-info"><strong>OpenAI API</strong><span>LLM Inference</span></div>
      </div>
    </div>
  </div>

  <!-- PROMPT EXAMPLE -->
  <div class="section">
    <div class="section-label">05 &mdash; Prompt Design</div>
    <div class="section-title">Example Few-Shot Prompt</div>
    <div class="prompt-block">
      <div class="prompt-header">
        <div class="prompt-dots"><span class="d1"></span><span class="d2"></span><span class="d3"></span></div>
        <div class="prompt-label">prompt_template.txt</div>
        <div></div>
      </div>
      <div class="prompt-body">
        <div class="p-comment"># Task instruction</div>
        <div>Classify the category of the following text.</div>
        <br/>
        <div class="p-comment"># Example 1 — English</div>
        <div><span class="p-key">Text:</span> <span class="p-val">"I love this product"</span></div>
        <div><span class="p-key">Category:</span> <span class="p-cat">Positive</span></div>
        <br/>
        <div class="p-comment"># Example 2 — Tamil</div>
        <div><span class="p-key">Text:</span> <span class="p-val">"இந்த சேவை மிகவும் மோசமாக உள்ளது"</span></div>
        <div><span class="p-key">Category:</span> <span class="p-cat">Negative</span></div>
        <br/>
        <div class="p-comment"># Inference target</div>
        <div><span class="p-key">Text:</span> <span class="p-var">"{input_text}"</span></div>
        <div><span class="p-key">Category:</span> <span style="color: var(--muted);">← model predicts here</span></div>
      </div>
    </div>
  </div>

  <!-- LANGUAGES -->
  <div class="section">
    <div class="section-label">06 &mdash; Languages</div>
    <div class="section-title">Supported Input Languages</div>
    <div class="lang-grid">
      <span class="lang-chip">🇬🇧 English</span>
      <span class="lang-chip">🇮🇳 Tamil</span>
      <span class="lang-chip">🇮🇳 Hindi</span>
      <span class="lang-chip">🌐 Any OpenAI-supported language</span>
    </div>
  </div>

  <!-- USE CASES -->
  <div class="section">
    <div class="section-label">07 &mdash; Use Cases</div>
    <div class="section-title">Real-World Applications</div>
    <div class="use-cases">
      <div class="use-card">
        <div class="uc-icon">💬</div>
        <h4>Sentiment Analysis</h4>
        <p>Classify customer reviews across multiple languages in real time.</p>
      </div>
      <div class="use-card">
        <div class="uc-icon">📋</div>
        <h4>Feedback Classification</h4>
        <p>Auto-sort multilingual feedback into actionable categories.</p>
      </div>
      <div class="use-card">
        <div class="uc-icon">🛡️</div>
        <h4>Content Moderation</h4>
        <p>Flag harmful or policy-violating content without language barriers.</p>
      </div>
      <div class="use-card">
        <div class="uc-icon">🚀</div>
        <h4>NLP Prototyping</h4>
        <p>Rapidly prototype text classification systems without labeling datasets.</p>
      </div>
    </div>
  </div>

  <!-- SETUP -->
  <div class="section">
    <div class="section-label">08 &mdash; Setup</div>
    <div class="section-title">Running Locally</div>
    <div class="setup-steps">
      <div class="setup-item">
        <div class="setup-item-head">
          <div class="setup-num">1</div>
          <h3>Install Dependencies</h3>
        </div>
        <div class="code-block"><div class="cmd">npm install</div></div>
      </div>
      <div class="setup-item">
        <div class="setup-item-head">
          <div class="setup-num">2</div>
          <h3>Configure API Key — create <code style="color:var(--accent);font-size:0.8em">.env.local</code></h3>
        </div>
        <div class="code-block">
          <div><span class="env-key">OPENAI_API_KEY</span>=<span class="env-val">your_api_key_here</span></div>
        </div>
      </div>
      <div class="setup-item">
        <div class="setup-item-head">
          <div class="setup-num">3</div>
          <h3>Start Development Server</h3>
        </div>
        <div class="code-block"><div class="cmd">npm run dev</div></div>
      </div>
    </div>
  </div>

  <!-- FUTURE -->
  <div class="section">
    <div class="section-label">09 &mdash; Roadmap</div>
    <div class="section-title">Future Improvements</div>
    <div class="future-list">
      <div class="future-item">
        <div class="future-dot"></div>
        <p>Dynamic prompt tuning based on input feedback</p>
      </div>
      <div class="future-item">
        <div class="future-dot"></div>
        <p>Confidence score extraction per classification</p>
      </div>
      <div class="future-item">
        <div class="future-dot"></div>
        <p>Logging and analytics integration dashboard</p>
      </div>
      <div class="future-item">
        <div class="future-dot"></div>
        <p>Full web deployment as a production app</p>
      </div>
    </div>
  </div>

</div>

<!-- FOOTER -->
<footer class="footer">
  <div class="footer-inner">
    <div class="footer-badge">
      <div class="av">T</div>
      <div class="info">
        <strong>Final Year B.Tech Project</strong>
        <span>Prompt Engineering · Multilingual NLP · LLMs · Educational Research</span>
      </div>
    </div>
    <p class="footer-note">Intended for educational and research purposes only.</p>
  </div>
</footer>

<script>
  // Scroll reveal
  const sections = document.querySelectorAll('.section');
  const io = new IntersectionObserver(entries => {
    entries.forEach(e => { if (e.isIntersecting) { e.target.classList.add('visible'); io.unobserve(e.target); } });
  }, { threshold: 0.1 });
  sections.forEach(s => io.observe(s));
</script>
</body>
</html>
