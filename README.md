<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Group Project App</title>
<style>
  body {
    font-family: Arial, sans-serif;
    background-color: #b3daff; /* aerial blue */
    color: #003366; /* dark blue text */
    margin: 0;
    padding: 0;
  }
  .container {
    width: 400px;
    margin: 30px auto;
    background-color: #cceeff;
    padding: 20px;
    border-radius: 12px;
    box-shadow: 0 0 15px rgba(0,0,0,0.1);
  }
  h2 {
    text-align: center;
    margin-bottom: 15px;
  }
  .auth-fields {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    justify-content: space-between;
    margin-bottom: 10px;
  }
  .auth-fields input,
  .auth-fields select {
    flex: 1 1 48%;
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 6px;
    box-sizing: border-box;
  }
  .password-container {
    position: relative;
    flex: 1 1 100%;
  }
  .password-container input {
    width: 100%;
    padding: 8px;
    padding-right: 30px;
    border: 1px solid #ccc;
    border-radius: 6px;
    box-sizing: border-box;
  }
  .eye-icon {
    position: absolute;
    top: 50%;
    right: 10px;
    transform: translateY(-50%);
    cursor: pointer;
    user-select: none;
    font-size: 16px;
    color: #0059b3;
  }
  button {
    width: 100%;
    padding: 10px;
    background-color: #3399ff;
    border: none;
    color: white;
    border-radius: 6px;
    cursor: pointer;
    font-size: 16px;
    margin-top: 10px;
  }
  button:hover {
    background-color: #287acc;
  }
  #chatSection {
    margin-top: 10px;
  }
  .progress-container {
    text-align: center;
    margin-bottom: 15px;
  }
  input[type="range"] {
    width: 80%;
  }
  #progressLabel {
    font-weight: bold;
    margin-left: 10px;
    color: #003366;
  }
  #messages {
    height: 250px;
    overflow-y: auto;
    background: #e6f7ff;
    border-radius: 10px;
    padding: 10px;
    margin-bottom: 10px;
    box-sizing: border-box;
  }
  .message {
    max-width: 70%;
    margin-bottom: 12px;
    padding: 8px 12px;
    border-radius: 15px;
    display: flex;
    flex-direction: column;
  }
  .left {
    align-self: flex-start;
    background-color: #d9f0ff;
    color: #003366;
  }
  .right {
    align-self: flex-end;
    background-color: #99ccff;
    color: #002244;
  }
  .meta {
    font-size: 11px;
    font-weight: 700;
    margin-bottom: 5px;
    color: #004080;
  }
  #chatForm {
    display: flex;
    gap: 10px;
  }
  #messageInput {
    flex-grow: 1;
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 6px;
    font-size: 14px;
    box-sizing: border-box;
  }
  .logout-btn {
    background-color: #ff6666;
    margin-top: 15px;
    font-weight: bold;
  }
  .logout-btn:hover {
    background-color: #cc5252;
  }
