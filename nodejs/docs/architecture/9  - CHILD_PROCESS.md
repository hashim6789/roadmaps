# 👶 Child Processes in Node.js

Child Processes in Node.js let you run separate tasks without slowing down your chat app! 😊 This guide explains the `child_process` module, its methods, and real-life scenarios, using a simple manager-assistant analogy. Let’s dive in! 🚀

## 🤔 What Are Child Processes?

- **Child Processes** let Node.js run separate programs or scripts in their own processes, independent of the main Node.js process.
- **Analogy**: Imagine a manager (main process) in an office who needs to create a report. She hires an assistant (child process) to do the task separately, so she can keep managing the office without waiting.
- **Why Use Them?** They’re great for running external commands (e.g., shell scripts) or heavy tasks that don’t need to share memory with the main app.

**Chat App Context**:

- You want to generate a daily report of chat activity (e.g., number of messages) without blocking the chat app’s main thread. A Child Process can handle this task independently.

## 🛠️ Methods of the `child_process` Module

The `child_process` module provides several methods to create and manage child processes. Here’s a quick overview:

| **Method**     | **Description**                                   | **Use Case in Chat App**                 |
| -------------- | ------------------------------------------------- | ---------------------------------------- |
| **`exec`**     | Runs a command in a shell and buffers the output. | Generate a report using a shell command. |
| **`execFile`** | Runs an executable file without a shell.          | Run a script to analyze chat logs.       |
| **`spawn`**    | Launches a new process with a command.            | Stream large chat logs for processing.   |
| **`fork`**     | Creates a new Node.js process for a script.       | Run a Node.js script to count messages.  |

### Key Points:

- **`exec` and `execFile`**: Best for short tasks with small output (buffered in memory).
- **`spawn`**: Best for streaming large data (e.g., real-time log processing).
- **`fork`**: Best for running Node.js scripts with communication between parent and child.

## 🌟 Real-Life Scenario: Generating a Chat Activity Report

### The Problem

Your chat app logs messages in `chat.txt`. Every day at 9:16 PM IST (like today, May 28, 2025), you need to generate a report (`report-2025-05-28.txt`) with the total number of messages and unique users. Doing this in the main thread could slow down the app, especially if the log file is large.

### The Solution

Use the `child_process` module’s `fork` method to run a separate Node.js script that counts messages and users, while the main app stays responsive.

### Code Solution

#### `chat-report.js` (Main Process)

```js
const http = require("node:http");
const { fork } = require("node:child_process");

// Create a timestamp for the report file
const date = new Date();
const timestamp =
  date.getFullYear() +
  "-" +
  String(date.getMonth() + 1).padStart(2, "0") +
  "-" +
  String(date.getDate()).padStart(2, "0");
const reportFile = `report-${timestamp}.txt`;

const server = http.createServer((req, res) => {
  res.setHeader("Content-Type", "text/plain");
  if (req.url === "/message") {
    // Main thread stays responsive
    res.statusCode = 200;
    res.end("[Your Name]: Message sent!");
  } else if (req.url === "/report") {
    // Fork a child process to generate the report
    const child = fork("./report-worker.js");
    child.send({ logFile: "chat.txt", reportFile });
    child.on("message", (result) => {
      res.statusCode = 200;
      res.end(
        `Report generated at ${reportFile}:\nMessages: ${result.messages}\nUsers: ${result.users}`
      );
    });
    child.on("error", () => {
      res.statusCode = 500;
      res.end("Error generating report!");
    });
    child.on("exit", () => console.log("Report generation complete."));
  } else {
    res.statusCode = 404;
    res.end("Not found!");
  }
});

server.listen(3000, () => {
  console.log("Server at http://127.0.0.1:3000/");
});
```

#### report-worker.js (Child Process)

```js
const fs = require("node:fs");
process.on("message", ({ logFile, reportFile }) => {
  try {
    // Read the chat log
    const data = fs.readFileSync(logFile, "utf8");
    const lines = data.split("\n").filter((line) => line.trim());

    // Count messages (lines) and unique users
    const messages = lines.length;
    const users = new Set(lines.map((line) => line.split(":")[0].trim())).size;

    // Write the report
    const reportContent = `Chat Activity Report - ${new Date().toISOString()}\nTotal Messages: ${messages}\nUnique Users: ${users}`;
    fs.writeFileSync(reportFile, reportContent);

    // Send result back to the main process
    process.send({ messages, users });
  } catch (err) {
    console.error("Error in child process:", err);
    process.exit(1);
  }
});
```

