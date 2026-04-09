# 吃豆人游戏实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 用单文件 HTML + Canvas + JavaScript 实现经典吃豆人游戏，浏览器双击即玩

**Architecture:** 单文件增量式开发，每个任务完成后文件都应可独立运行和测试。从骨架到完整游戏共 8 个任务。

**Tech Stack:** HTML5, CSS3, Vanilla JavaScript, Canvas 2D API

---

## 文件结构

只有一个文件，每个任务向其增量添加代码：
- **`index.html`** — 唯一文件，包含全部 HTML + CSS + JS

---

### Task 1: HTML 骨架 + 游戏常量 + 迷宫数据

**Files:**
- Create: `index.html`

- [ ] **Step 1: 创建 HTML 骨架，包含 CSS 样式和 JS 游戏常量/迷宫数据**

用以下内容创建 `index.html`：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>吃豆人</title>
<style>
* { margin: 0; padding: 0; box-sizing: border-box; }
body {
  background: #000;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  font-family: 'Courier New', monospace;
  color: #fff;
}
#hud {
  display: flex;
  gap: 40px;
  padding: 10px 0;
  font-size: 18px;
}
#hud span { min-width: 150px; text-align: center; }
#hud .label { color: #aaa; font-size: 14px; }
#hud .value { color: #ff0; font-size: 22px; font-weight: bold; }
canvas {
  border: 2px solid #2121DE;
  image-rendering: pixelated;
}
#lives {
  display: flex;
  gap: 10px;
  padding: 10px 0;
}
.life-icon {
  width: 20px; height: 20px;
  background: #ff0;
  border-radius: 50%;
  clip-path: polygon(100% 50%, 50% 50%, 100% 15%, 100% 85%, 50% 50%);
}
#message {
  position: absolute;
  font-size: 24px;
  color: #ff0;
  text-align: center;
  pointer-events: none;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.8);
}
</style>
</head>
<body>
<div id="hud">
  <div>
    <div class="label">SCORE</div>
    <div class="value" id="score">0</div>
  </div>
  <div>
    <div class="label">HIGH SCORE</div>
    <div class="value" id="highScore">0</div>
  </div>
  <div>
    <div class="label">LEVEL</div>
    <div class="value" id="level">1</div>
  </div>
</div>
<canvas id="gameCanvas"></canvas>
<div id="lives"></div>
<div id="message">按 Enter 开始游戏</div>

<script>
// ===== 游戏常量 =====
const TILE = 20; // 每格像素大小
const COLS = 28;
const ROWS = 31;
const WIDTH = COLS * TILE;
const HEIGHT = ROWS * TILE;

// 迷宫格子类型
const WALL = 1;
const DOT = 2;
const POWER_PELLET = 3;
const EMPTY = 0;
const GATE = 4;

// 方向
const DIR = {
  UP: { x: 0, y: -1 },
  DOWN: { x: 0, y: 1 },
  LEFT: { x: -1, y: 0 },
  RIGHT: { x: 1, y: 0 },
  NONE: { x: 0, y: 0 }
};

