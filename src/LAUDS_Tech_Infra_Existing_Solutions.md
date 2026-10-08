# LAUDS Tech-Infra Support – Existing Solutions

<pre class="mermaid">
flowchart TD
    A["Trigger: Need, Problem or Opportunity Identified"]
    A --> B["Input Collection from Users &amp; Team"]
    B --> C["Contextual Understanding (Observation, Experience, Trends)"]
    C --> D["Research &amp; Exploration of Available Tools"]
    D --> E["Consolidation of Options (Filtering &amp; Shortlisting)"]
    E --> F["Evaluation Based on Shared Criteria"]
    F --> G["Decision-Making (Team, Community, Org-Level)"]
    G --> H["Acquisition: Buy, Build, Receive or Repair"]
    H --> I["Integration into Workshop (Installation, Training)"]
    I --> J["Evaluation in Practice (Feedback, Usage, Fit)"]
    J --> K["Continuous Learning &amp; Process Adaptation"]
    K --> A
</pre>

---

<div class="steps">

| Step | Description |
| --- | --- |
| 1. Trigger | A need arises from practical use, strategic objectives, or new conceptual inputs. |
| 2-3. Input & Context | Feedback and direct observation contribute to a contextualized understanding. |
| 4-5. Research | Potential solutions are identified, explored, and consolidated. |
| 6. Evaluation | Options are assessed based on collaboratively defined selection criteria. |
| 7. Decision | Decisions are made transparently, either collectively or by designated actors. |
| 8. Acquisition | Tools are procured through purchase, repair, donation, or in-house development. |
| 9. Integration | Implementation includes technical setup, user onboarding, and spatial alignment. |
| 10. Practical Testing | Tools are applied in everyday practice; user feedback is collected. |
| 11. Learning | Insights gained feed back into the process for future iterations. |

</div>

## 2. Tool Categories

