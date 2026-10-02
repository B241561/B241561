<!-- ===================== HEADER ===================== -->

<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0F172A,50:1E3A8A,100:2563EB&height=220&section=header&text=ARMAAN%20KAUSHIK&fontSize=48&fontColor=FFFFFF&fontAlignY=36&animation=fadeIn&desc=B.Tech%20CSE%20%7C%20AI%2FML%20%7C%20Data%20Science%20%7&descSize=17&descAlignY=58" width="100%" />

<a href="https://git.io/typing-svg">
<img src="https://readme-typing-svg.demolab.com/?font=JetBrains+Mono&weight=600&size=21&duration=3000&pause=1000&color=2563EB&center=true&vCenter=true&width=750&height=55&lines=Building+things+that+solve+real+problems.;+%7C+AI%2FML+%7C+Data+Science;Learning+by+building%2C+testing+%26+improving." />
</a>

<br>

<a href="https://github.com/B241561">
<img src="https://img.shields.io/badge/GitHub-B241561-111827?style=for-the-badge&logo=github&logoColor=white"/>
</a>
&nbsp;
<a href="https://www.linkedin.com/in/arman-kaushik-29b5b6370/">
<img src="https://img.shields.io/badge/LinkedIn-Arman%20Kaushik-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white"/>
</a>

</div>

<p align="center">
  <img src="assets/cat.gif" width="180">
</p>

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>FeynML — 4D Hypercube Hero Preview</title>
<style>
  :root {
    --fy-bg: #0B0D10;
    --fy-amber: #FFB020;
    --fy-phosphor: #C9F227;
    --fy-steel: #3D5A73;
    --fy-text: #E8E6DF;
    --fy-text-dim: #6B6E74;
    --fy-border: #1C1F24;
  }
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    background: var(--fy-bg);
    color: var(--fy-text);
    font-family: 'Inter', system-ui, sans-serif;
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 40px;
  }
  .hero-4d-wrap {
    position: relative;
    width: 100%;
    max-width: 560px;
  }
  #fy4d-canvas {
    display: block;
    width: 100%;
    aspect-ratio: 1 / 0.82;
    border: 1px solid var(--fy-border);
  }
  .hud-tag {
    position: absolute;
    font-family: 'JetBrains Mono', monospace;
    font-size: 11px;
    color: var(--fy-text-dim);
    border: 1px solid var(--fy-border);
    background: rgba(11,13,16,0.85);
    padding: 6px 12px;
    line-height: 1.6;
  }
  .hud-tag strong { color: var(--fy-amber); font-size: 13px; display: block; }
  .hud-top-left     { top: 14px; left: 14px; }
  .hud-top-right    { top: 14px; right: 14px; }
  .hud-bottom-left  { bottom: 14px; left: 14px; }
  .hud-bottom-right { bottom: 14px; right: 14px; }
  .hud-center-label {
    position: absolute;
    bottom: 14px; left: 50%;
    transform: translateX(-50%);
    font-family: 'JetBrains Mono', monospace;
    font-size: 10px;
    letter-spacing: 0.1em;
    color: var(--fy-text-dim);
    text-transform: uppercase;
  }
</style>
</head>
<body>

<div class="hero-4d-wrap">
  <canvas id="fy4d-canvas"></canvas>
  <div class="hud-tag hud-top-left">Model Space<strong>4D Projection</strong></div>
  <div class="hud-tag hud-top-right">Drift Vector<strong id="hud-drift">Δ 0.041</strong></div>
  <div class="hud-tag hud-bottom-left">Dimensions<strong>16 Nodes</strong></div>
  <div class="hud-center-label">Rotating hyperplane · audit trace live</div>
</div>

