<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Reaction Clash 2P</title>
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
  #result {
    font-size: 1.5em;
    margin-top: 20px;
  }
  #best {
    margin-top: 10px;
    font-size: 1.1em;
    color: #aaa;
  }
</style>
</head>
<body>
  <div id="message">👾 2인 반응속도 대결 👾<br><br>Player 1: <b>A</b> 키<br>Player 2: <b>L</b> 키<br><br>시작하려면 버튼을 누르세요.</div>
  <button id="startBtn">Start</button>
  <div id="result"></div>
  <div id="best"></div>

<script>
  const msg = document.getElementById('message');
  const startBtn = document.getElementById('startBtn');
  const resultDiv = document.getElementById('result');
  const bestDiv = document.getElementById('best');

  let startTime, timeoutId;
  let ready = false;
  let winnerDeclared = false;
  let bestP1 = localStorage.getItem('bestP1') || null;
  let bestP2 = localStorage.getItem('bestP2') || null;

  const updateBest = () => {
    bestDiv.innerHTML = `📈 최고기록<br>
    Player 1: ${bestP1 ? bestP1 + " ms" : "기록 없음"}<br>
    Player 2: ${bestP2 ? bestP2 + " ms" : "기록 없음"}`;
  };

  updateBest();

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
      msg.innerHTML = "지금 눌러요! 💥";
      document.body.style.backgroundColor = "#2ecc71";
      startTime = Date.now();
      ready = true;
    }, randomDelay);
  });

  document.addEventListener('keydown', (e) => {
    if (winnerDeclared) return; // 이미 끝났으면 무시

    const key = e.key.toLowerCase();

    if (!ready) {
      // 조기 입력
      clearTimeout(timeoutId);
      document.body.style.backgroundColor = "#e74c3c";
      if (key === 'a') {
        msg.textContent = "Player 1이 너무 빨랐어요! Player 2 승! 🏆";
      } else if (key === 'l') {
        msg.textContent = "Player 2가 너무 빨랐어요! Player 1 승! 🏆";
      }
      startBtn.style.display = "block";
      winnerDeclared = true;
      return;
    }

    // 정상 입력 (ready 상태)
    const reaction = Date.now() - startTime;
    document.body.style.backgroundColor = "#3498db";
    let winner = "";
    if (key === 'a') {
      winner = "Player 1";
      msg.textContent = `🎯 ${winner} 승! 반응속도: ${reaction} ms`;
      if (!bestP1 || reaction < bestP1) {
        bestP1 = reaction;
        localStorage.setItem('bestP1', bestP1);
      }
    } else if (key === 'l') {
      winner = "Player 2";
      msg.textContent = `🎯 ${winner} 승! 반응속도: ${reaction} ms`;
      if (!bestP2 || reaction < bestP2) {
        bestP2 = reaction;
        localStorage.setItem('bestP2', bestP2);
      }
    } else {
      return; // 다른 키면 무시
    }

    updateBest();
    winnerDeclared = true;
    ready = false;
    startBtn.style.display = "block";
  });
</script>
</body>
</html>
