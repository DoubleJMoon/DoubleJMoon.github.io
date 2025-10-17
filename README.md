<html lang="ko">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>랜덤 선택기</title>
<style>
:root{
  --accent:#a855f7;
  --bg:#071021;
  --card:#0f1724;
  --text:#e6eef8;
  --muted:#9aa9c6;
  --btnBg: transparent;
}
body.light{
  --accent:#6366f1;
  --bg:#f5f7fb;
  --card:#ffffff;
  --text:#0b1220;
  --muted:#6b7280;
}
body{
  margin:0;padding:20px;font-family:"Noto Sans KR","Segoe UI",system-ui,Roboto,-apple-system;
  background:var(--bg);color:var(--text);transition:all .2s ease;
}
.wrap{max-width:1200px;margin:0 auto}
.header{display:flex;justify-content:space-between;align-items:center;gap:12px;flex-wrap:wrap}
h1{margin:0;color:var(--accent);font-size:20px}
.controls{display:flex;gap:0.6rem;align-items:center;flex-wrap:wrap;margin:12px 0}
label.file-btn, button {
  display:inline-flex;align-items:center;gap:8px;padding:8px 12px;border-radius:10px;border:1px solid var(--accent);
  background:var(--btnBg);color:var(--accent);cursor:pointer;font-weight:700;transition:all .12s ease;
}
label.file-btn:hover, button:hover{background:var(--accent);color:var(--bg)}
label.file-btn input[type="file"]{display:none}
.file-name{font-size:13px;color:var(--muted);opacity:0.9}
.file-list{display:flex;flex-wrap:wrap;gap:14px;margin-top:12px}
.card{background:var(--card);border-radius:12px;padding:12px;width:340px;box-shadow:0 8px 30px rgba(0,0,0,0.45);display:flex;flex-direction:column;gap:8px;transition:outline .2s ease}
.card .titleRow{display:flex;gap:8px;align-items:center;justify-content:space-between}
.card input.cardTitle{background:transparent;border:1px dashed rgba(255,255,255,0.06);padding:6px;border-radius:8px;color:var(--text);font-weight:700;width:65%}
.card .meta{display:flex;gap:8px;align-items:center;justify-content:space-between}
.card textarea{width:95%;height:120px;resize:vertical;background:transparent;color:var(--text);border:1px solid rgba(255,255,255,0.04);border-radius:8px;padding:8px;font-size:13px;resize:none;}
.card .slots{display:flex;gap:8px;flex-wrap:wrap;align-items:center}
.slot{min-width:78px;flex:1;padding:8px;border-radius:8px;background:rgba(255,255,255,0.02);text-align:center;font-weight:700;min-height:44px;display:flex;align-items:center;justify-content:center}
.slot.anim{box-shadow:0 8px 18px rgba(0,0,0,0.45);transform:translateY(-4px);transition:transform .12s;animation: slotSpin 0.4s ease-in-out;}
@keyframes slotSpin{
  0%{transform: translateY(0); opacity:1;}
  50%{transform: translateY(-10px); opacity:0.5;}
  100%{transform: translateY(0); opacity:1;}
}
.result{margin-top:6px;padding:8px;border-radius:8px;background:linear-gradient(90deg,rgba(168,85,247,0.12),transparent);font-weight:700;text-align:center;min-height:28px}
.history{font-size:12px;color:var(--muted);background:rgba(255,255,255,0.03);padding:8px;border-radius:8px;max-height:86px;overflow:auto}
#allResults{margin-top:20px;padding:12px;border-top:2px solid var(--accent);border-radius:8px;background:transparent}
#allResults h3{margin:0 0 8px 0}
.footer{margin-top:18px;color:var(--muted);font-size:13px;display:flex;justify-content:space-between;flex-wrap:wrap}
.smallGhost{background:transparent;border:1px solid rgba(255,255,255,0.06);padding:6px 8px;border-radius:8px;color:var(--text);cursor:pointer}
@media(max-width:820px){.card{width:100%}}

