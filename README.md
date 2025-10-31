<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Reaction Clash 2P - Name Edition</title>
<style>
  body {
    font-family: 'Arial', sans-serif;
    background-color: #111;
    color: white;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;
    margin: 0;
    transition: background-color 0.2s;
    text-align: center;
  }
  #message {
    font-size: 2em;
    margin-bottom: 30px;
  }
  button {
    background: #444;
    color: white;
    border: none;
    padding: 15px 30px;
    font-size: 1.2em;
    cursor: pointer;
    border-radius: 10px;
  }
  button:hover {
    background: #666;
  }
  input {
    padding: 10px;
    font-size: 1.1em;
    border-radius: 5px;
    border: none;
    margin: 5px;
    text-align: center;
  }
  #result {
    font-size: 1.5em;
    margin-top: 20px;
  }
  #best {
    margin-top: 10px;
    font-size: 1.1em;
    color: #aaa;
  }
  #nameForm {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 10px;
  }
  #warning {
    color: #e74c3c;
    font-size: 1em;
    margin-top: 5px;
  }
</style>
</head>
<body>
  <div id="message">코더즈 자체제작 반응속도 게임</div>

  <div id="nameForm">
    <div>
      Player 1 이름: <input type="text" id="p1Name" placeholder="예: 미녀">
    </div>
    <div>
      Player 2 이름: <input type="text" id="p2Name" placeholder="예: 야수">
    </div>
    <button id="saveNames">이름 저장</button>
    <div id="warning"></div>
  </div>

  <button id="startBtn" style="display:none;">Start</button>
  <div id="result"></div>
  <div id="best"></div>

<script>
  const msg = document.getElementById('message');
  const startBtn = document.getElementById('startBtn');
  const resultDiv = document.getElementById('result');
  const bestDiv = document.getElementById('best');
  const nameForm = document.getElementById('nameForm');
  const p1Input = document.getElementById('p1Name');
  const p2Input = document.getElementById('p2Name');
  const saveNamesBtn = document.getElementById('saveNames');
  const warning = document.getElementById('warning');

  let startTime, timeoutId;
  let ready = false;
  let winnerDeclared = false;
  let p1Name = "", p2Name = "";
  let bestScores = JSON.parse(localStorage.getItem('reactionBestScores') || "{}");

  function updateBest() {
    const p1Best = bestScores[p1Name] ? bestScores[p1Name] + " ms" : "기록 없음";
    const p2Best = bestScores[p2Name] ? bestScores[p2Name] + " ms" : "기록 없음";
    bestDiv.innerHTML = `📈 최고기록<br>${p1Name}: ${p1Best}<br>${p2Name}: ${p2Best}`;
  }

  saveNamesBtn.addEventListener('click', () => {
    const name1 = p1Input.value.trim();
    const name2 = p2Input.value.trim();

    if (!name1 || !name2) {
      warning.textContent = "⚠️ 두 플레이어 이름을 모두 입력하세요!";
      return;
    }

    warning.textContent = "";
    p1Name = name1;
    p2Name = name2;

    nameForm.style.display = "none";
    startBtn.style.display = "block";
    msg.innerHTML = `${p1Name} (A 키) vs ${p2Name} (L 키)<br><br>시작하려면 버튼을 누르세요.`;
    updateBest();
  });

  startBtn.addEventListener('click', (e) => {
    e.stopPropagation();
    msg.innerHTML = "기다리세요...";
    startBtn.style.display = "none";
    resultDiv.textContent = "";
    document.body.style.backgroundColor = "#111";
    ready = false;
    winnerDeclared = false;

    const randomDelay = Math.random() * 3000 + 2000; // 2~5초 대기
    timeoutId = setTimeout(() => {
      msg.innerHTML = "지금!";
      document.body.style.backgroundColor = "#2ecc71";
      startTime = Date.now();
      ready = true;
    }, randomDelay);
  });

  document.addEventListener('keydown', (e) => {
    // 🔒 이름 입력창에 포커스되어 있으면 게임 입력 무시
    if (document.activeElement === p1Input || document.activeElement === p2Input) {
      return;
    }

    if (winnerDeclared) return;
    const key = e.key.toLowerCase();

    if (!ready) {
      // 조기 입력
      if (key !== 'a' && key !== 'l') return;
      clearTimeout(timeoutId);
      document.body.style.backgroundColor = "#e74c3c";
      if (key === 'a') {
        msg.textContent = `어허... ${p1Name} ,시작도 전에 누르다니. ${p2Name} 승! 🏆`;
      } else if (key === 'l') {
        msg.textContent = `어허... ${p2Name} ,시작도 전에 누르다니. ${p1Name} 승! 🏆`;
      }
      startBtn.style.display = "block";
      winnerDeclared = true;
      return;
    }

    // 정상 입력
    if (key !== 'a' && key !== 'l') return;

    const reaction = Date.now() - startTime;
    document.body.style.backgroundColor = "#3498db";
    let winner = "";
    if (key === 'a') {
      winner = p1Name;
      msg.textContent = `🎯 ${winner} 승! 반응속도: ${reaction} ms`;
      if (!bestScores[p1Name] || reaction < bestScores[p1Name]) {
        bestScores[p1Name] = reaction;
      }
    } else if (key === 'l') {
      winner = p2Name;
      msg.textContent = `🎯 ${winner} 승! 반응속도: ${reaction} ms`;
      if (!bestScores[p2Name] || reaction < bestScores[p2Name]) {
        bestScores[p2Name] = reaction;
      }
    }

    localStorage.setItem('reactionBestScores', JSON.stringify(bestScores));
    updateBest();
    winnerDeclared = true;
    ready = false;
    startBtn.style.display = "block";
  });
</script>
</body>
</html>