</style>
</head>
<body>
  <div class="container">

    <div id="authSection">
      <h2 id="formTitle">Login</h2>
      <form id="authForm">
        <div class="auth-fields">
          <input type="text" id="username" placeholder="Username" required />
          <div class="password-container">
            <input type="password" id="password" placeholder="Password" required />
            <span class="eye-icon" onclick="togglePassword()">👁️</span>
          </div>
          <input type="text" id="fullname" placeholder="Full Name" style="display:none;" />
          <select id="role" style="display:none;">
            <option value="student">Student</option>
            <option value="teacher">Teacher</option>
          </select>
        </div>
        <button type="submit" id="authButton">Login</button>
        <p style="text-align:center;">
          <a href="#" id="toggleAuthBtn">Switch to Signup</a>
        </p>
      </form>
    </div>

    <div id="chatSection" style="display:none;">
      <div class="progress-container">
        <label><strong>Project Progress:</strong></label><br/>
        <input type="range" id="progressSlider" min="0" max="100" value="0" />
        <span id="progressLabel">0%</span>
      </div>

      <div id="messages"></div>

      <form id="chatForm">
        <input type="text" id="messageInput" placeholder="Type message" autocomplete="off" />
        <button type="submit">Send</button>
      </form>

      <button class="logout-btn" onclick="logout()">Logout</button>
    </div>
  </div>

  <!-- Firebase SDK -->
  <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
  <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-database.js"></script>

  <script>
    // Firebase config
    const firebaseConfig = {
      apiKey: "AIzaSyBj8-QrFiVu6H1YjEXkSuZojb0whFwWZZk",
      authDomain: "group-project-app-6ef69.firebaseapp.com",
      databaseURL: "https://group-project-app-6ef69-default-rtdb.firebaseio.com",
      projectId: "group-project-app-6ef69",
      storageBucket: "group-project-app-6ef69.appspot.com",
      messagingSenderId: "396548018075",
      appId: "1:396548018075:web:89951da751a5271ed29b3e",
      measurementId: "G-HQPRKLDSDX"
    };

    // Initialize Firebase
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();

    let isLogin = true;
    let currentUser = null;
    let usersCache = {};

    const authForm = document.getElementById("authForm");
    const usernameInput = document.getElementById("username");
    const passwordInput = document.getElementById("password");
    const fullnameInput = document.getElementById("fullname");
    const roleSelect = document.getElementById("role");
    const authButton = document.getElementById("authButton");
    const toggleAuthBtn = document.getElementById("toggleAuthBtn");
    const formTitle = document.getElementById("formTitle");
    const chatSection = document.getElementById("chatSection");
    const authSection = document.getElementById("authSection");
    const chatForm = document.getElementById("chatForm");
    const messageInput = document.getElementById("messageInput");
    const messagesContainer = document.getElementById("messages");
    const progressSlider = document.getElementById("progressSlider");
    const progressLabel = document.getElementById("progressLabel");

    // Toggle Login/Signup form
    toggleAuthBtn.onclick = () => {
      isLogin = !isLogin;
      formTitle.textContent = isLogin ? "Login" : "Signup";
      authButton.textContent = isLogin ? "Login" : "Signup";
      fullnameInput.style.display = isLogin ? "none" : "inline-block";
      roleSelect.style.display = isLogin ? "none" : "inline-block";
      fullnameInput.value = "";
      roleSelect.value = "student";
    };

    // Toggle password visibility
    function togglePassword() {
      passwordInput.type = passwordInput.type === "password" ? "text" : "password";
    }

    // Load users from Firebase
    async function loadUsers() {
      try {
        const snapshot = await db.ref("users").get();
        usersCache = snapshot.exists() ? snapshot.val() : {};
      } catch (e) {
        alert("Error loading users");
      }
    }

    // Save user to Firebase
    async function saveUser(username, userData) {
      await db.ref("users/" + username).set(userData);
    }

    // Append a message bubble to chat window
    function appendMessage(msg) {
      const div = document.createElement("div");
      div.className = "message " + (msg.username === currentUser.username ? "right" : "left");

      const meta = document.createElement("div");
      meta.className = "meta";

      // Format time 12hr with AM/PM
      const date = new Date(msg.timestamp || Date.now());
      let hours = date.getHours();
      const ampm = hours >= 12 ? "PM" : "AM";
      hours = hours % 12 || 12;
      const minutes = date.getMinutes().toString().padStart(2, "0");
      meta.textContent = `${msg.fullname} (${msg.role}) - ${hours}:${minutes} ${ampm}`;

      div.appendChild(meta);

      const text = document.createElement("div");
      text.textContent = msg.text;
      div.appendChild(text);

      messagesContainer.appendChild(div);
      messagesContainer.scrollTop = messagesContainer.scrollHeight;
    }

    // Load chat messages from Firebase and listen for new
    function listenForMessages() {
      db.ref("messages").on("child_added", snapshot => {
        const msg = snapshot.val();
        appendMessage(msg);
      });
    }

    // Load project progress slider and listen for changes
    function listenForProgress() {
      db.ref("progress").on("value", snapshot => {
        const val = snapshot.val() || 0;
        progressSlider.value = val;
        progressLabel.textContent = val + "%";
      });
    }

    // Update progress in Firebase on slider change
    progressSlider.addEventListener("input", e => {
      const val = e.target.value;
      progressLabel.textContent = val + "%";
      if (currentUser && currentUser.role === "teacher") {
        db.ref("progress").set(Number(val));
      }
    });

    // Handle login/signup submit
    authForm.addEventListener("submit", async (e) => {
      e.preventDefault();
      const username = usernameInput.value.trim();
      const password = passwordInput.value.trim();

      if (!username || !password) {
        alert("Please fill in username and password");
        return;
      }

      await loadUsers();

      if (isLogin) {
        // Login
        if (!(username in usersCache)) {
          alert("User not found. Please signup.");
          return;
        }
        if (usersCache[username].password !== password) {
          alert("Incorrect password.");
          return;
        }
        currentUser = { username, ...usersCache[username] };
        afterLogin();
      } else {
        // Signup
        if (username in usersCache) {
          alert("Username already taken.");
          return;
        }
        const fullname = fullnameInput.value.trim();
        const role = roleSelect.value;
        if (!fullname) {
          alert("Please enter your full name.");
          return;
        }
        const newUser = { password, fullname, role };
        await saveUser(username, newUser);
        currentUser = { username, ...newUser };
        afterLogin();
      }
    });

    // After login/show chat
    function afterLogin() {
      authSection.style.display = "none";
      chatSection.style.display = "block";
      loadMessagesAndProgress();
    }

    // Load chat and progress listeners
    function loadMessagesAndProgress() {
      messagesContainer.innerHTML = "";
      listenForMessages();
      listenForProgress();
    }

    // Handle chat message sending
    chatForm.addEventListener("submit", async (e) => {
      e.preventDefault();
      const text = messageInput.value.trim();
      if (!text) return;
      const msgData = {
        username: currentUser.username,
        fullname: currentUser.fullname,
        role: currentUser.role,
        text,
        timestamp: Date.now(),
      };
      await db.ref("messages").push(msgData);
      messageInput.value = "";
    });

    // Logout
    function logout() {
      currentUser = null;
      authSection.style.display = "block";
      chatSection.style.display = "none";
      usernameInput.value = "";
      passwordInput.value = "";
      fullnameInput.value = "";
      progressLabel.textContent = "0%";
      progressSlider.value = 0;
      messagesContainer.innerHTML = "";
    }
  </script>
</body>
</html>















  

