# Graceful Shutdown in Node.js Application

## Overview

This guide demonstrates how to implement a graceful shutdown in a Node.js application to ensure minimal disruption and data integrity during server shutdown.

## What is Graceful Shutdown?

Graceful shutdown involves handling server shutdown in a way that ensures all pending requests are processed, ongoing data operations are completed, and resources are released before terminating the server.

## Why Graceful Shutdown?

- **Data Integrity**: Prevents data loss or corruption by ensuring all data operations are completed before shutting down.
- **Smooth Transition**: Allows the server to stop accepting new requests while finishing pending ones, leading to a seamless shutdown process.
- **Resource Cleanup**: Efficiently releases resources such as database connections and file system handles.
- **Minimal Disruption**: Reduces the risk of disrupting other processes or services running on the system.

## Implementation Steps

1. **Handle Process Kill Signal**: Detect when the server needs to shut down, usually triggered by a process kill signal (e.g., SIGINT or SIGTERM).
2. **Stop Accepting New Requests**: Cease accepting new requests from clients while allowing ongoing requests to complete.
3. **Close Data Processing**: Release resources and stop ongoing data operations (e.g., close database connections, file system workers, etc.).
4. **Exit the Process**: Gracefully exit the Node.js process once all operations are completed.

## Example

See the provided example code for a simple Node.js server using Express.js that implements graceful shutdown.

## Example Code

```javascript
const express = require('express');
const app = express();
const port = 3000;

const reCheckShuttingDownTime = 10000;
let shuttingDown = false;

/* Here we check server is process to shutting down */
app.use((_req, res, next) => {
  if (!shuttingDown) return next();

  res.status(503).send(`
  <html>
  <head>
     <title>503 Service Unavailable</title>
  </head>
  <body style="text-align: center;">
     <h1>503 Service Unavailable</h1>
     <p>Server is in the process of restarting.</p>
  </body>
  </html>`);
});

// Routes
app.get('/', (req, res) => {
    res.send('Hello World!');
});

// Start server
const server = app.listen(port, () => {
    console.log(`Server listening at http://localhost:${port}`);
});

/* Run when close server using kill command. */
process.on("SIGTERM", shutDown);
/* Run when close server using `ctr+c`. */
process.on("SIGINT", shutDown);

function shutDown() {
  shuttingDown = true;
  /* Here we get current connection count */
  server.getConnections((err, connections) => {
    if (connections === 0) {
      server.close(() => {
        process.exit(0);
      });
    }
  });
  /* Here we check if connection is still alive then re-check after some time */
  setTimeout(() => {
    shutDown();
  }, reCheckShuttingDownTime);
}
