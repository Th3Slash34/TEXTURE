# Texture Generator Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file HTML web application that generates geometric textures based on a grid of squares with corner-to-corner connections.

**Architecture:** Single standalone HTML file with inline CSS and vanilla JS. Canvas 2D for preview and PNG export, SVG generation for vector export. All state lives in JS variables, no external dependencies.

**Tech Stack:** HTML5, CSS3, Vanilla JavaScript, Canvas 2D

---

### Task 1: Create HTML skeleton and UI layout

**Files:**
- Create: `C:\Users\franc\Desktop\UNIVERSITA'\DESIGN 2\WORKSHOP ROCCO MODUGNO\OBSIDIAN\SITO\TEXTURE O PATTERN\index.html`

- [ ] **Step 1: Write the HTML skeleton and CSS**

```html
<!DOCTYPE html>
<html lang="it">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Texture Generator</title>
<style>
* { margin: 0; padding: 0; box-sizing: border-box; }
body { font-family: 'Segoe UI', sans-serif; background: #f5f5f5; display: flex; justify-content: center; align-items: center; min-height: 100vh; }
.container { background: #fff; border-radius: 8px; box-shadow: 0 4px 20px rgba(0,0,0,0.1); display: flex; padding: 24px; gap: 24px; }
.preview { border: 1px solid #ccc; border-radius: 4px; overflow: hidden; }
.preview canvas { display: block; }
.controls { width: 280px; display: flex; flex-direction: column; gap: 14px; }
.controls h1 { font-size: 20px; font-weight: 600; color: #222; margin-bottom: 4px; }
.control-group { display: flex; flex-direction: column; gap: 4px; }
.control-group label { font-size: 13px; font-weight: 500; color: #444; }
.control-group input[type="number"] { width: 70px; padding: 4px 8px; border: 1px solid #ccc; border-radius: 4px; font-size: 14px; }
.control-group input[type="range"] { width: 100%; }
.control-row { display: flex; align-items: center; gap: 8px; }
.control-row .value { font-size: 13px; color: #666; min-width: 30px; }
.btn-row { display: flex; gap: 8px; margin-top: 8px; }
.btn { padding: 8px 16px; border: none; border-radius: 4px; cursor: pointer; font-size: 14px; font-weight: 500; }
.btn-primary { background: #222; color: #fff; }
.btn-secondary { background: #e0e0e0; color: #222; }
.btn:hover { opacity: 0.85; }
</style>
</head>
<body>
<div class="container">
  <div class="preview">
    <canvas id="preview" width="600" height="600"></canvas>
  </div>
  <div class="controls">
    <h1>Texture Generator</h1>

    <div class="control-group">
      <label>Griglia</label>
      <div class="control-row">
        <input type="number" id="gridCols" min="2" max="20" value="5">
        <span>×</span>
        <input type="number" id="gridRows" min="2" max="20" value="5">
      </div>
    </div>

    <div class="control-group">
      <label>Dimensione quadrato</label>
      <div class="control-row">
        <input type="range" id="squareSize" min="10" max="100" value="40">
        <span class="value" id="squareSizeVal">40</span>
      </div>
    </div>

    <div class="control-group">
      <label>Spaziatura (× lato)</label>
      <div class="control-row">
        <input type="range" id="spacing" min="0.1" max="2.0" step="0.1" value="0.5">
        <span class="value" id="spacingVal">0.5</span>
      </div>
    </div>

    <div class="control-group">
      <label>Linee min</label>
      <div class="control-row">
        <input type="range" id="linesMin" min="1" max="10" value="1">
        <span class="value" id="linesMinVal">1</span>
      </div>
    </div>

    <div class="control-group">
      <label>Linee max</label>
      <div class="control-row">
        <input type="range" id="linesMax" min="1" max="10" value="3">
        <span class="value" id="linesMaxVal">3</span>
      </div>
    </div>

    <div class="control-group">
      <label>Quadrati</label>
      <div class="control-row">
        <select id="fillMode">
          <option value="empty">Vuoti</option>
          <option value="filled">Pieni</option>
        </select>
      </div>
    </div>

    <div class="btn-row">
      <button class="btn btn-primary" id="generateBtn">Genera</button>
      <button class="btn btn-secondary" id="exportPngBtn">PNG</button>
      <button class="btn btn-secondary" id="exportSvgBtn">SVG</button>
    </div>
  </div>
</div>
<script>
// JS will go here in subsequent tasks
</script>
</body>
</html>
```

