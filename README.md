<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>머더 미스터리 추첨기 - BETA</title>
<style>
  :root {
    --bg-color: radial-gradient(circle at center, #1a1a1a 0%, #050505 100%);
    --card-bg: #f5f5f5;
    --accent-red: #cc0000;
    --zombie-green: #32CD32;
    --chill-blue: #87CEEB;
  }

  body { 
    font-family: 'Segoe UI', sans-serif; background: var(--bg-color);
    margin: 0; padding: 20px; text-align: center; color: #e0e0e0; 
    min-height: 100vh; display: flex; flex-direction: column; justify-content: center;
  }

  #menu, #draw-area { 
    background: rgba(30, 30, 30, 0.9); padding: 30px; border-radius: 20px; 
    display: inline-block; border: 1px solid #333; box-shadow: 0 10px 30px rgba(0,0,0,0.5);
    max-width: 90%;
  }

  /* 설명 박스 */
  .info-container { margin: 15px 0; display: flex; justify-content: center; gap: 8px; flex-wrap: wrap; }
  .info-btn { 
    padding: 6px 12px; border-radius: 6px; border: 1px solid #444; background: #222; 
    color: #888; cursor: pointer; font-size: 11px;
  }

  /* 토글 스위치 */
  .switch-container { margin: 15px 0; display: flex; align-items: center; justify-content: center; gap: 10px; }
  .switch { position: relative; display: inline-block; width: 44px; height: 20px; }
  .switch input { opacity: 0; width: 0; height: 0; }
  .slider { position: absolute; cursor: pointer; top: 0; left: 0; right: 0; bottom: 0; background-color: #444; transition: .4s; border-radius: 20px; }
  .slider:before { position: absolute; content: ""; height: 14px; width: 14px; left: 3px; bottom: 3px; background-color: white; transition: .4s; border-radius: 50%; }
  input:checked + .slider { background-color: var(--accent-red); }
  input:checked + .slider:before { transform: translateX(24px); }

  /* 결과 카드 */
  #result { 
    display: none; background: var(--card-bg); color: #111; padding: 40px 20px; 
    border-radius: 25px; margin: 20px auto; max-width: 380px; box-shadow: 0 20px 50px rgba(0,0,0,0.8);
  }

  /* 엔딩 화면 */
  #final-screen { display: none; background: #000; padding: 100px 20px; border-top: 2px solid #1a1a1a; }
  .final-murder-type { font-size: 70px; font-weight: 900; margin-bottom: 20px; text-transform: uppercase; }
  .type-standard { color: var(--accent-red); }
  .type-zombie { background: linear-gradient(to bottom, #44ff44, #003300); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
  .type-chillsear { color: #fff; text-shadow: 0 0 15px var(--chill-blue), 0 0 30px var(--chill-blue); }

  #modal { display: none; position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); background: #222; border: 1px solid #444; padding: 25px; border-radius: 15px; z-index: 100; width: 260px; }
  #modal-overlay { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.8); z-index: 99; }
</style>
</head>
<body>

<div id="modal-overlay" onclick="closeModal()"></div>
<div id="modal"><h3 id="modal-title" style="color:#fff;"></h3><p id="modal-body" style="color:#bbb;"></p><button onclick="closeModal()">닫기</button></div>

<div id="game-container">
  <h1 style="color:#444; letter-spacing:8px;">MURDER BETA</h1>

  <div id="menu">
    <div class="info-container">
      <div class="info-btn" onclick="openModal('Standard', '기본 모드입니다. 머더는 랜덤 특수 능력을 가집니다.')">STANDARD</div>
      <div class="info-btn" onclick="openModal('Zombie', '감염 모드. 죽은 생존자는 좀비가 되며, 의사가 치료하면 시민으로 돌아옵니다.')">ZOMBIE</div>
      <div class="info-btn" onclick="openModal('Chillsear', '암살 모드. 의사의 치료가 불가능한 강력한 타입입니다.')">CHILLSEAR</div>
    </div>

    <div class="switch-container">
      <span style="font-size: 13px; color: #777;">능력/타입 활성화</span>
      <label class="switch"><input type="checkbox" id="useSkills" checked><span class="slider"></span></label>
    </div>

    <label>인원 수 : <input type="number" id="playerCount" min="4" max="20" value="6"></label><br>
    <button onclick="startGame()" style="margin-top:20px; padding:15px 40px; border-radius:12px; border:none; background:#222; color:#fff; font-weight:bold;">GAME START</button>
  </div>

  <div id="draw-area" style="display:none;">
    <div id="playerNum" style="font-size: 28px; margin-bottom: 20px; color: #666;"></div>
    <button onclick="draw()" style="background:var(--accent-red); padding:15px 40px; border-radius:12px; border:none; color:#fff; font-weight:bold;">역할 확인</button>
  </div>

  <div id="result">
    <div id="role" style="font-size: 48px; font-weight: 900;"></div>
    <div id="skill" style="font-size: 32px; font-weight: bold; margin: 15px 0;"></div>
    <div id="desc" style="font-size: 18px; color: #666; margin-bottom: 30px;"></div>
    <button onclick="nextPlayer()" style="background:#000; color:#fff; width:100%; padding:15px; border-radius:10px; border:none;">확인 완료</button>
  </div>

  <div id="final-screen">
    <div style="color: #666; font-size: 20px; margin-bottom: 25px;">이번 라운드 머더는...</div>
    <div id="finalType" class="final-murder-type"></div>
    <div id="finalDetail" style="font-size: 22px; color: #555; font-weight:bold;"></div>
    <br><br>
    <button onclick="location.reload()" style="background:none; border:1px solid #333; color:#333; padding:10px 20px;">새 게임 시작</button>
  </div>
</div>

<script>
// 원본 설명 그대로 복구
const skills = {
  머더: [
    { name: "방탄", desc: "한 번 공격을 버팁니다. (2목숨)" },
    { name: "가방", desc: "죽인 대상이 살아있는 것처럼 행동하다가 마지막에 사망합니다. (1명)" },
    { name: "총잡이", desc: "보안관과 같은 방식으로 처치합니다." },
    { name: "무능력", desc: "능력이 없습니다." }
  ],
  보안관: [
    { name: "강인한", desc: "목숨이 2개가 됩니다." },
    { name: "마피아", desc: "머더와 모든 시민을 제거하면 즉시 승리합니다." },
    { name: "무능력", desc: "능력이 없습니다." }
  ],
  의사: [
    { name: "자가치유", desc: "자신을 치료할 수 있습니다. (1회)" },
    { name: "전문 의사", desc: "최대 2명까지 치료할 수 있습니다." },
    { name: "구급상자", desc: "생존자 1명에게 미리 지급해 스스로 치료하게 합니다." },
    { name: "무능력", desc: "능력이 없습니다." }
  ],
  시민: [
    { name: "스파이", desc: "시민/보안관을 죽일수록 최대 2목숨까지 증가합니다. 머더를 죽이면 자신도 사망하며 시민 승리." },
    { name: "퍼블win", desc: "가장 먼저 머더에게 죽으면 즉시 시민 승리." },
    { name: "패링", desc: "한 번 공격을 막아냅니다." },
    { name: "무능력", desc: "능력이 없습니다." }
  ]
};

const murderTypes = [
  { name: "Standard", weight: 60, desc: "추가 능력이 포함된 기본 라운드입니다.", class: "type-standard" },
  { name: "Chillsear", weight: 30, desc: "의사가 치료할 수 없는 무자비한 라운드입니다.", class: "type-chillsear" },
  { name: "Zombie", weight: 10, desc: "죽은 자들이 좀비가 되는 감염 라운드입니다.", class: "type-zombie" }
];

let totalPlayers, current, roles, roundMurderType, useSkills;

function openModal(t, b) { document.getElementById('modal-title').textContent = t; document.getElementById('modal-body').textContent = b; document.getElementById('modal').style.display = 'block'; document.getElementById('modal-overlay').style.display = 'block'; }
function closeModal() { document.getElementById('modal').style.display = 'none'; document.getElementById('modal-overlay').style.display = 'none'; }

function startGame() {
  totalPlayers = parseInt(document.getElementById("playerCount").value);
  useSkills = document.getElementById("useSkills").checked;
  roles = ["머더", "보안관", "의사"];
  for (let i = 4; i <= totalPlayers; i++) roles.push("시민");
  roles.sort(() => Math.random() - 0.5);
  const rand = Math.random() * 100;
  let cum = 0;
  for (const t of murderTypes) { cum += t.weight; if (rand < cum) { roundMurderType = t; break; } }
  current = 0;
  document.getElementById("menu").style.display = "none";
  document.getElementById("draw-area").style.display = "block";
  document.getElementById("playerNum").textContent = `PLAYER 1`;
}

function draw() {
  const role = roles[current];
  let dSkill = "능력 없음", dDesc = "무능력 상태입니다.";

  if (useSkills) {
    if (role === "머더" && roundMurderType.name === "Standard") {
      const s = skills.머더[Math.floor(Math.random() * skills.머더.length)];
      dSkill = s.name; dDesc = s.desc;
    } else if (role === "머더") {
      dSkill = "SPECIAL TYPE"; dDesc = "특수 타입입니다. 정체는 마지막에 공개됩니다.";
    } else {
      const list = skills[role] || skills.시민;
      const s = list[Math.floor(Math.random() * list.length)];
      dSkill = s.name; dDesc = s.desc;
    }
  }

  const rDiv = document.getElementById("role");
  rDiv.textContent = role;
  rDiv.style.color = role === "머더" ? "#cc0000" : role === "보안관" ? "#2b57ff" : role === "의사" ? "#28a745" : "#666";
  document.getElementById("skill").textContent = dSkill;
  document.getElementById("desc").textContent = dDesc;
  document.getElementById("draw-area").style.display = "none";
  document.getElementById("result").style.display = "block";
}

function nextPlayer() {
  current++;
  if (current >= totalPlayers) {
    if (useSkills) {
      document.getElementById("result").style.display = "none";
      document.getElementById("final-screen").style.display = "block";
      const tElt = document.getElementById("finalType");
      tElt.textContent = roundMurderType.name;
      tElt.className = "final-murder-type " + roundMurderType.class;
      document.getElementById("finalDetail").textContent = roundMurderType.desc;
      if ("vibrate" in navigator) navigator.vibrate(200);
    } else { alert("게임 시작! (순수 모드)"); location.reload(); }
    return;
  }
  document.getElementById("result").style.display = "none";
  document.getElementById("draw-area").style.display = "block";
  document.getElementById("playerNum").textContent = `PLAYER ${current + 1}`;
}
</script>
</body>
</html>
