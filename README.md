<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Batman — Arkham Escape</title>
<style>
:root{
  --bg:#07070a; --panel:#121318; --muted:#9aa0a6; --accent:#1abc9c;
  --danger:#ff6b6b; --crit:#ffd24d; --text:#e6eef0;
}
*{box-sizing:border-box}
body{
  margin:0;font-family:Inter,system-ui,Arial;
  background:linear-gradient(180deg,#020204,#0b0f12);
  color:var(--text); display:flex; justify-content:center; padding:12px;
}
.wrap{
  width:100%; max-width:420px; background:var(--panel);
  border-radius:12px; padding:12px; border:1px solid rgba(255,255,255,0.03);
  box-shadow:0 8px 28px rgba(0,0,0,.7);
}
header{text-align:center}
h1{margin:6px 0; font-size:18px; color:var(--accent)}
.row{display:flex; gap:10px; align-items:flex-start; margin-top:8px}
.col{flex:1}
.panel{background:rgba(255,255,255,0.02); padding:10px; border-radius:8px;}
.avatar{font-size:46px; display:inline-block; width:72px; height:72px; line-height:72px; text-align:center; transition:transform .35s ease;}
.avatar.hit{ animation:hitAnim .32s ease; }
@keyframes hitAnim{ 0%{ transform:scale(1);} 50%{ transform:scale(1.18);} 100%{ transform:scale(1);} }
@keyframes lungeRight { 0%{transform:translateX(0);} 50%{transform:translateX(60px);} 100%{transform:translateX(0);} }
@keyframes lungeLeft { 0%{transform:translateX(0);} 50%{transform:translateX(-60px);} 100%{transform:translateX(0);} }
.avatar.lunge-right{ animation:lungeRight 0.5s ease; }
.avatar.lunge-left{ animation:lungeLeft 0.5s ease; }
.hpbar{height:12px;background:#1b1d20;border-radius:10px;overflow:hidden;border:1px solid rgba(255,255,255,.03)}
.hpfill{height:100%; transition:width .35s linear; background:linear-gradient(90deg,#2ecc71,#1abc9c)}
.hpfill.enemy{ background:linear-gradient(90deg,#ff6b6b,#ff4d4d) }
button{
  background:#0f1316; color:var(--text);
  border:1px solid rgba(255,255,255,.03);
  padding:10px 14px; border-radius:8px;
  cursor:pointer; font-weight:700; font-size:15px;
  flex:1; min-width:90px;
}
button:hover{ border-color:var(--accent); color:var(--accent) }
#actions{margin-top:10px; display:flex; gap:8px; flex-wrap:wrap; justify-content:center}
#log{ margin-top:12px; background:rgba(255,255,255,0.02); padding:10px; border-radius:8px; height:140px; overflow:auto; font-size:13px; color:var(--muted) }
.floating-dmg{ position:fixed; font-weight:900; pointer-events:none; text-shadow:0 6px 18px rgba(0,0,0,.6); transform:translate(-50%,-50%); animation:floatUp 900ms ease-out forwards; }
@keyframes floatUp{ from{ opacity:1; transform:translate(-50%,0) scale(1);} to{ opacity:0; transform:translate(-50%,-60px) scale(1.05);} }
.small{ font-size:12px; color:var(--muted) }
.inventory{ display:flex; gap:6px; flex-wrap:wrap; justify-content:center; margin-top:6px }
.item{ background:#0f1316; padding:6px 8px; border-radius:8px; border:1px solid rgba(255,255,255,.03); cursor:pointer; font-size:12px }
.hidden{display:none}
.progress-emoji{ font-size:24px; text-align:center; margin-top:8px; }
</style>
</head>
<body>
<div class="wrap">
  <header><h1>🦇 Batman — Arkham Escape</h1></header>

  <div id="startPanel" class="panel" style="margin-top:10px;">
    <div class="small">Choisis la difficulté :</div>
    <div style="margin-top:8px; display:flex; gap:8px; flex-wrap:wrap; justify-content:center">
      <button onclick="menuStart('facile')">😃 Facile</button>
      <button onclick="menuStart('normal')">😐 Normal</button>
      <button onclick="menuStart('difficile')">😈 Difficile</button>
    </div>
  </div>

  <div id="gamePanel" class="panel hidden" style="margin-top:10px;">
    <div class="row">
      <div class="col" style="max-width:160px">
        <div class="small">BATMAN</div>
        <div id="playerAvatar" class="avatar">🦇</div>
        <div class="small" id="playerPVtxt">PV: 40 / 40</div>
        <div class="hpbar" style="margin-top:4px"><div id="playerHP" class="hpfill" style="width:100%"></div></div>
        <div class="small" style="margin-top:6px">Objets</div>
        <div id="inventory" class="inventory"><div class="small">Aucun</div></div>
      </div>
      <div class="col">
        <div id="sceneText" class="small">Choisis une sortie :</div>
        <div id="choiceButtons" style="margin-top:8px;">
          <button onclick="chooseDoor('gauche')">🚪 Gauche</button>
          <button onclick="chooseDoor('droite')">🚪 Droite</button>
          <button onclick="chooseDoor('ascenseur')">⬆️ Ascenseur</button>
        </div>
        <div id="encounterArea" style="margin-top:8px; display:none">
          <div style="display:flex; align-items:center; gap:6px; justify-content:center">
            <div id="enemyAvatar" class="avatar">👤</div>
            <div style="text-align:left">
              <div id="enemyName" class="small">Ennemi</div>
              <div class="small" id="enemyPVtxt">PV: 0 / 0</div>
              <div class="hpbar" style="margin-top:4px"><div id="enemyHP" class="hpfill enemy" style="width:0%"></div></div>
            </div>
          </div>
          <div id="actions" style="margin-top:10px"></div>
        </div>
        <div id="progressEmoji" class="progress-emoji hidden"></div>
      </div>
    </div>
    <div id="log" aria-live="polite"></div>
  </div>

  <div id="endPanel" class="panel hidden" style="margin-top:10px; text-align:center">
    <h3 id="endTitle"></h3>
    <div class="small" id="endText"></div>
    <div style="margin-top:10px">
      <button onclick="resetAll()">🔄 Rejouer</button>
    </div>
  </div>
</div>

<script>
// ------------------- Données -------------------
const riddlesByDiff = {
  facile:[
    {q:"Je peux être brisé sans être touché, qui suis-je ?", answers:["Un secret","Un verre","Un cœur"], correct:0},
    {q:"Je suis toujours devant toi mais on ne peut me voir, qui suis-je ?", answers:["L'avenir","Un miroir","Le vent"], correct:0},
    {q:"Plus je prends de la place, moins on me voit, qui suis-je ?", answers:["L'obscurité","La fumée","La poussière"], correct:0}
  ],
  normal:[
    {q:"Plus je sèche, plus je deviens mouillé. Qui suis-je ?", answers:["Une serviette","Une éponge","La pluie"], correct:0},
    {q:"Je grandis sans être vivant, j’ai besoin d’air. Qui suis-je ?", answers:["Le feu","Une ombre","Une vague"], correct:0},
    {q:"Je voyage autour du monde tout en restant dans un coin. Qui suis-je ?", answers:["Un timbre","Un oiseau","Une boussole"], correct:0}
  ],
  difficile:[
    {q:"On me prend avant de me donner. Qui suis-je ?", answers:["Une photo","Un cadeau","Un billet"], correct:0},
    {q:"Je parle sans bouche, j’entends sans oreilles. Qui suis-je ?", answers:["Un écho","Un fantôme","Un secret"], correct:0},
    {q:"Moins j’existe, plus vous me voyez. Qui suis-je ?", answers:["Un trou","Une étoile","L'ombre"], correct:0}
  ]
};

let difficulty='normal',batman=null,encounters=0,currentEnemy=null,riddleIndex=0,combatLock=false;

// ------------------- Utils -------------------
function $(id){return document.getElementById(id);}
function rand(min,max){return Math.floor(Math.random()*(max-min+1))+min;}
function log(msg){const d=$('log');const el=document.createElement('div');el.innerHTML=msg;d.appendChild(el);d.scrollTop=d.scrollHeight;}
function spawnFloatingAt(el,txt,color='white'){if(!el)return;const r=el.getBoundingClientRect();const f=document.createElement('div');f.className='floating-dmg';f.textContent=txt;f.style.left=(r.left+r.width/2)+'px';f.style.top=(r.top+8)+'px';f.style.color=color;document.body.appendChild(f);setTimeout(()=>f.remove(),900);}

// ------------------- Initialisation -------------------
function menuStart(diff){
  difficulty=diff;
  const hp=(diff==='facile'?50:(diff==='difficile'?30:40));
  batman={pv:hp,max:hp,hasEvade:false};
  encounters=0; currentEnemy=null; riddleIndex=0;
  $('startPanel').classList.add('hidden'); $('gamePanel').classList.remove('hidden');
  $('progressEmoji').classList.add('hidden'); $('log').innerHTML='';
  $('sceneText').textContent='➡️ Choisis une sortie pour commencer (Gauche / Droite / Ascenseur).';
  updateUI();
}

// ------------------- UI -------------------
function updateUI(){
  $('playerPVtxt').textContent=`PV: ${batman.pv} / ${batman.max}`;
  $('playerHP').style.width=Math.max(0,(batman.pv/batman.max*100))+'%';
  if(currentEnemy && currentEnemy.type==='combat'){
    $('encounterArea').style.display='block';
    $('enemyAvatar').textContent=currentEnemy.emoji;
    $('enemyName').textContent=currentEnemy.name;
    $('enemyPVtxt').textContent=`PV: ${currentEnemy.pv} / ${currentEnemy.max}`;
    $('enemyHP').style.width=Math.max(0,(currentEnemy.pv/currentEnemy.max*100))+'%';
  }
}

// ------------------- Choix de porte -------------------
function chooseDoor(dir){
  $('choiceButtons').style.display='none';
  $('sceneText').textContent='... tu avances ...';
  setTimeout(()=>{
    encounters++;
    if(encounters===1){
      const base=(difficulty==='facile'?18:(difficulty==='difficile'?25:20));
      currentEnemy={type:'combat',name:'Joker',emoji:'🤡',pv:base,max:base,dmgRange:(difficulty==='facile'? [1,4]:(difficulty==='difficile'? [3,6]:[2,5]))};
      startCombat();
    } else if(encounters===2){
      const base=(difficulty==='facile'?20:(difficulty==='difficile'?30:25));
      currentEnemy={type:'combat',name:'Bane',emoji:'💪',pv:base,max:base,dmgRange:(difficulty==='facile'? [2,5]:(difficulty==='difficile'? [4,8]:[3,7]))};
      startCombat();
    } else{
      currentEnemy={type:'riddle',name:"L'Homme-Mystère",emoji:'🧩'};
      startRiddleSequence();
    }
    updateUI();
  },400);
}

// ------------------- Reset -------------------
function resetAll(){
  $('endPanel').classList.add('hidden');
  $('gamePanel').classList.add('hidden');
  $('startPanel').classList.remove('hidden');
  $('log').innerHTML='';
  $('choiceButtons').style.display='flex';
  $('sceneText').textContent='Choisis une sortie :';
}

// ------------------- Combat -------------------
function startCombat(){
  $('sceneText').textContent=`⚔️ Combat contre ${currentEnemy.name}`;
  showCombatOptions();
}

function showCombatOptions(){
  if(batman.pv<=0){endEncounter(false); return;}
  if(currentEnemy.pv<=0){endEncounter(true); return;}
  const actionsDiv=$('actions'); actionsDiv.innerHTML='';
  ['💥 Coup léger (2-4)','🦇 Coup fort (4-7)','🛡️ Défense (+évasion)'].forEach((txt,i)=>{
    const btn=document.createElement('button'); btn.textContent=txt; btn.onclick=()=>playerAttack(i); actionsDiv.appendChild(btn);
  });
}

function playerAttack(idx){
  if(combatLock) return; combatLock=true;
  let dmg=0;
  if(idx===0)dmg=rand(2,4);
  else if(idx===1)dmg=rand(4,7);
  else{batman.hasEvade=true; log('🛡️ Batman prépare une défense !'); combatLock=false; return;}
  // Animation Batman qui fonce vers l’ennemi
  $('playerAvatar').classList.add('lunge-right');
  setTimeout(()=>$('playerAvatar').classList.remove('lunge-right'),500);

  currentEnemy.pv-=dmg; if(currentEnemy.pv<0)currentEnemy.pv=0;
  log(`🦇 Batman inflige ${dmg} PV à ${currentEnemy.name}`);
  spawnFloatingAt($('enemyAvatar'),`-${dmg}`,'red');
  updateUI();
  setTimeout(enemyTurn,600);
}

function enemyTurn(){
  if(currentEnemy.pv<=0){combatLock=false; endEncounter(true); return;}
  let dmg=rand(currentEnemy.dmgRange[0],currentEnemy.dmgRange[1]);
  if(batman.hasEvade){dmg=Math.max(0,dmg-3); batman.hasEvade=false;}
  // Animation ennemi qui fonce vers Batman
  $('enemyAvatar').classList.add('lunge-left');
  setTimeout(()=>$('enemyAvatar').classList.remove('lunge-left'),500);

  batman.pv-=dmg; if(batman.pv<0)batman.pv=0;
  log(`💀 ${currentEnemy.name} inflige ${dmg} PV à Batman`);
  spawnFloatingAt($('playerAvatar'),`-${dmg}`,'red');
  updateUI();
  combatLock=false; showCombatOptions();
}

// ------------------- Riddler -------------------
function startRiddleSequence(){
  $('encounterArea').style.display='block';
  $('actions').innerHTML=''; riddleIndex=0;
  showRiddle();
}

function showRiddle(){
  const riddles=riddlesByDiff[difficulty];
  if(riddleIndex>=riddles.length){endEncounter(true); return;}
  const riddle=riddles[riddleIndex];
  $('sceneText').textContent=riddle.q;
  const actionsDiv=$('actions'); actionsDiv.innerHTML='';
  riddle.answers.forEach((ans,i)=>{
    const btn=document.createElement('button'); btn.textContent=ans;
    btn.onclick=()=>{
      if(i===riddle.correct){log(`✔️ Correct: ${ans}`); spawnFloatingAt($('enemyAvatar'),'+1','green');}
      else{log(`❌ Faux: ${ans}`); spawnFloatingAt($('playerAvatar'),'-5','red'); batman.pv-=5;if(batman.pv<0)batman.pv=0;}
      updateUI(); riddleIndex++; setTimeout(showRiddle,400);
    };
    actionsDiv.appendChild(btn);
  });
}

// ------------------- Fin -------------------
function endEncounter(victory){
  $('actions').innerHTML=''; $('encounterArea').style.display='none';
  if(victory){log('✅ Batman remporte cette rencontre !'); $('progressEmoji').textContent+='✅'; $('progressEmoji').classList.remove('hidden');}
  else{log('💀 Batman a été vaincu...'); $('progressEmoji').classList.add('hidden');}
  setTimeout(()=>{
    if(encounters<3 && victory){$('choiceButtons').style.display='flex'; $('sceneText').textContent='➡️ Choisis une sortie :';}
    else{
      $('gamePanel').classList.add('hidden'); $('endPanel').classList.remove('hidden');
      $('endTitle').textContent=victory?'🏆 Batman a échappé à Arkham !':'💀 Batman est capturé !';
      $('endText').textContent='Appuie sur Rejouer pour recommencer.';
    }
  },600);
}
</script>
</body>
</html>
