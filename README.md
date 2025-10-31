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
  let clickable = false; // 🔒 클릭 가능한지 여부 (보호용)
  let bestScore = localStorage.getItem('bestReaction') || null;

  if (bestScore) {
    bestDiv.textContent = `📈 최고기록: ${bestScore} ms`;
  }

  startBtn.addEventListener('click', (e) => {
    e.stopPropagation(); // body 클릭 이벤트와 충돌 방지
    msg.textContent = "기다리세요...";
    startBtn.style.display = "none";
    document.body.style.backgroundColor = "#111";
    clickable = false; // 아직 클릭하면 안 됨

    const randomDelay = Math.random() * 3000 + 2000; // 2~5초 사이
    timeoutId = setTimeout(() => {
      msg.textContent = "지금 클릭!";
      document.body.style.backgroundColor = "#2ecc71";
      startTime = Date.now();
      clickable = true; // 이제 클릭 가능
    }, randomDelay);
  });

  document.body.addEventListener('click', () => {
    if (!clickable) {
      // 아직 클릭하면 안 되는 상태
      clearTimeout(timeoutId);
      msg.textContent = "너무 빨랐어요! 😅";
      document.body.style.backgroundColor = "#e74c3c";
      startBtn.style.display = "block";
      scoreDiv.textContent = "";
    } else if (clickable && msg.textContent === "지금 클릭!") {
      // 정상적인 반응
      const reaction = Date.now() - startTime;
      msg.textContent = `반응속도: ${reaction} ms`;
      document.body.style.backgroundColor = "#3498db";
      scoreDiv.textContent = reaction < 200 ? "⚡ 반응신이시군요!" : "";
      startBtn.style.display = "block";
      clickable = false;

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
