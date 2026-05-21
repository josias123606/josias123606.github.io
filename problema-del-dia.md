---
layout: default
title: Problema del Día
---

<script>
window.MathJax = {
  tex: {
    inlineMath: [['$', '$'], ['\\(', '\\)']],
    displayMath: [['$$', '$$'], ['\\[', '\\]']],
    processEscapes: true
  }
};
</script>
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>

<style>
.pdd-intro{color:#888;font-size:13px;margin-bottom:20px;line-height:1.6}
.pdd-section{margin-bottom:36px}

/* Contenedor del cronómetro */
.timer-container {
    background: #1a2636;
    border: 1px solid #2e4a6e;
    border-radius: 8px;
    padding: 12px 16px;
    margin-bottom: 24px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-size: 13px;
    color: #c8dff5;
}
.timer-clock {
    font-family: monospace;
    font-size: 16px;
    font-weight: bold;
    color: #8ab0d8;
    letter-spacing: 0.5px;
}

/* Control de Pestañas (Hoy / Ayer) */
.pdd-tabs {
    display: flex;
    gap: 8px;
    border-bottom: 1px solid #333;
    margin-bottom: 20px;
}
.pdd-tab {
    padding: 10px 20px;
    color: #888;
    font-size: 13px;
    font-weight: 600;
    cursor: pointer;
    background: none;
    border: none;
    border-bottom: 2px solid transparent;
    transition: all 0.15s;
}
.pdd-tab:hover { color: #d0d0d0; }
.pdd-tab.active {
    color: #fff;
    border-bottom-color: #3a6090;
}

/* Selector de Idioma Optimizado */
.lang-selector {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    font-size: 12px;
    color: #aaa;
    background: #1c1c1c;
    border: 1px solid #383838;
    padding: 4px 10px;
    border-radius: 6px;
    height: 28px;
}
.lang-selector select {
    background: transparent;
    border: none;
    color: #e0e0e0;
    font-size: 12px;
    font-weight: 600;
    cursor: pointer;
    outline: none;
    padding-right: 4px;
}
.lang-selector select option {
    background: #1e1e1e;
    color: #fff;
}

/* Bloqueo estricto de selección de texto */
.no-select {
    user-select: none !important;
    -webkit-user-select: none !important;
    -moz-user-select: none !important;
    -ms-user-select: none !important;
}

/* Estructura de tarjetas */
.problem-card{background:#1e1e1e;border:1px solid #333;border-radius:10px;overflow:hidden}
.problem-head{padding:12px 20px;background:#242424;border-bottom:1px solid #2e2e2e;display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:12px}
.meta-badges {display:flex;flex-wrap:wrap;gap:8px;align-items:center}
.p-badge{display:inline-flex;align-items:center;gap:5px;padding:4px 10px;border-radius:4px;font-size:11px;font-weight:500}
.p-badge.comp{background:#2a3a2a;color:#8ec88e;border:1px solid #3a5a3a}
.p-badge.country{background:#2a2a3a;color:#8e9ec8;border:1px solid #3a3a5a}
.p-badge.topic{background:#3a2a2a;color:#c88e8e;border:1px solid #5a3a3a}
.p-badge.locked {background:#222;color:#666;border:1px solid #333}
.problem-body{padding:24px}
.problem-text{color:#d8d8d8;font-size:15px;line-height:1.85}
.problem-text p{margin:10px 0}
.problem-text p:first-child{margin-top:0}

/* Caja de Soluciones */
.sol-toggle{margin-top: 20px; padding:9px 18px;background:#252525;border:1px solid #383838;border-radius:6px;color:#999;font-size:13px;cursor:pointer;transition:all .15s}
.sol-toggle:hover{background:#2e2e2e;color:#c0c0c0;border-color:#4a4a4a}
.solution-box{margin-top:16px;padding:18px;background:#161616;border-left:3px solid #3a5a3a;border-radius:0 6px 6px 0;display:none}
.solution-box.show{display:block}
.sol-text{color:#b0b0b0;font-size:14px;line-height:1.85}
.sol-text p{margin:10px 0}

.spinner{display:inline-block;width:14px;height:14px;border:2px solid #4a6a9a;border-top-color:#8ab0d8;border-radius:50%;animation:spin .7s linear infinite;vertical-align:middle;margin-right:7px}
@keyframes spin{to{transform:rotate(360deg)}}
.status-msg{font-size:13px;color:#888;margin-top:14px;min-height:20px}
.status-msg.err{color:#e07070}
.loading-msg{color:#666;font-size:13px;padding:20px 0}
</style>

<p class="pdd-intro">
    🌟 Problemas diarios obtenidos del compendio <strong style="color:#c0c0c0">MathNet</strong>. Intenta resolverlos por tu cuenta usando papel y lapiź antes de que se publique la solución oficial.
</p>

<div class="timer-container">
    <span>⏳ Tiempo restante para el próximo reto:</span>
    <span class="timer-clock" id="dailyTimer">00:00:00</span>
</div>

<div class="pdd-tabs">
    <button class="pdd-tab active" id="tabToday" onclick="switchTab('today')">📅 Problema de Hoy</button>
    <button class="pdd-tab" id="tabYesterday" onclick="switchTab('yesterday')">📜 Ver Problema de Ayer (y Solución)</button>
</div>

<div class="pdd-section">
    <div id="problemCard" style="display:none">
        <div class="problem-head">
            <div class="meta-badges" id="problemMeta"></div>
            <div class="lang-selector">
                🌐 <select id="langSelect" onchange="changeLanguage(this.value)">
                    <option value="original">Original</option>
                    <option value="es">Español</option>
                </select>
            </div>
        </div>
        <div class="problem-body">
            <div id="problemTextContainer">
                <div class="problem-text" id="problemText"></div>
            </div>
            
            <div id="solutionArea" style="display:none">
                <button class="sol-toggle" id="solToggle" onclick="toggleSolution()">👁 Ver solución</button>
                <div class="solution-box" id="solutionBox">
                    <div class="sol-text" id="solutionText"></div>
                </div>
            </div>
        </div>
    </div>
    
    <div id="loadingMsg" class="loading-msg"><span class="spinner"></span> Cargando panel diario...</div>
    <div class="status-msg" id="statusMsg"></div>
</div>

<script>
(function(){
    var DS = 'ShadenA/MathNet';
    var BASE = 'https://datasets-server.huggingface.co';
    var currentMode = 'today'; 
    var currentLang = 'original';
    var cachedData = { today: null, yesterday: null };
    var translationCache = {}; // Almacena traducciones para no repetir peticiones API

    function startTimer() {
        var clock = document.getElementById('dailyTimer');
        setInterval(function(){
            var now = new Date();
            var midnight = new Date(now.getFullYear(), now.getMonth(), now.getDate() + 1, 0, 0, 0);
            var diff = midnight - now;

            if (diff <= 0) {
                location.reload();
                return;
            }

            var hours = String(Math.floor(diff / 3600000)).padStart(2, '0');
            var minutes = String(Math.floor((diff % 3600000) / 60000)).padStart(2, '0');
            var seconds = String(Math.floor((diff % 60000) / 1000)).padStart(2, '0');
            clock.textContent = hours + ":" + minutes + ":" + seconds;
        }, 1000);
    }

    function getIndicesForDate(date) {
        var y = date.getFullYear();
        var m = String(date.getMonth() + 1).padStart(2, '0');
        var d = String(date.getDate()).padStart(2, '0');
        var dateStr = y + '-' + m + '-' + d;

        var hash = 0;
        for (var i = 0; i < dateStr.length; i++) {
            hash = dateStr.charCodeAt(i) + ((hash << 5) - hash);
        }
        hash = Math.abs(hash);

        var offset = hash % 27500; 
        var itemIndex = (hash >> 3) % 40;
        return { offset: offset, index: itemIndex };
    }

    window.switchTab = function(mode) {
        if(mode === currentMode) return;
        currentMode = mode;
        
        document.getElementById('tabToday').classList.toggle('active', mode === 'today');
        document.getElementById('tabYesterday').classList.toggle('active', mode === 'yesterday');
        
        // Resetear selector de idioma al cambiar de pestaña si lo deseas, o mantener el estado
        renderCurrentState();
    };

    window.changeLanguage = function(lang) {
        if(lang === currentLang) return;
        currentLang = lang;
        renderCurrentState();
    };

    async function loadDailyPanels() {
        var now = new Date();
        var yesterday = new Date(); 
        yesterday.setDate(now.getDate() - 1);

        var todayParams = getIndicesForDate(now);
        var yesterdayParams = getIndicesForDate(yesterday);

        try {
            var rToday = await fetch(BASE + '/rows?dataset='+encodeURIComponent(DS)+'&config=all&split=train&offset='+todayParams.offset+'&length=100');
            if(rToday.ok) {
                var dToday = await rToday.json();
                if(dToday.rows && dToday.rows.length > todayParams.index) {
                    cachedData.today = dToday.rows[todayParams.index].row;
                }
            }

            var rYest = await fetch(BASE + '/rows?dataset='+encodeURIComponent(DS)+'&config=all&split=train&offset='+yesterdayParams.offset+'&length=100');
            if(rYest.ok) {
                var dYest = await rYest.json();
                if(dYest.rows && dYest.rows.length > yesterdayParams.index) {
                    cachedData.yesterday = dYest.rows[yesterdayParams.index].row;
                }
            }

            document.getElementById('loadingMsg').style.display = 'none';
            document.getElementById('problemCard').style.display = 'block';
            renderCurrentState();

        } catch(e) {
            document.getElementById('loadingMsg').style.display = 'none';
            document.getElementById('statusMsg').innerHTML = '⚠ Error al sincronizar el servidor de problemas: ' + e.message;
            document.getElementById('statusMsg').className = 'status-msg err';
        }
    }

    // Procesa la traducción asíncrona aislando bloques matemáticos para que no se arruinen
    async function fetchTranslation(text, cacheKey) {
        if (!text) return '';
        
        var mathBlocks = [];
        // Aislamos tanto $$...$$ como $...$ en un único arreglo ordenado
        var placeholderMd = text.replace(/\$\$[\s\S]*?\$\$|\$[\s\S]*?\$/g, function(match) {
            mathBlocks.push(match);
            return ' ___MATH' + (mathBlocks.length - 1) + '___ ';
        });

        try {
            var url = 'https://api.mymemory.translated.net/get?q=' + encodeURIComponent(placeholderMd) + '&langpair=en|es';
            var res = await fetch(url);
            if(!res.ok) throw new Error();
            var data = await res.json();
            var translated = data.responseData.translatedText;

            // Restauramos las fórmulas matemáticas originales en sus posiciones exactas
            for(var i = 0; i < mathBlocks.length; i++) {
                // Soportamos posibles variaciones de espacios creadas por el traductor en el token
                var rx = new RegExp('___ ?MATH' + i + ' ?___', 'g');
                translated = translated.replace(rx, mathBlocks[i]);
            }
            
            translationCache[cacheKey] = translated;
            return translated;
        } catch(e) {
            return text; // Si falla la API por cuota o red, muestra el original sin romper la app
        }
    }

    function renderCurrentState() {
        var p = (currentMode === 'today') ? cachedData.today : cachedData.yesterday;
        var textContainer = document.getElementById('problemTextContainer');
        var textEl = document.getElementById('problemText');
        var solEl = document.getElementById('solutionText');
        var metaEl = document.getElementById('problemMeta');
        var solArea = document.getElementById('solutionArea');

        if(!p) {
            textEl.innerHTML = "<p>Problema temporalmente no disponible en este bloque de datos.</p>";
            metaEl.innerHTML = "";
            solArea.style.display = "none";
            return;
        }

        document.getElementById('solutionBox').classList.remove('show');
        document.getElementById('solToggle').textContent = '👁 Ver solución';

        if(currentMode === 'today') {
            metaEl.innerHTML = '<span class="p-badge locked">🔒 Fuente y Origen: Desbloqueables en 24 horas</span>';
            textContainer.className = "no-select";
            textContainer.oncontextmenu = function(e){ e.preventDefault(); return false; };
            textContainer.oncopy = function(e){ e.preventDefault(); return false; };
            textContainer.onkeydown = function(e){
                if(e.ctrlKey && (e.keyCode === 67 || e.keyCode === 65 || e.keyCode === 85)) { e.preventDefault(); return false; }
            };
            solArea.style.display = 'none';
        } else {
            var m = '';
            if(p.competition) m += '<span class="p-badge comp">🏅 ' + _esc(p.competition) + '</span>';
            if(p.country) m += '<span class="p-badge country">🌍 ' + _esc(p.country) + '</span>';
            var ts = p.topics_flat; 
            if(!Array.isArray(ts)) ts = ts ? [ts] : [];
            ts.slice(0,3).forEach(function(t){ 
                m += '<span class="p-badge topic">📐 ' + _esc(String(t).split('>').pop().trim()) + '</span>'; 
            });
            metaEl.innerHTML = m;

            textContainer.className = "";
            textContainer.oncontextmenu = null;
            textContainer.oncopy = null;
            textContainer.onkeydown = null;

            solArea.style.display = 'block';
            
            var rawSol = p.solutions_markdown;
            var solContent = "";
            if (Array.isArray(rawSol) && rawSol.length > 0) {
                solContent = rawSol.join('\n\n---\n\n').trim();
            } else if (typeof rawSol === 'string' && rawSol.trim() !== '') {
                solContent = rawSol.trim();
            }
            if (!solContent) {
                solContent = '*Este problema específico no cuenta con una solución almacenada en el registro.* 😔';
            }
            solEl.innerHTML = _md(solContent);
        }

        // CONTROL DE RENDERIZADO SEGÚN EL IDIOMA SELECCIONADO
        var rawProblemMd = p.problem_markdown || '';
        var cacheKey = p.id + '_' + currentMode;

        if (currentLang === 'es') {
            if (translationCache[cacheKey]) {
                textEl.innerHTML = _md(translationCache[cacheKey]);
                runMathJax();
            } else {
                textEl.innerHTML = '<p><span class="spinner"></span> Traduciendo enunciado de forma segura...</p>';
                fetchTranslation(rawProblemMd, cacheKey).then(function(translatedText) {
                    // Verificación de concurrencia: que el usuario no haya cambiado de vista mientras cargaba
                    var activeP = (currentMode === 'today') ? cachedData.today : cachedData.yesterday;
                    if(activeP && activeP.id === p.id && currentLang === 'es') {
                        textEl.innerHTML = _md(translatedText);
                        runMathJax();
                    }
                });
            }
        } else {
            textEl.innerHTML = _md(rawProblemMd);
            runMathJax();
        }

        function runMathJax() {
            if(window.MathJax && MathJax.typesetPromise){
                var targets = [textEl];
                if(currentMode === 'yesterday') targets.push(solEl);
                MathJax.typesetPromise(targets).catch(function(err){});
            }
        }
    }

    window.toggleSolution = function(){
        var box = document.getElementById('solutionBox');
        var btn = document.getElementById('solToggle');
        btn.textContent = box.classList.toggle('show') ? '🙈 Ocultar solución' : '👁 Ver solución';
    };

    startTimer();
    loadDailyPanels();
})();

function _esc(s){ return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;'); }

function _md(md){
    if(!md) return '';
    var mathBlocks = [];
    md = md.replace(/\$\$([\s\S]*?)\$\$/g, function(match) {
        mathBlocks.push(match);
        return '\n%%%MATH_BLOCK_' + (mathBlocks.length - 1) + '%%%\n';
    });

    var lines=md.split('\n'),out=[],inL=false;
    for(var i=0;i<lines.length;i++){
        var l=lines[i];
        if(l.trim()==='') {
            if(inL){out.push('</ul>');inL=false;}
            out.push('');
            continue;
        }
        if(/^[-*+]\s/.test(l)||/^\d+\.\s/.test(l)){
            if(!inL){out.push('<ul>');inL=true;}
            out.push('<li>'+_fmt(l.replace(/^[-*+\d.]+\s/,''))+'</li>');
        } else if(l.indexOf('%%%MATH_BLOCK_') !== -1) {
            if(inL){out.push('</ul>');inL=false;}
            out.push(l); 
        } else {
            if(inL){out.push('</ul>');inL=false;}
            if(/^#+\s/.test(l)) out.push('<strong>'+_fmt(l.replace(/^#+\s/,''))+'</strong>');
            else out.push('<p>'+_fmt(l)+'</p>');
        }
    }
    if(inL) out.push('</ul>');
    var result = out.join('\n');

    for(var j=0; j<mathBlocks.length; j++){
        result = result.replace('%%%MATH_BLOCK_' + j + '%%%', mathBlocks[j]);
    }
    return result;
}

function _fmt(s){
    var inlineMath = [];
    s = s.replace(/\$([\s\S]*?)\$/g, function(match) {
        inlineMath.push(match);
        return '%%%INLINE_MATH_' + (inlineMath.length - 1) + '%%%';
    });

    s = s.replace(/\*\*(.+?)\*\*/g,'<strong>$1</strong>')
         .replace(/\*(.+?)\*/g,'<em>$1</em>')
         .replace(/`(.+?)`/g,'<code>$1</code>');

    for(var j=0; j<inlineMath.length; j++){
        s = s.replace('%%%INLINE_MATH_' + j + '%%%', inlineMath[j]);
    }
    return s;
}
</script>
