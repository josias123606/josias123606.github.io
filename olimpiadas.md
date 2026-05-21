---
layout: default
title: Olimpiadas
---

<style>
.olim-header{margin-bottom:28px}
.olim-header h2{color:#f0f0f0;font-size:22px;margin-bottom:6px}
.olim-header p{color:#888;font-size:13px;line-height:1.6}
.filter-section{margin-bottom:18px}
.filter-label{font-size:11px;color:#666;text-transform:uppercase;letter-spacing:.8px;margin-bottom:8px;font-weight:600}
.filter-group{display:flex;flex-wrap:wrap;gap:7px}
.f-btn{padding:6px 14px;background:#2a2a2a;border:1px solid #404040;border-radius:20px;color:#a0a0a0;font-size:12px;cursor:pointer;transition:all .15s;user-select:none}
.f-btn:hover{background:#333;color:#d0d0d0;border-color:#555}
.f-btn.on{background:#3a3a3a;border-color:#888;color:#fff;font-weight:600}
.main-btn{margin-top:22px;padding:11px 28px;background:#2e4a6e;border:1px solid #3a6090;border-radius:8px;color:#c8dff5;font-size:14px;font-weight:600;cursor:pointer;transition:all .2s;letter-spacing:.3px}
.main-btn:hover:not(:disabled){background:#3a5e8a;border-color:#4d7ab0;color:#e0efff;transform:translateY(-1px)}
.main-btn:disabled{opacity:.5;cursor:not-allowed;transform:none}
.status-msg{margin-top:14px;font-size:13px;color:#888;min-height:20px}
.status-msg.err{color:#e07070}
.problem-card{margin-top:28px;background:#1e1e1e;border:1px solid #333;border-radius:10px;overflow:hidden;display:none}
.problem-card.show{display:block}
.problem-head{padding:16px 20px;background:#242424;border-bottom:1px solid #2e2e2e;display:flex;flex-wrap:wrap;gap:8px;align-items:center}
.p-badge{display:inline-flex;align-items:center;gap:5px;padding:4px 10px;border-radius:4px;font-size:11px;font-weight:500}
.p-badge.comp{background:#2a3a2a;color:#8ec88e;border:1px solid #3a5a3a}
.p-badge.country{background:#2a2a3a;color:#8e9ec8;border:1px solid #3a3a5a}
.p-badge.topic{background:#3a2a2a;color:#c88e8e;border:1px solid #5a3a3a}
.p-badge.lang{background:#2a3a3a;color:#8eccc8;border:1px solid #3a5a5a}
.problem-body{padding:24px}
.problem-text{color:#d8d8d8;font-size:15px;line-height:1.85}
.problem-text p{margin:12px 0}
.problem-text p:first-child{margin-top:0}
.sol-toggle{margin-top:20px;padding:9px 18px;background:#252525;border:1px solid #383838;border-radius:6px;color:#999;font-size:13px;cursor:pointer;transition:all .15s}
.sol-toggle:hover{background:#2e2e2e;color:#c0c0c0;border-color:#4a4a4a}
.solution-box{margin-top:16px;padding:18px;background:#161616;border-left:3px solid #3a5a3a;border-radius:0 6px 6px 0;display:none}
.solution-box.show{display:block}
.solution-box .sol-text{color:#b0b0b0;font-size:14px;line-height:1.85}
.spinner{display:inline-block;width:14px;height:14px;border:2px solid #4a6a9a;border-top-color:#8ab0d8;border-radius:50%;animation:spin .7s linear infinite;vertical-align:middle;margin-right:7px}
@keyframes spin{to{transform:rotate(360deg)}}
.counter-badge{display:inline-block;background:#1e2e1e;border:1px solid #2e4e2e;border-radius:12px;padding:3px 10px;font-size:11px;color:#78a878;margin-left:auto}
</style>

<div class="olim-header">
    <h2>🏆 Problemas de Olimpiadas</h2>
    <p>Selecciona un problema al azar del compendio <strong style="color:#c0c0c0">MathNet</strong> — más de 27,000 problemas de competencias matemáticas internacionales. Filtra por tema y nivel de competencia.</p>
</div>

<div class="filter-section">
    <div class="filter-label">Tema</div>
    <div class="filter-group" id="topicGroup">
        <div class="f-btn on" data-topic="todas">Todas</div>
        <div class="f-btn" data-topic="Algebra">Álgebra</div>
        <div class="f-btn" data-topic="Geometry">Geometría</div>
        <div class="f-btn" data-topic="Combinatorics">Combinatoria</div>
        <div class="f-btn" data-topic="Number Theory">T. de Números</div>
        <div class="f-btn" data-topic="Analysis">Análisis</div>
    </div>
</div>

<div class="filter-section">
    <div class="filter-label">Nivel de competencia</div>
    <div class="filter-group" id="diffGroup">
        <div class="f-btn on" data-diff="todas">Todas</div>
        <div class="f-btn" data-diff="nacional">Nacional</div>
        <div class="f-btn" data-diff="regional">Regional</div>
        <div class="f-btn" data-diff="internacional">Internacional (IMO / USAMO / APMO)</div>
    </div>
</div>

<button class="main-btn" id="randomBtn" onclick="fetchProblem()">
    🎲 Problema aleatorio
</button>

<div class="status-msg" id="statusMsg"></div>

<div class="problem-card" id="problemCard">
    <div class="problem-head" id="problemMeta"></div>
    <div class="problem-body">
        <div class="problem-text" id="problemText"></div>
        <button class="sol-toggle" id="solToggle" onclick="toggleSolution()">
            👁 Ver solución
        </button>
        <div class="solution-box" id="solutionBox">
            <div class="sol-text" id="solutionText"></div>
        </div>
    </div>
</div>

<script>
(function() {
    var BASE     = 'https://datasets-server.huggingface.co';
    var DS       = 'ShadenA%2FMathNet';
    var CFG      = 'default';
    var SPLIT    = 'train';
    var lastOffset = -1;
    var activeTopic = 'todas';
    var activeDiff  = 'todas';

    var DIFF_WHERE = {
        'nacional':       "(competition NOT LIKE '%IMO%' AND competition NOT LIKE '%APMO%' AND competition NOT LIKE '%USAMO%' AND competition NOT LIKE '%ISL%' AND competition NOT LIKE '%Balkan%' AND competition NOT LIKE '%Ibero%' AND competition NOT LIKE '%Nordic%' AND competition NOT LIKE '%Baltic%' AND competition NOT LIKE '%Benelux%' AND competition NOT LIKE '%Putnam%')",
        'regional':       "(competition LIKE '%Balkan%' OR competition LIKE '%Ibero%' OR competition LIKE '%Nordic%' OR competition LIKE '%Baltic%' OR competition LIKE '%Benelux%' OR competition LIKE '%APMO%' OR competition LIKE '%Caucasus%' OR competition LIKE '%Mediterran%')",
        'internacional':  "(competition LIKE '%IMO%' OR competition LIKE '%USAMO%' OR competition LIKE '%ISL%' OR competition LIKE '%Putnam%' OR competition LIKE '%Shortlist%')"
    };

    // Filter buttons
    document.getElementById('topicGroup').addEventListener('click', function(e) {
        var btn = e.target.closest('.f-btn');
        if (!btn) return;
        document.querySelectorAll('#topicGroup .f-btn').forEach(function(b) { b.classList.remove('on'); });
        btn.classList.add('on');
        activeTopic = btn.getAttribute('data-topic');
    });

    document.getElementById('diffGroup').addEventListener('click', function(e) {
        var btn = e.target.closest('.f-btn');
        if (!btn) return;
        document.querySelectorAll('#diffGroup .f-btn').forEach(function(b) { b.classList.remove('on'); });
        btn.classList.add('on');
        activeDiff = btn.getAttribute('data-diff');
    });

    function buildWhere() {
        var parts = ["language = 'English'"];
        if (activeTopic !== 'todas') {
            parts.push("topics_flat LIKE '%" + activeTopic + "%'");
        }
        if (activeDiff !== 'todas' && DIFF_WHERE[activeDiff]) {
            parts.push(DIFF_WHERE[activeDiff]);
        }
        return parts.join(' AND ');
    }

    function setStatus(msg, isErr) {
        var el = document.getElementById('statusMsg');
        el.innerHTML = msg;
        el.className = 'status-msg' + (isErr ? ' err' : '');
    }

    function setLoading(on) {
        var btn = document.getElementById('randomBtn');
        btn.disabled = on;
        btn.innerHTML = on
            ? '<span class="spinner"></span>Buscando...'
            : '🎲 Problema aleatorio';
    }

    window.fetchProblem = async function() {
        setLoading(true);
        setStatus('');
        document.getElementById('solutionBox').classList.remove('show');
        document.getElementById('solToggle').textContent = '👁 Ver solución';

        try {
            var where = encodeURIComponent(buildWhere());
            var countUrl = BASE + '/filter?dataset=' + DS + '&config=' + CFG + '&split=' + SPLIT
                         + '&where=' + where + '&offset=0&length=1';

            var countResp = await fetch(countUrl);
            if (!countResp.ok) throw new Error('Error al conectar con la API (' + countResp.status + ')');
            var countData = await countResp.json();
            var total = countData.num_rows_total || 0;

            if (total === 0) {
                setStatus('No se encontraron problemas con estos filtros. Intenta una combinación diferente.', true);
                setLoading(false);
                return;
            }

            // Pick a different offset from last time
            var offset;
            var attempts = 0;
            do {
                offset = Math.floor(Math.random() * total);
                attempts++;
            } while (offset === lastOffset && total > 1 && attempts < 10);
            lastOffset = offset;

            var rowUrl = BASE + '/filter?dataset=' + DS + '&config=' + CFG + '&split=' + SPLIT
                       + '&where=' + where + '&offset=' + offset + '&length=1';

            var rowResp = await fetch(rowUrl);
            if (!rowResp.ok) throw new Error('Error al obtener el problema');
            var rowData = await rowResp.json();

            if (!rowData.rows || rowData.rows.length === 0) throw new Error('Respuesta vacía');

            var prob = rowData.rows[0].row;
            displayProblem(prob, offset + 1, total);

        } catch(e) {
            setStatus('⚠ ' + e.message + '. Verifica tu conexión e intenta de nuevo.', true);
        }
        setLoading(false);
    };

    function displayProblem(p, num, total) {
        // Meta badges
        var meta = '';
        if (p.competition) meta += '<span class="p-badge comp">🏅 ' + escHtml(p.competition) + '</span>';
        if (p.country)     meta += '<span class="p-badge country">🌍 ' + escHtml(p.country) + '</span>';
        if (p.topics_flat) {
            var topics = Array.isArray(p.topics_flat) ? p.topics_flat : [p.topics_flat];
            topics.slice(0,3).forEach(function(t) {
                meta += '<span class="p-badge topic">📐 ' + escHtml(String(t).split('>').pop().trim()) + '</span>';
            });
        }
        meta += '<span class="counter-badge">Problema #' + num + ' de ' + total + '</span>';
        document.getElementById('problemMeta').innerHTML = meta;

        // Problem text
        var probEl = document.getElementById('problemText');
        probEl.innerHTML = markdownToHtml(p.problem_markdown || '');
        document.getElementById('problemCard').className = 'problem-card show';

        // Solution
        var solEl = document.getElementById('solutionText');
        solEl.innerHTML = markdownToHtml(p.solutions_markdown || 'Solución no disponible.');
        document.getElementById('solutionBox').classList.remove('show');
        document.getElementById('solToggle').textContent = '👁 Ver solución';

        // Re-render MathJax
        if (window.MathJax && MathJax.typesetPromise) {
            MathJax.typesetPromise([probEl, solEl]).catch(function(){});
        }

        setStatus('');
        document.getElementById('problemCard').scrollIntoView({ behavior: 'smooth', block: 'start' });
    }

    window.toggleSolution = function() {
        var box = document.getElementById('solutionBox');
        var btn = document.getElementById('solToggle');
        var open = box.classList.toggle('show');
        btn.textContent = open ? '🙈 Ocultar solución' : '👁 Ver solución';
    };

    function escHtml(s) {
        return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;');
    }

    // Minimal markdown → HTML (preserves LaTeX, handles bold, lists, paragraphs)
    function markdownToHtml(md) {
        if (!md) return '';
        var lines = md.split('\n');
        var out = [];
        var inList = false;
        for (var i = 0; i < lines.length; i++) {
            var line = lines[i];
            // List item
            if (/^[-*+]\s/.test(line) || /^\d+\.\s/.test(line)) {
                if (!inList) { out.push('<ul>'); inList = true; }
                var item = line.replace(/^[-*+]\s/, '').replace(/^\d+\.\s/, '');
                out.push('<li>' + inlineFormat(item) + '</li>');
            } else {
                if (inList) { out.push('</ul>'); inList = false; }
                if (line.trim() === '') {
                    out.push('');
                } else if (/^#+\s/.test(line)) {
                    out.push('<strong>' + inlineFormat(line.replace(/^#+\s/, '')) + '</strong>');
                } else {
                    out.push('<p>' + inlineFormat(line) + '</p>');
                }
            }
        }
        if (inList) out.push('</ul>');
        // Merge consecutive <p> lines separated by empty lines
        return out.join('\n');
    }

    function inlineFormat(s) {
        // Preserve LaTeX, apply bold/italic
        return s
            .replace(/\*\*(.+?)\*\*/g, '<strong>$1</strong>')
            .replace(/\*(.+?)\*/g, '<em>$1</em>')
            .replace(/`(.+?)`/g, '<code>$1</code>');
    }
})();
</script>
