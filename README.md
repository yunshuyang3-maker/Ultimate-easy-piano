<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Ultimate Easy Piano</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
:root{--bg:#0b1020;--card:rgba(255,255,255,.08);--text:#e5e7eb;--accent:#7c7cff;--glow:0 0 18px rgba(124,124,255,.8)}
body.light{--bg:#f8fafc;--card:#fff;--text:#0f172a}
*{box-sizing:border-box}
body{margin:0;font-family:system-ui,sans-serif;background:radial-gradient(circle at top,var(--accent),var(--bg) 45%);color:var(--text);transition:.4s;animation:bgShift 30s infinite alternate}
@keyframes bgShift{0%{background-position:0% 50%}50%{background-position:100% 50%}100%{background-position:0% 50%}}
header{padding:2rem 1rem;text-align:center;text-shadow:0 0 10px rgba(124,124,255,.5)}
header h1{margin:0;font-size:2.2rem}
header p{opacity:.8}
button{background:var(--accent);color:#fff;border:none;padding:.5rem .9rem;border-radius:12px;cursor:pointer;margin:.3rem;box-shadow:var(--glow);transition:.3s}
button:hover{transform:scale(1.05);box-shadow:0 0 25px var(--accent)}
main{max-width:1100px;margin:auto;padding:1rem}
.card{background:var(--card);backdrop-filter:blur(15px);border-radius:18px;padding:1rem;margin-bottom:1rem;box-shadow:0 15px 40px rgba(0,0,0,.4);transition:.3s}
.card:hover{transform:translateY(-3px);box-shadow:0 25px 60px rgba(124,124,255,.4)}
input{width:100%;padding:.7rem;border-radius:14px;border:none;outline:none;background:rgba(255,255,255,.1);color:var(--text)}
input::placeholder{opacity:.6}
small{opacity:.7}
.piano{display:flex;justify-content:center;position:relative;margin:2rem auto;width:420px}
.key{cursor:pointer;display:flex;justify-content:center;align-items:flex-end;font-size:.7rem;transition:.15s}
.white{width:60px;height:220px;background:linear-gradient(#fff,#ddd);border-radius:8px;border:1px solid #333;z-index:1}
.white:hover{box-shadow:0 0 15px var(--accent)}
.black{width:40px;height:140px;background:linear-gradient(#111,#000);position:absolute;top:0;z-index:2;margin-left:-20px;border-radius:6px}
.black:hover{box-shadow:0 0 15px var(--accent)}
.key.active{box-shadow:var(--glow);transform:translateY(2px)}
.song{margin-top:.7rem;transition:.3s}
.song:hover{transform:translateX(5px);text-shadow:0 0 5px var(--accent)}
.pattern{font-family:monospace;padding:.3rem .5rem;background:rgba(0,0,0,.3);border-radius:8px;display:inline-block}
.favorite{float:right;cursor:pointer;font-size:1.2rem;transition:.3s}
.favorite:hover{transform:scale(1.2);color:var(--accent);text-shadow:var(--glow)}
.autocomplete{background:var(--card);border-radius:12px;margin-top:.3rem;overflow:hidden}
.autocomplete div{padding:.4rem .6rem;cursor:pointer;transition:.2s}
.autocomplete div:hover{background:var(--accent)}
#lane{position:relative;height:260px;overflow:hidden;background:rgba(0,0,0,.35);border-radius:16px;margin:1rem 0;box-shadow:0 10px 30px rgba(0,0,0,.4)}
.note{position:absolute;left:50%;transform:translateX(-50%);width:50px;height:20px;border-radius:8px;background:var(--accent);box-shadow:var(--glow)}
#hitLine{position:absolute;bottom:40px;left:0;right:0;height:2px;background:#fff;opacity:.6}
.particle{position:absolute;width:6px;height:6px;border-radius:50%;background:var(--accent);pointer-events:none;animation:fadeParticle .5s forwards}
@keyframes fadeParticle{0%{opacity:1;transform:translateY(0)}100%{opacity:0;transform:translateY(-30px)}}
#combo{position:fixed;top:10px;right:10px;font-size:1.4rem;color:var(--accent);text-shadow:var(--glow)}
</style>
</head>
<body>
<header>
<h1>🎹 Ultimate Easy Piano</h1>
<p>Modern • Trending • Playable</p>
<button onclick="document.body.classList.toggle('light')">🌙 Theme</button>
</header>
<main>
<div class="card">
<h2>Piano</h2>
<div class="piano">
<div class="key white" data-f="261.63">C</div>
<div class="key black" style="left:45px" data-f="277.18"></div>
<div class="key white" data-f="293.66">D</div>
<div class="key black" style="left:105px" data-f="311.13"></div>
<div class="key white" data-f="329.63">E</div>
<div class="key white" data-f="349.23">F</div>
<div class="key black" style="left:225px" data-f="369.99"></div>
<div class="key white" data-f="392.00">G</div>
<div class="key black" style="left:285px" data-f="415.30"></div>
<div class="key white" data-f="440.00">A</div>
<div class="key black" style="left:345px" data-f="466.16"></div>
<div class="key white" data-f="493.88">B</div>
</div>
</div>

<div class="card">
<h2>Songs</h2>
<input id="search" placeholder="Search song, chord, mood..." oninput="searchSongs()">
<small>Only 10 latest searches are saved.</small>
<div id="auto" class="autocomplete"></div>
<div id="songs"></div>
</div>

<div class="card">
<h2>Play Along</h2>
<div id="lane"><div id="hitLine"></div></div>
<p>Score: <span id="score">0</span></p>
<button onclick="startPlayAlong()">▶ Start Play-Along</button>
</div>

<div id="combo">Combo: 0</div>

<div class="card">
<h2>Playlists</h2>
<input id="playlistName" placeholder="New playlist name">
<button onclick="addPlaylist()">➕ Add Playlist</button>
<div id="playlists"></div>
</div>
</main>

<script>
const ctx=new (window.AudioContext||window.webkitAudioContext)();
const chords=["C","G","Am","F","Dm","Em","A","Bb"],trending=["Blinding Lights","Espresso","Good Luck Babe","Meme Song","Rickroll","Coffin Dance","Dance Monkey","Stay","As It Was","Bad Habits"],songs=[];
for(let i=1;i<=1000;i++){let n=i<=500?trending[i%trending.length]+(i>trending.length?" "+i:""):"Song "+i;songs.push({name:n,pattern:[chords[i%8],chords[(i+1)%8],chords[(i+2)%8],chords[(i+3)%8]]})}
let fav=JSON.parse(localStorage.getItem("fav")||"[]"),history=JSON.parse(localStorage.getItem("hist")||"[]");
let notes=[],playing=false,score=0,combo=0,playlists=JSON.parse(localStorage.getItem("playlists")||"[]");

function play(f,key){const o=ctx.createOscillator(),g=ctx.createGain();o.type="triangle";o.frequency.value=f;g.gain.setValueAtTime(0,ctx.currentTime);g.gain.linearRampToValueAtTime(.2,ctx.currentTime+.05);g.gain.linearRampToValueAtTime(0,ctx.currentTime+.25);o.connect(g).connect(ctx.destination);o.start();key.classList.add("active");setTimeout(()=>key.classList.remove("active"),200);o.stop(ctx.currentTime+.26)}
document.querySelectorAll(".key").forEach(k=>k.onclick=()=>play(k.dataset.f,k));

function render(list){const s=document.getElementById("songs");s.innerHTML="";list.slice(0,50).forEach(x=>{const d=document.createElement("div");d.className="song";d.innerHTML=`<span class="favorite" onclick="favToggle('${x.name}')">${fav.includes(x.name)?"★":"☆"}</span><b>${x.name}</b><br><span class="pattern">${x.pattern.join(" – ")}</span>`;s.appendChild(d)})}
render(songs);

function searchSongs(){const q=search.value.toLowerCase().trim();if(q&&!history.includes(q)){history.unshift(q);history=history.slice(0,10);localStorage.setItem("hist",JSON.stringify(history))}render(songs.filter(s=>s.name.toLowerCase().includes(q)||s.pattern.join(" ").toLowerCase().includes(q)).slice(0,50))}

function favToggle(n){fav.includes(n)?fav=fav.filter(f=>f!==n):fav.push(n);localStorage.setItem("fav",JSON.stringify(fav));searchSongs()}

function startPlayAlong(){
  notes=[]; score=0; combo=0;
  document.getElementById("score").innerText=score;
  document.getElementById("combo").innerText="Combo: "+combo;
  playing=true;
  const song=songs[Math.floor(Math.random()*songs.length)];
  const melody=song.pattern;
  const freqsMap={"C":261.63,"D":293.66,"E":329.63,"F":349.23,"G":392.00,"A":440.00,"B":493.88,"Am":440.00,"Dm":293.66,"Em":329.63,"Bb":466.16};
  for(let i=0;i<50;i++){
    const chord=melody[i%melody.length];
    notes.push({y:-i*60,hit:false,freq:freqsMap[chord]||261.63});
  }
  requestAnimationFrame(updateNotes);
}

function updateNotes(){if(!playing)return;const l=document.getElementById("lane");l.querySelectorAll(".note").forEach(n=>n.remove());notes.forEach(n=>{n.y+=2;if(n.y>260&&!n.hit){n.hit=true;combo=0;document.getElementById("combo").innerText="Combo: "+combo}if(n.y<260){const d=document.createElement("div");d.className="note";d.style.top=n.y+"px";l.appendChild(d)}});requestAnimationFrame(updateNotes)}

function createParticles(x,y){for(let i=0;i<5;i++){const p=document.createElement("div");p.className="particle";p.style.left=x+"px";p.style.top=y+"px";document.body.appendChild(p);setTimeout(()=>p.remove(),500)}}

document.querySelectorAll(".key").forEach(k=>{k.addEventListener("click",()=>{const f=parseFloat(k.dataset.f);notes.forEach(n=>{if(!n.hit&&Math.abs(n.freq-f)<1&&Math.abs(n.y-220)<15){n.hit=true;score+=10;combo++;document.getElementById("score").innerText=score;document.getElementById("combo").innerText="Combo: "+combo;const r=k.getBoundingClientRect();createParticles(r.left+r.width/2,r.top+r.height/2)}})})})

function renderPlaylists(){const p=document.getElementById("playlists");p.innerHTML="";playlists.forEach(pl=>{const d=document.createElement("div");d.innerHTML=`<b>${pl}</b>`;p.appendChild(d)})}
renderPlaylists();
function addPlaylist(){const n=document.getElementById("playlistName").value.trim();if(n&&!playlists.includes(n)){playlists.push(n);localStorage.setItem("playlists",JSON.stringify(playlists));renderPlaylists();document.getElementById("playlistName").value=""}}
</script>
</body>
</html>
