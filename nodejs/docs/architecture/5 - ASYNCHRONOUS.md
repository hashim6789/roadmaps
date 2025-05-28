# ⏳ Synchronous vs. Asynchronous in Node.js

Node.js is awesome for handling tasks in a chat app, but some tasks wait while others don’t! 😊 This guide explains synchronous vs. asynchronous, blocking vs. non-blocking, and how to manage async code with callbacks, Promises, and async/await. Let’s dive in! 🚀

## 🤔 Synchronous vs. Asynchronous: What’s the Difference?

- **Synchronous (Sync)**: Tasks happen one after another, waiting for each to finish.
  - **Analogy**: In a restaurant, you order food and wait at the counter until it’s ready—no one else gets served until you’re done.
- **Asynchronous (Async)**: Tasks start and don’t wait; other tasks can happen while waiting.
  - **Analogy**: You order food, get a buzzer, and sit down—others can order while your food is being prepared.

**Chat App Example**:

- **Sync**: Loading a chat log and waiting before showing the app (app freezes).
- **Async**: Loading the chat log in the background while the app stays responsive.

## 🛑 Blocking vs. Non-Blocking: How Tasks Wait

- **Blocking (Sync)**: The program pauses until a task finishes.
  - **Example**: Reading a file with `fs.readFileSync()`—nothing else happens until the file is read.
- **Non-Blocking (Async)**: The program keeps running while the task happens.
  - **Example**: Reading a file with `fs.readFile()`—you can handle other tasks while the file loads.

```js
// Blocking: Waits for file
const fs = require("node:fs");
const data = fs.readFileSync("chat.txt", "utf8"); // Blocks
console.log(data);
console.log("This waits!");

// Non-Blocking: Doesn’t wait
fs.readFile("chat.txt", "utf8", (err, data) => {
  console.log(data); // Happens later
});
console.log("This runs immediately!");
```

## 🌟 Ways to Handle Asynchronous Code in Node.js

Node.js provides three main ways to manage async tasks: callbacks, Promises, and async/await. Let’s explore each with a chat app example!

### 1. Callbacks

- **What**: A function passed as an argument that runs when the async task finishes.
- **Pros**: Simple for small tasks.
- **Cons**: Can lead to “callback hell” (nested callbacks) for complex tasks.
- **Chat** App Example: Load a chat log and display it.

```js
// Using a callback
const fs = require("node:fs");
fs.readFile("chat.txt", "utf8", (err, data) => {
  if (err) {
    console.log("Error loading chat!");
    return;
  }
  console.log(`Chat Log: ${data}`);
});
```

### 2. Promises

- **What**: An object that represents a future result (success or failure).
- **Pros**: Cleaner than callbacks, avoids nesting with .then().
- **Cons**: Still requires chaining for multiple tasks.
- **Chat** App Example: Load a chat log and then notify users.

```js
// Using a Promise
const fs = require("node:fs").promises;
fs.readFile("chat.txt", "utf8")
  .then((data) => {
    console.log(`Chat Log: ${data}`);
    return "Notifying users...";
  })
  .then((message) => console.log(message))
  .catch((err) => console.log("Error loading chat!"));
```

### 3. Async/Await

- **What**: A modern syntax to write async code like sync code, using async and await.
- **Pros**: Cleanest and most readable, great for multiple async tasks.
- **Cons**: Requires understanding Promises (it’s built on them).
- **Chat** App Example: Load a chat log and save a new message.

```js
// Using async/await
const fs = require("node:fs").promises;
async function chatTasks() {
  try {
    const data = await fs.readFile("chat.txt", "utf8");
    console.log(`Chat Log: ${data}`);
    await fs.writeFile("chat.txt", `${data}\nNew message!`);
    console.log("Message saved!");
  } catch (err) {
    console.log("Error in chat tasks!");
  }
}
chatTasks();
```

# 📋 Comparison of Methods

A quick look at different ways to handle asynchronous code in Node.js:

| **Method**      | **Readability**   | **Error Handling**   | **Best For**                 |
| --------------- | ----------------- | -------------------- | ---------------------------- |
| **Callbacks**   | Low (nested)      | Manual (if/else)     | Simple tasks                 |
| **Promises**    | Medium (chaining) | Built-in (.catch)    | Chaining multiple tasks      |
| **Async/Await** | High (like sync)  | Built-in (try/catch) | Complex tasks, readable code |

## 🎯 Follow-Up Activity: Manage Chat Tasks

Let’s try all three methods to handle chat app tasks!

Create a folder called chat-async:

```bash
mkdir chat-async
cd chat-async
```

Create chat.txt:[Your Name]: Welcome!

Create chat-tasks.js:

```js
const fs = require("node:fs");
const fsPromises = require("node:fs").promises;

// 1. Callback: Read chat log
console.log("Using Callback:");
fs.readFile("chat.txt", "utf8", (err, data) => {
  if (err) {
    console.log("Error!");
    return;
  }
  console.log(data);
});

// 2. Promise: Read and append
console.log("\nUsing Promise:");
fsPromises
  .readFile("chat.txt", "utf8")
  .then((data) => {
    console.log(data);
    return fsPromises.writeFile("chat.txt", `${data}\n[Your Name]: Hi!`);
  })
  .then(() => console.log("Message added!"))
  .catch((err) => console.log("Error!"));

// 3. Async/Await: Read, append, and read again
console.log("\nUsing Async/Await:");
async function manageChat() {
  try {
    const data = await fsPromises.readFile("chat.txt", "utf8");
    console.log(data);
    await fsPromises.writeFile("chat.txt", `${data}\n[Your Name]: Bye!`);
    const updated = await fsPromises.readFile("chat.txt", "utf8");
    console.log("Updated Log:");
    console.log(updated);
  } catch (err) {
    console.log("Error!");
  }
}
manageChat();
```

Run it:

```bash
node chat-tasks.js
```

Challenge: Add error handling for a missing file:
Modify the async/await section to handle a file named missing.txt.

## Why Do This?

Practice sync and async with all three methods.
Manage a chat app’s tasks.
Share your output on X with #NodeJSBeginner.

## 🚀 What’s Next?

You’ve mastered sync and async! Head to node-dns-module.md to explore:

The dns module for networking.
More Node.js built-in modules.
Building bigger apps.

Tip: Check Node.js docs for more on async programming.
