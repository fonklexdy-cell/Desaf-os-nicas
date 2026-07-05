
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MateKen2 - Desafío Nica</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Fredoka+One&family=Nunito:wght@400;700&display=swap');
        body { font-family: 'Nunito', sans-serif; background-color: #f0fdf4; margin: 0; display: flex; justify-content: center; align-items: center; min-height: 100vh; }
        .game-card { background: white; border-radius: 2rem; box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1); width: 95%; max-width: 600px; border: 4px solid #22c55e; overflow: hidden; }
        .header { background: linear-gradient(135deg, #22c55e, #0ea5e9); padding: 1.5rem; text-align: center; color: white; }
        h1 { font-family: 'Fredoka One', cursive; font-size: 2rem; }
        .robot-container { font-size: 5rem; margin: 1rem 0; animation: float 3s ease-in-out infinite; }
        @keyframes float { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-10px); } }
        .btn-option { transition: all 0.2s; border: 2px solid #e2e8f0; width: 100%; cursor: pointer; padding: 1rem; border-radius: 1rem; font-weight: bold; background: white; }
        .btn-option:hover { transform: scale(1.02); border-color: #0ea5e9; background-color: #f0f9ff; }
        .progress-bar { height: 12px; background-color: #e2e8f0; border-radius: 6px; overflow: hidden; }
        .progress-fill { height: 100%; background: linear-gradient(90deg, #fbbf24, #f97316); transition: width 0.5s ease; }
        .feedback { display: none; padding: 1rem; border-radius: 1rem; margin-top: 1rem; text-align: center; font-weight: bold; }
        .correct { background-color: #dcfce7; color: #166534; display: block; }
        .incorrect { background-color: #fee2e2; color: #991b1b; display: block; }
        .btn-start { background-color: #f97316; color: white; padding: 1rem 3rem; font-size: 1.5rem; border-radius: 9999px; font-family: 'Fredoka One', cursive; box-shadow: 0 4px 0px #c2410c; }
        .score-display { font-family: 'Fredoka One', cursive; font-size: 4rem; color: #0ea5e9; }
        .btn-flotante-mateken {
    position: fixed;
    bottom: 25px;
    right: 25px;
    background-color: #FFD166;
    color: #073B4C;
    padding: 15px 22px;
    font-size: 1.1rem;
    font-weight: bold;
    text-decoration: none;
    border-radius: 50px;
    box-shadow: 0 6px 15px rgba(0,0,0,0.4);
    display: flex;
    align-items: center;
    gap: 10px;
    transition: transform 0.2s, background-color 0.2s;
    z-index: 9999;
    font-family: sans-serif;
}
.btn-flotante-mateken:hover {
    transform: scale(1.1);
    background-color: #fffde7;
}
    
    </style>
</head>
<body>

<div class="game-card">
    <div class="header">
        <h1>MateKen2</h1>
        <p>¡Nuevos desafíos nicas!</p>
    </div>

    <div class="p-6 text-center">
        <div id="start-screen">
            <div class="robot-container">🤖</div>
            <h2 class="text-2xl font-bold text-gray-800 mb-4">¡Hola! Soy Ken</h2>
            <p class="text-gray-600 mb-8">¿Listo para resolver las cuentas de la casa?</p>
            <button onclick="startGame()" class="btn-start">✨ EMPEZAR ✨</button>
        </div>

        <div id="game-screen" class="hidden">
            <div class="robot-container" id="robot">🤖</div>
            <div class="mb-4">
                <div class="progress-bar"><div id="energy-bar" class="progress-fill" style="width: 0%"></div></div>
            </div>
            <div id="question-box" class="bg-blue-50 p-4 rounded-xl mb-6 border-l-4 border-blue-500 text-left">
                <p id="question-text" class="text-lg text-blue-900 font-bold"></p>
            </div>
            <div id="options-container" class="grid grid-cols-1 gap-3"></div>
            <div id="feedback-msg" class="feedback"></div>
        </div>

        <div id="win-screen" class="hidden">
            <div class="text-6xl mb-4">🏆</div>
            <h2 class="text-3xl font-bold text-gray-800">¡Lo lograste!</h2>
            <div id="final-score" class="score-display">0 pts</div>
            <button onclick="startGame()" class="mt-6 bg-blue-500 text-white font-bold py-3 px-8 rounded-full shadow-lg">Jugar otra vez</button>
        </div>
    </div>
</div>

<script>
    const soundCorrect = new Audio('https://assets.mixkit.co/active_storage/sfx/2000/2000-preview.mp3');
    const soundWrong = new Audio('https://assets.mixkit.co/active_storage/sfx/2005/2005-preview.mp3');
    const soundWin = new Audio('https://assets.mixkit.co/active_storage/sfx/2020/2020-preview.mp3');

    const bancoPreguntas = [
        { q: "4 plátanos a C$5 c/u. ¿Total?", o: ["C$20", "C$15", "C$25"], c: 0, r: "🍌" },
        { q: "3 frescos a C$7 c/u. ¿Total?", o: ["C$14", "C$21", "C$28"], c: 1, r: "🥤" },
        { q: "2 Vigorón a C$60 c/u. ¿Total?", o: ["C$100", "C$120", "C$110"], c: 1, r: "🍽️" },
        { q: "Pasaje de bus: C$2.50 x 4 viajes. ¿Total?", o: ["C$8", "C$10", "C$12"], c: 1, r: "🚌" },
        { q: "C$100 menos C$45 de carne. ¿Te queda?", o: ["C$55", "C$65", "C$45"], c: 0, r: "🥩" },
        { q: "6 aguacates a C$10 c/u. ¿Total?", o: ["C$50", "C$60", "C$70"], c: 1, r: "🥑" },
        { q: "3 libras de arroz a C$18. ¿Total?", o: ["C$54", "C$44", "C$64"], c: 0, r: "🍚" },
        { q: "2 galones de leche a C$80. ¿Total?", o: ["C$150", "C$160", "C$170"], c: 1, r: "🥛" },
        { q: "Compraste pan por C$25 y pagaste con C$50. ¿Vuelto?", o: ["C$20", "C$25", "C$35"], c: 1, r: "🍞" },
        { q: "Taxi: C$50 + C$20 de propina. ¿Total?", o: ["C$60", "C$80", "C$70"], c: 2, r: "🚕" }
    ];

    let currentLevel = 0, correctAnswers = 0, activeChallenges = [];

    function startGame() {
        // Barajamos el banco y tomamos 8 preguntas aleatorias cada vez
        activeChallenges = [...bancoPreguntas].sort(() => Math.random() - 0.5).slice(0, 8);
        currentLevel = 0;
        correctAnswers = 0;
        document.getElementById('start-screen').classList.add('hidden');
        document.getElementById('win-screen').classList.add('hidden');
        document.getElementById('game-screen').classList.remove('hidden');
        loadChallenge();
    }

    function loadChallenge() {
        const c = activeChallenges[currentLevel];
        document.getElementById('question-text').innerText = c.q;
        document.getElementById('robot').innerText = c.r;
        const cont = document.getElementById('options-container');
        cont.innerHTML = '';
        document.getElementById('feedback-msg').className = 'feedback';

        c.o.forEach((op, i) => {
            const b = document.createElement('button');
            b.innerText = op;
            b.className = 'btn-option';
            b.onclick = () => {
                if (i === c.c) {
                    soundCorrect.play();
                    correctAnswers++;
                    document.getElementById('feedback-msg').innerText = "✨ ¡Correcto! ✨";
                    document.getElementById('feedback-msg').className = "feedback correct";
                    [...document.querySelectorAll('.btn-option')].forEach(btn => btn.disabled = true);
                    setTimeout(nextLevel, 1200);
                } else {
                    soundWrong.play();
                    document.getElementById('feedback-msg').innerText = "❌ ¡Inténtalo de nuevo!";
                    document.getElementById('feedback-msg').className = "feedback incorrect";
                }
            };
            cont.appendChild(b);
        });
        document.getElementById('energy-bar').style.width = (currentLevel / activeChallenges.length) * 100 + '%';
    }

    function nextLevel() {
        currentLevel++;
        if (currentLevel < activeChallenges.length) {
            loadChallenge();
        } else {
            soundWin.play();
            document.getElementById('game-screen').classList.add('hidden');
            document.getElementById('win-screen').classList.remove('hidden');
            document.getElementById('final-score').innerText = `${Math.round((correctAnswers/activeChallenges.length)*100)} pts`;
        }
    }
</script>

<a href="https://tinyurl.com/mathken2" target="_blank" class="btn-flotante-mateken">
        🤖 Ir a MateKen2
    </a>
</body>
</html>