<style>
.lauds .tools{display:flex;flex-wrap:wrap;gap:8px;margin:10px 0;align-items:center}
.lauds .tools input,.lauds .tools select,.lauds .tools button{padding:7px 10px;border:1px solid #bbb;border-radius:6px;font:inherit}
.lauds .tools input{flex:1;min-width:200px}
.lauds .cnt{color:#666;font-size:.9em}
.lauds .q{display:none!important}
.lauds{width:100%;max-width:none}
.lauds .wrap{overflow-x:auto;max-height:75vh;border:1px solid #ccc;border-radius:8px;width:100%;margin:0 auto}
.lauds table{border-collapse:collapse;width:max-content;min-width:100%;font-size:.88em;display:table}
.lauds th{position:sticky;top:0;background:#f0f0f0;color:#222;text-align:left;padding:8px;cursor:pointer;white-space:nowrap;border-bottom:2px solid #ccc}
.lauds td{padding:6px 8px;border-top:1px solid #ddd;vertical-align:top}
.lauds tbody tr:nth-child(even){background:rgba(128,128,128,.08)}
.steps{width:100%}
.steps table{width:100%;margin:0;table-layout:auto}
.steps th,.steps td{text-align:left}
.steps th:first-child,.steps td:first-child{white-space:nowrap;width:1%}
</style>
<div class="lauds"><div class="lauds-table" data-src="files/Tool_Categories.csv" data-filters=""></div></div>

## 3. Existing LAUDS-related Tools

License legend: FOSS: Free/Open Source Software OSH: Open Source Hardware CC: Creative Commons PROP: proprietary

<div class="lauds"><div class="lauds-table" data-src="files/Existing_LAUDS_Related_Tools.csv" data-require="Tool" data-filters="Type|Sub-Category|License|Integration Level|Lifecycle Phase|CAx classification" data-link="Ref-Link"></div></div>

<script>
(function(){
var el=document.querySelectorAll('pre.mermaid');
if(!el.length)return;
var sc=document.createElement('script');
sc.src='https://cdnjs.cloudflare.com/ajax/libs/mermaid/10.9.3/mermaid.min.js';
sc.onload=function(){
mermaid.initialize({startOnLoad:false,theme:'dark',securityLevel:'loose'});
mermaid.run({nodes:el});
};
document.head.appendChild(sc);
})();
</script>

<script>
(function(){
var cache={};
function parseCSV(t){var rows=[],r=[],c='',q=false,i=0;t=t.replace(/^\uFEFF/,'');
for(;i<t.length;i++){var ch=t[i];
if(q){if(ch=='"'){if(t[i+1]=='"'){c+='"';i++}else q=false}else c+=ch}
else if(ch=='"')q=true;else if(ch==','){r.push(c);c=''}
else if(ch=='\n'||ch=='\r'){if(ch=='\r'&&t[i+1]=='\n')i++;r.push(c);rows.push(r);r=[];c=''}
else c+=ch}
if(c||r.length){r.push(c);rows.push(r)}return rows}
function load(src){return cache[src]||(cache[src]=fetch(src).then(function(r){if(!r.ok)throw 0;return r.text()}).then(parseCSV))}
function esc(s){return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;')}
function prep(rows,req){
var all=rows.map(function(r){return r.map(function(c){return String(c).trim()})}).filter(function(r){return r.some(Boolean)});
var hi=0;while(hi<all.length&&all[hi].filter(Boolean).length<3)hi++;
var head=all[hi].map(function(h){return h.indexOf('FOSS:')==0?'License':h==='Exisiting Tools'?'Tool':h});
var data=all.slice(hi+1);var ri=head.indexOf(req);
data=data.filter(function(r){return ri>=0?r[ri]:r.join('')});
var keep=head.map(function(h,i){return h&&data.some(function(r){return r[i]})});
return {head:head.filter(function(_,i){return keep[i]}),data:data.map(function(r){return head.map(function(_,i){return r[i]||''}).filter(function(_,i){return keep[i]})})}}
function build(box,T){
var head=T.head,data=T.data;
var fcols=(box.dataset.filters||'').split('|').filter(Boolean).map(function(n){return head.indexOf(n)}).filter(function(i){return i>=0});
var lcol=head.indexOf(box.dataset.link);
var h='';
if(fcols.length){
h='<div class="tools">';
fcols.forEach(function(c){var v={};data.forEach(function(r){(r[c]||'').split(';').forEach(function(x){x=x.trim();if(x)v[x]=1})});
h+='<select data-c="'+c+'"><option value="">'+esc(head[c])+': all</option>'+Object.keys(v).sort().map(function(x){return '<option>'+esc(x)+'</option>'}).join('')+'</select>'});
h+='<button type="button" class="reset">Reset</button><span class="cnt"></span></div>'}
h+='<div class="wrap"><table><thead><tr>'+head.map(function(x,i){return '<th data-c="'+i+'">'+esc(x)+' <i></i></th>'}).join('')+'</tr></thead><tbody></tbody></table></div>';
box.innerHTML=h;
var tb=box.querySelector('tbody'),sels=[].slice.call(box.querySelectorAll('select')),cnt=box.querySelector('.cnt'),dir={};
function draw(){var n=0;tb.innerHTML=data.filter(function(r){return sels.every(function(s){return !s.value||(r[s.dataset.c]||'').split(';').map(function(x){return x.trim()}).indexOf(s.value)>=0})}).map(function(r){n++;
return '<tr>'+head.map(function(_,i){var v=r[i]||'';return '<td>'+(i==lcol&&/^https?:/.test(v)?'<a href="'+esc(v)+'" target="_blank">link</a>':esc(v))+'</td>'}).join('')+'</tr>'}).join('');
if(cnt)cnt.textContent=n+' of '+data.length+' rows'}
if(sels.length){sels.forEach(function(s){s.onchange=draw});
var reset=box.querySelector('.reset');
if(reset)reset.onclick=function(){sels.forEach(function(s){s.value=''});draw();}}
[].forEach.call(box.querySelectorAll('th'),function(th){th.onclick=function(){var c=+th.dataset.c;dir[c]=!dir[c];
data.sort(function(a,b){return (a[c]||'').localeCompare(b[c]||'',undefined,{numeric:true})*(dir[c]?1:-1)});
[].forEach.call(box.querySelectorAll('i'),function(i){i.textContent=''});th.querySelector('i').textContent=dir[c]?'\u25B2':'\u25BC';draw()}});
draw()}
[].forEach.call(document.querySelectorAll('.lauds-table'),function(box){
load(box.dataset.src).then(function(rows){build(box,prep(rows,box.dataset.require))}).catch(function(e){box.innerHTML='<p>Could not read '+esc(box.dataset.src)+' ('+esc(e&&e.message||'load failed')+'). Keep the CSV files in the files/ folder next to this file and open the page through a web server (e.g. run <code>python3 -m http.server</code> in this folder).</p>'})});
})();
</script>