<script>
(function () {
  var canvas = document.getElementById('fy4d-canvas');
  var ctx = canvas.getContext('2d');
  var wrap = canvas.parentElement;

  function resize() {
    var w = wrap.clientWidth;
    var h = Math.round(w * 0.82);
    canvas.width = w * devicePixelRatio;
    canvas.height = h * devicePixelRatio;
    canvas.style.width = w + 'px';
    canvas.style.height = h + 'px';
    ctx.setTransform(devicePixelRatio, 0, 0, devicePixelRatio, 0, 0);
  }
  window.addEventListener('resize', resize);
  resize();

  /* ---- Build tesseract: 16 vertices in 4D space (-1,1)^4 ---- */
  var verts4 = [];
  for (var i = 0; i < 16; i++) {
    verts4.push([
      (i & 1) ? 1 : -1,
      (i & 2) ? 1 : -1,
      (i & 4) ? 1 : -1,
      (i & 8) ? 1 : -1
    ]);
  }
  /* edges: connect vertices differing in exactly 1 bit */
  var edges = [];
  for (var a = 0; a < 16; a++) {
    for (var b = a + 1; b < 16; b++) {
      var diff = a ^ b;
      if (diff === 1 || diff === 2 || diff === 4 || diff === 8) edges.push([a, b]);
    }
  }
  /* inner cube (w = -1) vs outer cube (w = 1) for two-tone coloring */
  function isInner(idx) { return (idx & 8) === 0; }

  function rotate4D(v, planes, angles) {
    var p = v.slice();
    planes.forEach(function (pl, i) {
      var ang = angles[i];
      var c = Math.cos(ang), s = Math.sin(ang);
      var i0 = pl[0], i1 = pl[1];
      var x = p[i0], y = p[i1];
      p[i0] = x * c - y * s;
      p[i1] = x * s + y * c;
    });
    return p;
  }

  function project(v4, w, h) {
    /* perspective project 4D -> 3D -> 2D */
    var wDist = 2.6;
    var wScale = 1 / (wDist - v4[3]);
    var x3 = v4[0] * wScale, y3 = v4[1] * wScale, z3 = v4[2] * wScale;

    var zDist = 3.2;
    var zScale = 1 / (zDist - z3);
    var x2 = x3 * zScale, y2 = y3 * zScale;

    var scale = Math.min(w, h) * 0.30;
    return {
      x: w / 2 + x2 * scale,
      y: h / 2 + y2 * scale,
      depth: z3 // for opacity shading
    };
  }

  var planes = [ [0,1], [0,2], [1,3], [2,3] ]; // rotate across several 4D planes at once
  var t = 0;

  function draw() {
    t += 0.006;
    var w = canvas.width / devicePixelRatio;
    var h = canvas.height / devicePixelRatio;
    ctx.clearRect(0, 0, w, h);

    var angles = [ t * 0.7, t * 0.5, t * 0.35, t * 0.9 ];
    var projected = verts4.map(function (v) {
      var r = rotate4D(v, planes, angles);
      return project(r, w, h);
    });

    /* edges */
    edges.forEach(function (e) {
      var A = projected[e[0]], B = projected[e[1]];
      var innerEdge = isInner(e[0]) && isInner(e[1]);
      var outerEdge = !isInner(e[0]) && !isInner(e[1]);
      var avgDepth = (A.depth + B.depth) / 2;
      var alpha = 0.25 + (avgDepth + 1) * 0.28;

      ctx.beginPath();
      ctx.moveTo(A.x, A.y);
      ctx.lineTo(B.x, B.y);
      if (innerEdge) {
        ctx.strokeStyle = 'rgba(255,176,32,' + alpha.toFixed(3) + ')'; // amber
        ctx.lineWidth = 1.1;
      } else if (outerEdge) {
        ctx.strokeStyle = 'rgba(61,90,115,' + (alpha + 0.15).toFixed(3) + ')'; // steel
        ctx.lineWidth = 1;
      } else {
        ctx.strokeStyle = 'rgba(232,230,223,' + (alpha * 0.4).toFixed(3) + ')'; // faint bone connector
        ctx.lineWidth = 0.6;
      }
      ctx.stroke();
    });

    /* vertices */
    projected.forEach(function (p, i) {
      var alpha = 0.5 + (p.depth + 1) * 0.25;
      var r = isInner(i) ? 2.6 : 1.8;
      ctx.beginPath();
      ctx.arc(p.x, p.y, r, 0, Math.PI * 2);
      ctx.fillStyle = isInner(i)
        ? 'rgba(255,176,32,' + alpha.toFixed(3) + ')'
        : 'rgba(201,242,39,' + (alpha * 0.7).toFixed(3) + ')';
      ctx.fill();
    });

    requestAnimationFrame(draw);
  }
  draw();

  /* cosmetic HUD flicker */
  var driftEl = document.getElementById('hud-drift');
  setInterval(function () {
    driftEl.textContent = 'Δ ' + (Math.random() * 0.05 + 0.02).toFixed(3);
  }, 1800);
})();
</script>

