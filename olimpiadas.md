---
layout: default
title: Generador de Problemas al Azar
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
.problem-body{padding:24px}
.problem-text{color:#d8d8d8;font-size:15px;line-height:1.85}
.problem-text p{margin:10px 0}
.problem-text p:first-child{margin-top:0}
.sol-toggle{margin-top:20px;padding:9px 18px;background:#252525;border:1px solid #383838;border-radius:6px;color:#999;font-size:13px;cursor:pointer;transition:all .15s}
.sol-toggle:hover{background:#2e2e2e;color:#c0c0c0;border-color:#4a4a4a}
.solution-box{margin-top:16px;padding:18px;background:#161616;border-left:3px solid #3a5a3a;border-radius:0 6px 6px 0;display:none}
.solution-box.show{display:block}
.sol-text{color:#b0b0b0;font-size:14px;line-height:1.85}
.sol-text p{margin:10px 0}
.spinner{display:inline-block;width:14px;height:14px;border:2px solid #4a6a9a;border-top-color:#8ab0d8;border-radius:50%;animation:spin .7s linear infinite;vertical-align:middle;margin-right:7px}
@keyframes spin{to{transform:rotate(360deg)}}
.attempt-hint{font-size:11px;color:#666;margin-top:6px}
</style>

<div class="olim-header">
    <h2>🎲 Generador de Problemas al Azar</h2>
    <p>Selecciona un problema al azar del compendio <strong style="color:#c0c0c0">MathNet</strong> — más de 27,000 problemas de competencias matemáticas internacionales. Filtra por tema y nivel de competencia.</p>
</div>

<div class="filter-section">
    <div class="filter-label">Tema</div>
    <div class="filter-group" id="topicGroup">
        <div class="f-btn on" data-topic="todas">Todas</div>
        <div class="f-btn" data-topic="algebra">Álgebra</div>
        <div class="f-btn" data-topic="geometry">Geometría</div>
        <div class="f-btn" data-topic="combinatorics">Combinatoria</div>
        <div class="f-btn" data-topic="number theory">T. de Números</div>
        <div class="f-btn" data-topic="analysis">Análisis</div>
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

<button class="main-btn" id="randomBtn" onclick="fetchRandomProblem()">
    🎲 Problema aleatorio
</button>
<div class="attempt-hint" id="attemptHint"></div>
<div class="status-msg" id="statusMsg"></div>

<div class="problem-card" id="problemCard">
    <div class="problem-head" id="problemMeta"></div>
    <div class="problem-body">
        <div class="problem-text" id="problemText"></div>
        <button class="sol-toggle" id="solToggle" onclick="toggleSolution()">👁 Ver solución</button>
        <div class="solution-box" id="solutionBox">
            <div class="sol-text" id="solutionText"></div>
        </div>
    </div>
</div>

<script>
(function(){
    var DS         = 'ShadenA/MathNet';
    var BASE       = 'https://datasets-server.huggingface.co';
    var BATCH      = 100;
    var MAX_TRY    = 6;
    var lastId     = null;
    var activeTopic= 'todas';
    var activeDiff = 'todas';

    var _cfg   = 'all';
    var _split = 'train';
    var _total = 27817;

    var DIFF_INTL    = ['imo','usamo','isl','shortlist','putnam'];
    var DIFF_REG     = ['balkan','ibero','apmo','nordic','baltic','benelux','caucasus','mediterran','pan african','south east'];


    function rowsUrl(offset){
        return BASE + '/rows?dataset=' + encodeURIComponent(DS)
             + '&config=' + encodeURIComponent(_cfg)
             + '&split='  + encodeURIComponent(_split)
             + '&offset=' + offset + '&length=' + BATCH;
    }

    document.getElementById('topicGroup').addEventListener('click', function(e){
        var b=e.target.closest('.f-btn'); if(!b) return;
        document.querySelectorAll('#topicGroup .f-btn').forEach(function(x){x.classList.remove('on');});
        b.classList.add('on'); activeTopic=b.dataset.topic;
    });
    document.getElementById('diffGroup').addEventListener('click', function(e){
        var b=e.target.closest('.f-btn'); if(!b) return;
        document.querySelectorAll('#diffGroup .f-btn').forEach(function(x){x.classList.remove('on');});
        b.classList.add('on'); activeDiff=b.dataset.diff;
    });

    function matchesTopic(row){
        if(activeTopic==='todas') return true;
        return JSON.stringify(row.topics_flat||'').toLowerCase().indexOf(activeTopic)!==-1;
    }
    function matchesDiff(row){
        if(activeDiff==='todas') return true;
        var c=(row.competition||'').toLowerCase();
        if(activeDiff==='internacional') return DIFF_INTL.some(function(k){return c.indexOf(k)!==-1;});
        if(activeDiff==='regional')      return DIFF_REG.some(function(k){return c.indexOf(k)!==-1;});
        if(activeDiff==='nacional'){
            return !DIFF_INTL.some(function(k){return c.indexOf(k)!==-1;})
                && !DIFF_REG.some(function(k){return c.indexOf(k)!==-1;});
        }
        return true;
    }

    function setStatus(msg,err){
        var el=document.getElementById('statusMsg');
        el.innerHTML=msg; el.className='status-msg'+(err?' err':'');
    }
    function setLoading(on){
        var btn=document.getElementById('randomBtn');
        btn.disabled=on;
        btn.innerHTML=on?'<span class="spinner"></span>Buscando...':'🎲 Problema aleatorio';
    }

    window.fetchRandomProblem = async function(){
        setLoading(true); setStatus('');
        document.getElementById('solutionBox').classList.remove('show');
        document.getElementById('solToggle').textContent='👁 Ver solución';
        document.getElementById('attemptHint').textContent='';

        var found=null, seen=[];
        for(var attempt=0; attempt<MAX_TRY && !found; attempt++){
            var offset;
            do { offset=Math.floor(Math.random()*Math.max(1,_total-BATCH)); }
            while(seen.indexOf(offset)!==-1);
            seen.push(offset);

            try {
                var r=await fetch(rowsUrl(offset));
                if(!r.ok){ if(attempt===MAX_TRY-1) throw new Error('HTTP '+r.status); continue; }
                var d=await r.json();
                if(!d.rows||!d.rows.length) continue;

                var cands=d.rows.map(function(x){return x.row;}).filter(function(row){
                    return row.problem_markdown && matchesTopic(row) && matchesDiff(row);
                });
                var fresh=cands.filter(function(r){return r.id!==lastId;});
                if(fresh.length) cands=fresh;
                if(cands.length) found=cands[Math.floor(Math.random()*cands.length)];
            } catch(e){
                if(attempt===MAX_TRY-1){
                    setStatus('⚠ Error: '+e.message+'. Verifica tu conexión e intenta de nuevo.',true);
                    setLoading(false); return;
                }
            }
            if(!found && attempt<MAX_TRY-1)
                document.getElementById('attemptHint').textContent='Intento '+(attempt+1)+'/'+MAX_TRY+' — ampliando búsqueda...';
        }

        document.getElementById('attemptHint').textContent='';

        if(!found){
            setStatus('No se encontraron problemas con estos filtros. Prueba una combinación diferente.',true);
            setLoading(false); return;
        }

        lastId=found.id;
        _renderProblem(found,
            document.getElementById('problemText'),
            document.getElementById('solutionText'),
            document.getElementById('problemMeta'));
        document.getElementById('problemCard').className='problem-card show';
        document.getElementById('problemCard').scrollIntoView({behavior:'smooth',block:'start'});
        setLoading(false);
    };

    window.toggleSolution = function(){
        var box=document.getElementById('solutionBox');
        var btn=document.getElementById('solToggle');
        btn.textContent=box.classList.toggle('show')?'🙈 Ocultar solución':'👁 Ver solución';
    };
})();

/* Shared helpers exposed globally for problema-del-dia too */
function _renderProblem(p, textEl, solEl, metaEl){
    if(metaEl){
        var m='';
        if(p.competition) m+='<span class="p-badge comp">🏅 '+_esc(p.competition)+'</span>';
        if(p.country)     m+='<span class="p-badge country">🌍 '+_esc(p.country)+'</span>';
        var ts=p.topics_flat; if(!Array.isArray(ts)) ts=ts?[ts]:[];
        ts.slice(0,3).forEach(function(t){ m+='<span class="p-badge topic">📐 '+_esc(String(t).split('>').pop().trim())+'</span>'; });
        metaEl.innerHTML=m;
    }
    textEl.innerHTML = _md(p.problem_markdown||'');
    if(solEl) solEl.innerHTML = _md(p.solutions_markdown||'Solución no disponible.');
    if(window.MathJax&&MathJax.typesetPromise){
        var els=[textEl]; if(solEl) els.push(solEl);
        MathJax.typesetPromise(els).catch(function(){});
    }
}
function _esc(s){ return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;'); }
function _md(md){
    if(!md) return '';
    var lines=md.split('\n'),out=[],inL=false;
    for(var i=0;i<lines.length;i++){
        var l=lines[i];
        if(/^[-*+]\s/.test(l)||/^\d+\.\s/.test(l)){
            if(!inL){out.push('<ul>');inL=true;}
            out.push('<li>'+_fmt(l.replace(/^[-*+\d.]+\s/,''))+'</li>');
        } else {
            if(inL){out.push('</ul>');inL=false;}
            if(l.trim()==='') out.push('');
            else if(/^#+\s/.test(l)) out.push('<strong>'+_fmt(l.replace(/^#+\s/,''))+'</strong>');
            else out.push('<p>'+_fmt(l)+'</p>');
        }
    }
    if(inL) out.push('</ul>');
    return out.join('\n');
}
function _fmt(s){
    return s.replace(/\*\*(.+?)\*\*/g,'<strong>$1</strong>')
            .replace(/\*(.+?)\*/g,'<em>$1</em>')
            .replace(/`(.+?)`/g,'<code>$1</code>');
}
</script>
