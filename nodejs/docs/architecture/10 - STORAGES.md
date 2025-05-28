# 📦 Browser and Backend Storage in Node.js

Storage helps your chat app remember user data, like preferences or login sessions! 😊 This guide explains browser storage (Cookies, Local Storage, Session Storage, IndexedDB) and backend storage (Sessions, Databases), with real-life examples and implementations. Let’s dive in! 🚀

## 🤔 What Are Browser and Backend Storage?

- **Browser Storage**: Saves data on the user’s browser (client-side).
- **Backend Storage**: Saves data on the server (server-side).
- **Analogy**: Think of a notebook. Browser storage is like writing notes in a personal notebook you carry (client-side), while backend storage is like storing notes in the teacher’s filing cabinet (server-side).

**Chat App Context**:

- **Browser Storage**: Save a user’s theme preference, temporary chat drafts, or offline messages.
- **Backend Storage**: Track user login sessions or store chat messages permanently.

## 🛠️ Types of Storage and Their Implementations

### 1. Browser Storage

#### Cookies

- **What**: Small pieces of data stored in the browser, sent with every HTTP request.
- **Use Case**: Remember a user’s login token or track their activity.
- **Pros**: Works across browser tabs; can set expiration.
- **Cons**: Limited size (~4KB); sent with every request (overhead).

**Example**: Track a user’s last login time in a chat app.

```js
// Set a cookie in a Node.js server
const http = require("node:http");
const server = http.createServer((req, res) => {
  if (req.url === "/login") {
    res.setHeader(
      "Set-Cookie",
      `lastLogin=${new Date().toISOString()}; Path=/; HttpOnly`
    );
    res.statusCode = 200;
    res.end("Logged in! Cookie set.");
  } else if (req.url === "/last-login") {
    const cookies = req.headers.cookie?.split(";").reduce((acc, cookie) => {
      const [key, value] = cookie.trim().split("=");
      acc[key] = value;
      return acc;
    }, {});
    res.statusCode = 200;
    res.end(`Last Login: ${cookies?.lastLogin || "Never"}`);
  } else {
    res.statusCode = 404;
    res.end("Not found!");
  }
});
server.listen(3000, () => console.log("Server at http://127.0.0.1:3000/"));
```

#### Local Storage

- **What**: Stores data in the browser with no expiration (until cleared).
- **Use Case**: Save user preferences, like dark mode.
- **Pros**: Larger size (~5-10MB); stays until deleted.
- **Cons**: Not sent with requests; only accessible in the same domain.

Example: Save a user’s theme preference in a chat app.

<!-- theme.html -->
<!DOCTYPE html>
<html>
<head>
  <title>Chat App Theme</title>
</head>
<body>
  <button onclick="setTheme('dark')">Dark Mode</button>
  <button onclick="setTheme('light')">Light Mode</button>
  <p>Current Theme: <span id="theme">Loading...</span></p>
  <script>
    // Load theme on page load
    document.addEventListener("DOMContentLoaded", () => {
      const theme = localStorage.getItem("theme") || "light";
      document.getElementById("theme").textContent = theme;
      document.body.style.background = theme === "dark" ? "#333" : "#fff";
    });
    // Save theme on button click
    function setTheme(theme) {
      localStorage.setItem("theme", theme);
      document.getElementById("theme").textContent = theme;
      document.body.style.background = theme === "dark" ? "#333" : "#fff";
    }
  </script>
</body>
</html>

Serve this with a Node.js server:

```js
const http = require("node:http");
const fs = require("node:fs");
http
  .createServer((req, res) => {
    if (req.url === "/theme") {
      fs.createReadStream("theme.html").pipe(res);
    } else {
      res.statusCode = 404;
      res.end("Not found!");
    }
  })
  .listen(3001, () => console.log("Server at http://127.0.0.1:3001/"));
```

#### Session Storage

What: Stores data in the browser for the current session (clears when the tab closes).

- **Use Case**: Save a temporary chat draft while the user is typing.
  Pros: Cleared on tab close; isolated per tab.
  Cons: Not shared across tabs; limited size (~5MB).

Example: Save a chat draft in a chat app.