// 经典吃豆人迷宫布局 (28列 x 31行)
const MAZE_TEMPLATE = [
  [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
  [1,2,2,2,2,2,2,2,2,2,2,2,2,1,1,2,2,2,2,2,2,2,2,2,2,2,2,1],
  [1,2,1,1,1,1,2,1,1,1,1,1,2,1,1,2,1,1,1,1,1,2,1,1,1,1,2,1],
  [1,3,1,1,1,1,2,1,1,1,1,1,2,1,1,2,1,1,1,1,1,2,1,1,1,1,3,1],
  [1,2,1,1,1,1,2,1,1,1,1,1,2,1,1,2,1,1,1,1,1,2,1,1,1,1,2,1],
  [1,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,1],
  [1,2,1,1,1,1,2,1,1,2,1,1,1,1,1,1,1,1,2,1,1,2,1,1,1,1,2,1],
  [1,2,1,1,1,1,2,1,1,2,1,1,1,1,1,1,1,1,2,1,1,2,1,1,1,1,2,1],
  [1,2,2,2,2,2,2,1,1,2,2,2,2,1,1,2,2,2,2,1,1,2,2,2,2,2,2,1],
  [1,1,1,1,1,1,2,1,1,1,1,1,0,1,1,0,1,1,1,1,1,2,1,1,1,1,1,1],
  [0,0,0,0,0,1,2,1,1,1,1,1,0,1,1,0,1,1,1,1,1,2,1,0,0,0,0,0],
  [0,0,0,0,0,1,2,1,1,0,0,0,0,0,0,0,0,0,0,1,1,2,1,0,0,0,0,0],
  [0,0,0,0,0,1,2,1,1,0,1,1,1,4,4,1,1,1,0,1,1,2,1,0,0,0,0,0],
  [1,1,1,1,1,1,2,1,1,0,1,0,0,0,0,0,0,1,0,1,1,2,1,1,1,1,1,1],
  [0,0,0,0,0,0,2,0,0,0,1,0,0,0,0,0,0,1,0,0,0,2,0,0,0,0,0,0],
  [1,1,1,1,1,1,2,1,1,0,1,0,0,0,0,0,0,1,0,1,1,2,1,1,1,1,1,1],
  [0,0,0,0,0,1,2,1,1,0,1,1,1,1,1,1,1,1,0,1,1,2,1,0,0,0,0,0],
  [0,0,0,0,0,1,2,1,1,0,0,0,0,0,0,0,0,0,0,1,1,2,1,0,0,0,0,0],
  [0,0,0,0,0,1,2,1,1,0,1,1,1,1,1,1,1,1,0,1,1,2,1,0,0,0,0,0],
  [1,1,1,1,1,1,2,1,1,0,1,1,1,1,1,1,1,1,0,1,1,2,1,1,1,1,1,1],
  [1,2,2,2,2,2,2,2,2,2,2,2,2,1,1,2,2,2,2,2,2,2,2,2,2,2,2,1],
  [1,2,1,1,1,1,2,1,1,1,1,1,2,1,1,2,1,1,1,1,1,2,1,1,1,1,2,1],
  [1,2,1,1,1,1,2,1,1,1,1,1,2,1,1,2,1,1,1,1,1,2,1,1,1,1,2,1],
  [1,3,2,2,1,1,2,2,2,2,2,2,2,0,0,2,2,2,2,2,2,2,1,1,2,2,3,1],
  [1,1,1,2,1,1,2,1,1,2,1,1,1,1,1,1,1,1,2,1,1,2,1,1,2,1,1,1],
  [1,1,1,2,1,1,2,1,1,2,1,1,1,1,1,1,1,1,2,1,1,2,1,1,2,1,1,1],
  [1,2,2,2,2,2,2,1,1,2,2,2,2,1,1,2,2,2,2,1,1,2,2,2,2,2,2,1],
  [1,2,1,1,1,1,1,1,1,1,1,1,2,1,1,2,1,1,1,1,1,1,1,1,1,1,2,1],
  [1,2,1,1,1,1,1,1,1,1,1,1,2,1,1,2,1,1,1,1,1,1,1,1,1,1,2,1],
  [1,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,1],
  [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1]
];

// 游戏状态
const STATE = {
  READY: 'ready',
  PLAYING: 'playing',
  PAUSED: 'paused',
  GAME_OVER: 'game_over',
  WIN: 'win'
};

// ===== 全局游戏状态 =====
let gameState = STATE.READY;
let maze = [];
let score = 0;
let highScore = parseInt(localStorage.getItem('pacmanHighScore')) || 0;
let level = 1;
let totalDots = 0;
let dotsEaten = 0;

// ===== Canvas 初始化 =====
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
canvas.width = WIDTH;
canvas.height = HEIGHT;

// 初始化迷宫（深拷贝）
function initMaze() {
  maze = MAZE_TEMPLATE.map(row => [...row]);
  totalDots = 0;
  dotsEaten = 0;
  for (let y = 0; y < ROWS; y++) {
    for (let x = 0; x < COLS; x++) {
      if (maze[y][x] === DOT || maze[y][x] === POWER_PELLET) {
        totalDots++;
      }
    }
  }
}

// ===== 工具函数 =====
function isWall(x, y) {
  if (y < 0 || y >= ROWS) return true;
  // 隧道处理
  if (x < 0 || x >= COLS) return false;
  return maze[y][x] === WALL;
}

function isWalkable(x, y, allowGate) {
  if (y < 0 || y >= ROWS) return false;
  // 隧道处理
  if (x < 0 || x >= COLS) return true;
  const tile = maze[y][x];
  if (tile === WALL) return false;
  if (tile === GATE && !allowGate) return false;
  return true;
}

// ===== 验证 =====
initMaze();
console.log('迷宫初始化完成, 总豆子数:', totalDots);
console.log('迷宫尺寸:', COLS, 'x', ROWS);
</script>
</body>
</html>
```

- [ ] **Step 2: 验证文件可打开**

在浏览器中打开 `index.html`，应该看到：
- 顶部 HUD 显示 SCORE: 0, HIGH SCORE: 0, LEVEL: 1
- 中间黑色 Canvas 区域
- 底部 "按 Enter 开始游戏" 提示
- 控制台输出 "迷宫初始化完成, 总豆子数: 244"

- [ ] **Step 3: 提交**

```bash
git add index.html
git commit -m "feat: 添加游戏骨架、常量定义和迷宫数据"
```

---

### Task 2: 迷宫渲染

**Files:**
- Modify: `index.html`（在 `initMaze()` 后、`initMaze()` 调用前添加渲染代码）

- [ ] **Step 1: 添加迷宫渲染函数**

在 `</script>` 标签前添加以下代码（在 `initMaze()` 调用之前）：

```javascript
// ===== 渲染函数 =====
function drawMaze() {
  for (let y = 0; y < ROWS; y++) {
    for (let x = 0; x < COLS; x++) {
      const tile = maze[y][x];
      const px = x * TILE;
      const py = y * TILE;

      if (tile === WALL) {
        ctx.fillStyle = '#2121DE';
        ctx.fillRect(px, py, TILE, TILE);
        // 内边框效果
        ctx.fillStyle = '#1919AA';
        ctx.fillRect(px + 2, py + 2, TILE - 4, TILE - 4);
      } else if (tile === DOT) {
        ctx.fillStyle = '#FFB8AE';
        ctx.beginPath();
        ctx.arc(px + TILE / 2, py + TILE / 2, 2, 0, Math.PI * 2);
        ctx.fill();
      } else if (tile === POWER_PELLET) {
        // 闪烁效果
        const pulse = Math.sin(Date.now() / 200) * 0.3 + 0.7;
        ctx.fillStyle = `rgba(255, 184, 174, ${pulse})`;
        ctx.beginPath();
        ctx.arc(px + TILE / 2, py + TILE / 2, 6, 0, Math.PI * 2);
        ctx.fill();
      } else if (tile === GATE) {
        ctx.fillStyle = '#FFB8FF';
        ctx.fillRect(px, py + TILE / 2 - 2, TILE, 4);
      }
    }
  }
}

function render() {
  // 清屏
  ctx.fillStyle = '#000';
  ctx.fillRect(0, 0, WIDTH, HEIGHT);
  // 绘制迷宫
  drawMaze();
}

// 渲染循环
function gameLoop() {
  render();
  requestAnimationFrame(gameLoop);
}
```

修改 `initMaze()` 调用部分，改为：

```javascript
initMaze();
document.getElementById('highScore').textContent = highScore;
console.log('迷宫初始化完成, 总豆子数:', totalDots);
updateLivesDisplay();
requestAnimationFrame(gameLoop);
```

添加 `updateLivesDisplay` 函数（在 `updateHighScore` 附近）：

```javascript
function updateLivesDisplay() {
  const livesEl = document.getElementById('lives');
  livesEl.innerHTML = '';
  for (let i = 0; i < pacman.lives; i++) {
    const dot = document.createElement('div');
    dot.className = 'life-icon';
    livesEl.appendChild(dot);
  }
}

function updateScoreDisplay() {
  document.getElementById('score').textContent = score;
}

function updateLevelDisplay() {
  document.getElementById('level').textContent = level;
}

function showMessage(text) {
  document.getElementById('message').textContent = text;
}

function hideMessage() {
  document.getElementById('message').textContent = '';
}
```

- [ ] **Step 2: 验证**

在浏览器中打开 `index.html`，应该看到：
- 完整的经典吃豆人迷宫（蓝色墙壁、粉色豆子、闪烁能量球）
- 底部显示 3 个黄色生命图标
- 豆子会闪烁

- [ ] **Step 3: 提交**

```bash
git add index.html
git commit -m "feat: 实现迷宫渲染和渲染循环"
```

---

### Task 3: 吃豆人渲染 + 键盘输入 + 移动

**Files:**
- Modify: `index.html`

- [ ] **Step 1: 添加吃豆人对象和渲染/移动逻辑**

在 `// ===== 全局游戏状态 =====` 区块后添加：

```javascript
// ===== 吃豆人 =====
const pacman = {
  x: 14,
  y: 23,
  direction: DIR.LEFT,
  nextDirection: DIR.LEFT,
  lives: 3,
  mouthAngle: 0.25,
  mouthSpeed: 0.08,
  mouthOpen: true
};
```

在 `// ===== 渲染函数 =====` 区块中添加 `drawPacman`：

```javascript
function drawPacman() {
  const px = pacman.x * TILE + TILE / 2;
  const py = pacman.y * TILE + TILE / 2;
  const radius = TILE / 2 - 1;

  // 计算嘴巴朝向角度
  let angle = 0;
  if (pacman.direction === DIR.RIGHT) angle = 0;
  else if (pacman.direction === DIR.DOWN) angle = Math.PI / 2;
  else if (pacman.direction === DIR.LEFT) angle = Math.PI;
  else if (pacman.direction === DIR.UP) angle = -Math.PI / 2;

  // 嘴巴动画
  if (pacman.mouthOpen) {
    pacman.mouthAngle += pacman.mouthSpeed;
    if (pacman.mouthAngle >= 0.25) pacman.mouthOpen = false;
  } else {
    pacman.mouthAngle -= pacman.mouthSpeed;
    if (pacman.mouthAngle <= 0.02) pacman.mouthOpen = true;
  }

  const mouth = pacman.mouthAngle * Math.PI;

  ctx.fillStyle = '#FFFF00';
  ctx.beginPath();
  ctx.arc(px, py, radius, angle + mouth, angle + Math.PI * 2 - mouth);
  ctx.lineTo(px, py);
  ctx.closePath();
  ctx.fill();
}
```

在 `render()` 函数中，`drawMaze()` 后添加：

```javascript
  drawPacman();
```

- [ ] **Step 2: 添加吃豆人移动逻辑**

在渲染函数之后添加：

```javascript
// ===== 吃豆人移动 =====
let moveTimer = 0;
const MOVE_INTERVAL = 150; // 毫秒，移动间隔

function movePacman(dt) {
  moveTimer += dt;
  if (moveTimer < MOVE_INTERVAL) return;
  moveTimer = 0;

  // 尝试转向
  const nx = pacman.x + pacman.nextDirection.x;
  const ny = pacman.y + pacman.nextDirection.y;
  if (isWalkable(nx, ny, false)) {
    pacman.direction = pacman.nextDirection;
  }

  // 按当前方向移动
  const newX = pacman.x + pacman.direction.x;
  const newY = pacman.y + pacman.direction.y;

  // 隧道处理
  if (newX < 0) pacman.x = COLS - 1;
  else if (newX >= COLS) pacman.x = 0;
  else if (isWalkable(newX, newY, false)) {
    pacman.x = newX;
    pacman.y = newY;
  }
}
```

修改 `gameLoop()` 函数：

```javascript
let lastTime = 0;
function gameLoop(timestamp) {
  const dt = timestamp - lastTime;
  lastTime = timestamp;

  if (gameState === STATE.PLAYING) {
    movePacman(dt);
  }

  render();
  requestAnimationFrame(gameLoop);
}
```

- [ ] **Step 3: 添加键盘输入处理**

在文件底部 `requestAnimationFrame(gameLoop);` 之前添加：

```javascript
// ===== 键盘输入 =====
document.addEventListener('keydown', (e) => {
  switch (e.key) {
    case 'ArrowUp': case 'w': case 'W':
      pacman.nextDirection = DIR.UP;
      e.preventDefault();
      break;
    case 'ArrowDown': case 's': case 'S':
      pacman.nextDirection = DIR.DOWN;
      e.preventDefault();
      break;
    case 'ArrowLeft': case 'a': case 'A':
      pacman.nextDirection = DIR.LEFT;
      e.preventDefault();
      break;
    case 'ArrowRight': case 'd': case 'D':
      pacman.nextDirection = DIR.RIGHT;
      e.preventDefault();
      break;
    case ' ':
      if (gameState === STATE.PLAYING) {
        gameState = STATE.PAUSED;
        showMessage('游戏暂停 - 按空格继续');
      } else if (gameState === STATE.PAUSED) {
        gameState = STATE.PLAYING;
        hideMessage();
      }
      e.preventDefault();
      break;
    case 'Enter':
      if (gameState === STATE.READY) {
        gameState = STATE.PLAYING;
        hideMessage();
      } else if (gameState === STATE.GAME_OVER || gameState === STATE.WIN) {
        resetGame();
        gameState = STATE.PLAYING;
        hideMessage();
      }
      e.preventDefault();
      break;
  }
});
```

添加 `resetGame` 函数：

```javascript
function resetGame() {
  score = 0;
  level = 1;
  initMaze();
  resetPositions();
  updateScoreDisplay();
  updateLevelDisplay();
  updateLivesDisplay();
}

function resetPositions() {
  pacman.x = 14;
  pacman.y = 23;
  pacman.direction = DIR.LEFT;
  pacman.nextDirection = DIR.LEFT;
}
```

- [ ] **Step 4: 验证**

打开 `index.html`：
- 按 Enter 开始游戏
- 方向键/WASD 控制吃豆人移动
- 吃豆人嘴巴有张合动画
- 空格暂停
- 墙壁不可穿越
- 可以穿过左右隧道

- [ ] **Step 5: 提交**

```bash
git add index.html
git commit -m "feat: 实现吃豆人渲染、键盘输入和移动逻辑"
```

---

### Task 4: 鬼魂渲染 + 移动 + AI

**Files:**
- Modify: `index.html`

- [ ] **Step 1: 添加鬼魂数据**

在 `resetPositions()` 函数后添加：

```javascript
// ===== 鬼魂 =====
const ghostColors = ['#FF0000', '#FFB8FF', '#00FFFF', '#FFB852'];
const ghostNames = ['Blinky', 'Pinky', 'Inky', 'Clyde'];

// 鬼魂起始位置
const GHOST_STARTS = [
  { x: 14, y: 11 },
  { x: 13, y: 14 },
  { x: 14, y: 14 },
  { x: 15, y: 14 }
];

// 散射目标（各自角落）
const SCATTER_TARGETS = [
  { x: 25, y: 0 },   // Blinky - 右上
  { x: 2, y: 0 },    // Pinky - 左上
  { x: 27, y: 30 },  // Inky - 右下
  { x: 0, y: 30 }    // Clyde - 左下
];

let ghosts = [];
let ghostMode = 'scatter'; // scatter | chase
let modeTimer = 0;
let frightenedTimer = 0;
let frightenedGhostIndex = 0; // 连续吃鬼魂的计数器

function initGhosts() {
  ghosts = [];
  for (let i = 0; i < 4; i++) {
    ghosts.push({
      x: GHOST_STARTS[i].x,
      y: GHOST_STARTS[i].y,
      direction: DIR.UP,
      state: 'scatter', // scatter | chase | frightened | eaten
      color: ghostColors[i],
      inHouse: i > 0 // 第一个鬼魂在外面，其他在鬼屋
    });
  }
  ghostMode = 'scatter';
  modeTimer = 0;
}
```

在 `resetPositions()` 中添加鬼魂重置：

```javascript
  initGhosts();
```

- [ ] **Step 2: 添加鬼魂渲染**

在 `drawPacman()` 后添加：

```javascript
function drawGhost(ghost) {
  const px = ghost.x * TILE;
  const py = ghost.y * TILE;
  const cx = px + TILE / 2;
  const cy = py + TILE / 2;
  const r = TILE / 2 - 1;

  // 颜色
  if (ghost.state === 'frightened') {
    ctx.fillStyle = frightenedTimer < 2000 && Math.floor(Date.now() / 200) % 2 ? '#fff' : '#2121DE';
  } else if (ghost.state === 'eaten') {
    // 只画眼睛，不画身体
    drawGhostEyes(cx, cy, ghost.direction);
    return;
  } else {
    ctx.fillStyle = ghost.color;
  }

  // 身体（圆顶+波浪底）
  ctx.beginPath();
  ctx.arc(cx, cy - 2, r, Math.PI, 0);
  ctx.lineTo(cx + r, cy + r - 2);
  // 波浪底部
  const segments = 3;
  const segWidth = (r * 2) / segments;
  for (let i = segments; i > 0; i--) {
    const sx = cx + r - (segments - i) * segWidth;
    ctx.quadraticCurveTo(sx - segWidth / 2, cy + r + 2, sx - segWidth, cy + r - 2);
  }
  ctx.closePath();
  ctx.fill();

  // 眼睛
  if (ghost.state !== 'frightened') {
    drawGhostEyes(cx, cy, ghost.direction);
  } else {
    // 惊吓表情
    ctx.fillStyle = '#fff';
    ctx.beginPath();
    ctx.arc(cx - 3, cy - 3, 2, 0, Math.PI * 2);
    ctx.arc(cx + 3, cy - 3, 2, 0, Math.PI * 2);
    ctx.fill();
    // 波浪嘴
    ctx.strokeStyle = '#fff';
    ctx.lineWidth = 1;
    ctx.beginPath();
    ctx.moveTo(cx - 5, cy + 3);
    for (let i = 0; i < 5; i++) {
      ctx.lineTo(cx - 5 + i * 2.5, cy + (i % 2 === 0 ? 3 : 5));
    }
    ctx.stroke();
  }
}

function drawGhostEyes(cx, cy, direction) {
  // 眼白
  ctx.fillStyle = '#fff';
  ctx.beginPath();
  ctx.ellipse(cx - 4, cy - 3, 4, 5, 0, 0, Math.PI * 2);
  ctx.ellipse(cx + 4, cy - 3, 4, 5, 0, 0, Math.PI * 2);
  ctx.fill();

  // 瞳孔（跟随方向偏移）
  ctx.fillStyle = '#00f';
  const ox = direction.x * 2;
  const oy = direction.y * 2;
  ctx.beginPath();
  ctx.arc(cx - 4 + ox, cy - 3 + oy, 2, 0, Math.PI * 2);
  ctx.arc(cx + 4 + ox, cy - 3 + oy, 2, 0, Math.PI * 2);
  ctx.fill();
}
```

在 `render()` 中，`drawPacman()` 后添加：

```javascript
  for (const ghost of ghosts) {
    drawGhost(ghost);
  }
```

- [ ] **Step 3: 添加鬼魂移动和 AI**

```javascript
// ===== 鬼魂 AI =====
const GHOST_MOVE_INTERVAL = 180;
const GHOST_FRIGHTENED_INTERVAL = 250;
const FRIGHTENED_DURATION = 8000; // 8秒
const MODE_SWITCH_INTERVAL = 20000; // 20秒切换 scatter/chase

let ghostMoveTimers = [0, 0, 0, 0];

function getGhostTarget(ghost, index) {
  if (ghost.state === 'eaten') {
    return { x: 14, y: 14 }; // 鬼魂老家
  }
  if (ghost.state === 'frightened') {
    return null; // 随机移动
  }
  if (ghost.state === 'scatter') {
    return SCATTER_TARGETS[index];
  }
  // CHASE 模式 - 追踪吃豆人
  return { x: pacman.x, y: pacman.y };
}

function manhattanDistance(x1, y1, x2, y2) {
  return Math.abs(x1 - x2) + Math.abs(y1 - y2);
}

function getOppositeDirection(dir) {
  if (dir === DIR.UP) return DIR.DOWN;
  if (dir === DIR.DOWN) return DIR.UP;
  if (dir === DIR.LEFT) return DIR.RIGHT;
  if (dir === DIR.RIGHT) return DIR.LEFT;
  return DIR.NONE;
}

function chooseDirection(ghost, index) {
  const target = getGhostTarget(ghost, index);
  const opposite = getOppositeDirection(ghost.direction);
  const allowGate = ghost.state === 'eaten' || ghost.inHouse;

  // 获取所有可行方向（不能回头）
  const possibleDirs = [DIR.UP, DIR.DOWN, DIR.LEFT, DIR.RIGHT].filter(dir => {
    if (dir === opposite) return false;
    const nx = ghost.x + dir.x;
    const ny = ghost.y + dir.y;
    return isWalkable(nx, ny, allowGate);
  });

  if (possibleDirs.length === 0) {
    // 如果无路可走（死胡同），允许回头
    const nx = ghost.x + opposite.x;
    const ny = ghost.y + opposite.y;
    if (isWalkable(nx, ny, allowGate)) return opposite;
    return ghost.direction;
  }

  if (ghost.state === 'frightened') {
    // 随机选择
    return possibleDirs[Math.floor(Math.random() * possibleDirs.length)];
  }

  // 选择曼哈顿距离最短的方向
  let bestDir = possibleDirs[0];
  let bestDist = Infinity;

  for (const dir of possibleDirs) {
    const nx = ghost.x + dir.x;
    const ny = ghost.y + dir.y;
    const dist = manhattanDistance(nx, ny, target.x, target.y);
    if (dist < bestDist) {
      bestDist = dist;
      bestDir = dir;
    }
  }

  return bestDir;
}

function moveGhosts(dt) {
  // 模式计时器
  modeTimer += dt;
  if (modeTimer > MODE_SWITCH_INTERVAL) {
    modeTimer = 0;
    ghostMode = ghostMode === 'scatter' ? 'chase' : 'scatter';
    for (const ghost of ghosts) {
      if (ghost.state !== 'frightened' && ghost.state !== 'eaten') {
        ghost.state = ghostMode;
      }
    }
  }

  // 惊吓计时器
  if (frightenedTimer > 0) {
    frightenedTimer -= dt;
    if (frightenedTimer <= 0) {
      frightenedTimer = 0;
      frightenedGhostIndex = 0;
      for (const ghost of ghosts) {
        if (ghost.state === 'frightened') {
          ghost.state = ghostMode;
        }
      }
    }
  }

  // 移动每个鬼魂
  for (let i = 0; i < ghosts.length; i++) {
    const ghost = ghosts[i];
    const interval = ghost.state === 'frightened'
      ? GHOST_FRIGHTENED_INTERVAL
      : GHOST_MOVE_INTERVAL;

    ghostMoveTimers[i] += dt;
    if (ghostMoveTimers[i] < interval) continue;
    ghostMoveTimers[i] = 0;

    // 鬼屋释放
    if (ghost.inHouse) {
      ghost.inHouse = false;
      ghost.x = 14;
      ghost.y = 11;
      ghost.direction = DIR.LEFT;
      continue;
    }

    // 选择方向
    ghost.direction = chooseDirection(ghost, i);

    // 移动
    const nx = ghost.x + ghost.direction.x;
    const ny = ghost.y + ghost.direction.y;

    // 到达老家（被吃后）
    if (ghost.state === 'eaten' && nx === 14 && ny >= 13 && ny <= 15) {
      ghost.state = ghostMode;
      ghost.x = 14;
      ghost.y = 11;
      continue;
    }

    // 隧道
    if (nx < 0) ghost.x = COLS - 1;
    else if (nx >= COLS) ghost.x = 0;
    else {
      ghost.x = nx;
      ghost.y = ny;
    }
  }
}

function activateFrightened() {
  frightenedTimer = FRIGHTENED_DURATION;
  frightenedGhostIndex = 0;
  for (const ghost of ghosts) {
    if (ghost.state !== 'eaten') {
      ghost.state = 'frightened';
      // 反转方向
      ghost.direction = getOppositeDirection(ghost.direction);
    }
  }
}
```

在 `gameLoop()` 中，`movePacman(dt)` 后添加：

```javascript
    moveGhosts(dt);
```

修改 `initGhosts()` 的调用位置，在 `initMaze()` 后添加：

```javascript
  initGhosts();
```

- [ ] **Step 4: 验证**

打开 `index.html`，按 Enter 开始：
- 4 个不同颜色的鬼魂出现
- 鬼魂从鬼屋出发
- 鬼魂会追踪吃豆人
- 吃能量球后鬼魂变蓝
- 鬼魂在 scatter/chase 间切换

- [ ] **Step 5: 提交**

```bash
git add index.html
git commit -m "feat: 实现鬼魂渲染、移动和曼哈顿距离AI"
```

---

### Task 5: 碰撞检测 - 吃豆子 + 计分

**Files:**
- Modify: `index.html`

- [ ] **Step 1: 添加吃豆子碰撞检测**

在 `movePacman()` 函数末尾（移动逻辑后）添加：

```javascript
  // 吃豆子
  const tile = maze[pacman.y][pacman.x];
  if (tile === DOT) {
    maze[pacman.y][pacman.x] = EMPTY;
    score += 10;
    dotsEaten++;
    updateScoreDisplay();
    checkHighScore();
    checkWin();
  } else if (tile === POWER_PELLET) {
    maze[pacman.y][pacman.x] = EMPTY;
    score += 50;
    dotsEaten++;
    activateFrightened();
    updateScoreDisplay();
    checkHighScore();
    checkWin();
  }
```

添加辅助函数：

```javascript
function checkHighScore() {
  if (score > highScore) {
    highScore = score;
    localStorage.setItem('pacmanHighScore', highScore);
    document.getElementById('highScore').textContent = highScore;
  }
}

function checkWin() {
  if (dotsEaten >= totalDots) {
    gameState = STATE.WIN;
    level++;
    showMessage('恭喜通关！按 Enter 进入第 ' + level + ' 关');
  }
}
```

- [ ] **Step 2: 验证**

打开 `index.html`：
- 吃豆人经过豆子时豆子消失
- 分数增加 10 分
- 吃能量球得 50 分并激活鬼魂惊吓
- 吃完所有豆子显示通关消息

- [ ] **Step 3: 提交**

```bash
git add index.html
git commit -m "feat: 实现吃豆子碰撞检测和计分系统"
```

---

### Task 6: 吃豆人 vs 鬼魂碰撞 + 吃鬼魂

**Files:**
- Modify: `index.html`

- [ ] **Step 1: 添加吃豆人与鬼魂碰撞检测**

在 `moveGhosts()` 调用后、`gameLoop` 的 `render()` 前添加：

```javascript
    // 碰撞检测：吃豆人 vs 鬼魂
    checkPacmanGhostCollision();
```

添加碰撞检测函数：

```javascript
function checkPacmanGhostCollision() {
  for (const ghost of ghosts) {
    if (ghost.inHouse) continue;

    if (ghost.x === pacman.x && ghost.y === pacman.y) {
      if (ghost.state === 'frightened') {
        // 吃掉鬼魂
        ghost.state = 'eaten';
        const ghostScore = 200 * Math.pow(2, frightenedGhostIndex);
        score += ghostScore;
        frightenedGhostIndex++;
        updateScoreDisplay();
        checkHighScore();
      } else if (ghost.state !== 'eaten') {
        // 被鬼魂抓到
        pacmanHit();
        return;
      }
    }
  }
}

function pacmanHit() {
  pacman.lives--;
  updateLivesDisplay();

  if (pacman.lives <= 0) {
    gameState = STATE.GAME_OVER;
    showMessage('GAME OVER! 得分: ' + score + ' - 按 Enter 重新开始');
  } else {
    // 重置位置但保留迷宫状态
    gameState = STATE.READY;
    resetPositions();
    initGhosts();
    showMessage('按 Enter 继续');
  }
}
```

- [ ] **Step 2: 验证**

打开 `index.html`：
- FRIGHTENED 状态下碰到鬼魂，鬼魂变成眼睛回老家
- 分数增加（200, 400, 800, 1600）
- 正常状态下碰到鬼魂，减少生命
- 3 条命用完显示 GAME OVER

- [ ] **Step 3: 提交**

```bash
git add index.html
git commit -m "feat: 实现吃豆人与鬼魂碰撞检测和生命系统"
```

---

### Task 7: 游戏状态完善 + 关卡系统

**Files:**
- Modify: `index.html`

- [ ] **Step 1: 完善关卡系统**

修改 `checkWin()` 函数：

```javascript
function checkWin() {
  if (dotsEaten >= totalDots) {
    level++;
    updateLevelDisplay();
    nextLevel();
  }
}

function nextLevel() {
  gameState = STATE.READY;
  initMaze();
  resetPositions();
  initGhosts();
  frightenedTimer = 0;
  modeTimer = 0;
  updateLivesDisplay();
  showMessage('第 ' + level + ' 关 - 按 Enter 开始');
}
```

- [ ] **Step 2: 完善游戏状态显示**

修改 `render()` 函数，在不同状态显示不同内容：

```javascript
function render() {
  // 清屏
  ctx.fillStyle = '#000';
  ctx.fillRect(0, 0, WIDTH, HEIGHT);
  // 绘制迷宫
  drawMaze();

  if (gameState === STATE.READY || gameState === STATE.PLAYING || gameState === STATE.PAUSED) {
    drawPacman();
    for (const ghost of ghosts) {
      drawGhost(ghost);
    }
  }

  if (gameState === STATE.PAUSED) {
    ctx.fillStyle = 'rgba(0, 0, 0, 0.5)';
    ctx.fillRect(0, 0, WIDTH, HEIGHT);
    ctx.fillStyle = '#ff0';
    ctx.font = 'bold 36px Courier New';
    ctx.textAlign = 'center';
    ctx.fillText('PAUSED', WIDTH / 2, HEIGHT / 2);
  }

  if (gameState === STATE.GAME_OVER) {
    ctx.fillStyle = 'rgba(0, 0, 0, 0.6)';
    ctx.fillRect(0, 0, WIDTH, HEIGHT);
    ctx.fillStyle = '#f00';
    ctx.font = 'bold 42px Courier New';
    ctx.textAlign = 'center';
    ctx.fillText('GAME OVER', WIDTH / 2, HEIGHT / 2 - 20);
    ctx.fillStyle = '#fff';
    ctx.font = '24px Courier New';
    ctx.fillText('得分: ' + score, WIDTH / 2, HEIGHT / 2 + 20);
  }

  if (gameState === STATE.WIN) {
    ctx.fillStyle = 'rgba(0, 0, 0, 0.4)';
    ctx.fillRect(0, 0, WIDTH, HEIGHT);
    ctx.fillStyle = '#0f0';
    ctx.font = 'bold 36px Courier New';
    ctx.textAlign = 'center';
    ctx.fillText('LEVEL CLEAR!', WIDTH / 2, HEIGHT / 2);
  }
}
```

- [ ] **Step 3: 验证**

- 吃完所有豆子进入下一关
- 新关卡迷宫重置
- 暂停时显示 PAUSED 覆盖层
- GAME OVER 显示得分
- 按 Enter 可重新开始

- [ ] **Step 4: 提交**

```bash
git add index.html
git commit -m "feat: 完善关卡系统和游戏状态显示"
```

---

### Task 8: 细节优化 + 最终验证

**Files:**
- Modify: `index.html`

- [ ] **Step 1: 添加初始鬼魂延迟释放**

修改 `initGhosts()`，让鬼魂依次出屋：

```javascript
function initGhosts() {
  ghosts = [];
  for (let i = 0; i < 4; i++) {
    ghosts.push({
      x: GHOST_STARTS[i].x,
      y: GHOST_STARTS[i].y,
      direction: DIR.UP,
      state: 'scatter',
      color: ghostColors[i],
      inHouse: i > 0,
      releaseDelay: i * 3000 // 每个鬼魂延迟3秒出屋
    });
  }
  ghostMode = 'scatter';
  modeTimer = 0;
}
```

在 `gameLoop` 开头添加鬼魂释放逻辑（在状态判断之前）：

```javascript
  // 释放鬼魂
  if (gameState === STATE.PLAYING) {
    for (const ghost of ghosts) {
      if (ghost.inHouse && ghost.releaseDelay > 0) {
        ghost.releaseDelay -= dt;
        if (ghost.releaseDelay <= 0) {
          ghost.inHouse = false;
          ghost.x = 14;
          ghost.y = 11;
          ghost.direction = DIR.LEFT;
        }
      }
    }
  }
```

- [ ] **Step 2: 添加分数弹出效果**

在 `// ===== 全局游戏状态 =====` 后添加：

```javascript
let floatingTexts = [];

function addFloatingText(text, x, y, color) {
  floatingTexts.push({ text, x: x * TILE, y: y * TILE, color, life: 1000 });
}
```

在吃豆子和吃鬼魂的地方添加浮动文字：

在 Task 5 的吃豆子逻辑中：
```javascript
    addFloatingText('+10', pacman.x, pacman.y, '#FFB8AE');
```

吃能量球：
```javascript
    addFloatingText('+50', pacman.x, pacman.y, '#FFB8AE');
```

吃鬼魂：
```javascript
        addFloatingText('+' + ghostScore, ghost.x, ghost.y, '#0ff');
```

在 `render()` 中 `drawGhost` 循环后添加：

```javascript
  drawFloatingTexts(dt);
```

添加浮动文字渲染函数：

```javascript
function drawFloatingTexts(dt) {
  ctx.textAlign = 'center';
  ctx.font = 'bold 14px Courier New';

  floatingTexts = floatingTexts.filter(ft => {
    ft.life -= dt;
    ft.y -= dt * 0.03;
    const alpha = Math.max(0, ft.life / 1000);
    ctx.fillStyle = ft.color;
    ctx.globalAlpha = alpha;
    ctx.fillText(ft.text, ft.x + TILE / 2, ft.y);
    ctx.globalAlpha = 1;
    return ft.life > 0;
  });
}
```

- [ ] **Step 3: 最终整体验证**

按以下清单逐项验证：

| # | 验收项 | 状态 |
|---|--------|------|
| 1 | 迷宫正确渲染，墙壁不可穿越 | [ ] |
| 2 | 吃豆人可用方向键/WASD控制 | [ ] |
| 3 | 4个鬼魂正常移动并追踪吃豆人 | [ ] |
| 4 | 豆子被吃掉后消失并加分 | [ ] |
| 5 | 能量球使鬼魂变脆弱 | [ ] |
| 6 | 吃豆人被鬼魂碰到后减少生命 | [ ] |
| 7 | 3条命用完显示 GAME OVER | [ ] |
| 8 | 所有豆子吃完进入下一关 | [ ] |
| 9 | 分数显示正确 | [ ] |
| 10 | 空格暂停/继续正常 | [ ] |
| 11 | 浏览器双击即可玩 | [ ] |
| 12 | 吃鬼魂分数递增(200/400/800/1600) | [ ] |
| 13 | 鬼魂被吃后变眼睛回老家复活 | [ ] |
| 14 | 隧道左右穿越正常 | [ ] |

- [ ] **Step 4: 最终提交**

```bash
git add index.html
git commit -m "feat: 添加浮动分数效果和鬼魂延迟释放，完成最终优化"
```

---

## 自审

1. **规范覆盖**: 设计文档中所有需求都有对应 Task：
   - 迷宫渲染 → Task 2
   - 吃豆人控制 → Task 3
   - 鬼魂 AI → Task 4
   - 吃豆子/计分 → Task 5
   - 碰撞/吃鬼魂 → Task 6
   - 关卡/状态 → Task 7
   - 优化 → Task 8

2. **占位符扫描**: 无 TBD/TODO/placeholder

3. **类型一致性**: 所有变量名（`pacman`, `ghosts`, `maze`, `score`）全计划统一

4. **类型签名**: 函数签名一致，无前后矛盾
