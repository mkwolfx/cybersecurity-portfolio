<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>How I Found a Reflected XSS by Understanding Context</title>
<link href="https://fonts.googleapis.com/css2?family=Lora:ital,wght@0,400;0,600;1,400&family=IBM+Plex+Mono:wght@400;500&family=DM+Sans:wght@400;500;600&display=swap" rel="stylesheet">
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: #f9f7f4;
    color: #1a1a1a;
    font-family: 'Lora', Georgia, serif;
    line-height: 1.8;
    min-height: 100vh;
  }

  .hero {
    background: #0e1117;
    color: #fff;
    padding: 80px 24px 60px;
    text-align: center;
    position: relative;
    overflow: hidden;
  }

  .hero::before {
    content: '';
    position: absolute;
    inset: 0;
    background: radial-gradient(ellipse at 30% 60%, rgba(255,80,50,0.12) 0%, transparent 60%),
                radial-gradient(ellipse at 70% 30%, rgba(80,120,255,0.1) 0%, transparent 55%);
    pointer-events: none;
  }

  .hero-tag {
    display: inline-block;
    background: rgba(255,80,50,0.15);
    color: #ff6b4a;
    border: 1px solid rgba(255,80,50,0.3);
    border-radius: 4px;
    font-family: 'IBM Plex Mono', monospace;
    font-size: 11px;
    letter-spacing: 0.12em;
    text-transform: uppercase;
    padding: 4px 12px;
    margin-bottom: 28px;
  }

  .hero h1 {
    font-family: 'Lora', serif;
    font-size: clamp(28px, 5vw, 48px);
    font-weight: 600;
    line-height: 1.2;
    max-width: 760px;
    margin: 0 auto 24px;
    color: #fff;
  }

  .hero-sub {
    font-family: 'DM Sans', sans-serif;
    font-size: 16px;
    color: rgba(255,255,255,0.55);
    max-width: 520px;
    margin: 0 auto 36px;
    line-height: 1.6;
  }

  .hero-meta {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 20px;
    flex-wrap: wrap;
  }

  .meta-item {
    font-family: 'DM Sans', sans-serif;
    font-size: 13px;
    color: rgba(255,255,255,0.4);
    display: flex;
    align-items: center;
    gap: 6px;
  }

  .meta-dot {
    width: 4px;
    height: 4px;
    border-radius: 50%;
    background: rgba(255,255,255,0.2);
  }

  .container {
    max-width: 720px;
    margin: 0 auto;
    padding: 64px 24px 96px;
  }

  .intro-box {
    background: #fff;
    border-left: 3px solid #ff6b4a;
    border-radius: 0 8px 8px 0;
    padding: 24px 28px;
    margin-bottom: 48px;
    box-shadow: 0 1px 4px rgba(0,0,0,0.06);
  }

  .intro-box p {
    font-size: 17px;
    color: #333;
    line-height: 1.75;
  }

  h2 {
    font-family: 'DM Sans', sans-serif;
    font-size: 22px;
    font-weight: 600;
    color: #0e1117;
    margin: 56px 0 20px;
    padding-bottom: 10px;
    border-bottom: 1px solid #e8e4df;
  }

  h3 {
    font-family: 'DM Sans', sans-serif;
    font-size: 16px;
    font-weight: 600;
    color: #0e1117;
    margin: 32px 0 12px;
  }

  p {
    font-size: 17px;
    color: #2d2d2d;
    margin-bottom: 20px;
    line-height: 1.8;
  }

  .step-grid {
    display: flex;
    flex-direction: column;
    gap: 0;
    margin: 32px 0 48px;
    border: 1px solid #e8e4df;
    border-radius: 12px;
    overflow: hidden;
    background: #fff;
    box-shadow: 0 1px 4px rgba(0,0,0,0.05);
  }

  .step-item {
    display: flex;
    gap: 0;
    border-bottom: 1px solid #f0ece7;
  }

  .step-item:last-child { border-bottom: none; }

  .step-num {
    background: #0e1117;
    color: #ff6b4a;
    font-family: 'IBM Plex Mono', monospace;
    font-size: 13px;
    font-weight: 500;
    min-width: 56px;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 16px 0;
    letter-spacing: 0.04em;
  }

  .step-content {
    padding: 16px 20px;
    flex: 1;
  }

  .step-title {
    font-family: 'DM Sans', sans-serif;
    font-size: 14px;
    font-weight: 600;
    color: #0e1117;
    margin-bottom: 4px;
  }

  .step-desc {
    font-family: 'DM Sans', sans-serif;
    font-size: 13px;
    color: #666;
    line-height: 1.6;
    margin: 0;
  }

  .code-block {
    background: #0e1117;
    border-radius: 8px;
    overflow: hidden;
    margin: 24px 0;
    box-shadow: 0 2px 8px rgba(0,0,0,0.15);
  }

  .code-header {
    background: #1a1f2e;
    padding: 10px 16px;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  .code-dots {
    display: flex;
    gap: 6px;
  }

  .dot { width: 10px; height: 10px; border-radius: 50%; }
  .dot-red { background: #ff5f57; }
  .dot-yellow { background: #febc2e; }
  .dot-green { background: #28c840; }

  .code-label {
    font-family: 'IBM Plex Mono', monospace;
    font-size: 11px;
    color: rgba(255,255,255,0.35);
    letter-spacing: 0.08em;
  }

  .code-body {
    padding: 20px 20px;
    font-family: 'IBM Plex Mono', monospace;
    font-size: 14px;
    line-height: 1.7;
    overflow-x: auto;
  }

  .code-body .tag { color: #f92672; }
  .code-body .attr { color: #a6e22e; }
  .code-body .val { color: #e6db74; }
  .code-body .comment { color: rgba(255,255,255,0.3); font-style: italic; }
  .code-body .highlight { color: #ff6b4a; font-weight: 500; }
  .code-body .normal { color: #f8f8f2; }

  .payload-anatomy {
    background: #fff;
    border: 1px solid #e8e4df;
    border-radius: 12px;
    padding: 28px;
    margin: 24px 0 40px;
    box-shadow: 0 1px 4px rgba(0,0,0,0.05);
  }

  .payload-display {
    font-family: 'IBM Plex Mono', monospace;
    font-size: 18px;
    font-weight: 500;
    color: #0e1117;
    margin-bottom: 24px;
    padding: 16px;
    background: #f4f1ee;
    border-radius: 6px;
    letter-spacing: 0.02em;
    word-break: break-all;
    line-height: 1.6;
  }

  .payload-display .p-quote { color: #e53e3e; }
  .payload-display .p-close { color: #dd6b20; }
  .payload-display .p-tag { color: #2b6cb0; }
  .payload-display .p-attr { color: #276749; }
  .payload-display .p-val { color: #744210; }
  .payload-display .p-js { color: #6b46c1; }

  .anatomy-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
    gap: 12px;
  }

  .anatomy-item {
    background: #f9f7f4;
    border-radius: 8px;
    padding: 12px 14px;
  }

  .anatomy-code {
    font-family: 'IBM Plex Mono', monospace;
    font-size: 13px;
    font-weight: 500;
    color: #0e1117;
    margin-bottom: 4px;
  }

  .anatomy-desc {
    font-family: 'DM Sans', sans-serif;
    font-size: 12px;
    color: #888;
    line-height: 1.4;
  }

  .insight-box {
    background: #0e1117;
    color: #fff;
    border-radius: 12px;
    padding: 36px 32px;
    margin: 48px 0;
    text-align: center;
  }

  .insight-box .formula {
    font-family: 'IBM Plex Mono', monospace;
    font-size: 20px;
    color: #ff6b4a;
    margin-bottom: 12px;
    letter-spacing: 0.04em;
  }

  .insight-box p {
    color: rgba(255,255,255,0.6);
    font-size: 15px;
    margin: 0;
  }

  .flow-steps {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
    gap: 0;
    margin: 28px 0 40px;
    border: 1px solid #e8e4df;
    border-radius: 10px;
    overflow: hidden;
    background: #fff;
  }

  .flow-step {
    padding: 20px 16px;
    text-align: center;
    border-right: 1px solid #f0ece7;
    position: relative;
  }

  .flow-step:last-child { border-right: none; }

  .flow-icon {
    font-size: 22px;
    margin-bottom: 8px;
  }

  .flow-num {
    font-family: 'IBM Plex Mono', monospace;
    font-size: 10px;
    color: #bbb;
    margin-bottom: 4px;
  }

  .flow-label {
    font-family: 'DM Sans', sans-serif;
    font-size: 13px;
    font-weight: 600;
    color: #0e1117;
  }

  .disclaimer {
    background: #fff8f6;
    border: 1px solid #fdd0c4;
    border-radius: 8px;
    padding: 16px 20px;
    margin: 48px 0 0;
    display: flex;
    gap: 12px;
    align-items: flex-start;
  }

  .disclaimer-icon { font-size: 16px; margin-top: 2px; }

  .disclaimer p {
    font-family: 'DM Sans', sans-serif;
    font-size: 14px;
    color: #c05621;
    margin: 0;
    line-height: 1.6;
  }

  .author-card {
    background: #fff;
    border: 1px solid #e8e4df;
    border-radius: 12px;
    padding: 24px 28px;
    margin-top: 40px;
    display: flex;
    align-items: center;
    gap: 20px;
    box-shadow: 0 1px 4px rgba(0,0,0,0.05);
  }

  .author-avatar {
    width: 52px;
    height: 52px;
    border-radius: 50%;
    background: #0e1117;
    display: flex;
    align-items: center;
    justify-content: center;
    font-family: 'IBM Plex Mono', monospace;
    font-size: 14px;
    color: #ff6b4a;
    font-weight: 500;
    flex-shrink: 0;
  }

  .author-info .name {
    font-family: 'DM Sans', sans-serif;
    font-size: 15px;
    font-weight: 600;
    color: #0e1117;
    margin-bottom: 3px;
  }

  .author-info .bio {
    font-family: 'DM Sans', sans-serif;
    font-size: 13px;
    color: #888;
    margin: 0;
  }

  .tags {
    display: flex;
    gap: 8px;
    flex-wrap: wrap;
    margin: 40px 0 0;
  }

  .tag-pill {
    font-family: 'DM Sans', sans-serif;
    font-size: 12px;
    background: #f0ece7;
    color: #555;
    border-radius: 100px;
    padding: 5px 14px;
    text-decoration: none;
  }

  .owasp-link {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    font-family: 'DM Sans', sans-serif;
    font-size: 14px;
    color: #2563eb;
    text-decoration: none;
    border-bottom: 1px solid rgba(37,99,235,0.3);
    transition: border-color 0.2s;
  }

  .owasp-link:hover { border-color: #2563eb; }

  @media (max-width: 600px) {
    .flow-steps { grid-template-columns: repeat(2, 1fr); }
    .anatomy-grid { grid-template-columns: repeat(2, 1fr); }
  }
</style>
</head>
<body>

<div class="hero">
  <div class="hero-tag">Bug Bounty · Web Security</div>
  <h1>How I Found a Reflected XSS by Understanding Context</h1>
  <p class="hero-sub">A real-world walkthrough of methodical thinking over random payload spraying — from reflection discovery to confirmed execution.</p>
  <div class="hero-meta">
    <span class="meta-item">First write-up</span>
    <span class="meta-dot"></span>
    <span class="meta-item">5 min read</span>
    <span class="meta-dot"></span>
    <span class="meta-item">Reflected XSS</span>
  </div>
</div>

<div class="container">

  <div class="intro-box">
    <p>When I started testing the <code style="font-family:'IBM Plex Mono',monospace;font-size:14px;background:#f4f1ee;padding:2px 6px;border-radius:4px;">/search?q=</code> parameter, I didn't immediately start firing complex payloads. My first step was to understand <em>how the application handled user input</em> — and that patience made all the difference.</p>
  </div>

  <h2>Step 1 — Finding the Reflection</h2>

  <p>I entered a simple test string like <code style="font-family:'IBM Plex Mono',monospace;font-size:14px;background:#f4f1ee;padding:2px 6px;border-radius:4px;">test123</code> and opened DevTools. The goal at this stage isn't exploitation — it's reconnaissance. I searched for my string in the page source to trace exactly where and how it was reflected.</p>

  <p>The input appeared in multiple places, but one stood out: my value was being placed <strong>directly inside an HTML attribute</strong>.</p>

  <div class="code-block">
    <div class="code-header">
      <div class="code-dots">
        <span class="dot dot-red"></span>
        <span class="dot dot-yellow"></span>
        <span class="dot dot-green"></span>
      </div>
      <span class="code-label">HTML — page source</span>
    </div>
    <div class="code-body">
      <span class="tag">&lt;input</span> <span class="attr">value</span>=<span class="val">"test123"</span><span class="tag">&gt;</span><br>
      <span class="comment">         ↑ my input is reflected here, inside an attribute value</span>
    </div>
  </div>

  <h2>Step 2 — Understanding the Context</h2>

  <p>This is the most important step. <strong>Context determines everything</strong> in XSS — the same string that works in one context does nothing in another. In an HTML attribute context, a few things are true:</p>

  <div class="step-grid">
    <div class="step-item">
      <div class="step-num">01</div>
      <div class="step-content">
        <div class="step-title">Script tags won't work directly</div>
        <p class="step-desc">Injecting <code style="font-family:monospace;font-size:12px;">&lt;script&gt;</code> tags inside an attribute value will be rendered as plain text, not executed.</p>
      </div>
    </div>
    <div class="step-item">
      <div class="step-num">02</div>
      <div class="step-content">
        <div class="step-title">You must escape the attribute first</div>
        <p class="step-desc">To get control of the HTML structure, the attribute's closing quote must be terminated before anything else.</p>
      </div>
    </div>
    <div class="step-item">
      <div class="step-num">03</div>
      <div class="step-content">
        <div class="step-title">Then exit the tag entirely</div>
        <p class="step-desc">After breaking out of the attribute, the containing tag must also be closed before new HTML can be injected.</p>
      </div>
    </div>
  </div>

  <p>The plan crystallised: <strong>Break the attribute → Close the tag → Inject a new element with an event handler.</strong></p>

  <h2>Step 3 — The Payload</h2>

  <p>Here is the payload I used, and why each character exists:</p>

  <div class="payload-anatomy">
    <div class="payload-display">
      <span class="p-quote">"</span><span class="p-close">&gt;</span><span class="p-tag">&lt;img </span><span class="p-attr">src</span>=<span class="p-val">x </span><span class="p-attr">onerror</span>=<span class="p-js">"alert(1)"</span><span class="p-tag">&gt;</span>
    </div>
    <div class="anatomy-grid">
      <div class="anatomy-item">
        <div class="anatomy-code">"</div>
        <div class="anatomy-desc">Closes the open attribute value</div>
      </div>
      <div class="anatomy-item">
        <div class="anatomy-code">&gt;</div>
        <div class="anatomy-desc">Closes the surrounding HTML tag</div>
      </div>
      <div class="anatomy-item">
        <div class="anatomy-code">&lt;img</div>
        <div class="anatomy-desc">Injects a new HTML element</div>
      </div>
      <div class="anatomy-item">
        <div class="anatomy-code">src=x</div>
        <div class="anatomy-desc">Intentionally invalid source to force a load error</div>
      </div>
      <div class="anatomy-item">
        <div class="anatomy-code">onerror=</div>
        <div class="anatomy-desc">Event handler that fires when the image fails</div>
      </div>
      <div class="anatomy-item">
        <div class="anatomy-code">alert(1)</div>
        <div class="anatomy-desc">Proof-of-concept JavaScript execution</div>
      </div>
    </div>
  </div>

  <h2>Step 4 — Confirming Execution</h2>

  <p>After injecting the payload into the <code style="font-family:'IBM Plex Mono',monospace;font-size:14px;background:#f4f1ee;padding:2px 6px;border-radius:4px;">q</code> parameter, the browser loaded the page, the image failed to load (as expected), the <code style="font-family:'IBM Plex Mono',monospace;font-size:14px;background:#f4f1ee;padding:2px 6px;border-radius:4px;">onerror</code> handler fired, and the alert box appeared. XSS confirmed.</p>

  <p>To demonstrate impact beyond a simple alert, I tested additional payloads:</p>

  <div class="code-block">
    <div class="code-header">
      <div class="code-dots">
        <span class="dot dot-red"></span>
        <span class="dot dot-yellow"></span>
        <span class="dot dot-green"></span>
      </div>
      <span class="code-label">Additional PoC payloads</span>
    </div>
    <div class="code-body">
      <span class="comment">// Access cookies</span><br>
      <span class="highlight">"&gt;&lt;img src=x onerror="alert(document.cookie)"&gt;</span><br><br>
      <span class="comment">// Alternative vector via iframe</span><br>
      <span class="highlight">"&gt;&lt;iframe src="javascript:alert(1)"&gt;&lt;/iframe&gt;</span>
    </div>
  </div>

  <p>Both confirmed: arbitrary JavaScript execution was achievable, including access to session cookies — which demonstrates real-world severity.</p>

  <h2>The Mental Model</h2>

  <p>This is the repeatable thought process that led to the find. It works for any injection context, not just HTML attributes:</p>

  <div class="flow-steps">
    <div class="flow-step">
      <div class="flow-num">01</div>
      <div class="flow-label">Find reflection</div>
    </div>
    <div class="flow-step">
      <div class="flow-num">02</div>
      <div class="flow-label">Map the context</div>
    </div>
    <div class="flow-step">
      <div class="flow-num">03</div>
      <div class="flow-label">Break structure</div>
    </div>
    <div class="flow-step">
      <div class="flow-num">04</div>
      <div class="flow-label">Craft payload</div>
    </div>
    <div class="flow-step">
      <div class="flow-num">05</div>
      <div class="flow-label">Confirm & report</div>
    </div>
  </div>

  <div class="insight-box">
    <div class="formula">XSS = Context + Grammar</div>
    <p>Random payloads produce random results. Understanding where your input lands — and what characters have structural meaning there — is the real skill.</p>
  </div>

  <p>For a deeper dive into how injection works across different contexts (HTML, SQL, LDAP, and more), the OWASP Injection Theory page is an excellent reference:</p>

  <p><a href="https://owasp.org/www-community/Injection_Theory" class="owasp-link" target="_blank">owasp.org — Injection Theory ↗</a></p>

  <div class="tags">
    <span class="tag-pill">XSS</span>
    <span class="tag-pill">Bug Bounty</span>
    <span class="tag-pill">Web Security</span>
    <span class="tag-pill">Pentesting</span>
    <span class="tag-pill">OWASP</span>
    <span class="tag-pill">Beginner Writeup</span>
  </div>

  <div class="disclaimer">
    <span class="disclaimer-icon">⚠️</span>
    <p><strong>Disclaimer:</strong> All testing was performed in an authorized environment only. Never test for vulnerabilities on systems you do not have explicit permission to test.</p>
  </div>

  <div class="author-card">
    <div class="author-avatar">AP</div>
    <div class="author-info">
      <div class="name">Aspiring Pentester</div>
      <p class="bio">Bug bounty learner · sharing findings, methods, and lessons along the way. First writeup ✓</p>
    </div>
  </div>

</div>
</body>
</html>
