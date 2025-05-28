# 🔄 Concurrency vs. Parallelism in Node.js

Concurrency and parallelism help Node.js handle many tasks, like managing users in a chat app! 😊 This guide explains both concepts simply, using a unique classroom analogy, and includes a real-life exercise to make it clear. Let’s dive in! 🚀

## 🤔 What Are Concurrency and Parallelism?

- **Concurrency**: Doing multiple tasks by switching between them quickly, but only one task runs at a time.
  - **Analogy**: A teacher in a classroom helping students with homework. She answers one student’s question, then moves to the next, switching between them while only helping one at a time.
- **Parallelism**: Doing multiple tasks at the exact same time using multiple workers.
  - **Analogy**: Two teachers in the same classroom, each helping a group of students simultaneously—both groups get help at the same time.

**Chat App Context**:

- **Concurrency**: Node.js handling 100 users sending messages by processing one message at a time but switching quickly (single thread, Event Loop).
- **Parallelism**: Using multiple Node.js processes to handle 100 users at the same time (e.g., with the `cluster` module).

## 🛠️ How Node.js Uses Them

- **Concurrency in Node.js**: Node.js uses a single-threaded Event Loop to manage tasks. It switches between tasks (e.g., reading a file, sending a response) without waiting, thanks to its asynchronous nature.
- **Parallelism in Node.js**: Node.js can achieve parallelism using the `cluster` module, which creates multiple processes (workers) to handle tasks simultaneously, utilizing multiple CPU cores.

```js
// Concurrency: Event Loop switches tasks
const http = require("node:http");
http
  .createServer((req, res) => {
    res.end("Message processed!"); // Event Loop handles one request at a time
  })
  .listen(3000);

// Parallelism: Cluster for multiple processes
const cluster = require("node:cluster");
const http = require("node:http");
if (cluster.isMaster) {
  cluster.fork(); // Create a worker
  cluster.fork(); // Create another worker
} else {
  http
    .createServer((req, res) => {
      res.end("Message processed!");
    })
    .listen(3000);
}
```

# 🌟 Key Differences

Understanding Concurrency vs Parallelism:

| **Aspect**           | **Concurrency**                    | **Parallelism**                       |
| -------------------- | ---------------------------------- | ------------------------------------- |
| **Execution**        | One task at a time, switches fast. | Multiple tasks at the same time.      |
| **Resources**        | Single thread (Event Loop).        | Multiple threads/processes (Cluster). |
| **Chat App Example** | Handling 100 messages one by one.  | Handling 100 messages across workers. |

## 🧩 Types of Concurrency in Node.js

Node.js offers several ways to achieve concurrency. Here’s a quick list—we’ll explore each in detail in upcoming files:

- **Event Loop**: Manages tasks with a single thread (already covered above).
- **Worker Threads**: Runs JavaScript in separate threads for CPU-heavy tasks.
- **Child Processes**: Spawns separate processes for independent tasks.
- **Async I/O**: Handles I/O operations (e.g., file or network) without blocking.
- **Clustering**: Distributes tasks across multiple processes (also used for parallelism, as seen above).

What’s Next? We’ll dive into each type in separate guides, starting with Worker Threads in worker-threads.md.

## 🎯 Follow-Up Activity: Test Concurrency and Parallelism

Let’s simulate a chat app handling multiple users to see concurrency and parallelism in action!

Create a folder called chat-concurrent:

```bash
mkdir chat-concurrent
cd chat-concurrent
```

Create chat-concurrent.js (Concurrency with Event Loop):

```js
const http = require("node:http");
const server = http.createServer((req, res) => {
  if (req.url === "/message") {
    // Simulate a message with a delay
    setTimeout(() => {
      res.statusCode = 200;
      res.end(`[Your Name]: Message sent at ${new Date().toISOString()}!`);
    }, 1000);
  } else {
    res.statusCode = 404;
    res.end("Not found!");
  }
});
server.listen(3000, () => {
  console.log("Concurrency server at http://127.0.0.1:3000/");
});
```

Create chat-parallel.js (Parallelism with Cluster):

```js
const cluster = require("node:cluster");
const http = require("node:http");
const numCPUs = 2; // Simulate 2 CPUs
if (cluster.isMaster) {
  console.log("Parallelism server starting...");
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
} else {
  http
    .createServer((req, res) => {
      if (req.url === "/message") {
        setTimeout(() => {
          res.statusCode = 200;
          res.end(
            `[Your Name]: Message sent by worker ${
              cluster.worker.id
            } at ${new Date().toISOString()}!`
          );
        }, 1000);
      } else {
        res.statusCode = 404;
        res.end("Not found!");
      }
    })
    .listen(3000, () => {
      console.log(
        `Worker ${cluster.worker.id} running at http://127.0.0.1:3000/`
      );
    });
}
```

Run both servers separately:
First, concurrency:node chat-concurrent.js

Open multiple browser tabs and visit http://127.0.0.1:3000/message. Notice requests are handled one by one (Event Loop).
Then, parallelism:node chat-parallel.js

Open multiple tabs again and visit http://127.0.0.1:3000/message. Notice requests are handled by different workers at the same time.

Challenge: Add a /status route to both servers to show how many requests are being handled:else if (req.url === "/status") {
res.statusCode = 200;
res.end("Processing request...");
}

## Why Do This?

See concurrency (Event Loop) vs. parallelism (Cluster) in action.
Test how a chat app handles multiple users.
Share your results on X with #NodeJSBeginner.

## 🚀 What’s Next?

You’ve learned concurrency and parallelism! Head to worker-threads.md to explore:

How Worker Threads handle concurrency for CPU-heavy tasks.
More ways to manage tasks in Node.js.
Building bigger apps.

Tip: Check Node.js docs for more on the cluster module.
