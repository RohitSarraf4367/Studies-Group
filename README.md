# Studies-Group
Group study for student 
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Class Chat + AI Study Helper 🎉</title>
<link rel="stylesheet" href="style.css">

<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

</head>
<body>

<!-- LOGIN SCREEN -->
<div id="loginScreen">
  <h1>Welcome to Class Chat & AI Helper!</h1>
  <input type="text" id="usernameInput" placeholder="Enter your Instagram-style name (@name)">
  <button onclick="login()">Join Chat</button>
</div>

<!-- MAIN SCREEN -->
<div id="mainScreen" class="hidden">

  <!-- HEADER -->
  <div id="header">
    <h2>Class Chat Room</h2>
    <span id="userGreeting"></span>
  </div>

  <!-- CHAT WINDOW -->
  <div id="chatWindow"></div>

  <!-- CHAT INPUT AREA -->
  <div id="inputArea">
    <input type="text" id="messageInput" placeholder="Type a message...">
    <button onclick="sendMessage()">Send</button>
    <button onclick="addEmoji()">😊</button>
  </div>

  <!-- FUN SECTION -->
  <div id="funSection">
    <h3>💡 Fun Fact / Poll</h3>
    <p id="funFact">Did you know? The shortest war in history lasted 38 minutes!</p>
  </div>

  <!-- AI STUDY HELPER -->
  <div id="aiSection">
    <h3>🤖 AI Study Helper</h3>
    <input type="text" id="aiInput" placeholder="Ask me anything in any language">
    <button onclick="askAI()">Ask AI</button>
    <div id="aiAnswer"></div>
  </div>

</div>

<script src="app.js"></script>
</body>
</html>body {
  font-family: 'Arial', sans-serif;
  margin: 0;
  padding: 0;
  background: linear-gradient(135deg, #ff9a9e, #fad0c4);
}

.hidden { display: none; }

/* LOGIN SCREEN */
#loginScreen {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  color: #fff;
}

#loginScreen input {
  padding: 10px;
  margin: 10px 0;
  border-radius: 5px;
  border: none;
  font-size: 16px;
}

#loginScreen button {
  padding: 10px 20px;
  border-radius: 5px;
  border: none;
  background-color: #ff6f91;
  color: white;
  cursor: pointer;
  font-weight: bold;
}

/* MAIN SCREEN */
#header {
  background-color: #ff6f91;
  padding: 10px;
  color: white;
  display: flex;
  justify-content: space-between;
}

#chatWindow {
  flex: 1;
  height: 40vh;
  overflow-y: auto;
  padding: 10px;
  background-color: #fff4f8;
}

.message {
  padding: 8px 12px;
  margin: 5px 0;
  border-radius: 15px;
  background-color: #ffe6eb;
  max-width: 80%;
  word-wrap: break-word;
}

#inputArea {
  display: flex;
  padding: 10px;
  background-color: #fff0f5;
}

#inputArea input {
  flex: 1;
  padding: 10px;
  border-radius: 5px;
  border: none;
}

#inputArea button {
  padding: 10px;
  margin-left: 5px;
  border-radius: 5px;
  border: none;
  cursor: pointer;
  font-weight: bold;
}

#funSection, #aiSection {
  padding: 10px;
  background-color: #ffdee8;
  margin-top: 10px;
  border-radius: 10px;
}

#aiAnswer {
  margin-top: 5px;
  background-color: #fff0f5;
  padding: 8px;
  border-radius: 8px;
  min-height: 30px;
}// ----- FIREBASE SETUP -----
const firebaseConfig = {
  apiKey: "YOUR_FIREBASE_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  databaseURL: "https://YOUR_PROJECT_ID.firebaseio.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT_ID.appspot.com",
  messagingSenderId: "SENDER_ID",
  appId: "APP_ID"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.database();

// ----- GLOBALS -----
let username = "";

// ----- LOGIN -----
function login() {
  username = document.getElementById("usernameInput").value.trim();
  if (username) {
    document.getElementById("loginScreen").classList.add("hidden");
    document.getElementById("mainScreen").classList.remove("hidden");
    document.getElementById("userGreeting").innerText = `Hello, ${username}!`;
    loadMessages();
  }
}

// ----- CHAT -----
function sendMessage() {
  const message = document.getElementById("messageInput").value.trim();
  if (message) {
    db.ref('messages').push({
      user: username,
      text: message,
      time: Date.now()
    });
    document.getElementById("messageInput").value = "";
  }
}

function loadMessages() {
  const chatWindow = document.getElementById("chatWindow");
  db.ref('messages').on('child_added', snapshot => {
    const data = snapshot.val();
    const msgDiv = document.createElement("div");
    msgDiv.classList.add("message");
    msgDiv.innerHTML = `<strong>${data.user}:</strong> ${data.text}`;
    chatWindow.appendChild(msgDiv);
    chatWindow.scrollTop = chatWindow.scrollHeight;
  });
}

function addEmoji() {
  const emojis = ["😀","😂","😎","😍","🤩","👍","🎉"];
  const randomEmoji = emojis[Math.floor(Math.random() * emojis.length)];
  document.getElementById("messageInput").value += randomEmoji;
}

// ----- AI STUDY HELPER -----
async function askAI() {
  const question = document.getElementById("aiInput").value.trim();
  if (!question) return;

  document.getElementById("aiAnswer").innerText = "Thinking... 🤖";

  try {
    const response = await fetch("https://api.openai.com/v1/chat/completions", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Authorization": "Bearer YOUR_OPENAI_API_KEY"
      },
      body: JSON.stringify({
        model: "gpt-3.5-turbo",
        messages: [{ role: "user", content: question }],
        max_tokens: 150
      })
    });

    const data = await response.json();
    const answer = data.choices[0].message.content;
    document.getElementById("aiAnswer").innerText = answer;

  } catch (err) {
    document.getElementById("aiAnswer").innerText = "Error! Try again 😅";
  }

  document.getElementById("aiInput").value = "";
}