/* 다크 모드 input/textarea */
body:not(.light) .card textarea,
body:not(.light) .card input {
  background: #0f1724; /* 카드 배경과 유사하게 */
  color: #e6eef8;      /* 텍스트 밝게 */
  border: 1px solid rgba(255,255,255,0.06);
  outline-color: rgba(255,255,255,0.3);
}

/* 다크 모드 number input */
body:not(.light) .card input[type="number"] {
  background: #0f1724;
  color: #e6eef8;
  border: 1px solid rgba(255,255,255,0.06);
}

/* 라이트 모드 textarea 테두리 검정 */
body.light .card textarea {
  border-color: #000;
  outline-color: #000;
  color: #0b1220;
  background: #fff;
}
</style>
</head>
<body>
<div class="wrap">
  <div class="header">
    <div style="display:flex;align-items:center;gap:12px;flex-wrap:wrap">
      <h1>🎰 랜덤 선택기</h1>
      <div class="file-name" id="fileCount">(카드 없음)</div>
    </div>
    <div style="display:flex;gap:8px;align-items:center;flex-wrap:wrap">
      <label class="file-btn">📂 파일 불러오기
        <input id="fileInput" type="file" accept=".txt" multiple>
      </label>
      <button id="addCardBtn">➕ 새 카드 추가</button>
      <button id="runAllBtn">🎰 모두 추첨</button>
      <!-- <button id="rerunAllBtn" class="smallGhost">🔁 한번에 돌리기</button> -->
      <button id="themeToggle" class="smallGhost">🌗 테마</button>
    </div>
  </div>

  <div class="controls" style="margin-top:10px">
    <div style="font-size:13px;color:var(--muted)">※ 각 카드의 제목과 항목은 직접 수정 가능합니다.</div>
  </div>

  <div id="fileList" class="file-list"></div>

  <div id="allResults">
    <h3>📊 전체 최신 결과</h3>
    <div id="allResultsContent">아직 결과가 없습니다.</div>
  </div>

  <div class="footer">
    <div></div>
    <div style="color:var(--muted)">만든이: <a href="https://github.com/moondouble" style="text-decoration-line:none;color:var(--muted)" target="_blank">정마테</a></div>
  </div>
</div>

<script>
const fileInput = document.getElementById('fileInput');
const addCardBtn = document.getElementById('addCardBtn');
const fileList = document.getElementById('fileList');
const runAllBtn = document.getElementById('runAllBtn');
const rerunAllBtn = document.getElementById('rerunAllBtn');
const themeBtn = document.getElementById('themeToggle');
const fileCount = document.getElementById('fileCount');
const allResultsContent = document.getElementById('allResultsContent');

let fileData = [];
let idCounter = 1;
initTheme();
updateFileCount();
updateAllResults();

function updateFileCount(){
  fileCount.textContent = fileData.length ? `${fileData.length} 개의 카드` : '(카드 없음)';
}

fileInput.addEventListener('change', async (e) => {
  const files = [...e.target.files];
  for (const f of files) {
    const text = await f.text();
    const items = text.split(/\r?\n/).map(s=>s.trim()).filter(Boolean);
    const nameWithoutExt = f.name.replace(/\.[^/.]+$/, '');
    const obj = createDataObj(nameWithoutExt, items);
    addCardToDOM(obj, true);
    fileData.push(obj);
  }
  updateAllResults();
  updateFileCount();
});

addCardBtn.addEventListener('click', () => {
  const obj = createDataObj(`새 카드 ${idCounter}`, []);
  addCardToDOM(obj, false);
  fileData.push(obj);
  updateFileCount();
  setTimeout(()=>{ obj.card.querySelector('.cardTitle').focus(); }, 60);
});

function createDataObj(name, items){
  return {id:'card_'+(idCounter++),name,name,items:items.slice(),count:1,result:[],history:[],card:null};
}