- [ ] **Step 2: Verify the file opens in browser**

Open `index.html` in browser. Expected: layout renders with 600×600 canvas on left, controls panel on right, all labels and controls visible.

---

### Task 2: Implement core data model and generation algorithm

**Files:**
- Modify: `index.html` (add JS inside `<script>` tags)

- [ ] **Step 1: Add the data model and algorithm JS**

```javascript
function generateTexture(cols, rows, squareSize, spacingRatio, linesMin, linesMax) {
  const spacing = squareSize * spacingRatio;
  const cells = [];
  const activeCounts = [];

  // Step 1: assign active corner counts with horizontal constraint
  for (let r = 0; r < rows; r++) {
    activeCounts[r] = [];
    for (let c = 0; c < cols; c++) {
      const forbidden = new Set();
      if (c > 0) forbidden.add(activeCounts[r][c - 1]);
      if (c < cols - 1 && activeCounts[r][c + 1] !== undefined) forbidden.add(activeCounts[r][c + 1]);
      const allowed = [];
      for (let k = 0; k <= 4; k++) {
        if (!forbidden.has(k)) allowed.push(k);
      }
      activeCounts[r][c] = allowed[Math.floor(Math.random() * allowed.length)];
    }
  }

  // Step 2: assign which corners are active for each cell
  // corners order: [TL, TR, BL, BR]
  const corners = [
    { dr: -1, dc: -1 }, // TL → up-left
    { dr: -1, dc: 1 },  // TR → up-right
    { dr: 1, dc: -1 },  // BL → down-left
    { dr: 1, dc: 1 }    // BR → down-right
  ];

  for (let r = 0; r < rows; r++) {
    cells[r] = [];
    for (let c = 0; c < cols; c++) {
      const count = activeCounts[r][c];
      const cornerIndices = [];
      const pool = [0, 1, 2, 3];
      for (let i = 0; i < count; i++) {
        const idx = Math.floor(Math.random() * pool.length);
        cornerIndices.push(pool[idx]);
        pool.splice(idx, 1);
      }
      cells[r][c] = { row: r, col: c, activeCorners: cornerIndices };
    }
  }

  // Step 3: compute connections
  const connections = [];
  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      const cell = cells[r][c];
      const cx = c * (squareSize + spacing) + squareSize / 2;
      const cy = r * (squareSize + spacing) + squareSize / 2;
      for (const ci of cell.activeCorners) {
        const corner = corners[ci];
        const nr = r + corner.dr;
        const nc = c + corner.dc;
        if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
        const targetCell = cells[nr][nc];
        if (!targetCell) continue;
        // The corner position of source
        const srcCornerPos = getCornerPos(r, c, ci, squareSize, spacing);
        // Find the 2 closest corners of target cell to this position
        const targetCorners = getTargetCorners(nr, nc, srcCornerPos, squareSize, spacing);
        for (const tci of targetCorners) {
          const tgtCornerPos = getCornerPos(nr, nc, tci, squareSize, spacing);
          const multiplicity = linesMin + Math.floor(Math.random() * (linesMax - linesMin + 1));
          connections.push({
            x1: srcCornerPos.x, y1: srcCornerPos.y,
            x2: tgtCornerPos.x, y2: tgtCornerPos.y,
            multiplicity: multiplicity
          });
        }
      }
    }
  }

  return { cols, rows, squareSize, spacing, cells, connections, activeCounts };
}

function getCornerPos(row, col, cornerIndex, size, spacing) {
  const x0 = col * (size + spacing);
  const y0 = row * (size + spacing);
  switch (cornerIndex) {
    case 0: return { x: x0, y: y0 };               // TL
    case 1: return { x: x0 + size, y: y0 };         // TR
    case 2: return { x: x0, y: y0 + size };         // BL
    case 3: return { x: x0 + size, y: y0 + size };  // BR
  }
}

function getTargetCorners(row, col, srcPos, size, spacing) {
  const x0 = col * (size + spacing);
  const y0 = row * (size + spacing);
  const corners = [
    { x: x0, y: y0 },               // TL
    { x: x0 + size, y: y0 },         // TR
    { x: x0, y: y0 + size },         // BL
    { x: x0 + size, y: y0 + size }  // BR
  ];
  // Calculate distances
  const dists = corners.map((c, i) => ({
    idx: i,
    dist: Math.sqrt((c.x - srcPos.x) ** 2 + (c.y - srcPos.y) ** 2)
  }));
  dists.sort((a, b) => a.dist - b.dist);
  return [dists[0].idx, dists[1].idx];
}
```

