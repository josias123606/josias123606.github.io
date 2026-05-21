---
layout: default
title: Animaciones
---

<style>
.anim-block{margin-bottom:40px;padding-bottom:40px;border-bottom:1px solid #333}
.anim-block:last-child{border-bottom:none}
.anim-block h3{font-size:18px;color:#f0f0f0;margin-bottom:12px}
.anim-canvas-wrap{position:relative;width:100%;background:#0d0d0d;border-radius:8px;overflow:hidden;border:1px solid #333}
.anim-canvas-wrap canvas{display:block;width:100%;cursor:crosshair}
.anim-controls{display:flex;align-items:center;gap:10px;margin-top:10px;flex-wrap:wrap}
.anim-btn{padding:7px 16px;background:#2a2a2a;border:1px solid #444;border-radius:6px;color:#c0c0c0;font-size:13px;cursor:pointer;transition:background .15s,color .15s}
.anim-btn:hover{background:#383838;color:#e0e0e0}
.anim-btn.active{background:#3a3a3a;border-color:#666;color:#fff}
.anim-status{font-size:12px;color:#666;margin-left:auto}
.speed-wrap{display:flex;align-items:center;gap:8px;font-size:12px;color:#888}
.speed-wrap input[type=range]{accent-color:#888;width:80px}
.anim-desc{margin-top:16px;padding:14px 16px;background:#1e1e1e;border-left:3px solid #444;border-radius:0 6px 6px 0;font-size:13px;color:#a0a0a0;line-height:1.7}
.anim-desc strong{color:#c8c8c8}
.anim-badge{display:inline-block;background:#333;padding:2px 8px;border-radius:4px;font-size:11px;color:#999;margin-bottom:8px}
</style>

<h2>🎬 Animaciones Matemáticas</h2>
<p style="color:#888;margin-bottom:30px;">Cada animación se activa automáticamente cuando la ves y se pausa al salir del área visible, así la página siempre carga rápido sin importar cuántas animaciones haya.</p>

<!-- ═══════════════════════════════════════
     ANIMACIÓN 1 — Juego de la Vida
═══════════════════════════════════════ -->
<div class="anim-block">
    <span class="anim-badge">Sistemas Dinámicos · Autómatas Celulares</span>
    <h3>El Juego de la Vida — John Conway (1970)</h3>

    <div class="anim-canvas-wrap">
        <canvas id="gol-canvas" height="360"></canvas>
    </div>

    <div class="anim-controls">
        <button class="anim-btn active" id="gol-playpause">⏸ Pausar</button>
        <button class="anim-btn" id="gol-random">🎲 Aleatorio</button>
        <button class="anim-btn" id="gol-clear">✕ Limpiar</button>
        <div class="speed-wrap">
            Vel:
            <input type="range" id="gol-speed" min="1" max="30" value="10">
        </div>
        <span class="anim-status" id="gol-status">Dibuja en el tablero · Haz clic o arrastra</span>
    </div>

    <div class="anim-desc">
        <strong>El Juego de la Vida</strong> es un autómata celular inventado por el matemático John Horton Conway en 1970.
        No es un juego en el sentido tradicional — no hay jugadores ni objetivos. Es un sistema determinista que evoluciona
        siguiendo cuatro reglas simples aplicadas a cada celda en una cuadrícula infinita:<br><br>
        1. Una celda viva con <strong>2 ó 3 vecinos vivos</strong> sobrevive.<br>
        2. Una celda viva con <strong>menos de 2</strong> vecinos muere de soledad.<br>
        3. Una celda viva con <strong>más de 3</strong> vecinos muere de superpoblación.<br>
        4. Una celda muerta con <strong>exactamente 3</strong> vecinos vivos nace.<br><br>
        De estas cuatro reglas emerge una complejidad fascinante: patrones que oscilan, se mueven, se reproducen
        y producen comportamientos impredecibles. Conway demostró que el Juego de la Vida es
        <strong>Turing completo</strong> — es decir, capaz de simular cualquier computación posible.
        <strong>Interactúa:</strong> haz clic o arrastra para dibujar células vivas.
    </div>
</div>

<!-- ═══════════════════════════════════════
     Sistema de gestión de animaciones
     Añade más bloques arriba con la misma
     estructura y regístralos abajo.
═══════════════════════════════════════ -->

<script>
/* ─────────────────────────────────────────────
   GESTOR GLOBAL DE ANIMACIONES
   Cada animación se registra con:
     AnimManager.register(canvasId, { init, start, stop })
   El gestor usa IntersectionObserver para:
   - Inicializar la animación la primera vez que es visible
   - Reanudarla cuando entra en el viewport
   - Pausarla cuando sale del viewport
───────────────────────────────────────────── */
var AnimManager = (function() {
    var anims = {};
    var observer = new IntersectionObserver(function(entries) {
        entries.forEach(function(entry) {
            var id = entry.target.id;
            var a = anims[id];
            if (!a) return;
            if (entry.isIntersecting) {
                if (!a.initialized) { a.init(); a.initialized = true; }
                if (!a.paused) a.start();
            } else {
                a.stop();
            }
        });
    }, { threshold: 0.1 });

    return {
        register: function(canvasId, handlers) {
            var el = document.getElementById(canvasId);
            if (!el) return;
            anims[canvasId] = { init: handlers.init, start: handlers.start, stop: handlers.stop, initialized: false, paused: false };
            observer.observe(el);
        },
        setPaused: function(canvasId, paused) {
            if (anims[canvasId]) anims[canvasId].paused = paused;
        }
    };
})();


/* ─────────────────────────────────────────────
   JUEGO DE LA VIDA
───────────────────────────────────────────── */
(function() {
    var canvas = document.getElementById('gol-canvas');
    var ctx = canvas.getContext('2d');
    var CELL = 9;
    var cols, rows, grid, next;
    var raf = null;
    var running = true;
    var fps = 10;
    var lastTime = 0;
    var drawing = false;
    var drawValue = 1;
    var gen = 0;

    function resize() {
        canvas.width = canvas.offsetWidth;
        cols = Math.floor(canvas.width / CELL);
        rows = Math.floor(canvas.height / CELL);
    }

    function makeGrid() {
        return Array.from({length: cols}, function() { return new Uint8Array(rows); });
    }

    function randomize() {
        grid = makeGrid();
        for (var x = 0; x < cols; x++)
            for (var y = 0; y < rows; y++)
                grid[x][y] = Math.random() < 0.3 ? 1 : 0;
        gen = 0;
        updateStatus();
    }

    function clear() {
        grid = makeGrid();
        gen = 0;
        draw();
        updateStatus();
    }

    function countNeighbors(g, x, y) {
        var count = 0;
        for (var dx = -1; dx <= 1; dx++)
            for (var dy = -1; dy <= 1; dy++) {
                if (dx === 0 && dy === 0) continue;
                var nx = (x + dx + cols) % cols;
                var ny = (y + dy + rows) % rows;
                count += g[nx][ny];
            }
        return count;
    }

    function step() {
        next = makeGrid();
        var alive = 0;
        for (var x = 0; x < cols; x++) {
            for (var y = 0; y < rows; y++) {
                var n = countNeighbors(grid, x, y);
                var cell = grid[x][y];
                if (cell === 1) {
                    next[x][y] = (n === 2 || n === 3) ? 1 : 0;
                } else {
                    next[x][y] = (n === 3) ? 1 : 0;
                }
                alive += next[x][y];
            }
        }
        grid = next;
        gen++;
        updateStatus(alive);
    }

    function draw() {
        ctx.fillStyle = '#0d0d0d';
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        for (var x = 0; x < cols; x++) {
            for (var y = 0; y < rows; y++) {
                if (grid[x][y]) {
                    var brightness = 180 + Math.random() * 40;
                    ctx.fillStyle = 'rgb('+Math.round(brightness*.6)+','+Math.round(brightness*.9)+','+brightness+')';
                    ctx.fillRect(x * CELL + 1, y * CELL + 1, CELL - 1, CELL - 1);
                }
            }
        }
    }

    function updateStatus(alive) {
        var s = document.getElementById('gol-status');
        if (s) s.textContent = 'Gen ' + gen + (alive !== undefined ? ' · ' + alive + ' vivas' : '') + ' · ' + cols + '×' + rows;
    }

    function loop(ts) {
        raf = requestAnimationFrame(loop);
        if (!running) { draw(); return; }
        var interval = 1000 / fps;
        if (ts - lastTime < interval) return;
        lastTime = ts;
        step();
        draw();
    }

    function cellAt(e) {
        var r = canvas.getBoundingClientRect();
        var scaleX = canvas.width / r.width;
        var scaleY = canvas.height / r.height;
        var cx = e.touches ? e.touches[0].clientX : e.clientX;
        var cy = e.touches ? e.touches[0].clientY : e.clientY;
        return {
            x: Math.floor((cx - r.left) * scaleX / CELL),
            y: Math.floor((cy - r.top) * scaleY / CELL)
        };
    }

    function toggleCell(e) {
        var c = cellAt(e);
        if (c.x >= 0 && c.x < cols && c.y >= 0 && c.y < rows) {
            grid[c.x][c.y] = drawValue;
            draw();
        }
    }

    canvas.addEventListener('mousedown', function(e) {
        var c = cellAt(e);
        if (c.x >= 0 && c.x < cols && c.y >= 0 && c.y < rows) {
            drawValue = grid[c.x][c.y] ? 0 : 1;
            drawing = true;
            toggleCell(e);
        }
    });
    canvas.addEventListener('mousemove', function(e) { if (drawing) toggleCell(e); });
    canvas.addEventListener('mouseup', function() { drawing = false; });
    canvas.addEventListener('mouseleave', function() { drawing = false; });
    canvas.addEventListener('touchstart', function(e) { e.preventDefault(); drawing = true; toggleCell(e); }, {passive:false});
    canvas.addEventListener('touchmove', function(e) { e.preventDefault(); if (drawing) toggleCell(e); }, {passive:false});
    canvas.addEventListener('touchend', function() { drawing = false; });

    document.getElementById('gol-playpause').addEventListener('click', function() {
        running = !running;
        this.textContent = running ? '⏸ Pausar' : '▶ Reanudar';
        this.classList.toggle('active', running);
        AnimManager.setPaused('gol-canvas', !running);
    });
    document.getElementById('gol-random').addEventListener('click', function() { randomize(); draw(); });
    document.getElementById('gol-clear').addEventListener('click', function() { clear(); });
    document.getElementById('gol-speed').addEventListener('input', function() { fps = parseInt(this.value); });

    AnimManager.register('gol-canvas', {
        init: function() {
            resize();
            grid = makeGrid();
            randomize();
        },
        start: function() {
            if (!raf) raf = requestAnimationFrame(loop);
        },
        stop: function() {
            if (raf) { cancelAnimationFrame(raf); raf = null; }
        }
    });
})();
</script>

<!-- ═══════════════════════════════════════
     ENTRADAS CON CATEGORÍA "Animaciones"
     Se añaden aquí automáticamente
═══════════════════════════════════════ -->
{% assign anim_posts = site.posts | where_exp: "post", "post.categories contains 'Animaciones'" %}
{% if anim_posts.size > 0 %}
<div style="margin-top:40px;padding-top:30px;border-top:2px solid #333">
    <h2 style="color:#f0f0f0;font-size:20px;margin-bottom:6px">📝 Entradas de Animaciones</h2>
    <p style="color:#888;font-size:13px;margin-bottom:24px">Cada entrada incluye la animación interactiva y su explicación matemática.</p>
    <div style="display:grid;grid-template-columns:repeat(auto-fill,minmax(260px,1fr));gap:16px">
        {% for post in anim_posts %}
        <a href="{{ post.url }}" style="text-decoration:none;display:block;background:#2a2a2a;border:1px solid #383838;border-radius:8px;padding:20px;transition:border-color .2s,background .2s" onmouseover="this.style.borderColor='#555';this.style.background='#303030'" onmouseout="this.style.borderColor='#383838';this.style.background='#2a2a2a'">
            <p style="font-size:11px;color:#666;margin-bottom:8px">{{ post.date | date: "%d de %B, %Y" }}</p>
            <h3 style="color:#e0e0e0;font-size:15px;margin-bottom:10px;line-height:1.4">{{ post.title }}</h3>
            <p style="color:#888;font-size:12px;line-height:1.5;margin-bottom:12px">{{ post.excerpt | strip_html | truncatewords: 18 }}</p>
            <div style="display:flex;flex-wrap:wrap;gap:5px">
                {% for cat in post.categories %}
                <span style="background:#333;padding:2px 8px;border-radius:4px;font-size:11px;color:#999">{{ cat }}</span>
                {% endfor %}
            </div>
        </a>
        {% endfor %}
    </div>
</div>
{% endif %}
