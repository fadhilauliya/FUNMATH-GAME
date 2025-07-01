# FUNMATH-GAME
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>FunMath Game</title>
  <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap" rel="stylesheet">
  <style>
    * { box-sizing: border-box; }
    body {
      font-family: 'Poppins', sans-serif;
      margin: 0;
      background: url('https://img.freepik.com/free-vector/hand-drawn-chalkboard-math-background_23-2148499325.jpg') no-repeat center center fixed;
      background-size: cover;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
    }
    .slide {
      display: none;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      animation: fadeIn 1s ease;
      width: 100%;
      max-width: 500px;
    }
    .slide.active { display: flex; }
    .container {
      background: rgba(255, 255, 255, 0.95);
      border-radius: 20px;
      box-shadow: 0 12px 24px rgba(0,0,0,0.2);
      padding: 30px;
      width: 90%;
      max-width: 480px;
      text-align: center;
    }
    @keyframes fadeIn {
      from {opacity: 0; transform: scale(0.9);}
      to {opacity: 1; transform: scale(1);}
    }
    h2 { color: #3f3d56; margin-bottom: 10px; }
    p { font-size: 14px; text-align: left; color: #555; }
    select, input[type="number"], input[type="range"] {
      width: 100%; padding: 10px; margin-top: 10px; margin-bottom: 15px;
      border: 1px solid #ccc; border-radius: 10px; font-size: 16px;
    }
    button {
      background-color: #4a47a3; color: #fff;
      padding: 10px 20px; border: none;
      border-radius: 10px; font-size: 16px;
      cursor: pointer; transition: 0.3s;
      margin: 5px;
    }
    button:hover { background-color: #3c358d; }
    #question {
      font-size: 36px; font-weight: bold;
      color: #333; min-height: 60px;
      transition: transform 0.4s;
    }
    .correct {
      animation: celebratory 1s ease forwards;
      color: #28a745; font-weight: bold;
    }
    .wrong {
      animation: disappointed 1s ease forwards;
      color: #dc3545; font-weight: bold;
    }
    @keyframes celebratory {
      0% { transform: scale(1); }
      50% { transform: scale(1.2); background-color: #d4edda; }
      100% { transform: scale(1); background-color: transparent; }
    }
    @keyframes disappointed {
      0% { transform: translateY(0); }
      50% { transform: translateY(-10px); background-color: #f8d7da; }
      100% { transform: translateY(0); background-color: transparent; }
    }
    #timer { font-size: 18px; color: #374785; margin: 10px 0; }
    #input-timer {
      font-size: 18px; color: #e74c3c;
      animation: pulse 1s infinite;
    }
    @keyframes pulse {
      0% { transform: scale(1); }
      50% { transform: scale(1.1); }
      100% { transform: scale(1); }
    }
  </style>
</head>
<body>
  <div class="container">
    <div class="slide active" id="slide-peraturan">
      <h2>🎮 FunMath Game</h2>
      <p>
        🔢 Peraturan:<br>
        1. Angka dan operator muncul satu per satu<br>
        2. Hitung dari kiri ke kanan tanpa kaidah<br>
        3. Pilih jumlah angka soal sesuai keinginan<br>
      </p>
      <label for="jumlah"><strong>Jumlah angka yang ditampilkan:</strong></label>
      <select id="jumlah">
        <option value="10">10 Angka</option>
        <option value="15">15 Angka</option>
        <option value="20">20 Angka</option>
        <option value="25">25 Angka</option>
        <option value="30">30 Angka</option>
      </select>
      <label for="speed"><strong>Kecepatan tampilan angka (1 - 2 detik):</strong></label>
      <input type="range" id="speed" min="1000" max="2000" value="1000" step="100">
      <button onclick="mulaiGame()">Mulai Game</button>
    </div>

    <div class="slide" id="slide-main">
      <div id="timer"></div>
      <div id="question"></div>
    </div>

    <div class="slide" id="slide-input">
      <div id="input-timer"></div>
      <input type="number" id="userAnswer" placeholder="Masukkan jawaban kamu" />
      <button onclick="checkAnswer()">Kirim Jawaban</button>
      <div id="result"></div>
    </div>
  </div>

  <audio id="beep" src="https://www.soundjay.com/button/sounds/beep-07.mp3"></audio>
  <audio id="correctSound" src="https://www.soundjay.com/misc/sounds/bell-ringing-05.mp3"></audio>
  <audio id="wrongSound" src="https://www.soundjay.com/button/sounds/button-10.mp3"></audio>

  <script>
    let ekspresi = "";
    let hasil = null;
    let angka = [];
    let operator = [];
    let tampilList = [];
    let currentOp = null;
    let showIndex = 0;
    let inputTimerInterval;
    let timeLeft = 15;
    let delay = 1000;

    function mulaiGame() {
      ekspresi = "";
      hasil = null;
      angka = [];
      operator = [];
      tampilList = [];
      showIndex = 0;
      document.getElementById("result").innerHTML = "";
      document.getElementById("userAnswer").value = "";
      gantiSlide("slide-main");

      const jumlah = parseInt(document.getElementById("jumlah").value);
      delay = parseInt(document.getElementById("speed").value);
      const ops = ["+", "-", "x"];

      for (let i = 0; i < jumlah; i++) {
        if (i < jumlah - 1) {
          operator.push(ops[Math.floor(Math.random() * ops.length)]);
        }
        let value = Math.floor(Math.random() * 10) + 1;
        angka.push(value);
      }

      for (let i = 0; i < angka.length; i++) {
        tampilList.push(angka[i]);
        if (i < operator.length) tampilList.push(operator[i]);
      }

      tampilkanAngka();
    }

    function gantiSlide(id) {
      document.querySelectorAll(".slide").forEach(s => s.classList.remove("active"));
      document.getElementById(id).classList.add("active");
    }

    function tampilkanAngka() {
      const container = document.getElementById("question");
      const beep = document.getElementById("beep");

      if (showIndex < tampilList.length) {
        const item = tampilList[showIndex];
        container.textContent = item;
        beep.play();

        if (showIndex === 0) {
          hasil = item;
          ekspresi = item.toString();
        } else if (typeof item === "string") {
          currentOp = item;
          ekspresi += ` ${item}`;
        } else {
          ekspresi += ` ${item}`;
          switch (currentOp) {
            case "+": hasil += item; break;
            case "-": hasil -= item; break;
            case "x": hasil *= item; break;
          }
        }
        showIndex++;
        setTimeout(tampilkanAngka, delay);
      } else {
        setTimeout(() => {
          gantiSlide("slide-input");
          mulaiInputTimer();
        }, 500);
      }
    }

    function mulaiInputTimer() {
      timeLeft = 15;
      document.getElementById("input-timer").textContent = `⏳ Jawab dalam: ${timeLeft}s`;
      inputTimerInterval = setInterval(() => {
        timeLeft--;
        document.getElementById("input-timer").textContent = `⏳ Jawab dalam: ${timeLeft}s`;
        if (timeLeft <= 0) {
          clearInterval(inputTimerInterval);
          document.getElementById("result").innerHTML = `<div class='wrong'>⏰ Waktu habis! Jawaban: ${ekspresi} = ${hasil}</div><br><button onclick='mulaiGame()'>Main Lagi 🔁</button><button onclick='kembaliKeAwal()'>🔙 Kembali</button>`;
        }
      }, 1000);
    }

    function checkAnswer() {
      clearInterval(inputTimerInterval);
      const user = parseInt(document.getElementById("userAnswer").value);
      const res = document.getElementById("result");
      const correctSound = document.getElementById("correctSound");
      const wrongSound = document.getElementById("wrongSound");

      if (isNaN(user)) {
        res.innerHTML = "⚠️ Masukkan angka dulu!";
        return;
      }

      if (user === hasil) {
        res.innerHTML = `<div class="correct">✅ Benar!<br>${ekspresi} = ${hasil}</div>`;
        correctSound.play();
      } else {
        res.innerHTML = `<div class="wrong">❌ Salah.<br>Ekspresi: ${ekspresi} = ${hasil}</div>`;
        wrongSound.play();
      }
      res.innerHTML += `<br><button onclick="mulaiGame()">Main Lagi 🔁</button><button onclick="kembaliKeAwal()">🔙 Kembali</button>`;
    }

    function kembaliKeAwal() {
      gantiSlide("slide-peraturan");
    }
  </script>
</body>
</html>
