<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Feliz Aniversário, Lorena! 🌌</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }

    body {
      background: #000;
      overflow: hidden;
      font-family: 'Georgia', serif;
    }

    canvas {
      position: fixed;
      top: 0; left: 0;
      width: 100%; height: 100%;
    }

    #intro {
      position: fixed;
      inset: 0;
      background: radial-gradient(ellipse at center, #0d0221 0%, #000000 100%);
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      z-index: 100;
    }

    #intro h1 {
      color: #fff;
      font-size: clamp(1.5rem, 4vw, 2.5rem);
      text-align: center;
      padding: 0 1.5rem;
      line-height: 1.6;
      text-shadow: 0 0 20px #a78bfa, 0 0 40px #7c3aed;
    }

    #intro p {
      margin-top: 1.2rem;
      color: #c4b5fd;
      font-size: clamp(0.9rem, 2vw, 1.1rem);
      text-align: center;
      padding: 0 2rem;
      line-height: 1.6;
    }

    #btn-entrar {
      margin-top: 2.5rem;
      padding: 0.9rem 2.5rem;
      background: transparent;
      border: 2px solid #a78bfa;
      color: #a78bfa;
      font-size: 1.1rem;
      border-radius: 50px;
      cursor: pointer;
      letter-spacing: 1px;
      transition: all 0.3s ease;
    }

    #btn-entrar:hover {
      background: #a78bfa;
      color: #000;
      box-shadow: 0 0 25px #a78bfa;
    }

    #tooltip {
      position: fixed;
      background: rgba(15, 5, 40, 0.95);
      border: 1px solid #7c3aed;
      color: #e9d5ff;
      padding: 0.8rem 1.3rem;
      border-radius: 12px;
      font-size: 1rem;
      pointer-events: none;
      opacity: 0;
      transition: opacity 0.3s ease;
      max-width: 240px;
      text-align: center;
      box-shadow: 0 0 20px #7c3aed88;
      z-index: 50;
      display: none;
    }

    #tooltip.visible {
      opacity: 1;
      display: block;
    }

    #card-overlay {
      position: fixed;
      inset: 0;
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 200;
      background: rgba(0,0,0,0.4);
      opacity: 0;
      pointer-events: none;
      transition: opacity 0.4s ease;
    }

    #card-overlay.visible {
      opacity: 1;
      pointer-events: all;
    }

    #card {
      background: rgba(15, 5, 40, 0.97);
      border: 1px solid #7c3aed;
      border-radius: 20px;
      padding: 2.5rem 2rem;
      max-width: 340px;
      width: 90%;
      text-align: center;
      color: #e9d5ff;
      box-shadow: 0 0 50px #7c3aed99;
      transform: scale(0.8);
      transition: transform 0.4s ease;
    }

    #card-overlay.visible #card {
      transform: scale(1);
    }

    #card-emoji {
      font-size: 3rem;
      display: block;
      margin-bottom: 1rem;
    }

    #card-texto {
      font-size: 1.2rem;
      line-height: 1.6;
    }

    #card button {
      margin-top: 1.5rem;
      background: transparent;
      border: 1px solid #a78bfa;
      color: #a78bfa;
      padding: 0.5rem 1.5rem;
      border-radius: 50px;
      cursor: pointer;
      font-size: 0.95rem;
      transition: all 0.3s;
    }

    #card button:hover {
      background: #a78bfa;
      color: #000;
    }

    #instrucao {
      position: fixed;
      bottom: 1.5rem;
      left: 50%;
      transform: translateX(-50%);
      color: #a78bfaaa;
      font-size: 0.9rem;
      text-align: center;
      pointer-events: none;
      z-index: 10;
      white-space: nowrap;
      animation: pulsar 2.5s ease-in-out infinite;
    }

    @keyframes pulsar {
      0%, 100% { opacity: 0.3; }
      50% { opacity: 1; }
    }
  </style>