function addCardToDOM(obj, fromFile){
  const card = document.createElement('div'); card.className='card';
  card.innerHTML=`
  <div class="titleRow">
    <input class="cardTitle" value="${escapeHtml(obj.name)}" />
    <div style="display:flex;gap:6px;align-items:center">
      <button class="removeBtn smallGhost" title="제거">🗑️</button>
    </div>
  </div>
  <div class="meta">
    <label>개수: <input type="number" class="countInput" min="1" value="${obj.count}"></label>
    <div><button class="pickBtn">추첨</button></div>
  </div>
  <div class="slots" data-slots></div>
  <textarea class="itemsArea" placeholder="항목을 한 줄씩 입력">${escapeHtml(obj.items.join('\n'))}</textarea>
  <div class="result">—</div>
  <div class="history">기록 없음</div>
  `;
  obj.card = card;
  fileList.appendChild(card);

  const titleInput = card.querySelector('.cardTitle');
  const removeBtn = card.querySelector('.removeBtn');
  const countInput = card.querySelector('.countInput');
  const pickBtn = card.querySelector('.pickBtn');
  const slotsWrap = card.querySelector('[data-slots]');
  const itemsArea = card.querySelector('.itemsArea');

  titleInput.addEventListener('input', ()=>{ obj.name = titleInput.value; updateAllResults(); });
  removeBtn.addEventListener('click', ()=>{ fileList.removeChild(card); fileData = fileData.filter(x=>x!==obj); updateFileCount(); updateAllResults(); });
  countInput.addEventListener('input', ()=>{ obj.count = parseInt(countInput.value)||1; });
  itemsArea.addEventListener('input', ()=>{ obj.items = itemsArea.value.split(/\r?\n/).map(s=>s.trim()).filter(Boolean); });
  pickBtn.addEventListener('click', ()=>{ runCard(obj); });
}

function escapeHtml(str){ return str.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;'); }

/* 🌟 핵심: 점점 느려지는 슬롯 + 마지막에 결과 표시 */
function runCard(obj){
  const slotsWrap = obj.card.querySelector('[data-slots]');
  slotsWrap.innerHTML = '';
  const selected = [];
  const available = obj.items.slice();
  let i=0;

  function spinNext(){
    if(i>=obj.count || !available.length){
      // 모든 슬롯 애니메이션 끝난 후 결과 표시
      obj.result = selected;
      obj.history.unshift(selected.join(', '));
      if(obj.history.length>5) obj.history.pop();
      obj.card.querySelector('.result').textContent = selected.join(', ');
      obj.card.querySelector('.history').textContent = obj.history.join('\n');
      updateAllResults();
      return;
    }

    const idx = Math.floor(Math.random()*available.length);
    const val = available.splice(idx,1)[0];
    selected.push(val);

    const slot = document.createElement('div');
    slot.className='slot anim';
    slot.textContent='...';
    slotsWrap.appendChild(slot);

    function spinSlot(step=0, maxSteps=15, delay=30){
      if(step>=maxSteps){
        slot.textContent = val;
        i++;
        spinNext();
        return;
      }
      slot.textContent = available.length ? available[Math.floor(Math.random()*available.length)] : val;
      const nextDelay = delay + step*20;
      setTimeout(()=>spinSlot(step+1, maxSteps, delay), nextDelay);
    }

    spinSlot();
  }

  spinNext();
}

function runAllCardsSequentially(){
  let i=0;
  function next(){
    if(i>=fileData.length) return;
    runCard(fileData[i++]);
    setTimeout(next, 3000);
  }
  next();
}

runAllBtn.addEventListener('click', ()=>runAllCardsSequentially());
<!-- rerunAllBtn.addEventListener('click', ()=>fileData.forEach(runCard)); -->

function updateAllResults(){
  if(!fileData.length){ allResultsContent.textContent = '아직 결과가 없습니다.'; return; }
  allResultsContent.innerHTML = '';
  fileData.forEach(obj=>{
    const div = document.createElement('div');
    div.textContent = `${obj.name}: ${obj.result.length ? obj.result.join(', ') : '—'}`;
    allResultsContent.appendChild(div);
  });
}

/* 테마 토글 */
function initTheme(){
  if(localStorage.theme==='light'){ document.body.classList.add('light'); } 
  else { document.body.classList.remove('light'); }
}
themeBtn.addEventListener('click', ()=>{
  document.body.classList.toggle('light');
  localStorage.theme = document.body.classList.contains('light') ? 'light' : 'dark';
});
</script>
</body>
</html>
