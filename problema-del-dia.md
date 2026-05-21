---
layout: default
title: Problema del Día
---

<style>
.pdd-intro{color:#888;font-size:13px;margin-bottom:28px;line-height:1.6}
.pdd-section{margin-bottom:36px}
.pdd-section h3{color:#f0f0f0;font-size:17px;margin-bottom:16px;display:flex;align-items:center;gap:8px;justify-content:space-between;flex-wrap:wrap}
.problem-card{background:#1e1e1e;border:1px solid #333;border-radius:10px;overflow:hidden}
.problem-head{padding:16px 20px;background:#242424;border-bottom:1px solid #2e2e2e;display:flex;flex-wrap:wrap;gap:8px;align-items:center}
.p-badge{display:inline-flex;align-items:center;gap:5px;padding:4px 10px;border-radius:4px;font-size:11px;font-weight:500}
.p-badge.comp{background:#2a3a2a;color:#8ec88e;border:1px solid #3a5a3a}
.p-badge.country{background:#2a2a3a;color:#8e9ec8;border:1px solid #3a3a5a}
.p-badge.topic{background:#3a2a2a;color:#c88e8e;border:1px solid #5a3a3a}
.problem-body{padding:24px}
.problem-text{color:#d8d8d8;font-size:15px;line-height:1.85;margin-bottom:16px}
.problem-text p{margin:10px 0}
.problem-text p:first-child{margin-top:0}
.sol-toggle{padding:9px 18px;background:#252525;border:1px solid #383838;border-radius:6px;color:#999;font-size:13px;cursor:pointer;transition:all .15s}
.sol-toggle:hover{background:#2e2e2e;color:#c0c0c0;border-color:#4a4a4a}
.solution-box{margin-top:16px;padding:18px;background:#161616;border-left:3px solid #3a5a3a;border-radius:0 6px 6px 0;display:none}
.solution-box.show{display:block}
.sol-text{color:#b0b0b0;font-size:14px;line-height:1.85}
.sol-text p{margin:10px 0}
.pick-btn{display:inline-flex;align-items:center;gap:6px;padding:8px 14px;background:#2e4a6e;border:1px solid #3a6090;border-radius:6px;color:#c8dff5;font-size:12px;font-weight:600;cursor:pointer;transition:all .2s}
.pick-btn:hover:not(:disabled){background:#3a5e8a;border-color:#4d7ab0;color:#e0efff}
.pick-btn:disabled{opacity:.5;cursor:not-allowed}
.spinner{display:inline-block;width:12px;height:12px;border:2px solid #4a6a9a;border-top-color:#8ab0d8;border-radius:50%;animation:spin .7s linear infinite}
@keyframes spin{to{transform:rotate(360deg)}}
.status-msg{font-size:12px;color:#666;margin-top:8px}
.status-msg.err{color:#e07070}
.loading-msg{color:#666;font-size:13px;padding:20px 0}
</style>

<p class="pdd-intro">
    🌟 Problemas aleatorios del compendio <strong style="color:#c0c0c0">MathNet</strong>. Intenta resolver antes de ver la solución.
</p>

<div class="pdd-section">
    <h3>
        <span>📅 Problema Aleatorio</span>
        <button class="pick-btn" id="pickBtn" onclick="fetchRandomProblem()">🎲 Pick another</button>
    </h3>
    
    <div id="problemCard" style="display:none">
        <div class="problem-head" id="problemMeta"></div>
        <div class="problem-body">
            <div class="problem-text" id="problemText"></div>
            <button class="sol-toggle" id="solToggle" onclick="toggleSolution()">👁 Ver solución</button>
            <div class="solution-box" id="solutionBox">
                <div class="sol-text" id="solutionText"></div>
            </div>
        </div>
    </div>
    
    <div id="loadingMsg" class="loading-msg"><span class="spinner"></span> Cargando primer problema...</div>
    <div class="status-msg" id="statusMsg"></div>
</div>

<script>
var lastProblemId = null;
var isLoading = false;

async function fetchRandomProblem(){
    if(isLoading) return;
    isLoading = true;
    
    var btn = document.getElementById('pickBtn');
    btn.disabled = true;
    btn.innerHTML = '<span class="spinner"></span> Buscando...';
    document.getElementById('statusMsg').innerHTML = '';
    
    try {
        var found = null;
        for(var attempt = 0; attempt < 10; attempt++){
            var offset = Math.floor(Math.random() * 27717);
            var url = 'https://datasets-server.huggingface.co/rows?dataset=ShadenA/MathNet&config=all&split=train&offset=' + offset + '&length=100';
            
            try {
                var response = await fetch(url);
                if(!response.ok) continue;
                
                var data = await response.json();
                if(!data.rows || !data.rows.length) continue;
                
                var problems = data.rows
                    .map(function(x){return x.row;})
                    .filter(function(row){
                        return row.problem_markdown && row.id !== lastProblemId;
                    });
                
                if(problems.length > 0){
                    found = problems[Math.floor(Math.random() * problems.length)];
                    break;
                }
            } catch(e){
                continue;
            }
        }
        
        if(!found) throw new Error('No se encontró problema');
        
        lastProblemId = found.id;
        renderProblem(found);
        document.getElementById('problemCard').style.display = 'block';
        document.getElementById('loadingMsg').style.display = 'none';
        document.getElementById('problemCard').scrollIntoView({behavior:'smooth',block:'start'});
        
    } catch(e) {
        document.getElementById('statusMsg').innerHTML = '⚠ ' + e.message + '. Intenta de nuevo.';
        document.getElementById('statusMsg').classList.add('err');
    } finally {
        btn.disabled = false;
        btn.innerHTML = '🎲 Pick another';
        isLoading = false;
    }
}

function renderProblem(p){
    var metaEl = document.getElementById('problemMeta');
    var m = '';
    if(p.competition) m += '<span class="p-badge comp">🏅 ' + escapeHtml(p.competition) + '</span>';
    if(p.country) m += '<span class="p-badge country">🌍 ' + escapeHtml(p.country) + '</span>';
    var ts = p.topics_flat; 
    if(!Array.isArray(ts)) ts = ts?[ts]:[];
    ts.slice(0,3).forEach(function(t){ 
        m += '<span class="p-badge topic">📐 ' + escapeHtml(String(t).split('>').pop().trim()) + '</span>'; 
    });
    metaEl.innerHTML = m;
    
    document.getElementById('problemText').innerHTML = markdownToHtml(p.problem_markdown || '');
    document.getElementById('solutionText').innerHTML = markdownToHtml(p.solutions_markdown || 'Solución no disponible.');
    document.getElementById('solutionBox').classList.remove('show');
    document.getElementById('solToggle').textContent = '👁 Ver solución';
    
    if(window.MathJax && MathJax.typesetPromise){
        MathJax.typesetPromise([document.getElementById('problemText'), document.getElementById('solutionText')]).catch(function(){});
    }
}

function escapeHtml(s){
    return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;');
}

function markdownToHtml(md){
    if(!md) return '';
    var lines = md.split('\n'), out = [], inL = false;
    for(var i=0; i<lines.length; i++){
        var l = lines[i];
        if(/^[-*+]\s/.test(l) || /^\d+\.\s/.test(l)){
            if(!inL){out.push('<ul>');inL=true;}
            out.push('<li>' + formatText(l.replace(/^[-*+\d.]+\s/,'')) + '</li>');
        } else {
            if(inL){out.push('</ul>');inL=false;}
            if(l.trim()==='') out.push('');
            else if(/^#+\s/.test(l)) out.push('<strong>' + formatText(l.replace(/^#+\s/,'')) + '</strong>');
            else out.push('<p>' + formatText(l) + '</p>');
        }
    }
    if(inL) out.push('</ul>');
    return out.join('\n');
}

function formatText(s){
    return s.replace(/\*\*(.+?)\*\*/g,'<strong>$1</strong>')
            .replace(/\*(.+?)\*/g,'<em>$1</em>')
            .replace(/`(.+?)`/g,'<code>$1</code>');
}

function toggleSolution(){
    var box = document.getElementById('solutionBox');
    var btn = document.getElementById('solToggle');
    btn.textContent = box.classList.toggle('show') ? '🙈 Ocultar solución' : '👁 Ver solución';
}

// Cargar primer problema al entrar
fetchRandomProblem();
</script>
