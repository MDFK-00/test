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
}

.quiz-item:hover {
    background: #2a2a33;
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
    min-width: 0;
    display: flex;
    flex-direction: column;
}

.topbar {
    padding: 15px;
    border-bottom: 1px solid #2a2a2a;
    background: rgba(20,20,25,0.6);
}

.content {
    flex: 1;
    display: flex;
    justify-content: center;
    align-items: center;
}

.card {
    width: min(700px, 90%);
    background: rgba(30,30,40,0.7);
    padding: 25px;
    border-radius: 16px;
    border: 1px solid #333;
}

/* HUD */
.hud {
    position: fixed;
    top: 10px;
    right: 10px;
    background: rgba(20,20,25,0.85);
    border: 1px solid #333;
    padding: 10px;
    border-radius: 12px;
    font-size: 13px;
}

/* OPTIONS */
.option {
    background: #222;
    padding: 10px;
    margin: 8px 0;
    border-radius: 10px;
    cursor: pointer;
}

.option:hover {
    background: #333;
}

input.answer-box {
    width: 100%;
    padding: 10px;
    border-radius: 10px;
    background: #1b1b1b;
    color: white;
    border: 1px solid #333;
}

/* FLASH */
@keyframes red {
    0% { background: transparent; }
    40% { background: rgba(255,0,0,0.25); }
    100% { background: transparent; }
}

@keyframes green {
    0% { background: transparent; }
    40% { background: rgba(0,255,0,0.25); }
    100% { background: transparent; }
}

.flash-red { animation: red 0.4s; }
.flash-green { animation: green 0.4s; }

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

    <button class="danger-btn" onclick="clearAll()">
        🗑 Clear All
    </button>
</div>

<!-- MAIN -->
<div class="main">
    <div class="topbar">Dark Quiz Engine</div>

    <div class="content">
        <div class="card" id="card">
            <h2>Select or upload a quiz</h2>
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
function load() {
    const saved = localStorage.getItem("quizzes");
    if (saved) {
        try {
            quizzes = JSON.parse(saved).filter(isValidQuiz);
        } catch {
            quizzes = [];
        }
    }
    save();
    render();
}
load();

/* VALIDATION (IMPORTANT FIX) */
function isValidQuiz(q) {
    return q &&
        typeof q.title === "string" &&
        Array.isArray(q.questions) &&
        q.questions.length > 0;
}

/* SAVE */
function save() {
    localStorage.setItem("quizzes", JSON.stringify(quizzes));
}

/* HUD */
function updateHUD() {
    const total = currentQuiz ? currentQuiz.questions.length : 0;
    const progress = total ? Math.round((index / total) * 100) : 0;

    document.getElementById("hud").innerHTML =
        `Score: ${score} | Mistakes: ${mistakes} | Progress: ${progress}%`;
}

/* RENDER LIST */
function render() {
    const list = document.getElementById("quizList");
    list.innerHTML = "";

    quizzes.forEach((q, i) => {
        const div = document.createElement("div");
        div.className = "quiz-item";

        const title = document.createElement("span");
        title.textContent = q.title;
        title.onclick = () => start(i);

        const del = document.createElement("button");
        del.textContent = "✖";
        del.style.background = "transparent";
        del.style.color = "#ff5c5c";
        del.style.border = "none";
        del.onclick = (e) => {
            e.stopPropagation();
            quizzes.splice(i, 1);
            save();
            render();
        };

        div.appendChild(title);
        div.appendChild(del);
        list.appendChild(div);
    });
}

/* UPLOAD SAFE */
document.getElementById("upload").addEventListener("change", e => {
    const file = e.target.files[0];
    const reader = new FileReader();

    reader.onload = ev => {
        try {
            const quiz = JSON.parse(ev.target.result);

            if (!isValidQuiz(quiz)) {
                alert("Invalid quiz format");
                return;
            }

            quizzes.push(quiz);
            save();
            render();

            if (!currentQuiz) start(0);

        } catch {
            alert("Invalid JSON");
        }
    };

    reader.readAsText(file);
});

/* START QUIZ (FIXED SAFE VERSION) */
function start(i) {
    const q = quizzes[i];

    if (!isValidQuiz(q)) {
        alert("Broken quiz detected");
        return;
    }

    currentQuiz = q;
    index = 0;
    score = 0;
    mistakes = 0;

    show();
}

/* NORMALIZE */
function norm(x) {
    return String(x).toLowerCase().trim();
}

/* FLASH */
function flash(type) {
    const c = document.getElementById("card");
    c.classList.add(type);
    setTimeout(() => c.classList.remove(type), 400);
}

/* SHOW */
function show() {
    const c = document.getElementById("card");

    if (!currentQuiz) {
        c.innerHTML = "<h2>Select a quiz</h2>";
        return;
    }

    const q = currentQuiz.questions[index];

    if (!q) {
        c.innerHTML = `
            <h2>Finished 🎉</h2>
            <p>Score: ${score}</p>
            <p>Mistakes: ${mistakes}</p>
        `;
        updateHUD();
        return;
    }

    let html = `<div><b>Q${index+1}:</b> ${q.question}</div>`;

    if (q.type === "mcq") {
        q.choices.forEach(a => {
            html += `<div class="option" onclick="mcq('${a}')">${a}</div>`;
        });
    } else if (q.type === "tf") {
        html += `
            <div class="option" onclick="tf(true)">True</div>
            <div class="option" onclick="tf(false)">False</div>
        `;
    } else {
        html += `
            <input class="answer-box" id="ans"/>
            <button onclick="id()">Submit</button>
        `;
    }

    c.innerHTML = html;
    updateHUD();
}

/* ANSWERS */
function mcq(a) {
    if (norm(a) === norm(currentQuiz.questions[index].answer)) {
        score++; flash("flash-green");
    } else {
        mistakes++; flash("flash-red");
    }
    index++; show();
}

function tf(a) {
    if (a === currentQuiz.questions[index].answer) {
        score++; flash("flash-green");
    } else {
        mistakes++; flash("flash-red");
    }
    index++; show();
}

function id() {
    const v = norm(document.getElementById("ans").value);
    const c = norm(currentQuiz.questions[index].answer);

    if (v === c) {
        score++; flash("flash-green");
    } else {
        mistakes++; flash("flash-red");
    }
    index++; show();
}

/* CLEAR ALL */
function clearAll() {
    quizzes = [];
    currentQuiz = null;
    localStorage.removeItem("quizzes");
    render();
    document.getElementById("card").innerHTML = "<h2>All cleared</h2>";
}

</script>

</body>
</html>