- [ ] **Step 2: Verify syntax**

Open in browser and check dev console for errors. Expected: no errors.

---

### Task 3: Implement Canvas rendering

**Files:**
- Modify: `index.html` (add rendering functions inside `<script>`)

- [ ] **Step 1: Add rendering function**

```javascript
function renderPreview(data) {
  const canvas = document.getElementById('preview');
  const ctx = canvas.getContext('2d');
  const fillMode = document.getElementById('fillMode').value;
  const w = data.cols * (data.squareSize + data.spacing);
  const h = data.rows * (data.squareSize + data.spacing);
  canvas.width = w;
  canvas.height = h;

  // Background
  ctx.fillStyle = '#fff';
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  ctx.strokeStyle = '#000';
  ctx.fillStyle = '#000';
  ctx.lineWidth = 1;

  // Draw squares
  for (let r = 0; r < data.rows; r++) {
    for (let c = 0; c < data.cols; c++) {
      const x = c * (data.squareSize + data.spacing);
      const y = r * (data.squareSize + data.spacing);
      if (fillMode === 'filled') {
        ctx.fillRect(x, y, data.squareSize, data.squareSize);
      } else {
        ctx.strokeRect(x, y, data.squareSize, data.squareSize);
      }
    }
  }

  // Draw connections
  for (const conn of data.connections) {
    ctx.lineWidth = 1;
    for (let i = 0; i < conn.multiplicity; i++) {
      const offset = (i - (conn.multiplicity - 1) / 2) * 2;
      ctx.beginPath();
      // Offset perpendicular to line direction
      const dx = conn.x2 - conn.x1;
      const dy = conn.y2 - conn.y1;
      const len = Math.sqrt(dx * dx + dy * dy);
      if (len > 0) {
        const nx = -dy / len * offset;
        const ny = dx / len * offset;
        ctx.moveTo(conn.x1 + nx, conn.y1 + ny);
        ctx.lineTo(conn.x2 + nx, conn.y2 + ny);
      }
      ctx.stroke();
    }
  }
}
```

- [ ] **Step 2: Wire up generate function**

```javascript
function generate() {
  const cols = parseInt(document.getElementById('gridCols').value);
  const rows = parseInt(document.getElementById('gridRows').value);
  const squareSize = parseInt(document.getElementById('squareSize').value);
  const spacingRatio = parseFloat(document.getElementById('spacing').value);
  const linesMin = parseInt(document.getElementById('linesMin').value);
  const linesMax = parseInt(document.getElementById('linesMax').value);

  const data = generateTexture(cols, rows, squareSize, spacingRatio, linesMin, linesMax);
  renderPreview(data);
  window._textureData = data;
}

document.getElementById('generateBtn').addEventListener('click', generate);
document.querySelectorAll('.controls input, .controls select').forEach(el => {
  el.addEventListener('change', generate);
  el.addEventListener('input', () => {
    if (el.type === 'range') {
      const valEl = document.getElementById(el.id + 'Val');
      if (valEl) valEl.textContent = el.value;
    }
  });
});

window.addEventListener('load', generate);
```

