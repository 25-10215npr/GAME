<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Reaction Clash</title>
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
  #score {
    margin-top: 20px;
    font-size: 1.3em;
  }
  #best {
    margin-top: 10px;
    font-size: 1.1em;
    color: #aaa;
  }
</style>
</head>
<body>
  <div id="message">시작하려면 아래 버튼을 누르세요</div>
  <button id="startBtn">Start</button>
  <div id="score"></div>
  <div id="best"></div>

<script>
  const msg = document.getElementById('message');
  const startBtn = document.getElementById('startBtn');
  const scoreDiv = document.getElementById('score');
  const bestDiv = document.getElementById('best');

  let startTime, timeoutId;
  let bestScore = localStorage.getItem('bestReaction') || null;

  if (bestScore) {
    bestDiv.textContent = `📈 최고기록: ${bestScore} ms`;
  }

  startBtn.addEventListener('click', () => {
    msg.textContent = "기다리세요...";
    startBtn.style.display = "none";
    document.body.style.backgroundColor = "#111";

    const randomDelay = Math.random() * 3000 + 2000; // 2~5초 사이
    timeoutId = setTimeout(() => {
      msg.textContent = "지금 클릭!";
      document.body.style.backgroundColor = "#2ecc71";
      startTime = Date.now();
    }, randomDelay);
  });

  document.body.addEventListener('click', () => {
    if (msg.textContent === "기다리세요...") {
      // 조기 클릭 (페널티)
      clearTimeout(timeoutId);
      msg.textContent = "너무 빨랐어요! 😅";
      document.body.style.backgroundColor = "#e74c3c";
      startBtn.style.display = "block";
      scoreDiv.textContent = "";
    } else if (msg.textContent === "지금 클릭!") {
      // 성공적인 클릭
      const reaction = Date.now() - startTime;
      msg.textContent = `반응속도: ${reaction} ms`;
      document.body.style.backgroundColor = "#3498db";
      scoreDiv.textContent = reaction < 200 ? "⚡ 반응신이시군요!" : "";
      startBtn.style.display = "block";

      if (!bestScore || reaction < bestScore) {
        bestScore = reaction;
        localStorage.setItem('bestReaction', bestScore);
        bestDiv.textContent = `📈 최고기록: ${bestScore} ms`;
      }
    }
  });
</script>
</body>
</html>
