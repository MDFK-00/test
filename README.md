<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Dark Quiz Engine</title>

<style>
    body {
        margin: 0;
        font-family: Arial, sans-serif;
        background: radial-gradient(circle at top, #1a1a1f, #0a0a0c);
        color: #eaeaea;
        display: flex;
        height: 100vh;
        overflow: hidden;
    }

    /* SIDEBAR */
    .sidebar {
        width: 280px;
        background: rgba(20,20,25,0.85);
        backdrop-filter: blur(10px);
        border-right: 1px solid #2a2a2a;
        padding: 15px;
        display: flex;
        flex-direction: column;
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
        transition: 0.2s;
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

    button {
        cursor: pointer;
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

    /* MAIN */
    .main {
        flex: 1;
        display: flex;
        flex-direction: column;
        position: relative;
    }

    .topbar {
        padding: 15px;
        border-bottom: 1px solid #2a2a2a;
        background: rgba(20,20,25,0.6);
        backdrop-filter: blur(10px);
    }

    .content {
        flex: 1;
        padding: 25px;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .card {
        width: 60%;
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

    .hud {
        position: fixed;
        top: 10px;
        right: 10px;
        background: rgba(20,20,25,0.85);
        border: 1px solid #333;
        padding: 10px 14px;
        border-radius: 12px;
        font-size: 13px;
        color: #ddd;
        backdrop-filter: blur(10px);
    }

    @keyframes flashRed {
        0% { background: rgba(255,0,0,0.0); }
        30% { background: rgba(255,0,0,0.25); }
        100% { background: rgba(255,0,0,0.0); }
    }

    .flash {
        animation: flashRed 0.4s ease;
    }
</style>
</head>

<body>

<div class="hud" id="hud">
    Score: 0 | Mistakes: 0 | Progress: 0%
</div>

<div class="sidebar">
    <div class="title">📁 Quiz Library</div>

    <div id="quizList"></div>

    <input type="file" id="upload" accept=".json"/>

    <button class="danger-btn" onclick="clearAllQuizzes()">
        🗑 Clear All Quizzes
    </button>
</div>

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
        title.style.cursor = "pointer";
        title.onclick = () => startQuiz(i);

        const del = document.createElement("button");
        del.textContent = "✖";
        del.style.background = "transparent";
        del.style.border = "none";
        del.style.color = "#ff5c5c";
        del.onclick = (e) => {
            e.stopPropagation();
            deleteQuiz(i);
        };

        div.appendChild(title);
        div.appendChild(del);

        list.appendChild(div);
    });
}

/* UPLOAD */
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

        } catch (err) {
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

/* FLASH */
function flashRed() {
    const card = document.getElementById("quizCard");
    card.classList.add("flash");

    setTimeout(() => {
        card.classList.remove("flash");
    }, 400);
}

/* QUESTION */
function showQuestion() {
    const card = document.getElementById("quizCard");
    const q = currentQuiz.questions[index];

    if (!q) {
        card.innerHTML = `
            <h2>Quiz Finished 🎉</h2>
            <p>Score: ${score}</p>
            <p>Mistakes: ${mistakes}</p>
            <p>Final: ${score} / ${currentQuiz.questions.length}</p>
        `;
        updateHUD();
        return;
    }

    let html = `
        <div class="question">
            Q${index+1}: ${q.question}
        </div>
    `;

    if (q.type === "mcq") {
        q.choices.forEach(choice => {
            html += `<div class="option" onclick="answerMCQ('${choice}')">${choice}</div>`;
        });
    }

    else if (q.type === "tf") {
        html += `
            <div class="option" onclick="answerTF(true)">True</div>
            <div class="option" onclick="answerTF(false)">False</div>
        `;
    }

    else if (q.type === "id") {
        html += `
            <input class="answer-box" id="idAnswer" placeholder="Type answer..." />
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
    } else {
        mistakes++;
        flashRed();
    }

    index++;
    showQuestion();
    updateHUD();
}

/* DELETE SINGLE QUIZ */
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
    if (!confirm("Delete ALL quizzes?")) return;

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