```html
<!-- draft.html -->
<!DOCTYPE html>
<html>
  <head>
    <title>Chat Draft</title>
  </head>
  <body>
    <textarea id="draft" placeholder="Type your message..."></textarea>
    <p>Saved Draft: <span id="savedDraft">None</span></p>
    <script>
      const textarea = document.getElementById("draft");
      const savedDraft = document.getElementById("savedDraft");
      // Load draft on page load
      textarea.value = sessionStorage.getItem("draft") || "";
      savedDraft.textContent = textarea.value || "None";
      // Save draft on input
      textarea.addEventListener("input", () => {
        sessionStorage.setItem("draft", textarea.value);
        savedDraft.textContent = textarea.value || "None";
      });
    </script>
  </body>
</html>
```

Serve this with a Node.js server:

```js
const http = require("node:http");
const fs = require("node:fs");
http
  .createServer((req, res) => {
    if (req.url === "/draft") {
      fs.createReadStream("draft.html").pipe(res);
    } else {
      res.statusCode = 404;
      res.end("Not found!");
    }
  })
  .listen(3002, () => console.log("Server at http://127.0.0.1:3002/"));
```

#### IndexedDB

- **What**: A NoSQL database in the browser for storing large amounts of structured data.
- **Use Case**: Store offline chat messages to sync later when online.
- **Pros**: Large storage (~50MB+); supports complex queries; works offline.
- **Cons**: More complex API; asynchronous operations.

Example: Store and retrieve offline chat messages in a chat app.

```html
<!-- offline.html -->
<!DOCTYPE html>
<html>
<head>
  <title>Offline Chat Messages</title>
</head>
<body>
  <input id="message" placeholder="Type your message...">
  <button onclick="saveMessage()">Save Offline</button>
  <button onclick="getMessages()">Show Messages</button>
  <p>Offline Messages: <pre id="messages">None</pre></p>
  <script src="https://cdn.jsdelivr.net/npm/idb@7.0.2/build/umd.js"></script>
  <script>
    // Open IndexedDB database
    const dbPromise = idb.openDB("ChatAppDB", 1, {
      upgrade(db) {
        db.createObjectStore("messages", { keyPath: "id", autoIncrement: true });
      }
    });

    // Save a message to IndexedDB
    async function saveMessage() {
      const message = document.getElementById("message").value;
      if (!message) return alert("Please enter a message!");
      const db = await dbPromise;
      await db.add("messages", { message, timestamp: new Date().toISOString() });
      document.getElementById("message").value = "";
      alert("Message saved offline!");
      getMessages();
    }

    // Retrieve messages from IndexedDB
    async function getMessages() {
      const db = await dbPromise;
      const messages = await db.getAll("messages");
      document.getElementById("messages").textContent = messages.length
        ? JSON.stringify(messages, null, 2)
        : "None";
    }

  </script>
</body>
</html>
```

Serve this with a Node.js server:

```js
const http = require("node:http");
const fs = require("node:fs");
http
  .createServer((req, res) => {
    if (req.url === "/offline") {
      fs.createReadStream("offline.html").pipe(res);
    } else {
      res.statusCode = 404;
      res.end("Not found!");
    }
  })
  .listen(3003, () => console.log("Server at http://127.0.0.1:3003/"));
```

2. Backend Storage

#### Sessions (Using express-session)

- **What**: Stores user data on the server, tied to a session ID stored in a cookie.
  - **Use Case**: Track user login sessions securely.
- **Pros**: Secure (server-side); can store sensitive data.
- **Cons**: Requires server memory or a store (e.g., Redis); needs session management.

Example: Track a user’s login session in a chat app.

```js
const express = require("express");
const session = require("express-session");
const app = express();

// Set up session middleware
app.use(session({
secret: "chat-app-secret",
resave: false,
saveUninitialized: false,
cookie: { maxAge: 24 _ 60 _ 60 \* 1000 } // 1 day
}));

app.get("/login", (req, res) => {
req.session.user = "[Your Name]";
req.session.loginTime = new Date().toISOString();
res.send("Logged in! Session created.");
});

app.get("/profile", (req, res) => {
if (req.session.user) {
res.send(`Welcome, ${req.session.user}! Logged in at: ${req.session.loginTime}`);
} else {
res.status(401).send("Please log in!");
}
});

app.listen(3004, () => console.log("Server at http://127.0.0.1:3004/"));
```

Note: Install dependencies with npm install express express-session.

Databases (Using SQLite as an Example)

- **What**: Stores data persistently on the server (e.g., SQLite, MongoDB).
  - **Use Case**: Save chat messages permanently.