</head>
<body>

  <canvas id="galaxy"></canvas>

  <!-- Tela de abertura -->
  <div id="intro">
    <h1>✨ Feliz Aniversário, Lorena! ✨</h1>
    <p>Para Lorena, que ilumina a vida das pessoas<br>com suas aventuras e loucuras!</p>
    <button id="btn-entrar">Explorar o céu 🌌</button>
  </div>

  <!-- Tooltip -->
  <div id="tooltip"></div>

  <!-- Card -->
  <div id="card-overlay">
    <div id="card">
      <span id="card-emoji">⭐</span>
      <p id="card-texto"></p>
      <button onclick="fecharCard()">Fechar ✕</button>
    </div>
  </div>

  <!-- Instrução -->
  <div id="instrucao">✨ Clique nas estrelas roxas para descobrir segredos ✨</div>

<script>
  const motivos = [
    { texto: "Você é uma garota esforçada",         emoji: "💪" },
    { texto: "Amo sua energia",                      emoji: "⚡" },
    { texto: "Floripa com vc foi muito mais legal",  emoji: "🏖️" },
    { texto: "Aventureira",                          emoji: "🧭" },
    { texto: "Parceira e amiga para a vida inteira", emoji: "👯‍♀️" },
    { texto: "Amorosa",                              emoji: "💜" },
  ];

  const canvas  = document.getElementById('galaxy');
  const ctx     = canvas.getContext('2d');
  const tooltip = document.getElementById('tooltip');
  const overlay = document.getElementById('card-overlay');
  const cardTexto = document.getElementById('card-texto');
  const cardEmoji = document.getElementById('card-emoji');

  let W, H;
  let stars = [];
  let specialStars = [];
  let mouseX = -999, mouseY = -999;
  let cardAberto = false;
  let animFrame;

  function resize() {
    W = canvas.width  = window.innerWidth;
    H = canvas.height = window.innerHeight;
    criarEstrelas();
  }

  function criarEstrelas() {
    stars = [];
    specialStars = [];

    // Estrelas comuns
    for (let i = 0; i < 300; i++) {
      stars.push({
        x:     Math.random() * W,
        y:     Math.random() * H,
        r:     0.5 + Math.random() * 1.8,
        alpha: 0.3 + Math.random() * 0.7,
        phase: Math.random() * Math.PI * 2,
        speed: 0.005 + Math.random() * 0.01,
      });
    }

    // Estrelas especiais — distribuídas em grade para não se sobrepor
    const cols = 3;
    const rows = 2;
    const cellW = W / cols;
    const cellH = H / rows;
    const margem = 60;

    for (let i = 0; i < motivos.length; i++) {
      const col = i % cols;
      const row = Math.floor(i / cols);
      specialStars.push({
        x:      margem + col * cellW + Math.random() * (cellW - margem * 2),
        y:      margem + row * cellH + Math.random() * (cellH - margem * 2),
        r:      8,
        phase:  Math.random() * Math.PI * 2,
        speed:  0.015 + Math.random() * 0.01,
        motivo: motivos[i],
      });
    }
  }

  function draw(t) {
    ctx.clearRect(0, 0, W, H);

    // Fundo
    const bg = ctx.createRadialGradient(W/2, H/2, 0, W/2, H/2, Math.max(W, H));
    bg.addColorStop(0,   '#0d0221');
    bg.addColorStop(0.6, '#080112');
    bg.addColorStop(1,   '#000000');
    ctx.fillStyle = bg;
    ctx.fillRect(0, 0, W, H);

    // Estrelas comuns
    stars.forEach(s => {
      const a = s.alpha * (0.5 + 0.5 * Math.sin(t * 0.001 * s.speed * 100 + s.phase));
      ctx.beginPath();
      ctx.arc(s.x, s.y, s.r, 0, Math.PI * 2);
      ctx.fillStyle = `rgba(200, 180, 255, \${a})`;
      ctx.fill();
    });

    // Estrelas especiais
    let hoveredStar = null;

    specialStars.forEach(s => {
      const dist = Math.hypot(mouseX - s.x, mouseY - s.y);
      const hovered = dist < 35;
      if (hovered) hoveredStar = s;

      const pulso = 0.75 + 0.25 * Math.sin(t * 0.001 * s.speed * 100 + s.phase);
      const raio  = hovered ? s.r * 1.8 : s.r * pulso;

      // Brilho externo
      const glow = ctx.createRadialGradient(s.x, s.y, 0, s.x, s.y, raio * 4);
      glow.addColorStop(0,   hovered ? 'rgba(220,200,255,0.6)' : 'rgba(167,139,250,0.35)');
      glow.addColorStop(0.5, hovered ? 'rgba(167,139,250,0.2)' : 'rgba(124,58,237,0.1)');
      glow.addColorStop(1,   'transparent');
      ctx.fillStyle = glow;
      ctx.beginPath();
      ctx.arc(s.x, s.y, raio * 4, 0, Math.PI * 2);
      ctx.fill();

      // Núcleo branco
      const core = ctx.createRadialGradient(s.x, s.y, 0, s.x, s.y, raio);
      core.addColorStop(0,   '#ffffff');
      core.addColorStop(0.3, hovered ? '#ddd6fe' : '#c4b5fd');
      core.addColorStop(1,   'transparent');
      ctx.fillStyle = core;
      ctx.beginPath();
      ctx.arc(s.x, s.y, raio, 0, Math.PI * 2);
      ctx.fill();

      // Raios (4 pontas)
      ctx.save();
      ctx.translate(s.x, s.y);
      ctx.strokeStyle = hovered ? 'rgba(220,200,255,0.8)' : 'rgba(167,139,250,0.5)';
      ctx.lineWidth   = hovered ? 2 : 1;
      const len = raio * (hovered ? 4 : 3);
      for (let a = 0; a < 4; a++) {
        ctx.rotate(Math.PI / 4);
        ctx.beginPath();
        ctx.moveTo(0, -len);
        ctx.lineTo(0,  len);
        ctx.stroke();
      }
      ctx.restore();
    });

    // Tooltip
    if (hoveredStar && !cardAberto) {
      tooltip.textContent = `\${hoveredStar.motivo.emoji}  \${hoveredStar.motivo.texto}`;
      tooltip.classList.add('visible');
      let tx = mouseX + 20, ty = mouseY - 20;
      if (tx + 250 > W) tx = mouseX - 260;
      if (ty < 10)       ty = mouseY + 25;
      tooltip.style.left = tx + 'px';
      tooltip.style.top  = ty + 'px';
    } else {
      tooltip.classList.remove('visible');
    }

    animFrame = requestAnimationFrame(draw);
  }

  function abrirCard(s) {
    cardTexto.textContent = s.motivo.texto;
    cardEmoji.textContent = s.motivo.emoji;
    overlay.classList.add('visible');
    cardAberto = true;
  }

  function fecharCard() {
    overlay.classList.remove('visible');
    cardAberto = false;
  }

  // Eventos mouse
  canvas.addEventListener('mousemove', e => {
    mouseX = e.clientX;
    mouseY = e.clientY;
  });

  canvas.addEventListener('mouseleave', () => {
    mouseX = -999;
    mouseY = -999;
  });

  canvas.addEventListener('click', e => {
    if (cardAberto) return;
    const s = specialStars.find(s => Math.hypot(e.clientX - s.x, e.clientY - s.y) < 35);
    if (s) abrirCard(s);
  });

  // Eventos touch
  canvas.addEventListener('touchmove', e => {
    e.preventDefault();
    mouseX = e.touches[0].clientX;
    mouseY = e.touches[0].clientY;
  }, { passive: false });

  canvas.addEventListener('touchend', e => {
    const tx = e.changedTouches[0].clientX;
    const ty = e.changedTouches[0].clientY;
    mouseX = -999; mouseY = -999;
    if (cardAberto) return;
    const s = specialStars.find(s => Math.hypot(tx - s.x, ty - s.y) < 45);
    if (s) abrirCard(s);
  });

  // Botão intro
  document.getElementById('btn-entrar').addEventListener('click', () => {
    const intro = document.getElementById('intro');
    intro.style.transition = 'opacity 1.2s ease';
    intro.style.opacity = '0';
    setTimeout(() => intro.style.display = 'none', 1200);
  });

  window.addEventListener('resize', resize);
  resize();
  requestAnimationFrame(draw);
</script>
</body>
</html>