</body>
</html>

<br><br>

---

<!-- ===================== ABOUT ===================== -->

## ⚡ About

> **B.Tech CSE student building , AI/ML, and Data Science projects.**

I enjoy turning ideas into working projects — from desktop applications and machine-learning systems to data-driven tools and automation.

**Current focus**

`DSA` · `Java` · `Python` · `SQL` · `Machine Learning` · `Data Science` ·

**Exploring**

`Power BI` · `Tableau` · `Data Visualization` · `AI Engineering`

---

<!-- ===================== TECH ===================== -->

## 🧰 Tech Stack

<div align="center">

<img src="https://skillicons.dev/icons?i=python,java,cpp,js,html,css,php,bootstrap,git,github,docker,jenkins,vscode,pytorch,tensorflow,sklearn&perline=16" />

<br><br>

<img src="https://img.shields.io/badge/SQL-4479A1?style=flat-square&logo=mysql&logoColor=white"/>
<img src="https://img.shields.io/badge/Power%20BI-F2C811?style=flat-square&logo=powerbi&logoColor=111827"/>
<img src="https://img.shields.io/badge/Tableau-E97627?style=flat-square&logo=tableau&logoColor=white"/>

</div>

---




🔗 Repository: FeynML

📊 GitHub Stats

<p align="center">
  <img src="https://github-readme-stats.vercel.app/api?username=B241561&show_icons=true&theme=dark&hide_border=false&include_all_commits=true&count_private=false" height="180" alt="GitHub Stats"/>
  <img src="https://github-readme-stats.vercel.app/api/top-langs/?username=B241561&layout=compact&theme=dark&hide_border=false&langs_count=8" height="180" alt="Top Languages"/>
</p>

🔥 Contribution Streak

<p align="center">
  <img src="https://streak-stats.demolab.com/?user=B241561&theme=dark&hide_border=false" alt="GitHub Streak"/>
</p>

🏆 GitHub Trophies

<p align="center">
  <img src="https://github-profile-trophy.vercel.app/?username=B241561&theme=onedark&no-frame=true&no-bg=true&margin-w=8&margin-h=8" alt="GitHub Trophies"/>
</p>

📌 Pinned Repositories

<p align="center">
  <a href="https://github.com/B241561/AI-Resume-Analyze">
    <img src="https://github-readme-stats.vercel.app/api/pin/?username=B241561&repo=AI-Resume-Analyze&theme=dark" alt="AI Resume Analyze"/>
  </a>
  <a href="https://github.com/B241561/FeynML">
    <img src="https://github-readme-stats.vercel.app/api/pin/?username=B241561&repo=FeynML&theme=dark" alt="FeynML"/>
  </a>
</p>
---
<!-- ===================== PROJECTS ===================== -->

## 🚀 Selected Projects

<table>
<tr>
<td width="50%">

### 🧠 AI Resume Analyzer

AI-powered desktop application for:

- PDF / DOCX resume parsing
- ATS-oriented analysis
- Job-description matching
- Skill extraction
- Gemini-assisted analysis
- OCR for scanned PDFs
- SQLite history
- PDF report generation

**[View Project →](https://github.com/B241561/AI-Resume-Analyze)**

</td>

<td width="50%">

### 🔬 FeynML

Machine-learning failure analysis engine focused on:

`Evaluation`

`Explainability`

`Root-Cause Analysis`

`Reliability`

**[View Project →](https://github.com/B241561/FeynML)**

</td>
</tr>
</table>



