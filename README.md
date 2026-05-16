<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Dark Quiz Engine</title>

<style>
    * {
        box-sizing: border-box;
    }

    body {
        margin: 0;
        font-family: Arial, sans-serif;
        background: radial-gradient(circle at top, #1a1a1f, #0a0a0c);
        color: #eaeaea;
        height: 100vh;
        display: flex;
        overflow: hidden;
    }

    /* SIDEBAR */
    .sidebar {
        width: 280px;
        min-width: 280px;
        height: 100vh;
        background: rgba(20,20,25,0.9);
        backdrop-filter: blur(10px);
        border-right: 1px solid #2a2a2a;
        padding: 15px;
        display: flex;
        flex-direction: column;
        overflow-y: auto;
    }

    .title {
        font-size: 14px;
        color: #aaa;
        margin-bottom: 10px;
    }

    .quiz-item {
        padding: 10px;
        margin: 6px 0;
        background: #1c1c22;
        border-radius: 8px;
        cursor: pointer;
        display: flex;
        justify-content: space-between;
        align-items: center;
    }

    .quiz-item:hover {
        background: #2a2a33;
    }

    input[type="file"] {
        margin-top: 10px;
        color: #aaa;
    }

    .danger-btn {
        margin-top: 10px;
        padding: 10px;
        border-radius: 10px;
        border: none;
        background: #ff3b3b;
        color: white;
        cursor: pointer;
    }

    /* MAIN AREA */
    .main {
        flex: 1;
        min-width: 0;
        display: flex;
        flex-direction: column;
        height: 100vh;
    }

    .topbar {
        padding: 15px;
        border-bottom: 1px solid #2a2a2a;
        background: rgba(20,20,25,0.6);
        backdrop-filter: blur(10px);
    }

    .content {
        flex: 1;
        display: flex;
        justify-content: center;
        align-items: center;
        padding: 20px;
    }

    .card {
        width: min(700px, 90%);
        background: rgba(30,30,40,0.7);
        padding: 25px;
        border-radius: 16px;
        border: 1px solid #333;
        box-shadow: 0 0 20px rgba(0,0,0,0.4);
        transition: 0.2s;
    }

    .question {
        font-size: 18px;
        margin-bottom: 20px;
    }

    .option {
        background: #222;
        padding: 10px;
        margin: 8px 0;
        border-radius: 10px;
        cursor: pointer;
        transition: 0.2s;
    }

    .option:hover {
        background: #333;
    }

    input.answer-box {
        width: 100%;
        padding: 10px;
        border-radius: 10px;
        border: 1px solid #333;
        background: #1b1b1b;
        color: white;
    }

    button {
        margin-top: 10px;
        padding: 10px;
        border-radius: 10px;
        border: none;
        background: #4c7dff;
        color: white;
        cursor: pointer;
    }

    /* HUD */
    .hud {
        position: fixed;
        top: 10px;
        right: 10px;
        background: rgba(20,20,25,0.85);
        border: 1px solid #333;
        padding: 10px 14px;
        border-radius: 12px;
        font-size: 13px;
    }

    /* FLASH EFFECTS */
    @keyframes flashRed {
        0% { background: rgba(255,0,0,0); }
        40% { background: rgba(255,0,0,0.25); }
        100% { background: rgba(255,0,0,0); }
    }

    @keyframes flashGreen {
        0% { background: rgba(0,255,0,0); }
        40% { background: rgba(0,255,0,0.25); }
        100% { background: rgba(0,255,0,0); }
    }

    .flash-red {
        animation: flashRed 0.4s ease;
    }

    .flash-green {
        animation: flashGreen 0.4s ease;
    }
</style>
</head>

<body>

<div class="hud" id="hud">
    Score: 0 | Mistakes: 0 | Progress: 0%
</div>

<!-- SIDEBAR -->
<div class="sidebar">
    <div class="title">📁 Quiz Library</div>

    <div id="quizList"></div>

    <input type="file" id="upload" accept=".json"/>

    <button class="danger-btn" onclick="clearAllQuizzes()">
        🗑 Clear All Quizzes
    </button>
</div>

<!-- MAIN -->
<div class="main">
    <div class="topbar">Dark Quiz Engine</div>

    <div class="content">
        <div class="card" id="quizCard">
            <h2>Upload or select a quiz</h2>
        </div>
    </div>
</div>

<script>
let quizzes = [];
let currentQuiz = null;
let index = 0;

let score = 0;
let mistakes = 0;

/* LOAD */
function loadSaved() {
    const saved = localStorage.getItem("quizzes");
    if (saved) quizzes = JSON.parse(saved);
    renderList();
}
loadSaved();

/* HUD */
function updateHUD() {
    const hud = document.getElementById("hud");

    const total = currentQuiz ? currentQuiz.questions.length : 0;
    const progress = total ? Math.round((index / total) * 100) : 0;

    hud.innerHTML = `
        Score: ${score} | Mistakes: ${mistakes} | Progress: ${progress}%
    `;
}

/* RENDER LIST */
function renderList() {
    const list = document.getElementById("quizList");
    list.innerHTML = "";

    quizzes.forEach((q, i) => {
        const div = document.createElement("div");
        div.className = "quiz-item";

        const title = document.createElement("span");
        title.textContent = q.title;
        title.onclick = () => startQuiz(i);

        const del = document.createElement("button");
        del.textContent = "✖";
        del.style.background = "transparent";
        del.style.color = "#ff5c5c";
        del.style.border = "none";
        del.onclick = (e) => {
            e.stopPropagation();
            deleteQuiz(i);
        };

        div.appendChild(title);
        div.appendChild(del);

        list.appendChild(div);
    });
}

/* SAFE UPLOAD */
document.getElementById("upload").addEventListener("change", function(e){
    const file = e.target.files[0];
    const reader = new FileReader();

    reader.onload = function(event){
        try {
            const quiz = JSON.parse(event.target.result);

            if (!quiz.title || !quiz.questions) {
                alert("Invalid quiz format");
                return;
            }

            quizzes.push(quiz);
            localStorage.setItem("quizzes", JSON.stringify(quizzes));
            renderList();

            if (!currentQuiz) startQuiz(0);

        } catch {
            alert("Invalid JSON file");
        }
    };

    reader.readAsText(file);
});

/* START */
function startQuiz(i) {
    currentQuiz = quizzes[i];
    index = 0;
    score = 0;
    mistakes = 0;
    showQuestion();
    updateHUD();
}

/* NORMALIZE */
function normalize(str) {
    return String(str).toLowerCase().trim();
}

/* FLASHES */
function flashRed() {
    const card = document.getElementById("quizCard");
    card.classList.add("flash-red");
    setTimeout(() => card.classList.remove("flash-red"), 400);
}

function flashGreen() {
    const card = document.getElementById("quizCard");
    card.classList.add("flash-green");
    setTimeout(() => card.classList.remove("flash-green"), 400);
}

/* QUESTION */
function showQuestion() {
    const card = document.getElementById("quizCard");

    if (!currentQuiz || !currentQuiz.questions) {
        card.innerHTML = "<h2>Select a quiz</h2>";
        return;
    }

    const q = currentQuiz.questions[index];

    if (!q) {
        card.innerHTML = `
            <h2>Finished 🎉</h2>
            <p>Score: ${score}</p>
            <p>Mistakes: ${mistakes}</p>
        `;
        updateHUD();
        return;
    }

    let html = `<div class="question">Q${index+1}: ${q.question}</div>`;

    if (q.type === "mcq") {
        q.choices.forEach(c => {
            html += `<div class="option" onclick="answerMCQ('${c}')">${c}</div>`;
        });
    } else if (q.type === "tf") {
        html += `
            <div class="option" onclick="answerTF(true)">True</div>
            <div class="option" onclick="answerTF(false)">False</div>
        `;
    } else {
        html += `
            <input class="answer-box" id="idAnswer"/>
            <button onclick="answerID()">Submit</button>
        `;
    }

    card.innerHTML = html;
    updateHUD();
}

/* ANSWERS */
function answerMCQ(ans) {
    const correct = normalize(currentQuiz.questions[index].answer);

    if (normalize(ans) === correct) {
        score++;
        flashGreen();
    } else {
        mistakes++;
        flashRed();
    }

    index++;
    showQuestion();
    updateHUD();
}

function answerTF(ans) {
    const correct = currentQuiz.questions[index].answer;

    if (ans === correct) {
        score++;
        flashGreen();
    } else {
        mistakes++;
        flashRed();
    }

    index++;
    showQuestion();
    updateHUD();
}

function answerID() {
    const user = normalize(document.getElementById("idAnswer").value);
    const correct = normalize(currentQuiz.questions[index].answer);

    if (user === correct) {
        score++;
        flashGreen();
    } else {
        mistakes++;
        flashRed();
    }

    index++;
    showQuestion();
    updateHUD();
}

/* DELETE */
function deleteQuiz(i) {
    quizzes.splice(i, 1);
    localStorage.setItem("quizzes", JSON.stringify(quizzes));
    renderList();

    if (quizzes.length === 0) {
        currentQuiz = null;
        document.getElementById("quizCard").innerHTML =
            "<h2>No quizzes available</h2>";
    }
}

/* CLEAR ALL */
function clearAllQuizzes() {
    if (!confirm("Delete all quizzes?")) return;

    quizzes = [];
    localStorage.removeItem("quizzes");
    renderList();

    currentQuiz = null;
    document.getElementById("quizCard").innerHTML =
        "<h2>All quizzes cleared</h2>";
}
</script>

</body>
</html>