- [ ] **Step 3: Verify rendering**

Open browser. Expected: texture appears in canvas. Changing controls and clicking "Genera" re-renders.

---

### Task 4: Implement SVG export

**Files:**
- Modify: `index.html` (add SVG export function inside `<script>`)

- [ ] **Step 1: Add SVG generation and export**

```javascript
function generateSVG(data) {
  const fillMode = document.getElementById('fillMode').value;
  const w = data.cols * (data.squareSize + data.spacing);
  const h = data.rows * (data.squareSize + data.spacing);
  let svg = `<svg xmlns="http://www.w3.org/2000/svg" width="${w}" height="${h}" viewBox="0 0 ${w} ${h}">\n`;
  svg += `<rect width="${w}" height="${h}" fill="white"/>\n`;

  // Squares
  for (let r = 0; r < data.rows; r++) {
    for (let c = 0; c < data.cols; c++) {
      const x = c * (data.squareSize + data.spacing);
      const y = r * (data.squareSize + data.spacing);
      if (fillMode === 'filled') {
        svg += `<rect x="${x}" y="${y}" width="${data.squareSize}" height="${data.squareSize}" fill="black"/>\n`;
      } else {
        svg += `<rect x="${x}" y="${y}" width="${data.squareSize}" height="${data.squareSize}" fill="none" stroke="black" stroke-width="1"/>\n`;
      }
    }
  }

  // Connections (as multiple lines with perpendicular offset)
  for (const conn of data.connections) {
    const dx = conn.x2 - conn.x1;
    const dy = conn.y2 - conn.y1;
    const len = Math.sqrt(dx * dx + dy * dy);
    for (let i = 0; i < conn.multiplicity; i++) {
      const offset = (i - (conn.multiplicity - 1) / 2) * 2;
      if (len > 0) {
        const nx = -dy / len * offset;
        const ny = dx / len * offset;
        svg += `<line x1="${conn.x1 + nx}" y1="${conn.y1 + ny}" x2="${conn.x2 + nx}" y2="${conn.y2 + ny}" stroke="black" stroke-width="1"/>\n`;
      } else {
        svg += `<line x1="${conn.x1}" y1="${conn.y1}" x2="${conn.x2}" y2="${conn.y2}" stroke="black" stroke-width="1"/>\n`;
      }
    }
  }

  svg += '</svg>';
  return svg;
}

function downloadFile(content, filename, mimeType) {
  const blob = new Blob([content], { type: mimeType });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
}

function exportPNG() {
  const canvas = document.getElementById('preview');
  canvas.toBlob(function(blob) {
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'texture.png';
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  });
}

function exportSVG() {
  const data = window._textureData;
  if (!data) return;
  const svg = generateSVG(data);
  downloadFile(svg, 'texture.svg', 'image/svg+xml');
}

document.getElementById('exportPngBtn').addEventListener('click', exportPNG);
document.getElementById('exportSvgBtn').addEventListener('click', exportSVG);
```

- [ ] **Step 2: Verify SVG export**

Click "SVG". Expected: browser downloads `texture.svg`. Open in browser/Illustrator - should look identical to canvas preview.

- [ ] **Step 3: Verify PNG export**

Click "PNG". Expected: browser downloads `texture.png`. Opens as image matching canvas preview.

---

### Task 5: Verify constraints and edge cases

- [ ] **Step 1: Verify horizontal constraint manually**

Set grid to 3×1 (or 5×1). Generate multiple times. For each row, inspect `activeCounts` in console. Verify no two adjacent cells have the same count.

- [ ] **Step 2: Verify corner connections at grid edges**

Set grid to 3×3. Generate. Inspect connections — verify connections only exist where source and target cells are both in bounds.

- [ ] **Step 3: Verify parameter boundary behavior**

Test grid 2×2 (minimum), grid 20×20 (maximum), square size 10 and 100, spacing 0.1 and 2.0, lines min=max (all lines same count), lines min=1, max=10. Expected: no errors, texture renders correctly for all combinations.
