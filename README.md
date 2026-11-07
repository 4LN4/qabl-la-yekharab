# qabl-la-yekharab
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>قبل لا يخرب</title>
<style>
:root{--bg:#f6f7f9;--fg:#1c2330;--mut:#667085;--card:#fff;--bd:#e2e5ea;--ac:#2f6fed;--ok:#1FA37A;--wr:#EF9F27;--bad:#E5484D;box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#10141b;--fg:#e8ecf3;--mut:#9aa4b5;--card:#1a212c;--bd:#2a3342;--ac:#6b9bff}}
*{box-sizing:border-box}
html{scroll-padding-top:env(safe-area-inset-top,0px)}
body{margin:0;background:var(--bg);color:var(--fg);font-family:system-ui,-apple-system,"Segoe UI",Tahoma,sans-serif;line-height:1.6}
#app{max-width:720px;margin:0 auto;padding:14px}
h1{font-size:20px;margin:6px 0 2px}
p.s{margin:0 0 12px;color:var(--mut);font-size:14px}
#st{position:relative;height:360px;border-radius:16px;background:var(--card);border:1px solid var(--bd);overflow:hidden;touch-action:none;user-select:none}
#st canvas{display:block;width:100%;height:100%}
#bn{position:absolute;inset:auto 8px 8px 8px;padding:10px 12px;border-radius:12px;background:var(--bg);border:1px solid var(--bd);font-size:14px}
#bn button{margin-top:6px;display:block}
.card{background:var(--card);border:1px solid var(--bd);border-radius:16px;padding:12px;margin-top:12px}
button{font:inherit;cursor:pointer;border:1px solid var(--bd);background:var(--card);color:var(--fg);padding:8px 14px;border-radius:10px}
button.on,#mb{background:var(--ac);color:#fff;border-color:var(--ac)}
.row2{display:flex;gap:10px;align-items:center}
input[type=range]{flex:1;accent-color:var(--ac)}
#sg{display:flex;gap:8px;margin-bottom:10px}
#sg button{flex:1}
.bts{display:flex;gap:8px;margin-top:10px;flex-wrap:wrap}
.bts button{flex:1;font-size:13px}
#rd{display:grid;grid-template-columns:1fr 1fr;gap:8px}
.rw{border:1px solid var(--bd);border-radius:12px;padding:8px 10px;cursor:pointer}
.rw.on{border-color:var(--ac);box-shadow:0 0 0 1px var(--ac)}
.rw small{display:block;color:var(--mut)}
.rw b{font-size:18px}
.dl{display:flex;justify-content:space-between;font-size:14px;color:var(--mut);margin-top:6px}
</style>
</head>
<body>
<div id="app">
<h1>قبل لا يخرب</h1>
<p class="s">حرّك الأيام أو شغّل المحاكاة، وقارن بين بدون قبل لا يخرب ومع قبل لا يخرب. اسحب الجهاز لتدويره.</p>
<div id="st"><div id="bn"></div></div>
<div class="card">
<div id="sg"><button data-m="no">بدون قبل لا يخرب</button><button data-m="with" class="on">مع قبل لا يخرب</button></div>
<div class="row2"><button id="pl">تشغيل</button><input id="sl" type="range" min="0" max="10" step="0.01" value="0"></div>
<div class="dl"><span id="dy"></span><span id="sv"></span></div>
<div class="bts"><button id="ma">قفز لوقت التنبيه</button><button id="mf">قفز لوقت العطل</button></div>
</div>
<div class="card"><div id="rd"></div></div>
</div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
var $=function(i){return document.getElementById(i)};
var DF=8,DA=DF*Math.pow(.3,1/1.5);
var d=0,play=0,maint=0,mode='with',sel='vib',saved=0,fs=1,ry=.6,el=.45,R=11,lastR=-1,lastS='',lastB='',H=360;
var SP={vib:{n:'الاهتزاز',u:'مم/ث',b:1.2,g:8,x:1},tmp:{n:'الحرارة',u:'°م',b:45,g:45,x:0},cur:{n:'التيار',u:'أمبير',b:3.2,g:3,x:1},snd:{n:'الصوت',u:'ديسيبل',b:55,g:30,x:0}};
function state(){var h=Math.max(0,100-100*Math.pow(d/DF,1.5));return{h:h,f:d>=DF,a:h<70}}
function setD(v){d=Math.max(0,Math.min(10,v));ui()}

/* ---- scene ---- */
var st=$('st'),W=st.clientWidth;
var rd=new THREE.WebGLRenderer({antialias:true,alpha:true});
rd.setPixelRatio(Math.min(devicePixelRatio||1,2));rd.setSize(W,H);st.insertBefore(rd.domElement,st.firstChild);
var scene=new THREE.Scene(),cam=new THREE.PerspectiveCamera(40,W/H,.1,100);
scene.add(new THREE.AmbientLight(0xffffff,.7));
var dl=new THREE.DirectionalLight(0xffffff,.8);dl.position.set(4,8,5);scene.add(dl);
var grid=new THREE.GridHelper(20,20,0x888888,0x888888);grid.material.transparent=true;grid.material.opacity=.25;scene.add(grid);

var dev=new THREE.Group(),unit=new THREE.Group();dev.add(unit);scene.add(dev);
var body=new THREE.Mesh(new THREE.BoxGeometry(2.8,1.8,1.9),new THREE.MeshStandardMaterial({color:0x3a4558,roughness:.6}));body.position.y=.9;unit.add(body);
var motor=new THREE.Mesh(new THREE.CylinderGeometry(.5,.5,.9,32),new THREE.MeshStandardMaterial({color:0x7F9BC4,roughness:.4,metalness:.3}));
motor.rotation.x=Math.PI/2;motor.position.set(-.7,.9,1.35);unit.add(motor);
var fan=new THREE.Group();fan.position.set(.65,.9,1.0);unit.add(fan);
var ring=new THREE.Mesh(new THREE.TorusGeometry(.62,.05,8,32),new THREE.MeshStandardMaterial({color:0x222a38}));ring.position.set(.65,.9,1.0);unit.add(ring);
var bm=new THREE.MeshStandardMaterial({color:0xcfd6e2});
for(var i=0;i<4;i++){var bl=new THREE.Mesh(new THREE.BoxGeometry(.5,.16,.04),bm);bl.position.x=.3;var g=new THREE.Group();g.add(bl);g.rotation.z=i*Math.PI/2;fan.add(g)}
fan.add(new THREE.Mesh(new THREE.CylinderGeometry(.1,.1,.1,16),new THREE.MeshStandardMaterial({color:0x222a38})).rotateX(Math.PI/2));

var RC=new THREE.Vector3(-4.4,2.4,-1.5);
var gw=new THREE.Mesh(new THREE.BoxGeometry(.9,.9,.9),new THREE.MeshStandardMaterial({color:0x2f6fed}));gw.position.copy(RC);scene.add(gw);
var an=new THREE.Mesh(new THREE.CylinderGeometry(.03,.03,.8,8),new THREE.MeshStandardMaterial({color:0x2f6fed}));an.position.set(RC.x,RC.y+.8,RC.z);scene.add(an);

var pos={vib:[-.7,.9,1.85],tmp:[0,1.85,0],cur:[1.45,.9,0],snd:[0,.9,-1.0]};
var sens={},halo={},beams={},hits=[],pk=[];
Object.keys(SP).forEach(function(k){
 var m=new THREE.Mesh(new THREE.SphereGeometry(.13,16,16),new THREE.MeshBasicMaterial({color:0x1FA37A}));
 m.position.set(pos[k][0],pos[k][1],pos[k][2]);m.userData.k=k;unit.add(m);sens[k]=m;hits.push(m);
 var h=new THREE.Mesh(new THREE.SphereGeometry(.2,16,16),new THREE.MeshBasicMaterial({color:0x1FA37A,transparent:true,opacity:.25}));m.add(h);halo[k]=h;
 var gm=new THREE.BufferGeometry();gm.setAttribute('position',new THREE.BufferAttribute(new Float32Array(6),3));
 var ln=new THREE.Line(gm,new THREE.LineBasicMaterial({color:0x2f6fed,transparent:true,opacity:.5}));ln.frustumCulled=false;scene.add(ln);beams[k]=ln;
 for(var j=0;j<2;j++){var p=new THREE.Mesh(new THREE.SphereGeometry(.07,8,8),new THREE.MeshBasicMaterial({color:0x2f6fed}));scene.add(p);pk.push({k:k,t:j*.5,m:p})}
});

/* health ring */
var rg=new THREE.Group();rg.position.set(0,3.5,0);scene.add(rg);
var track=new THREE.Mesh(new THREE.TorusGeometry(1,.1,8,64),new THREE.MeshBasicMaterial({color:0x888888,transparent:true,opacity:.25}));rg.add(track);
var arc=new THREE.Mesh(new THREE.TorusGeometry(1,.1,8,64,6.283),new THREE.MeshBasicMaterial({color:0x1FA37A}));arc.rotation.z=Math.PI/2;rg.add(arc);
var cv=document.createElement('canvas');cv.width=cv.height=256;var cx=cv.getContext('2d'),tex=new THREE.CanvasTexture(cv);
var pl=new THREE.Mesh(new THREE.PlaneGeometry(2.2,2.2),new THREE.MeshBasicMaterial({map:tex,transparent:true}));rg.add(pl);
function draw(h,sc,label){cx.clearRect(0,0,256,256);cx.textAlign='center';cx.fillStyle=sc;cx.direction='rtl';cx.font='bold 62px sans-serif';cx.fillText(Math.round(h)+'%',128,135);cx.font='bold 28px sans-serif';cx.fillText(label,128,180);tex.needsUpdate=true}

var sm=[];for(var i=0;i<6;i++){var s=new THREE.Mesh(new THREE.SphereGeometry(.2,8,8),new THREE.MeshBasicMaterial({color:0x555555,transparent:true,opacity:0}));s.userData.p=i/6;scene.add(s);sm.push(s)}
/* smoke follows the unit's world position (device at origin) */

/* ---- readouts ---- */
var rdEl=$('rd');
Object.keys(SP).forEach(function(k){var r=document.createElement('div');r.className='rw';r.dataset.k=k;r.innerHTML='<small>'+SP[k].n+'</small><b></b> <small style="display:inline"></small>';rdEl.appendChild(r)});

function ui(){
 var s=state(),w=(100-s.h)/100;
 $('sl').value=d;$('pl').textContent=play?'إيقاف':'تشغيل';
 $('dy').textContent='اليوم '+d.toFixed(1)+' من 10';
 $('sv').textContent='أعطال تم تفاديها: '+saved;
 var rows=rdEl.children;
 for(var i=0;i<rows.length;i++){var k=rows[i].dataset.k,p=SP[k],b=rows[i].children[1],u=rows[i].children[2];
  rows[i].classList.toggle('on',sel==k);
  if(mode=='with'){var v=p.b+w*p.g;b.textContent=v.toFixed(p.x);u.textContent=p.u;b.style.color=w>.7?'var(--bad)':w>.3?'var(--wr)':''}
  else{b.textContent='—';u.textContent='لا يوجد حساس';b.style.color=''}}
 var t;
 if(maint)t='🔧 جاري الصيانة وإعادة الجهاز لوضعه الطبيعي…';
 else if(s.f)t='🔴 تعطل الجهاز في اليوم '+DF+' وتوقف الإنتاج فجأة.'+(mode=='no'?' ما كان فيه أي إنذار قبلها.':'');
 else if(mode=='with'&&s.a)t='⚠️ <b>تنبيه مبكر:</b> صحة الجهاز '+Math.round(s.h)+'% والاهتزاز والحرارة فوق الطبيعي. العطل المتوقع بعد '+(DF-d).toFixed(1)+' يوم.<button id="mb">جدولة صيانة الآن</button>';
 else if(mode=='no'&&s.a)t='الجهاز يبدو شغّال، لكن ما عندك أي مؤشر على تدهوره.';
 else t='✅ الجهاز بحالة جيدة.';
 if(t!=lastB){$('bn').innerHTML=t;lastB=t}
}

/* ---- events ---- */
$('sl').oninput=function(){maint=0;play=0;setD(+this.value)};
$('pl').onclick=function(){if(d>=10)d=0;play=!play;maint=0;ui()};
$('ma').onclick=function(){play=0;maint=0;setD(DA+.15)};
$('mf').onclick=function(){play=0;maint=0;setD(DF+.1)};
$('sg').onclick=function(e){var b=e.target.closest('button');if(!b)return;mode=b.dataset.m;[].forEach.call($('sg').children,function(c){c.classList.toggle('on',c==b)});play=0;maint=0;ui()};
$('bn').onclick=function(e){if(e.target.id=='mb'){play=0;maint=1}};
$('rd').onclick=function(e){var b=e.target.closest('[data-k]');if(b){sel=b.dataset.k;ui()}};
var ps={},mv=0,pd=0,rc=new THREE.Raycaster(),mo=new THREE.Vector2();
st.addEventListener('pointerdown',function(e){if(e.target.closest('button'))return;ps[e.pointerId]={x:e.clientX,y:e.clientY};mv=0;var a=Object.keys(ps);if(a.length==2)pd=Math.hypot(ps[a[0]].x-ps[a[1]].x,ps[a[0]].y-ps[a[1]].y)});
st.addEventListener('pointermove',function(e){var p=ps[e.pointerId];if(!p)return;var a=Object.keys(ps);if(a.length==1){var dx=e.clientX-p.x,dy=e.clientY-p.y;mv+=Math.abs(dx)+Math.abs(dy);ry+=dx*.01;el=Math.max(.05,Math.min(1.2,el+dy*.006))}else{ps[e.pointerId]={x:e.clientX,y:e.clientY};var q=Math.hypot(ps[a[0]].x-ps[a[1]].x,ps[a[0]].y-ps[a[1]].y);R=Math.max(8,Math.min(16,R-(q-pd)*.03));pd=q;mv+=10}ps[e.pointerId]={x:e.clientX,y:e.clientY}});
function up(e){var h=ps[e.pointerId];delete ps[e.pointerId];if(h&&mv<6&&e.type=='pointerup'&&mode=='with'){var r=rd.domElement.getBoundingClientRect();mo.set((e.clientX-r.left)/r.width*2-1,-((e.clientY-r.top)/r.height)*2+1);rc.setFromCamera(mo,cam);var x=rc.intersectObjects(hits);if(x.length){sel=x[0].object.userData.k;ui()}}}
st.addEventListener('pointerup',up);st.addEventListener('pointercancel',up);
st.addEventListener('wheel',function(e){e.preventDefault();R=Math.max(8,Math.min(16,R+e.deltaY*.01))},{passive:false});
addEventListener('resize',function(){W=st.clientWidth;rd.setSize(W,H);cam.aspect=W/H;cam.updateProjectionMatrix()});

/* ---- loop ---- */
var t0=0,v1=new THREE.Vector3(),cH=new THREE.Color(0x7F9BC4),cB=new THREE.Color(0xD9644A);
function loop(){requestAnimationFrame(loop);t0+=.016;
 if(play){d+=.0133;if(d>=10){d=10;play=0}var s0=state();if(mode=='with'&&s0.a&&!maint){play=0}ui()}
 if(maint){d+=(0-d)*.06;if(d<.08){d=0;maint=0;saved++;fs=1}ui()}
 var s=state(),h=s.h,w=(100-h)/100,wm=mode=='with';
 var tf=s.f?0:1;fs+=(tf-fs)*.04;fan.rotation.z+=.4*fs*(h>30?1:Math.max(h,0)/30);
 var jit=w*.05*(s.f?0:1);unit.position.set((Math.random()-.5)*jit,0,(Math.random()-.5)*jit);
 motor.material.color.copy(cH).lerp(cB,Math.min(1,w*1.4));motor.material.emissive.setHex(0xE5484D);motor.material.emissiveIntensity=Math.min(.55,w*.8);
 var col=h<30?0xE5484D:h<70?0xEF9F27:0x1FA37A,sc='#'+col.toString(16).padStart(6,'0');
 arc.material.color.setHex(col);var rr=Math.round(h),key=rr+sc+mode+s.f;
 if(key!=lastS){lastS=key;arc.geometry.dispose();arc.geometry=new THREE.TorusGeometry(1,.1,8,64,Math.max(.02,h/100*6.283));draw(h,sc,mode=='no'&&!s.f?'غير معروف لك':s.f?'تعطل':h<70&&wm?'تنبيه':'صحة الجهاز')}
 rg.scale.setScalar(1+((s.a&&wm)||s.f?.07*Math.sin(t0*8):0));
 dev.rotation.y=ry;cam.position.set(0,R*Math.sin(el)+.4,R*Math.cos(el));cam.lookAt(0,1.4,0);rg.quaternion.copy(cam.quaternion);
 gw.visible=an.visible=wm;
 Object.keys(SP).forEach(function(k){var v=sens[k];v.visible=wm;beams[k].visible=wm;
  v.getWorldPosition(v1);var a=beams[k].geometry.attributes.position;a.setXYZ(0,v1.x,v1.y,v1.z);a.setXYZ(1,RC.x,RC.y-.1,RC.z);a.needsUpdate=true;
  halo[k].scale.setScalar((sel==k?1.5:1)*(1+.25*Math.sin(t0*4+k.length)));halo[k].material.opacity=sel==k?.5:.25;
  var c=w>.7?0xE5484D:w>.3?0xEF9F27:0x1FA37A;v.material.color.setHex(c);halo[k].material.color.setHex(c)});
 pk.forEach(function(p){p.m.visible=wm;p.t=(p.t+.012)%1;sens[p.k].getWorldPosition(v1);p.m.position.set(v1.x+(RC.x-v1.x)*p.t,v1.y+(RC.y-.1-v1.y)*p.t,v1.z+(RC.z-v1.z)*p.t)});
 sm.forEach(function(m){if(s.f){m.userData.p=(m.userData.p+.008)%1;var p=m.userData.p;m.position.set(-.6+Math.sin(p*9+m.id)*.4,1.9+p*2.2,1.3+Math.cos(p*7+m.id)*.2);m.scale.setScalar(.6+p*1.4);m.material.opacity=.5*(1-p)}else m.material.opacity=0});
 rd.render(scene,cam)}
ui();loop();
</script>
</body>
</html>