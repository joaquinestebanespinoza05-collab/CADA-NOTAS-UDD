[Notas Parciales.html](https://github.com/user-attachments/files/32963608/Notas.Parciales.html)
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Notas Parciales</title>
<link href="https://fonts.googleapis.com/css2?family=Open+Sans:wght@400;600;700&display=swap" rel="stylesheet">
<style>
:root{--link:#0b5394;--blue:#0a4a8a;--red:#d63b0c;--orange:#d97006;--green:#2f9a2f;--line:#dfe3e8;--txt:#6b6f75}
*{box-sizing:border-box}
body{margin:0;background:#fff;color:var(--txt);font:16px/1.4 "Open Sans",Arial,sans-serif}
main{max-width:1100px;margin:0 auto;padding:20px 18px 60px}
h1{font-weight:400;font-size:2rem;margin:0 0 6px;color:#6b6f75}
.hint{font-size:.85rem;margin:0 0 22px;color:#8a8f96}
.course{border:1px solid var(--line);border-radius:4px;margin-bottom:4px;background:#fff}
.chead{padding:20px 46px;color:var(--link);text-transform:uppercase;cursor:pointer;font-size:1.05rem;display:flex;align-items:center;gap:6px;flex-wrap:wrap}
.course.open .chead{border-bottom:1px solid var(--line)}
.badge{display:inline-block;color:#fff;font-size:.78rem;font-weight:700;border-radius:6px;padding:3px 8px;line-height:1.3;text-transform:none}
.badge.ok{background:var(--blue)}.badge.bad{background:var(--red)}
.cbody{padding:22px 24px 26px}
.row{display:flex;align-items:center;justify-content:space-between;min-height:56px}
.row.tog{cursor:pointer}
.tri{display:inline-block;width:22px;color:#888;font-size:.85rem}
.cat{background:var(--orange);color:#fff;font-weight:700;font-size:.78rem;text-transform:uppercase;border-radius:8px;padding:8px 8px;line-height:1.1}
.name{text-transform:none}
.name.up{text-transform:uppercase}
.g{background:var(--green);color:#fff;font-weight:700;font-size:.8rem;border-radius:7px;padding:5px 9px;min-width:52px;text-align:center}
.g.bad{background:var(--red)}
.g.edit,.badge.edit{cursor:pointer}
.g.edit:hover,.badge.edit:hover{outline:2px solid #9bb8dc}
.empty{padding:6px 0 6px 6px;font-size:.92rem}
</style>
</head>
<body>
<main>
<h1>Notas Parciales</h1>
<p class="hint">Toca un ramo o la flecha ▶ para abrir el detalle. Toca una nota para cambiarla; se guarda en este navegador.</p>
<div id="root"></div>
</main>

<script>
/* ===== TUS DATOS =====
   n: nombre | w: ponderación (%) | g: nota | pct: true si es porcentaje (asistencia)
   items: sub-grupo con sus propias notas | final: nota final si no tienes detalle */
const COURSES=[
 {n:"Taller de comunicación oral",groups:[
   {n:"Asistencia",items:[{n:"ASISTENCIA",w:80,g:100,pct:true}]},
   {n:"Nota final",items:[
     {n:"CERTAMEN APERTURABLE",w:40,g:6.8},
     {n:"talleres",w:60,items:[
       {n:"TALLERES 1: STORYTELLING",w:33,g:5.2},
       {n:"TALLERES 3: BITÁCORA",w:34,g:5.0},
       {n:"TALLERES 2: MESA REDONDA",w:33,g:7.0}]}]}]},
 {n:"Morfología",groups:[
   {n:"Nota final",items:[
     {n:"talleres",w:100,items:[
       {n:"TAREAS",g:6.3},
       {n:"CERTAMEN 3",g:5.5},
       {n:"CERTAMEN 1",g:4.0},
       {n:"CERTAMEN 2",g:5.0},
       {n:"CONTROLES",g:5.0},
       {n:"INFORMES",g:6.0},
       {n:"TAREAS",g:6.0}]}]}]},
 {n:"Taller comunicación escrita",groups:[
   {n:"Nota final",items:[
     {n:"talleres",w:100,items:[
       {n:"TALLERES 1",g:6.8},
       {n:"TALLERES 2",g:6.6},
       {n:"TALLERES 3",g:7.0},
       {n:"TALLERES 4",g:5.0}]}]}]},
 {n:"Biofísica",groups:[
   {n:"Nota final",items:[
     {n:"talleres",w:100,items:[
       {n:"CERTAMEN 1",g:5.4},
       {n:"CERTAMEN 2",g:3.8},
       {n:"EXAMEN",g:4.9},
       {n:"CONTROLES",g:6.2},
       {n:"LABORATORIO",g:6.3}]}]}]},
 {n:"Actividad física y deporte",groups:[
   {n:"Nota final",items:[
     {n:"talleres",w:100,items:[
       {n:"TALLERES 1",g:6.8},
       {n:"TALLERES 2",g:6.8},
       {n:"TALLERES 3",g:5.6}]}]}]},
 {n:"Práctica kinésica básica",groups:[
   {n:"Nota final",items:[
     {n:"talleres",w:100,items:[
       {n:"CERTAMEN 1",g:6.8},
       {n:"CERTAMEN 2",g:6.6},
       {n:"TAREAS",g:5.0},
       {n:"PROYECTOS",g:6.1}]}]}]},
 {n:"Morfología II",groups:[]},
 {n:"Bases químicas y biológicas",groups:[]},
 {n:"Lectura crítica",groups:[]},
 {n:"Biofísica II",groups:[]}
];
/* ===================== */

const KEY='notas-parciales-v1';
let ov={},open=new Set();
try{ov=JSON.parse(localStorage.getItem(KEY))||{}}catch(e){}
const save=()=>{try{localStorage.setItem(KEY,JSON.stringify(ov))}catch(e){}};
const fmt=n=>n.toFixed(1).replace('.',',');

function leaf(it,k){return ov[k]!==undefined?ov[k]:(it.g!==undefined?it.g:null)}
function val(it,k){return it.items?avg(it.items,k):leaf(it,k)}
function avg(items,k){let s=0,w=0;items.forEach((it,i)=>{const v=val(it,k+'.'+i);if(v!=null){const x=it.w??1;s+=v*x;w+=x}});return w?Math.round(s/w*10+1e-9)/10:null}
function final(c,ci){
  const gi=c.groups.findIndex(g=>g.n.toUpperCase()==='NOTA FINAL');
  if(gi>=0)return avg(c.groups[gi].items,ci+'.'+gi);
  return ov['f'+ci]!==undefined?ov['f'+ci]:(c.final!==undefined?c.final:null);
}

function items(list,k,depth){
  return list.map((it,i)=>{
    const key=k+'.'+i,pad=`style="padding-left:${6+depth*28}px"`;
    const v=val(it,key),bad=v!=null&&!it.pct&&v<4;
    const badge=v==null?`<span class="g edit" data-edit="${key}">–</span>`
      :`<span class="g ${bad?'bad':''} ${it.items?'':'edit'}" ${it.items?'':`data-edit="${key}"`}>${fmt(v)}</span>`;
    const label=it.w!==undefined?`${it.n} (${fmt(it.w)}%)`.replace(',0%','%'):it.n;
    if(it.items){
      const o=open.has(key);
      return `<div class="row tog" ${pad} data-tog="${key}"><span><span class="tri">${o?'▼':'▶'}</span><span class="name">${label}</span></span>${badge}</div>`+(o?items(it.items,key,depth+1):'');
    }
    return `<div class="row" ${pad}><span class="name">${label}</span>${badge}</div>`;
  }).join('');
}

function render(){
  document.getElementById('root').innerHTML=COURSES.map((c,ci)=>{
    const o=open.has('c'+ci),f=final(c,ci);
    const hasDetail=c.groups.length>0;
    const badge=f!=null?`<span class="badge ${f<4?'bad':'ok'} ${hasDetail?'':'edit'}" ${hasDetail?'':`data-edit="f${ci}"`}>Nota final: ${fmt(f)}</span>`:'';
    let body='';
    if(o){
      body='<div class="cbody">'+(hasDetail?c.groups.map((g,gi)=>{
        const key=ci+'.'+gi,go=open.has('g'+key);
        return `<div class="row tog" data-tog="g${key}" style="justify-content:flex-start"><span class="tri">${go?'▼':'▶'}</span><span class="cat">${g.n}</span></div>`+(go?items(g.items,key,1):'');
      }).join(''):'<p class="empty">Aún no hay detalle para este ramo.</p>')+'</div>';
    }
    return `<section class="course ${o?'open':''}"><div class="chead" data-tog="c${ci}">${c.n} ${badge}</div>${body}</section>`;
  }).join('');
}

document.getElementById('root').addEventListener('click',e=>{
  const ed=e.target.closest('[data-edit]');
  if(ed){
    e.stopPropagation();
    const k=ed.dataset.edit;
    const r=prompt('Nueva nota (deja vacío para borrar):',ov[k]!==undefined?ov[k]:'');
    if(r===null)return;
    const n=parseFloat(r.replace(',','.'));
    if(r.trim()===''||isNaN(n))delete ov[k];else ov[k]=n;
    save();render();return;
  }
  const t=e.target.closest('[data-tog]');
  if(t){const k=t.dataset.tog;open.has(k)?open.delete(k):open.add(k);render()}
});
render();
</script>
</body>
</html>
