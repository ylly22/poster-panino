# Panino Interattivo Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create an interactive HTML page from `poster.svg` where users drag ingredients onto a panino using Paper.js.

**Architecture:** Single HTML file loads Paper.js via CDN, imports the SVG, and handles all drag-and-drop interaction on canvas. No build tools, no server needed.

**Tech Stack:** Paper.js (CDN), vanilla JS, SVG import

**Files:**
- Modify: `poster.svg` (may need to add `id` attributes to ingredient groups for Paper.js identification)
- Create: `index.html` — the entire interactive page

---

### Task 1: Scaffold the HTML page with Paper.js

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create index.html with Paper.js boilerplate**

Create an HTML file that:
- Loads Paper.js from CDN (`https://unpkg.com/paper@0.12.17/dist/paper-full.min.js`)
- Contains a full-viewport `<canvas id="canvas">` element
- Has a `<script>` block for Paper.js code
- Positions the canvas centered, scales to fit viewport while maintaining A4 aspect ratio

**CSS:**
```html
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  html, body { width: 100%; height: 100%; overflow: hidden; background: #0f6846; }
  canvas { display: block; margin: 0 auto; }
</style>
```

**HTML structure:**
```html
<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Panino Interattivo</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    html, body { width: 100%; height: 100%; overflow: hidden; background: #0f6846; }
    canvas { display: block; margin: 0 auto; }
  </style>
</head>
<body>
  <canvas id="canvas" resize></canvas>
  <script src="https://unpkg.com/paper@0.12.17/dist/paper-full.min.js"></script>
  <script>
    paper.setup('canvas');
    // Resize canvas to fill viewport while maintaining A4 aspect ratio
    function resizeCanvas() {
      var canvas = document.getElementById('canvas');
      var w = window.innerWidth;
      var h = window.innerHeight;
      var svgAspect = 841.89 / 1190.55;
      var winAspect = w / h;
      if (winAspect > svgAspect) {
        canvas.style.height = h + 'px';
        canvas.style.width = (h * svgAspect) + 'px';
      } else {
        canvas.style.width = w + 'px';
        canvas.style.height = (w / svgAspect) + 'px';
      }
      canvas.width = parseInt(canvas.style.width);
      canvas.height = parseInt(canvas.style.height);
    }
    window.addEventListener('resize', resizeCanvas);
    resizeCanvas();
  </script>
</body>
</html>
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "feat: scaffold HTML page with Paper.js"
```

---

### Task 2: Load and import poster.svg into Paper.js

**Files:**
- Modify: `index.html` (add SVG import code)
- Reference: `poster.svg`

- [ ] **Step 1: Add SVG loading and import code**

After `paper.setup('canvas')`, fetch and import the SVG:

```js
// Load and import SVG
fetch('poster.svg')
  .then(function(response) { return response.text(); })
  .then(function(svgString) {
    var svg = paper.project.importSVG(svgString, { expandShapes: true });
    // Center the imported SVG on canvas
    svg.position = paper.view.center;
    initIngredients(svg);
  });
```

The `initIngredients` function will be defined in later tasks.

- [ ] **Step 2: Verify SVG renders correctly**

Open `index.html` in browser. The poster should appear exactly as in the SVG file, centered on the green canvas.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: import poster.svg into Paper.js"
```

---

### Task 3: Identify and isolate ingredient groups

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Define ingredient group IDs and add identification logic**

The SVG has these top-level groups (by `id` attribute):
- `Base` — background (fixed)
- `pane_base` — bottom bun (fixed, non-draggable, but acts as drop zone)
- `Buon_appetito` — text groups (handled specially)
- `cipolla`, `Insalata`, `Hamburger`, `Bacon`, `Formaggio`, `Panino_superiore`, `pomodoro` — ingredients (draggable)

Inside `initIngredients`, traverse the imported SVG and separate groups:

```js
var DRAGGABLE_IDS = ['cipolla', 'Insalata', 'Hamburger', 'Bacon', 'Formaggio', 'Panino_superiore', 'pomodoro'];
var dropZone;  // pane_base group
var draggableItems = [];
var texts;     // Buon_appetito group

function initIngredients(root) {
  root.children.forEach(function(child) {
    if (child.name === 'pane_base') {
      dropZone = child;
      dropZone.locked = true;
    } else if (child.name === 'Buon_appetito') {
      texts = child;
      // Hide "buon-appetito" initially, show "crea-il-tuo-panino"
      var buonApp = child.children['buon-appetito'];
      var creaPanino = child.children['crea-il-tuo-panino'];
      if (buonApp) buonApp.opacity = 0;
      if (creaPanino) creaPanino.opacity = 1;
    } else if (child.name === 'Base') {
      child.locked = true;  // background, non-interactive
    } else if (DRAGGABLE_IDS.indexOf(child.name) !== -1) {
      draggableItems.push(child);
      setupDrag(child, child.name);
    }
  });
}
```

- [ ] **Step 2: Verify identification**

Open the page and check the console. All ingredient groups should be identified correctly.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: identify and isolate ingredient groups"
```

---

### Task 4: Implement drag mechanics

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add drag-and-drop handlers to each ingredient**

Each ingredient stores its original position when drag starts. During drag, it follows the mouse. On release, check if it's on the panino.