- **Pros**: Scalable; permanent storage; queryable.
- **Cons**: Requires setup; can be slower than in-memory storage.

Example: Save and retrieve chat messages in a chat app.

```js
const express = require("express");
const sqlite3 = require("sqlite3").verbose();
const app = express();
app.use(express.json());

// Set up SQLite database
const db = new sqlite3.Database(":memory:");
db.serialize(() => {
  db.run("CREATE TABLE messages (user TEXT, message TEXT, timestamp TEXT)");
});

app.post("/message", (req, res) => {
  const { user, message } = req.body;
  const timestamp = new Date().toISOString();
  db.run(
    "INSERT INTO messages (user, message, timestamp) VALUES (?, ?, ?)",
    [user, message, timestamp],
    (err) => {
      if (err) return res.status(500).send("Error saving message!");
      res.status(201).send("Message saved!");
    }
  );
});

app.get("/messages", (req, res) => {
  db.all("SELECT * FROM messages", (err, rows) => {
    if (err) return res.status(500).send("Error fetching messages!");
    res.json(rows);
  });
});

app.listen(3005, () => console.log("Server at http://127.0.0.1:3005/"));
```

Note: Install dependencies with npm install express sqlite3.

## 🌍 Real-Life Scenarios in Development

Storage Type
Scenario
Why Use It?

Cookies
Track user’s last login time.
Persists across sessions; sent automatically.

Local Storage
Save user’s theme preference.
Persists until cleared; large storage size.

Session Storage
Save a temporary chat draft.
Clears on tab close; tab-specific.

IndexedDB
Store offline chat messages.
Large storage; works offline; structured data.

Sessions
Track user login sessions.
Secure server-side storage; tied to cookie.

Database
Store chat messages permanently.
Scalable; queryable; permanent storage.

## 🎯 Follow-Up Activity: Build a Chat Storage System with Offline Support

Let’s combine browser and backend storage, including IndexedDB, to enhance a chat app!

Create a folder called chat-storage:mkdir chat-storage
cd chat-storage

Initialize a Node.js project and install dependencies:

```bash
npm init -y
npm install express express-session sqlite3
```

Create chat.txt for initial messages:[Your Name]: Welcome to the chat!

Create app.js (Backend with Sessions and Database):

```js
const express = require("express");
const session = require("express-session");
const sqlite3 = require("sqlite3").verbose();
const app = express();
app.use(express.json());
app.use(express.static(".")); // Serve static files
app.use(session({
secret: "chat-app-secret",
resave: false,
saveUninitialized: false,
cookie: { maxAge: 24 _ 60 _ 60 _ 1000 }
}));
const db = new sqlite3.Database(":memory:");
db.serialize(() => {
db.run("CREATE TABLE messages (user TEXT, message TEXT, timestamp TEXT)");
});
app.get("/login", (req, res) => {
req.session.user = "[Your Name]";
res.setHeader("Set-Cookie", `lastLogin=${new Date().toISOString()}; Path=/; HttpOnly`);
res.send("Logged in!");
});
app.post("/message", (req, res) => {
if (!req.session.user) return res.status(401).send("Please log in!");
const { message } = req.body;
const timestamp = new Date().toISOString();
db.run("INSERT INTO messages (user, message, timestamp) VALUES (?, ?, ?)",
[req.session.user, message, timestamp], (err) => {
if (err) return res.status(500).send("Error saving message!");
res.status(201).send("Message saved!");
});
});
app.get("/messages", (req, res) => {
db.all("SELECT _ FROM messages", (err, rows) => {
if (err) return res.status(500).send("Error fetching messages!");
res.json(rows);
});
});
app.get("/logout", (req, res) => {
req.session.destroy((err) => {
if (err) return res.status(500).send("Error logging out!");
res.send("Logged out!");
});
});
app.listen(3000, () => console.log("Server at http://127.0.0.1:3000/"));
```

Create index.html (Frontend with Cookies, Local Storage, Session Storage, and IndexedDB):

