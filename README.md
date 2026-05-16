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
    }

    /* SIDEBAR */
    .sidebar {
        width: 280px;
        background: rgba(20,20,25,0.8);
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
    }

    .quiz-item:hover {
        background: #2a2a33;
    }

    input[type="file"] {
        margin-top: 10px;
        color: #aaa;
    }

    /* MAIN */
    .main {
        flex: 1;
        display: flex;
        flex-direction: column;
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
        flex-direction: column;
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
        margin-top: 15px;
        padding: 10px 15px;
        border: none;
        border-radius: 10px;
        background: #4c7dff;
        color: white;
        cursor: pointer;
    }

    button:hover {
        background: #3a66dd;
    }

    .progress {
        margin-bottom: 15px;
        color: #aaa;
    }
</style>
</head>

<body>

<div class="sidebar">
    <div class="title">📁 Quiz Library</div>
    <div id="quizList"></div>
    <input type="file" id="upload" accept=".json"/>
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

/* LOAD FROM LOCALSTORAGE */
function loadSaved() {
    const saved = localStorage.getItem("quizzes");
    if (saved) quizzes = JSON.parse(saved);
    renderList();
}
loadSaved();

/* RENDER LIST */
function renderList() {
    const list = document.getElementById("quizList");
    list.innerHTML = "";

    quizzes.forEach((q, i) => {
        const div = document.createElement("div");
        div.className = "quiz-item";
        div.textContent = q.title;
        div.onclick = () => startQuiz(i);
        list.appendChild(div);
    });
}

/* UPLOAD JSON */
document.getElementById("upload").addEventListener("change", function(e){
    const file = e.target.files[0];
    const reader = new FileReader();

    reader.onload = function(event){
        const quiz = JSON.parse(event.target.result);
        quizzes.push(quiz);
        localStorage.setItem("quizzes", JSON.stringify(quizzes));
        renderList();
    };

    reader.readAsText(file);
});

/* START QUIZ */
function startQuiz(i) {
    currentQuiz = quizzes[i];
    index = 0;
    score = 0;
    showQuestion();
}

/* NORMALIZE ANSWER */
function normalize(str) {
    return String(str).toLowerCase().trim();
}

/* RENDER QUESTION */
function showQuestion() {
    const card = document.getElementById("quizCard");
    const q = currentQuiz.questions[index];

    if (!q) {
        card.innerHTML = `
            <h2>Quiz Finished 🎉</h2>
            <p>Your Score: ${score} / ${currentQuiz.questions.length}</p>
        `;
        return;
    }

    let html = `<div class="progress">Question ${index+1} / ${currentQuiz.questions.length}</div>`;
    html += `<div class="question">${q.question}</div>`;

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
            <input class="answer-box" id="idAnswer" placeholder="Type your answer..." />
            <button onclick="answerID()">Submit</button>
        `;
    }

    card.innerHTML = html;
}

/* ANSWERS */
function answerMCQ(ans) {
    if (normalize(ans) === normalize(currentQuiz.questions[index].answer)) score++;
    index++;
    showQuestion();
}

function answerTF(ans) {
    if (ans === currentQuiz.questions[index].answer) score++;
    index++;
    showQuestion();
}

function answerID() {
    const user = normalize(document.getElementById("idAnswer").value);
    const correct = normalize(currentQuiz.questions[index].answer);

    if (user === correct) score++;
    index++;
    showQuestion();
}
</script>

</body>
</html>