```js
var originalPositions = {};

function setupDrag(item, name) {
  // Store original position
  originalPositions[name] = {
    x: item.position.x,
    y: item.position.y
  };
  
  var isPlaced = false;
  
  item.onMouseDown = function(event) {
    // Bring to front
    item.bringToFront();
  };
  
  item.onMouseDrag = function(event) {
    item.position.x += event.delta.x;
    item.position.y += event.delta.y;
  };
  
  item.onMouseUp = function(event) {
    if (isOnDropZone(item)) {
      if (!isPlaced) {
        placeIngredient(item);
        isPlaced = true;
      }
    } else {
      if (isPlaced && item.name !== 'Panino_superiore') {
        // Stay placed - ingredients don't return once placed
      } else {
        // Return to original position
        returnToOriginal(item, name);
        isPlaced = false;
      }
    }
  };
}
```

- [ ] **Step 2: Implement helper functions**

```js
function isOnDropZone(item) {
  if (!dropZone) return false;
  // Check if center of ingredient is within drop zone bounds
  var ingBounds = item.bounds;
  var dropBounds = dropZone.bounds;
  return ingBounds.intersects(dropBounds);
}

function placeIngredient(item) {
  // Scale up
  item.scale(1.8);
}

function returnToOriginal(item, name) {
  var orig = originalPositions[name];
  if (!orig) return;
  item.position.x = orig.x;
  item.position.y = orig.y;
}
```

- [ ] **Step 3: Verify drag works**

Open in browser. All ingredients should be draggable. When released on the panino, they should scale up. When released elsewhere, they should return to original position.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: implement drag-and-drop for ingredients"
```

---

### Task 5: Implement Buon Appetito transition

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add special handling for top bun (Panino_superiore)**

When Panino_superiore is placed on (or removed from) the panino, toggle text visibility.

```js
function setupDrag(item, name) {
  var isPlaced = false;
  
  item.onMouseDown = function(event) {
    item.bringToFront();
    
    // If top bun is being picked up after being placed, reset state
    if (name === 'Panino_superiore' && isPlaced) {
      hideBuonAppetito();
      isPlaced = false;
    }
  };
  
  item.onMouseDrag = function(event) {
    item.position.x += event.delta.x;
    item.position.y += event.delta.y;
  };
  
  item.onMouseUp = function(event) {
    if (isOnDropZone(item)) {
      if (!isPlaced) {
        placeIngredient(item);
        isPlaced = true;
        if (name === 'Panino_superiore') {
          showBuonAppetito();
        }
      }
    } else {
      if (name === 'Panino_superiore') {
        returnToOriginal(item, name);
        isPlaced = false;
        hideBuonAppetito();
      } else if (isPlaced) {
        // Other ingredients stay placed
      } else {
        returnToOriginal(item, name);
      }
    }
  };
}
```

- [ ] **Step 2: Add text toggle functions**

```js
function showBuonAppetito() {
  if (!texts) return;
  var buonApp = texts.children['buon-appetito'];
  var creaPanino = texts.children['crea-il-tuo-panino'];
  if (buonApp) buonApp.opacity = 1;
  if (creaPanino) creaPanino.opacity = 0;
}

function hideBuonAppetito() {
  if (!texts) return;
  var buonApp = texts.children['buon-appetito'];
  var creaPanino = texts.children['crea-il-tuo-panino'];
  if (buonApp) buonApp.opacity = 0;
  if (creaPanino) creaPanino.opacity = 1;
}
```

- [ ] **Step 3: Verify the full interaction flow**

Open in browser. Test:
1. Drag other ingredients → they place and scale
2. Drag top bun → "CREA IL TUO PANINO" disappears, "BUON APPETITO" appears
3. Drag top bun away → "BUON APPETITO" disappears, "CREA IL TUO PANINO" reappears
4. Other placed ingredients don't return when dragged away

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add Buon Appetito transition on top bun placement"
```

---

### Task 6: Final polish and testing

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add smooth transitions**

Use `requestAnimationFrame` for smoother return-to-original animation:

```js
function returnToOriginal(item, name) {
  var orig = originalPositions[name];
  if (!orig) return;
  var startX = item.position.x;
  var startY = item.position.y;
  var duration = 200; // ms
  var startTime = null;
  function animate(timestamp) {
    if (!startTime) startTime = timestamp;
    var elapsed = timestamp - startTime;
    var t = Math.min(elapsed / duration, 1);
    var ease = 1 - Math.pow(1 - t, 3); // ease out cubic
    item.position.x = startX + (orig.x - startX) * ease;
    item.position.y = startY + (orig.y - startY) * ease;
    if (t < 1) requestAnimationFrame(animate);
  }
  requestAnimationFrame(animate);
}
```

Also ensure placed ingredients other than the top bun cannot be dragged again:

```js
function setupDrag(item, name) {
  var isPlaced = false;
  
  item.onMouseDown = function(event) {
    if (isPlaced && name !== 'Panino_superiore') return; // Lock placed items
    item.bringToFront();
    // ... rest of logic
  };
}
```

- [ ] **Step 2: Touch support**

Paper.js handles touch events automatically through its event system, but ensure `touch-action: none` is set on the canvas:

```css
canvas { touch-action: none; }
```

- [ ] **Step 3: Verify everything works end-to-end**

Open in browser, test all interactions:
- Drag each ingredient type
- Place in different orders
- Place top bun first, then remove, add more ingredients, re-add top bun
- Resize browser window and verify scaling

- [ ] **Step 4: Final commit**

```bash
git add index.html
git commit -m "feat: final polish with smooth animations and touch support"
```