## How It Works

Main Process (chat-report.js): Runs an HTTP server for chat messages and forks a child process to generate the report.
Child Process (report-worker.js): Reads chat.txt, counts messages and users, writes the report to report-2025-05-28.txt, and sends the result back.
Result: The chat app stays responsive while the report is generated at 9:16 PM IST on May 28, 2025.

# 🌍 More Real-Life Scenarios in Development

Understanding when and why to use different child process methods in Node.js can greatly improve efficiency in real-world applications.

| **Scenario**          | **Child Process Method** | **Why Use It?**                                |
| --------------------- | ------------------------ | ---------------------------------------------- |
| Run a Python Script   | `exec` or `execFile`     | Analyze chat sentiment with a Python ML model. |
| Compress Chat Logs    | `spawn`                  | Use `tar` to compress logs for archiving.      |
| Run a Background Task | `fork`                   | Schedule daily user activity reports.          |

## 🎯 Follow-Up Activity: Generate Your Own Chat Report

Let’s create a chat server that uses a Child Process to generate a report!

Create a folder called chat-report:

```bash
mkdir chat-report
cd chat-report
```

Create chat.txt with some sample messages:[Your Name]: Hello, everyone!
[Friend]: Hey, how’s it going?
[Your Name]: Great, thanks!

#### Create chat-report.js (copy the Main Process code above):

```js
const http = require("node:http");
const { fork } = require("node:child_process");
const date = new Date();
const timestamp =
  date.getFullYear() +
  "-" +
  String(date.getMonth() + 1).padStart(2, "0") +
  "-" +
  String(date.getDate()).padStart(2, "0");
const reportFile = `report-${timestamp}.txt`;
const server = http.createServer((req, res) => {
  res.setHeader("Content-Type", "text/plain");
  if (req.url === "/message") {
    res.statusCode = 200;
    res.end("[Your Name]: Message sent!");
  } else if (req.url === "/report") {
    const child = fork("./report-worker.js");
    child.send({ logFile: "chat.txt", reportFile });
    child.on("message", (result) => {
      res.statusCode = 200;
      res.end(
        `Report generated at ${reportFile}:\nMessages: ${result.messages}\nUsers: ${result.users}`
      );
    });
    child.on("error", () => {
      res.statusCode = 500;
      res.end("Error generating report!");
    });
    child.on("exit", () => console.log("Report generation complete."));
  } else {
    res.statusCode = 404;
    res.end("Not found!");
  }
});
server.listen(3000, () => {
  console.log("Server at http://127.0.0.1:3000/");
});
```

#### Create report-worker.js (copy the Child Process code above):

```js
const fs = require("node:fs");
process.on("message", ({ logFile, reportFile }) => {
  try {
    const data = fs.readFileSync(logFile, "utf8");
    const lines = data.split("\n").filter((line) => line.trim());
    const messages = lines.length;
    const users = new Set(lines.map((line) => line.split(":")[0].trim())).size;
    const reportContent = `Chat Activity Report - ${new Date().toISOString()}\nTotal Messages: ${messages}\nUnique Users: ${users}`;
    fs.writeFileSync(reportFile, reportContent);
    process.send({ messages, users });
  } catch (err) {
    console.error("Error in child process:", err);
    process.exit(1);
  }
});
```

Run the server:

```bash
node chat-report.js
```

Visit these URLs in your browser:
http://127.0.0.1:3000/message (main thread stays responsive).
http://127.0.0.1:3000/report (child process generates the report).

Check the folder for report-2025-05-28.txt and verify its contents.
Challenge: Modify report-worker.js to include the most frequent message sender:
Add logic to count messages per user and include the top sender in the report.

## Why Do This?

Practice using Child Processes for independent tasks.
Generate a report for a chat app.
Share your report on X with #NodeJSBeginner.

## 🚀 What’s Next?

You’ve mastered Child Processes! Head to async-io.md to explore:

How Async I/O handles concurrency for I/O tasks.
More ways to manage tasks in Node.js.
Building bigger apps.

Tip: Check Node.js docs for more on the child_process module.
