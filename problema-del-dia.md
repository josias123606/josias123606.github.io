---
layout: default
title: Problema del Día
---

<style>
.pdd-intro{color:#888;font-size:13px;margin-bottom:28px;line-height:1.6}
.pdd-section{margin-bottom:36px}
.pdd-section-title{display:flex;align-items:center;gap:10px;margin-bottom:16px;padding-bottom:12px;border-bottom:1px solid #2a2a2a;flex-wrap:wrap}
.pdd-section-title h3{color:#f0f0f0;font-size:17px;margin:0}
.date-badge{padding:4px 12px;background:#2a2a2a;border:1px solid #404040;border-radius:12px;font-size:11px;color:#888}
.date-badge.today{background:#1e2e1e;border-color:#3a5a3a;color:#78a878}
.countdown{font-size:11px;color:#555;margin-left:4px}
.countdown span{color:#778}
.problem-card{background:#1e1e1e;border:1px solid #333;border-radius:10px;overflow:hidden}
.problem-head{padding:14px 20px;background:#242424;border-bottom:1px solid #2e2e2e;display:flex;flex-wrap:wrap;gap:8px;align-items:center}
.p-badge{display:inline-flex;align-items:center;gap:5px;padding:4px 10px;border-radius:4px;font-size:11px;font-weight:500}
.p-badge.comp{background:#2a3a2a;color:#8ec88e;border:1px solid #3a5a3a}
.p-badge.country{background:#2a2a3a;color:#8e9ec8;border:1px solid #3a3a5a}
.p-badge.topic{background:#3a2a2a;color:#c88e8e;border:1px solid #5a3a3a}
.problem-body{padding:22px}
.problem-text{color:#d8d8d8;font-size:15px;line-height:1.85}
.problem-text p,.sol-text p{margin:10px 0}
.problem-text p:first-child{margin-top:0}
.sol-toggle{margin-top:18px;padding:9px 18px;background:#252525;border:1px solid #383838;border-radius:6px;color:#999;font-size:13px;cursor:pointer;transition:all .15s}
.sol-toggle:hover{background:#2e2e2e;color:#c0c0c0;border-color:#4a4a4a}
.sol-box{margin-top:14px;padding:18px;background:#161616;border-left:3px solid #3a5a3a;border-radius:0 6px 6px 0;display:none}
.sol-box.show{display:block}
.sol-text{color:#b0b0b0;font-size:14px;line-height:1.85}
.spinner{display:inline-block;width:14px;height:14px;border:2px solid #4a6a9a;border-top-color:#8ab0d8;border-radius:50%;animation:spin .7s linear infinite;vertical-align:middle;margin-right:6px}
@keyframes spin{to{transform:rotate(360deg)}}
.loading-ph{color:#666;font-size:14px;padding:20px 0}
.divider{border:none;border-top:1px solid #222;margin:32px 0}
</style>

<p class="pdd-intro">
    Cada día a las 12:00 AM se publica un nuevo problema del compendio <strong style="color:#c0c0c0">MathNet</strong>.
    Intenta resolver el problema de hoy antes de ver la solución.
    Debajo encontrarás el problema de ayer con su enunciado y solución completa.
</p>

<div class="pdd-section">
    <div class="pdd-section-title">
        <h3>📅 Problema de hoy</h3>
        <span class="date-badge today" id="todayDate"></span>
        <span class="countdown" id="countdown"></span>
    </div>
    <div id="todayCard"><div class="loading-ph"><span class="spinner"></span>Cargando problema de hoy...</div></div>
</div>

<hr class="divider">

<div class="pdd-section">
    <div class="pdd-section-title">
        <h3>📖 Problema de ayer</h3>
        <span class="date-badge" id="yesterdayDate"></span>
    </div>
    <p style="font-size:13px;color:#666;margin-bottom:14px">Enunciado y solución completa.</p>
    <div id="yesterdayCard"><div class="loading-ph"><span class="spinner"></span>Cargando problema de ayer...</div></div>
</div>

<script>
(function(){
    var DS   = 'ShadenA/MathNet';
    var BASE = 'https://datasets-server.huggingface.co';
    var _cfg='all', _split='train', _total=27817;

    var MONTHS = ['enero','febrero','marzo','abril','mayo','junio',
                  'julio','agosto','septiembre','octubre','noviembre','diciembre'];
    var WDAYS  = ['domingo','lunes','martes','miércoles','jueves','viernes','sábado'];

    function fmtDate(d){
        return WDAYS[d.getDay()]+' '+d.getDate()+' de '+MONTHS[d.getMonth()]+', '+d.getFullYear();
    }

    /* Deterministic row for a given day offset (0=today, 1=yesterday).
       Uses prime multiplication + per-retry shift so each retry is also deterministic. */
    function seedIdx(daysAgo, retry){
        var d=new Date(); d.setHours(0,0,0,0);
        var dayNum=Math.floor(d.getTime()/86400000)-daysAgo;
        return ((dayNum*6271+8191+retry*1481)%_total+_total)%_total;
    }


    function rowUrl(offset){
        return BASE+'/rows?dataset='+encodeURIComponent(DS)
              +'&config='+encodeURIComponent(_cfg)
              +'&split='+encodeURIComponent(_split)
              +'&offset='+offset+'&length=1';
    }

    async function fetchDayProblem(daysAgo){
        for(var i=0;i<12;i++){
            var idx=seedIdx(daysAgo,i);
            try {
                var r=await fetch(rowUrl(idx));
                if(!r.ok) continue;
                var d=await r.json();
                if(!d.rows||!d.rows.length) continue;
                var row=d.rows[0].row;
                if(row.language==='English'&&row.problem_markdown) return row;
            } catch(e){ continue; }
        }
        /* Fallback: accept any language */
        for(var j=0;j<5;j++){
            var idx2=seedIdx(daysAgo,j+20);
            try {
                var r3=await fetch(rowUrl(idx2));
                if(!r3.ok) continue;
                var d3=await r3.json();
                if(d3.rows&&d3.rows.length&&d3.rows[0].row.problem_markdown) return d3.rows[0].row;
            } catch(e){ continue; }
        }
        throw new Error('No se pudo cargar el problema');
    }

    function renderCard(containerId, prob, showSol){
        var c=document.getElementById(containerId);
        var solId=containerId+'_sol', btnId=containerId+'_btn';

        var meta='';
        if(prob.competition) meta+='<span class="p-badge comp">🏅 '+esc(prob.competition)+'</span>';
        if(prob.country)     meta+='<span class="p-badge country">🌍 '+esc(prob.country)+'</span>';
        var ts=prob.topics_flat; if(!Array.isArray(ts)) ts=ts?[ts]:[];
        ts.slice(0,3).forEach(function(t){ meta+='<span class="p-badge topic">📐 '+esc(String(t).split('>').pop().trim())+'</span>'; });

        var solBtn='<button class="sol-toggle" id="'+btnId+'" onclick="pddToggle(\''+solId+'\',\''+btnId+'\')">'
                  +(showSol?'🙈 Ocultar solución':'👁 Ver solución')+'</button>';
        var solDiv='<div class="sol-box'+(showSol?' show':'')+'" id="'+solId+'">'
                  +'<div class="sol-text" id="'+solId+'_txt">'+pddMd(prob.solutions_markdown||'Solución no disponible.')+'</div></div>';

        c.innerHTML='<div class="problem-card">'
            +'<div class="problem-head">'+meta+'</div>'
            +'<div class="problem-body">'
            +'<div class="problem-text" id="'+containerId+'_txt">'+pddMd(prob.problem_markdown)+'</div>'
            +solBtn+solDiv
            +'</div></div>';

        if(window.MathJax&&MathJax.typesetPromise){
            var els=[document.getElementById(containerId+'_txt'),document.getElementById(solId+'_txt')].filter(Boolean);
            MathJax.typesetPromise(els).catch(function(){});
        }
    }

    window.pddToggle=function(boxId,btnId){
        var box=document.getElementById(boxId),btn=document.getElementById(btnId);
        if(!box||!btn) return;
        btn.textContent=box.classList.toggle('show')?'🙈 Ocultar solución':'👁 Ver solución';
    };

    /* Date labels */
    var today=new Date(); today.setHours(0,0,0,0);
    var yesterday=new Date(today); yesterday.setDate(yesterday.getDate()-1);
    document.getElementById('todayDate').textContent=fmtDate(today);
    document.getElementById('yesterdayDate').textContent=fmtDate(yesterday);

    /* Countdown */
    function tick(){
        var now=new Date(), nxt=new Date(now); nxt.setHours(24,0,0,0);
        var s=Math.floor((nxt-now)/1000);
        var h=String(Math.floor(s/3600)).padStart(2,'0');
        var m=String(Math.floor(s%3600/60)).padStart(2,'0');
        var sc=String(s%60).padStart(2,'0');
        document.getElementById('countdown').innerHTML='Próximo problema en <span>'+h+':'+m+':'+sc+'</span>';
    }
    tick(); setInterval(tick,1000);

    /* Auto-reload at midnight */
    (function(){
        var now=new Date(), nxt=new Date(now); nxt.setHours(24,0,0,0);
        setTimeout(function(){location.reload();},nxt-now);
    })();

    /* Load problems */
    (async function(){
        /* Load today and yesterday in parallel */
        var todayP     = fetchDayProblem(0);
        var yesterdayP = fetchDayProblem(1);

        todayP.then(function(p){ renderCard('todayCard',p,false); })
              .catch(function(){ document.getElementById('todayCard').innerHTML='<p style="color:#e07070;font-size:13px">⚠ No se pudo cargar el problema de hoy. Intenta recargar la página.</p>'; });

        yesterdayP.then(function(p){ renderCard('yesterdayCard',p,true); })
                  .catch(function(){ document.getElementById('yesterdayCard').innerHTML='<p style="color:#e07070;font-size:13px">⚠ No se pudo cargar el problema de ayer.</p>'; });
    })();

    function esc(s){ return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;'); }
    function pddMd(md){
        if(!md) return '';
        var lines=md.split('\n'),out=[],inL=false;
        for(var i=0;i<lines.length;i++){
            var l=lines[i];
            if(/^[-*+]\s/.test(l)||/^\d+\.\s/.test(l)){
                if(!inL){out.push('<ul>');inL=true;}
                out.push('<li>'+pf(l.replace(/^[-*+\d.]+\s/,''))+'</li>');
            } else {
                if(inL){out.push('</ul>');inL=false;}
                if(l.trim()==='') out.push('');
                else if(/^#+\s/.test(l)) out.push('<strong>'+pf(l.replace(/^#+\s/,''))+'</strong>');
                else out.push('<p>'+pf(l)+'</p>');
            }
        }
        if(inL) out.push('</ul>');
        return out.join('\n');
    }
    function pf(s){
        return s.replace(/\*\*(.+?)\*\*/g,'<strong>$1</strong>')
                .replace(/\*(.+?)\*/g,'<em>$1</em>')
                .replace(/`(.+?)`/g,'<code>$1</code>');
    }
})();
</script>
