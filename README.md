<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Sistem Rental PlayStation</title>
<style>
:root{--bg:#f5f7fa;--card:#fff;--tx:#1b2430;--mu:#5f6b7a;--bd:#d8dee6;--pr:#3b3fa8;--prt:#fff;--ok:#1b7f4b;--wa:#b26a00}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#12141d;--card:#1b1f2c;--tx:#e8eaf3;--mu:#9aa1b6;--bd:#333a50;--pr:#8a8ff0;--prt:#0b0d16;--ok:#4cc38a;--wa:#e0a040}}
:root[data-theme="dark"]{--bg:#12141d;--card:#1b1f2c;--tx:#e8eaf3;--mu:#9aa1b6;--bd:#333a50;--pr:#8a8ff0;--prt:#0b0d16;--ok:#4cc38a;--wa:#e0a040}
:root{box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
html{scroll-padding-top:env(safe-area-inset-top,0px)}
*,*::before,*::after{box-sizing:inherit}
body{margin:0;background:var(--bg);color:var(--tx);font-family:Arial,Helvetica,sans-serif;font-size:14px}
.wrap{max-width:1000px;margin:0 auto;padding:16px}
h1{font-size:20px;margin:0 0 4px}.sub{color:var(--mu);margin:0 0 16px}
.card{background:var(--card);border:1px solid var(--bd);border-radius:10px;padding:14px;margin-bottom:16px}
h2{font-size:15px;margin:0 0 10px}
.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(150px,1fr));gap:10px}
.box{border:1px solid var(--bd);border-radius:8px;padding:10px}.box b{display:block;font-size:18px;margin-top:2px}.box span{color:var(--mu);font-size:12px}
.unit b{font-size:14px}.unit small{display:block;color:var(--mu);margin:2px 0 4px}
.Kosong{color:var(--ok);font-weight:bold}.Dipakai,.Bermain{color:var(--wa);font-weight:bold}.Selesai{color:var(--ok);font-weight:bold}
form{display:grid;grid-template-columns:repeat(auto-fit,minmax(170px,1fr));gap:10px;align-items:end}
label{display:flex;flex-direction:column;gap:4px;font-size:12px;color:var(--mu)}
input,select{font:inherit;padding:8px;border:1px solid var(--bd);border-radius:6px;background:var(--bg);color:var(--tx);width:100%}
button{font:inherit;cursor:pointer;border:0;border-radius:6px;padding:9px 14px;background:var(--pr);color:var(--prt);font-weight:bold}
button.g{background:transparent;color:var(--mu);border:1px solid var(--bd);font-weight:normal}
.prev{grid-column:1/-1;color:var(--mu)}.prev b{color:var(--tx)}
.tw{overflow-x:auto}
table{border-collapse:collapse;width:100%;min-width:760px}
th,td{border-bottom:1px solid var(--bd);padding:8px;text-align:left;white-space:nowrap}
th{font-size:12px;color:var(--mu)}td select{width:auto;padding:4px}.r{text-align:right}
.del{background:transparent;color:var(--mu);padding:2px 8px;border:1px solid var(--bd);font-weight:normal}
.acts{display:flex;gap:8px;flex-wrap:wrap;margin-top:10px}
.hint{color:var(--mu);font-size:12px;margin-top:8px}
</style>
</head>
<body>
<div class="wrap">
<h1>Sistem Rental PlayStation</h1>
<p class="sub">Catat penyewa, hitung biaya sewa otomatis, dan pantau unit yang kosong atau sedang dipakai.</p>

<div class="card"><h2>Status unit</h2><div class="grid" id="units"></div></div>

<div class="card"><div class="grid">
<div class="box"><span>Jumlah transaksi</span><b id="s1">0</b></div>
<div class="box"><span>Total jam sewa</span><b id="s2">0</b></div>
<div class="box"><span>Total pendapatan</span><b id="s3">Rp0</b></div>
<div class="box"><span>Sedang bermain</span><b id="s4">0</b></div>
</div></div>

<div class="card"><h2>Input sewa baru</h2>
<form id="f">
<label>Tanggal<input type="date" id="tgl" required></label>
<label>Nama penyewa<input id="nama" required placeholder="Contoh: Rafi"></label>
<label>Unit PlayStation<select id="unit"></select></label>
<label>Jam mulai<input type="time" id="jam" required></label>
<label>Lama sewa (jam)<input type="number" id="lama" min="0.5" step="0.5" required placeholder="0.0"></label>
<button type="submit">Simpan sewa</button>
<div class="prev" id="prev"></div>
</form></div>

<div class="card"><h2>Daftar sewa</h2>
<div class="tw"><table><thead><tr>
<th>No</th><th>Tanggal</th><th>Penyewa</th><th>Unit</th><th>Mulai</th><th>Selesai</th><th class="r">Lama</th><th class="r">Diskon</th><th class="r">Total bayar</th><th>Status</th><th></th>
</tr></thead><tbody id="tb"></tbody></table></div>
<div class="acts"><button class="g" id="reset">Kembalikan data contoh</button><button class="g" id="clear">Hapus semua</button></div>
<div class="hint">Aturan: diskon 10% untuk sewa 3 jam atau lebih. Satu unit tidak bisa disewa dua kali selagi statusnya masih Bermain. Tarif adalah contoh asumsi. Data tersimpan di browser perangkat ini.</div>
</div>
</div>
<script>
var U=[{n:"PS3 - Unit 1",h:4000},{n:"PS4 - Unit 2",h:6000},{n:"PS4 - Unit 3",h:6000},{n:"PS5 - Unit 4",h:10000}];
var KEY="rentalps_v1",ST=["Bermain","Selesai"];
var $=function(i){return document.getElementById(i)};
function today(){return new Date().toISOString().slice(0,10)}
function seed(){var t=today();return [
{t:t,n:"Rafi",u:"PS4 - Unit 2",j:"09:00",l:2,s:"Selesai"},
{t:t,n:"Dimas",u:"PS3 - Unit 1",j:"10:30",l:1.5,s:"Selesai"},
{t:t,n:"Fajar",u:"PS5 - Unit 4",j:"13:00",l:3,s:"Bermain"},
{t:t,n:"Nanda",u:"PS4 - Unit 3",j:"14:00",l:1,s:"Bermain"}]}
var D;
try{var raw=localStorage.getItem(KEY);D=raw?JSON.parse(raw):null}catch(e){D=null}
if(!Array.isArray(D))D=seed();
function save(){try{localStorage.setItem(KEY,JSON.stringify(D))}catch(e){}}
function unit(n){return U.filter(function(x){return x.n===n})[0]}
function rp(v){return "Rp"+Math.round(v).toLocaleString("id-ID")}
function tgl(s){var a=s.split("-");return a[2]+"/"+a[1]+"/"+a[0]}
function esc(s){var d=document.createElement("div");d.textContent=s;return d.innerHTML}
function p2(n){return (n<10?"0":"")+n}
function akhir(j,l){var a=j.split(":"),m=(+a[0])*60+(+a[1])+Math.round(l*60);return p2(Math.floor(m/60)%24)+":"+p2(m%60)}
function hitung(u,l){var x=unit(u),b=x?x.h*l:0,d=l>=3?b*0.1:0;return {b:b,d:d,t:b-d,h:x?x.h:0}}
U.forEach(function(x){var o=document.createElement("option");o.value=x.n;o.textContent=x.n+" ("+rp(x.h)+"/jam)";$("unit").appendChild(o)});
$("tgl").value=today();var now=new Date();$("jam").value=p2(now.getHours())+":"+p2(now.getMinutes());
function prev(){var l=parseFloat($("lama").value),j=$("jam").value;
if(!(l>0)||!j){$("prev").innerHTML="Isi lama sewa untuk melihat biaya otomatis.";return}
var c=hitung($("unit").value,l);
$("prev").innerHTML="Tarif: <b>"+rp(c.h)+"/jam</b> · Diskon: <b>"+rp(c.d)+"</b> · Total bayar: <b>"+rp(c.t)+"</b> · Selesai pukul: <b>"+akhir(j,l)+"</b>"}
["unit","lama","jam"].forEach(function(i){$(i).addEventListener("input",prev)});
function pemakai(n){return D.filter(function(r){return r.u===n&&r.s==="Bermain"})[0]}
function render(){
var tot=0,jam=0,main=0,h="";
D.forEach(function(r,i){var c=hitung(r.u,r.l);tot+=c.t;jam+=r.l;if(r.s==="Bermain")main++;
h+="<tr><td>"+(i+1)+"</td><td>"+tgl(r.t)+"</td><td>"+esc(r.n)+"</td><td>"+esc(r.u)+"</td><td>"+r.j+"</td><td>"+akhir(r.j,r.l)+"</td><td class=r>"+r.l+" jam</td><td class=r>"+rp(c.d)+"</td><td class=r>"+rp(c.t)+"</td><td><select data-i='"+i+"' class='"+r.s+"'>"+ST.map(function(s){return "<option"+(s===r.s?" selected":"")+">"+s+"</option>"}).join("")+"</select></td><td><button class=del data-d='"+i+"' title='Hapus'>&times;</button></td></tr>"});
$("tb").innerHTML=h||"<tr><td colspan=11>Belum ada transaksi.</td></tr>";
$("units").innerHTML=U.map(function(x){var p=pemakai(x.n);return "<div class='box unit'><b>"+x.n+"</b><small>"+rp(x.h)+"/jam</small><span class='"+(p?"Dipakai":"Kosong")+"'>"+(p?"Dipakai: "+esc(p.n):"Kosong")+"</span></div>"}).join("");
$("s1").textContent=D.length;$("s2").textContent=Math.round(jam*10)/10;$("s3").textContent=rp(tot);$("s4").textContent=main;prev()}
$("f").addEventListener("submit",function(e){e.preventDefault();
var u=$("unit").value;
if(pemakai(u)){alert(u+" masih dipakai oleh "+pemakai(u).n+". Pilih unit lain atau ubah statusnya jadi Selesai dulu.");return}
D.push({t:$("tgl").value,n:$("nama").value.trim(),u:u,j:$("jam").value,l:parseFloat($("lama").value),s:"Bermain"});
save();render();$("nama").value="";$("lama").value="";prev();$("nama").focus()});
$("tb").addEventListener("change",function(e){var i=e.target.getAttribute("data-i");if(i!==null){D[i].s=e.target.value;save();render()}});
$("tb").addEventListener("click",function(e){var d=e.target.getAttribute("data-d");if(d!==null){D.splice(+d,1);save();render()}});
$("reset").addEventListener("click",function(){D=seed();save();render()});
$("clear").addEventListener("click",function(){D=[];save();render()});
render();
</script>
</body>
</html>

