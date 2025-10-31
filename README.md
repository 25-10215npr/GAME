<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Reaction Clash 2P - Ranking Edition</title>
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
    margin-bottom: 20px;
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
    margin-top: 15px;
  }
  #best {
    margin-top: 15px;
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
  #ranking {
    margin-top: 30px;
    color: #ffcc00;
    font-size: 1.1em;
  }
  #ranking h3 {
    margin-bottom: 5px;
    color: #fff;
  }
</style>
</head>
<body>
  <div id="message">코더즈 자체제작 반응속도 게임</div>

  <div id="nameForm">
    <div>
      Player 1 이름: <input type="text" id="p1Name" placeholder="예: 철수">
    </div>
    <div>
      Player 2 이름: <input type="text" id="p2Name" placeholder="예: 영희">
    </div>
    <button id="saveNames">이름 저장</button>
    <div id="warning"></div>
  </div>

  <button id="startBtn" style="display:none;">Start</button>
  <div id="result"></div>
  <div id="best"></div>

  <div id="ranking">
    <h3>🏁 TOP 3 반응속도 랭킹</h3>
    <ol id="rankingList"></ol>
  </div>

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
  const rankingList = document.getElementById('rankingList');

  let startTime, timeoutId;
  let ready = false;
  let winnerDeclared = false;
  let p1Name = "", p2Name = "";
  let bestScores = JSON.parse(localStorage.getItem('reactionBestScores') || "{}");
  let rankings = JSON.parse(localStorage.getItem('reactionRankings') || "[]");

  // --- 랭킹 업데이트 함수 ---
  function updateRanking() {
    rankingList.innerHTML = "";
    const top3 = rankings
      .sort((a, b) => a.score - b.score)
      .slice(0, 3);
    if (top3.length === 0) {
      rankingList.innerHTML = "<li>기록 없음</li>";
    } else {
      top3.forEach((r, i) => {
        const li = document.createElement("li");
        li.textContent = `${i + 1}. ${r.name} - ${r.score} ms`;
        rankingList.appendChild(li);
      });
    }
  }

  updateRanking();

  // --- 개별 최고 기록 표시 ---
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

    const randomDelay = Math.random() * 3000 + 2000;
    timeoutId = setTimeout(() => {
      msg.innerHTML = "지금! 💥";
      document.body.style.backgroundColor = "#2ecc71";
      startTime = Date.now();
      ready = true;
    }, randomDelay);
  });

  document.addEventListener('keydown', (e) => {
    // 입력창 포커스 중에는 무시
    if (document.activeElement === p1Input || document.activeElement === p2Input) return;

    if (winnerDeclared) return;
    const key = e.key.toLowerCase();

    if (!ready) {
      if (key !== 'a' && key !== 'l') return;
      clearTimeout(timeoutId);
      document.body.style.backgroundColor = "#e74c3c";
      if (key === 'a') {
        msg.textContent = `시작도 안했는데 ${p1Name} 얘가 누름; ${p2Name} 승! 🏆`;
      } else if (key === 'l') {
        msg.textContent = `시작도 안했는데 ${p2Name} 얘가 누름; ${p1Name} 승! 🏆`;
      }
      startBtn.style.display = "block";
      winnerDeclared = true;
      setTimeout(resetToNameInput, 1500);
      return;
    }

    if (key !== 'a' && key !== 'l') return;

    const reaction = Date.now() - startTime;
    document.body.style.backgroundColor = "#3498db";
    let winner = "";
    if (key === 'a') {
      winner = p1Name;
    } else if (key === 'l') {
      winner = p2Name;
    }

    msg.textContent = `🎯 ${winner} 승! 반응속도: ${reaction} ms`;

    // 개인 기록 갱신
    if (!bestScores[winner] || reaction < bestScores[winner]) {
      bestScores[winner] = reaction;
    }

    // 랭킹 추가
    rankings.push({ name: winner, score: reaction });
    rankings.sort((a, b) => a.score - b.score);
    if (rankings.length > 10) rankings = rankings.slice(0, 10); // 랭킹 상위 10까지만 유지

    // 저장
    localStorage.setItem('reactionBestScores', JSON.stringify(bestScores));
    localStorage.setItem('reactionRankings', JSON.stringify(rankings));

    updateBest();
    updateRanking();

    winnerDeclared = true;
    ready = false;
    startBtn.style.display = "block";
    setTimeout(resetToNameInput, 2000); // 2초 후 이름 다시 입력
  });

  // --- 다음 라운드 준비 ---
  function resetToNameInput() {
    document.body.style.backgroundColor = "#111";
    msg.textContent = " 새 라운드! 이름을 다시 입력하세요.";
    nameForm.style.display = "flex";
    startBtn.style.display = "none";
    resultDiv.textContent = "";
    bestDiv.textContent = "";
    p1Input.value = "";
    p2Input.value = "";
    p1Input.focus();
  }
</script>
</body>
</html>