```html
<!DOCTYPE html>

<html>
<head>
  <title>Chat App</title>
</head>
<body>
  <button onclick="login()">Login</button>
  <button onclick="logout()">Logout</button>
  <button onclick="setTheme('dark')">Dark Mode</button>
  <button onclick="setTheme('light')">Light Mode</button>
  <textarea id="draft" placeholder="Type your message..."></textarea>
  <button onclick="saveMessage()">Save Offline</button>
  <button onclick="sendMessage()">Send Online</button>
  <button onclick="getOfflineMessages()">Show Offline Messages</button>
  <p>Current Theme: <span id="theme">Loading...</span></p>
  <p>Saved Draft: <span id="savedDraft">None</span></p>
  <p>Offline Messages: <pre id="offlineMessages">None</pre></p>
  <p>Online Messages: <pre id="messages">Loading...</pre></p>
  <script src="https://cdn.jsdelivr.net/npm/idb@7.0.2/build/umd.js"></script>
  <script>
    // IndexedDB: Set up database
    const dbPromise = idb.openDB("ChatAppDB", 1, {
      upgrade(db) {
        db.createObjectStore("messages", { keyPath: "id", autoIncrement: true });
      }
    });

    // Cookies: Check last login
    async function login() {
      const res = await fetch("http://127.0.0.1:3000/login");
      if (res.ok) {
        const cookies = document.cookie.split(";").reduce((acc, cookie) => {
          const [key, value] = cookie.trim().split("=");
          acc[key] = value;
          return acc;
        }, {});
        alert(`Last Login: ${cookies.lastLogin}`);
        fetchMessages();
      }
    }

    async function logout() {
      const res = await fetch("http://127.0.0.1:3000/logout");
      if (res.ok) alert("Logged out!");
    }

    // Local Storage: Theme preference
    document.addEventListener("DOMContentLoaded", () => {
      const theme = localStorage.getItem("theme") || "light";
      document.getElementById("theme").textContent = theme;
      document.body.style.background = theme === "dark" ? "#333" : "#fff";
      fetchMessages();
      getOfflineMessages();
    });
    function setTheme(theme) {
      localStorage.setItem("theme", theme);
      document.getElementById("theme").textContent = theme;
      document.body.style.background = theme === "dark" ? "#333" : "#fff";
    }

    // Session Storage: Chat draft
    const textarea = document.getElementById("draft");
    const savedDraft = document.getElementById("savedDraft");
    textarea.value = sessionStorage.getItem("draft") || "";
    savedDraft.textContent = textarea.value || "None";
    textarea.addEventListener("input", () => {
      sessionStorage.setItem("draft", textarea.value);
      savedDraft.textContent = textarea.value || "None";
    });

    // IndexedDB: Save and retrieve offline messages
    async function saveMessage() {
      const message = textarea.value;
      if (!message) return alert("Please enter a message!");
      const db = await dbPromise;
      await db.add("messages", { message, timestamp: new Date().toISOString() });
      textarea.value = "";
      sessionStorage.removeItem("draft");
      savedDraft.textContent = "None";
      alert("Message saved offline!");
      getOfflineMessages();
    }

    async function getOfflineMessages() {
      const db = await dbPromise;
      const messages = await db.getAll("messages");
      document.getElementById("offlineMessages").textContent = messages.length
        ? JSON.stringify(messages, null, 2)
        : "None";
    }

    // Database: Send and fetch online messages
    async function sendMessage() {
      const message = textarea.value;
      if (!message) return alert("Please enter a message!");
      const res = await fetch("http://127.0.0.1:3000/message", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ message })
      });
      if (res.ok) {
        textarea.value = "";
        sessionStorage.removeItem("draft");
        savedDraft.textContent = "None";
        fetchMessages();
      } else {
        alert("Failed to send message. Save offline instead?");
      }
    }

    async function fetchMessages() {
      const res = await fetch("http://127.0.0.1:3000/messages");
      if (res.ok) {
        const messages = await res.json();
        document.getElementById("messages").textContent = JSON.stringify(messages, null, 2);
      }
    }

  </script>
</body>
</html>
```

Run the server:

```bash
node app.js
```

Open http://127.0.0.1:3000/ in your browser to interact with the chat app.
Challenge: Add a button to sync offline messages to the server:
Add a syncMessages function to send IndexedDB messages to the /message endpoint and clear them from IndexedDB.

## Why Do This?

Practice using browser and backend storage together, including IndexedDB.
Build a fully functional chat app with offline support.
Share your app on X with #NodeJSBeginner.

## 🚀 What’s Next?

You’ve mastered storage types! Head to async-io.md to explore:

How Async I/O handles concurrency for I/O tasks.
More ways to manage tasks in Node.js.
Building bigger apps.

Tip: Check Node.js docs for more on storage and Express.

```

```
