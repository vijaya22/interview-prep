# Node.js Interview Questions & Answers

---

## 1. What is Node.js?

Node.js is an open-source, cross-platform JavaScript runtime environment built on Chrome's V8 engine. It allows running JavaScript outside the browser, primarily for server-side applications.

Key characteristics:
- **Single-threaded** — uses one main thread for JavaScript execution
- **Event-driven** — uses an event loop for non-blocking I/O
- **Asynchronous** — I/O operations don't block the main thread
- **NPM ecosystem** — largest package registry in the world

---

## 2. How does the Node.js event loop work?

The event loop is the core mechanism that handles asynchronous operations. It runs in phases:

1. **Timers** — executes `setTimeout` and `setInterval` callbacks
2. **Pending callbacks** — executes I/O callbacks deferred from the previous loop
3. **Idle/Prepare** — internal use only
4. **Poll** — retrieves new I/O events, executes I/O callbacks
5. **Check** — executes `setImmediate` callbacks
6. **Close callbacks** — executes close event callbacks (e.g., `socket.on('close')`)

Between each phase, Node checks the microtask queue (`process.nextTick`, Promises).

```js
console.log('1');                           // Sync

setTimeout(() => console.log('2'), 0);      // Timers phase

setImmediate(() => console.log('3'));        // Check phase

process.nextTick(() => console.log('4'));    // Microtask (highest priority)

Promise.resolve().then(() => console.log('5')); // Microtask

console.log('6');                           // Sync

// Output: 1, 6, 4, 5, 2, 3
// (setTimeout vs setImmediate order can vary when not in an I/O cycle)
```

---

## 3. What is the difference between `process.nextTick()` and `setImmediate()`?

| Feature | `process.nextTick()` | `setImmediate()` |
|---|---|---|
| When it runs | After current operation, before event loop continues | During the Check phase of the event loop |
| Priority | Higher (runs before any I/O or timer) | Lower (runs after I/O events in current iteration) |
| Recursion risk | Can starve I/O if called recursively | Safe for recursion |

```js
setImmediate(() => console.log('setImmediate'));
process.nextTick(() => console.log('nextTick'));
// Output: nextTick, setImmediate

// Recursive nextTick can starve I/O
function dangerous() {
  process.nextTick(dangerous); // I/O never gets processed
}

// Safe alternative
function safe() {
  setImmediate(safe); // I/O gets a chance between iterations
}
```

Use `process.nextTick` for deferring work that must complete before I/O. Use `setImmediate` for breaking up long-running operations.

---

## 4. What is the difference between CommonJS and ES Modules in Node.js?

| Feature | CommonJS (CJS) | ES Modules (ESM) |
|---|---|---|
| Syntax | `require()` / `module.exports` | `import` / `export` |
| Loading | Synchronous | Asynchronous |
| Parsing | Dynamic (runtime) | Static (compile-time) |
| Tree shaking | Not possible | Supported |
| Top-level await | Not supported | Supported |
| `this` in module | `module.exports` | `undefined` |
| File extension | `.js` (default), `.cjs` | `.mjs` or `.js` with `"type": "module"` |

```js
// CommonJS
const fs = require('fs');
const { readFile } = require('fs');
module.exports = { myFunction };
exports.myFunction = function() {};

// ES Modules
import fs from 'fs';
import { readFile } from 'fs';
export function myFunction() {}
export default class MyClass {}
```

To use ESM in Node.js, either use `.mjs` extension or set `"type": "module"` in `package.json`.

---

## 5. What are streams in Node.js?

Streams are objects for handling reading/writing data continuously. They process data in chunks rather than loading everything into memory.

**Four types of streams:**

| Type | Description | Example |
|---|---|---|
| Readable | Source of data | `fs.createReadStream`, `http.IncomingMessage` |
| Writable | Destination for data | `fs.createWriteStream`, `http.ServerResponse` |
| Duplex | Both readable and writable | `net.Socket`, `WebSocket` |
| Transform | Duplex that modifies data passing through | `zlib.createGzip`, `crypto.createCipher` |

```js
const fs = require('fs');
const zlib = require('zlib');

// Piping streams
fs.createReadStream('input.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('input.txt.gz'));

// Reading a stream
const readable = fs.createReadStream('large-file.txt', { encoding: 'utf8' });

readable.on('data', (chunk) => {
  console.log(`Received ${chunk.length} bytes`);
});

readable.on('end', () => {
  console.log('Done reading');
});

readable.on('error', (err) => {
  console.error('Error:', err);
});

// Using async iteration (modern approach)
async function readFile(path) {
  const stream = fs.createReadStream(path, { encoding: 'utf8' });
  for await (const chunk of stream) {
    process.stdout.write(chunk);
  }
}
```

---

## 6. What is the `Buffer` class?

`Buffer` is a built-in class for handling raw binary data directly in memory, outside the V8 heap.

```js
// Creating buffers
const buf1 = Buffer.alloc(10);              // 10 zero-filled bytes
const buf2 = Buffer.from('Hello', 'utf8');  // from string
const buf3 = Buffer.from([72, 101, 108]);   // from array

// Converting
buf2.toString('utf8');    // 'Hello'
buf2.toString('base64');  // 'SGVsbG8='
buf2.toString('hex');     // '48656c6c6f'

// Operations
buf2.length;              // 5 (bytes, not characters)
buf2[0];                  // 72 (ASCII for 'H')
Buffer.concat([buf1, buf2]); // combine buffers

// Comparison
buf2.equals(Buffer.from('Hello')); // true
Buffer.compare(buf2, buf3);       // 1 (buf2 > buf3)

// JSON
const json = buf2.toJSON();
// { type: 'Buffer', data: [72, 101, 108, 108, 111] }
```

Common use cases: file I/O, network protocols, cryptography, binary data processing.

---

## 7. What is the cluster module?

The cluster module enables creating child processes that share the same server port, utilizing multiple CPU cores.

```js
const cluster = require('cluster');
const http = require('http');
const os = require('os');

if (cluster.isPrimary) {
  const numCPUs = os.cpus().length;

  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork(); // restart worker
  });
} else {
  // Workers share the same TCP port
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end(`Handled by worker ${process.pid}\n`);
  }).listen(3000);
}
```

Alternatives: **PM2** (process manager with clustering built-in), **worker_threads** (for CPU-intensive tasks without separate processes).

---

## 8. What is the `worker_threads` module?

`worker_threads` enables running JavaScript in parallel threads, useful for CPU-intensive operations.

```js
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');

if (isMainThread) {
  // Main thread
  const worker = new Worker(__filename, {
    workerData: { numbers: [1, 2, 3, 4, 5] }
  });

  worker.on('message', (result) => {
    console.log('Sum:', result); // Sum: 15
  });

  worker.on('error', (err) => {
    console.error('Worker error:', err);
  });

  worker.on('exit', (code) => {
    console.log(`Worker exited with code ${code}`);
  });
} else {
  // Worker thread
  const sum = workerData.numbers.reduce((a, b) => a + b, 0);
  parentPort.postMessage(sum);
}
```

**When to use:**
- CPU-intensive computations (image processing, encryption, parsing)
- Parallel processing of independent tasks

**When NOT to use:**
- I/O-bound operations (use async/await instead)
- Simple tasks (overhead of creating threads may outweigh benefits)

---

## 9. What is middleware in Express.js?

Middleware functions have access to `req`, `res`, and `next`. They execute in order and can modify the request/response or end the cycle.

```js
const express = require('express');
const app = express();

// Built-in middleware
app.use(express.json());                    // parse JSON body
app.use(express.urlencoded({ extended: true })); // parse URL-encoded body
app.use(express.static('public'));          // serve static files

// Custom middleware
function logger(req, res, next) {
  console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
  next(); // pass control to next middleware
}

// Error handling middleware (4 params)
function errorHandler(err, req, res, next) {
  console.error(err.stack);
  res.status(500).json({ error: 'Internal Server Error' });
}

// Application-level
app.use(logger);

// Route-level
app.get('/api/users', authenticate, (req, res) => {
  res.json(users);
});

// Error handler (must be last)
app.use(errorHandler);
```

Middleware execution order matters. They run sequentially in the order they are registered.

---

## 10. What is the difference between `req.params`, `req.query`, and `req.body`?

```js
// Route: PUT /api/users/42?role=admin
// Body: { "name": "Alice" }

app.put('/api/users/:id', (req, res) => {
  req.params  // { id: '42' }        — URL path parameters
  req.query   // { role: 'admin' }    — URL query string (?key=value)
  req.body    // { name: 'Alice' }    — request body (needs parser middleware)
});

// Multiple params
// GET /api/posts/123/comments/456
app.get('/api/posts/:postId/comments/:commentId', (req, res) => {
  req.params // { postId: '123', commentId: '456' }
});
```

---

## 11. How does error handling work in Node.js?

```js
// 1. try/catch — for synchronous and async/await code
async function fetchData() {
  try {
    const data = await db.query('SELECT * FROM users');
    return data;
  } catch (err) {
    console.error('Query failed:', err);
    throw err;
  }
}

// 2. Error-first callbacks — traditional Node pattern
const fs = require('fs');
fs.readFile('file.txt', (err, data) => {
  if (err) {
    console.error('Read failed:', err);
    return;
  }
  console.log(data.toString());
});

// 3. Event emitters — error event
const stream = fs.createReadStream('file.txt');
stream.on('error', (err) => {
  console.error('Stream error:', err);
});

// 4. Promise rejection
somePromise()
  .then(result => { /* ... */ })
  .catch(err => console.error(err));

// 5. Uncaught exceptions (last resort)
process.on('uncaughtException', (err) => {
  console.error('Uncaught:', err);
  process.exit(1); // should exit — state may be inconsistent
});

// 6. Unhandled rejections
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled rejection:', reason);
});

// Custom error classes
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.name = 'AppError';
  }
}
```

---

## 12. What is the `EventEmitter` class?

`EventEmitter` is the foundation of Node's event-driven architecture. Many built-in modules (streams, HTTP, etc.) extend it.

```js
const EventEmitter = require('events');

class OrderService extends EventEmitter {
  placeOrder(order) {
    // process order...
    this.emit('orderPlaced', order);
  }
}

const service = new OrderService();

// Register listeners
service.on('orderPlaced', (order) => {
  console.log('Send confirmation email for:', order.id);
});

service.on('orderPlaced', (order) => {
  console.log('Update inventory for:', order.id);
});

// One-time listener
service.once('orderPlaced', (order) => {
  console.log('First order bonus applied');
});

// Remove listener
const handler = (order) => console.log(order);
service.on('orderPlaced', handler);
service.off('orderPlaced', handler);

// Error handling
service.on('error', (err) => {
  console.error('Service error:', err);
});

// Max listeners (default 10, increase if needed)
service.setMaxListeners(20);
```

---

## 13. What is the `fs` module?

The `fs` module provides file system operations. It offers three APIs: callback, synchronous, and promise-based.

```js
const fs = require('fs');
const fsPromises = require('fs/promises');

// Callback API
fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});

// Synchronous API (blocks the event loop — avoid in servers)
const data = fs.readFileSync('file.txt', 'utf8');

// Promise API (recommended)
async function readFile() {
  const data = await fsPromises.readFile('file.txt', 'utf8');
  console.log(data);
}

// Common operations
await fsPromises.writeFile('out.txt', 'Hello');
await fsPromises.appendFile('out.txt', ' World');
await fsPromises.rename('old.txt', 'new.txt');
await fsPromises.unlink('temp.txt');              // delete file
await fsPromises.mkdir('new-dir', { recursive: true });
await fsPromises.rmdir('old-dir', { recursive: true });
const stats = await fsPromises.stat('file.txt');
const files = await fsPromises.readdir('.');
const exists = fs.existsSync('file.txt');         // sync check is fine

// Watch for changes
fs.watch('file.txt', (eventType, filename) => {
  console.log(`${filename} changed: ${eventType}`);
});
```

---

## 14. What is the `path` module?

The `path` module provides utilities for working with file and directory paths.

```js
const path = require('path');

path.join('/users', 'alice', 'docs', 'file.txt');
// '/users/alice/docs/file.txt'

path.resolve('src', 'utils', 'helper.js');
// '/absolute/path/to/src/utils/helper.js'

path.basename('/users/alice/file.txt');       // 'file.txt'
path.basename('/users/alice/file.txt', '.txt'); // 'file'

path.dirname('/users/alice/file.txt');        // '/users/alice'
path.extname('file.tar.gz');                  // '.gz'

path.parse('/users/alice/file.txt');
// { root: '/', dir: '/users/alice', base: 'file.txt', ext: '.txt', name: 'file' }

path.format({ dir: '/users/alice', base: 'file.txt' });
// '/users/alice/file.txt'

path.isAbsolute('/foo');  // true
path.isAbsolute('./foo'); // false

path.relative('/users/alice', '/users/bob'); // '../bob'

// __dirname and __filename
console.log(__dirname);  // directory of current file (CJS)
console.log(__filename); // full path of current file (CJS)

// ESM equivalent
import { fileURLToPath } from 'url';
const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);
```

---

## 15. What is the `http` module?

The `http` module creates HTTP servers and makes HTTP requests.

```js
const http = require('http');

// Create a server
const server = http.createServer((req, res) => {
  const { method, url, headers } = req;

  if (method === 'GET' && url === '/api/health') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ status: 'ok' }));
  } else if (method === 'POST' && url === '/api/data') {
    let body = '';
    req.on('data', chunk => { body += chunk; });
    req.on('end', () => {
      const data = JSON.parse(body);
      res.writeHead(201, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ received: data }));
    });
  } else {
    res.writeHead(404);
    res.end('Not Found');
  }
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});

// Make HTTP request (built-in, no external library)
const req = http.request('http://api.example.com/data', (res) => {
  let data = '';
  res.on('data', chunk => { data += chunk; });
  res.on('end', () => console.log(JSON.parse(data)));
});
req.end();

// Modern: fetch API (available in Node 18+)
const response = await fetch('http://api.example.com/data');
const data = await response.json();
```

---

## 16. What is environment variable management in Node.js?

```js
// Accessing environment variables
const port = process.env.PORT || 3000;
const nodeEnv = process.env.NODE_ENV;
const dbUrl = process.env.DATABASE_URL;

// Setting at runtime
// PORT=3000 NODE_ENV=production node app.js

// Using dotenv for .env files
require('dotenv').config();
// or with ESM:
import 'dotenv/config';

// .env file
// PORT=3000
// DATABASE_URL=postgres://localhost:5432/mydb
// JWT_SECRET=supersecretkey

// Validation pattern
function requireEnv(key) {
  const value = process.env[key];
  if (!value) {
    throw new Error(`Missing required environment variable: ${key}`);
  }
  return value;
}

const dbUrl = requireEnv('DATABASE_URL');
```

Best practices:
- Never commit `.env` files to version control
- Provide a `.env.example` with placeholder values
- Validate required variables at startup
- Use different `.env` files for different environments

---

## 17. What is the `child_process` module?

`child_process` allows spawning subprocesses to run system commands or other scripts.

```js
const { exec, execFile, spawn, fork } = require('child_process');

// exec — runs a shell command, buffers output
exec('ls -la', (err, stdout, stderr) => {
  if (err) throw err;
  console.log(stdout);
});

// execFile — runs a file directly (no shell, more secure)
execFile('node', ['script.js'], (err, stdout) => {
  console.log(stdout);
});

// spawn — streams output (better for large output)
const child = spawn('find', ['.', '-name', '*.js']);
child.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});
child.stderr.on('data', (data) => {
  console.error(`stderr: ${data}`);
});
child.on('close', (code) => {
  console.log(`exited with code ${code}`);
});

// fork — spawns a new Node.js process with IPC channel
// parent.js
const child = fork('./worker.js');
child.send({ task: 'compute', data: [1, 2, 3] });
child.on('message', (result) => {
  console.log('Result:', result);
});

// worker.js
process.on('message', (msg) => {
  const result = msg.data.reduce((a, b) => a + b, 0);
  process.send(result);
});
```

| Method | Shell | Buffered | IPC | Best for |
|---|---|---|---|---|
| `exec` | Yes | Yes | No | Short shell commands |
| `execFile` | No | Yes | No | Running executables safely |
| `spawn` | No | No (streams) | No | Long-running processes, large output |
| `fork` | No | No | Yes | Node.js child processes with messaging |

---

## 18. What is the `crypto` module?

The `crypto` module provides cryptographic operations.

```js
const crypto = require('crypto');

// Hashing
const hash = crypto.createHash('sha256')
  .update('password123')
  .digest('hex');

// HMAC (keyed hashing)
const hmac = crypto.createHmac('sha256', 'secret-key')
  .update('message')
  .digest('hex');

// Password hashing with scrypt (recommended for passwords)
async function hashPassword(password) {
  const salt = crypto.randomBytes(16).toString('hex');
  return new Promise((resolve, reject) => {
    crypto.scrypt(password, salt, 64, (err, derivedKey) => {
      if (err) reject(err);
      resolve(`${salt}:${derivedKey.toString('hex')}`);
    });
  });
}

async function verifyPassword(password, hash) {
  const [salt, key] = hash.split(':');
  return new Promise((resolve, reject) => {
    crypto.scrypt(password, salt, 64, (err, derivedKey) => {
      if (err) reject(err);
      resolve(crypto.timingSafeEqual(
        Buffer.from(key, 'hex'),
        derivedKey
      ));
    });
  });
}

// Random values
const uuid = crypto.randomUUID();
const randomBytes = crypto.randomBytes(32).toString('hex');
const randomInt = crypto.randomInt(1, 100);

// Encryption (AES-256-GCM)
function encrypt(text, key) {
  const iv = crypto.randomBytes(12);
  const cipher = crypto.createCipheriv('aes-256-gcm', key, iv);
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  const authTag = cipher.getAuthTag().toString('hex');
  return `${iv.toString('hex')}:${authTag}:${encrypted}`;
}
```

---

## 19. What is REST API design in Node.js?

```js
const express = require('express');
const app = express();
app.use(express.json());

// RESTful routes
app.get('/api/users',          getUsers);      // List all
app.get('/api/users/:id',      getUser);       // Get one
app.post('/api/users',         createUser);    // Create
app.put('/api/users/:id',      updateUser);    // Full update
app.patch('/api/users/:id',    patchUser);     // Partial update
app.delete('/api/users/:id',   deleteUser);    // Delete

// Controller example
async function getUsers(req, res) {
  try {
    const { page = 1, limit = 10, sort = 'name' } = req.query;
    const users = await User.find()
      .sort(sort)
      .skip((page - 1) * limit)
      .limit(Number(limit));

    res.json({
      data: users,
      page: Number(page),
      total: await User.countDocuments(),
    });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
}

async function createUser(req, res) {
  try {
    const user = await User.create(req.body);
    res.status(201).json({ data: user });
  } catch (err) {
    if (err.code === 11000) {
      res.status(409).json({ error: 'User already exists' });
    } else {
      res.status(400).json({ error: err.message });
    }
  }
}
```

REST best practices:
- Use nouns for resources (`/users`, not `/getUsers`)
- Use HTTP methods correctly (GET, POST, PUT, PATCH, DELETE)
- Return appropriate status codes (200, 201, 204, 400, 401, 403, 404, 500)
- Support pagination, filtering, sorting
- Version your API (`/api/v1/users`)
- Use consistent error response format

---

## 20. What is authentication in Node.js?

**JWT (JSON Web Token) authentication:**

```js
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

// Login — generate token
app.post('/api/login', async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email });

  if (!user || !(await bcrypt.compare(password, user.password))) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  const accessToken = jwt.sign(
    { userId: user.id, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: '15m' }
  );

  const refreshToken = jwt.sign(
    { userId: user.id },
    process.env.REFRESH_SECRET,
    { expiresIn: '7d' }
  );

  res.json({ accessToken, refreshToken });
});

// Auth middleware
function authenticate(req, res, next) {
  const authHeader = req.headers.authorization;
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'No token provided' });
  }

  const token = authHeader.split(' ')[1];

  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET);
    req.user = payload;
    next();
  } catch (err) {
    return res.status(401).json({ error: 'Invalid token' });
  }
}

// Protected route
app.get('/api/profile', authenticate, (req, res) => {
  res.json({ userId: req.user.userId });
});

// Authorization middleware
function authorize(...roles) {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
}

app.delete('/api/users/:id', authenticate, authorize('admin'), deleteUser);
```

---

## 21. What are common security practices for Node.js applications?

1. **Input validation** — validate and sanitize all user input

```js
const { body, validationResult } = require('express-validator');

app.post('/api/users',
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 8 }),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
  }
);
```

2. **Helmet** — set security HTTP headers

```js
const helmet = require('helmet');
app.use(helmet());
```

3. **Rate limiting** — prevent brute force and DDoS

```js
const rateLimit = require('express-rate-limit');
app.use('/api/', rateLimit({ windowMs: 15 * 60 * 1000, max: 100 }));
```

4. **CORS** — control cross-origin access

```js
const cors = require('cors');
app.use(cors({ origin: 'https://myapp.com', credentials: true }));
```

5. **SQL injection prevention** — use parameterized queries

```js
// Bad
db.query(`SELECT * FROM users WHERE id = ${req.params.id}`);
// Good
db.query('SELECT * FROM users WHERE id = $1', [req.params.id]);
```

6. **Other practices:**
- Hash passwords with bcrypt/scrypt (never store plaintext)
- Use HTTPS
- Keep dependencies updated (`npm audit`)
- Don't expose stack traces in production
- Use environment variables for secrets
- Implement CSRF protection for cookie-based auth

---

## 22. What is the difference between SQL and NoSQL databases with Node.js?

| Feature | SQL (PostgreSQL, MySQL) | NoSQL (MongoDB, Redis) |
|---|---|---|
| Data model | Tables with rows/columns | Documents, key-value, graph |
| Schema | Fixed schema | Flexible schema |
| Relationships | JOINs, foreign keys | Embedded documents, references |
| Scaling | Vertical (primarily) | Horizontal (sharding) |
| ACID | Full support | Varies (MongoDB supports transactions) |
| Use case | Complex queries, relationships | Flexible data, high throughput |

**Popular Node.js ORMs/ODMs:**
- **Prisma** — type-safe ORM for SQL databases
- **Drizzle** — lightweight TypeScript ORM
- **Sequelize** — traditional ORM for SQL
- **Knex.js** — SQL query builder
- **Mongoose** — ODM for MongoDB
- **TypeORM** — ORM supporting SQL and MongoDB

```js
// Prisma example
const user = await prisma.user.findUnique({
  where: { id: 1 },
  include: { posts: true },
});

// Mongoose example
const user = await User.findById(id).populate('posts');
```

---

## 23. What is caching in Node.js?

```js
// In-memory caching (simple)
const cache = new Map();

function getCached(key, fetchFn, ttl = 60000) {
  const cached = cache.get(key);
  if (cached && Date.now() - cached.timestamp < ttl) {
    return cached.data;
  }
  const data = fetchFn();
  cache.set(key, { data, timestamp: Date.now() });
  return data;
}

// Redis caching (distributed)
const Redis = require('ioredis');
const redis = new Redis();

async function getUser(id) {
  const cacheKey = `user:${id}`;

  // Check cache
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);

  // Fetch from DB
  const user = await db.query('SELECT * FROM users WHERE id = $1', [id]);

  // Cache with TTL
  await redis.set(cacheKey, JSON.stringify(user), 'EX', 3600); // 1 hour

  return user;
}

// Cache invalidation
async function updateUser(id, data) {
  await db.query('UPDATE users SET name = $1 WHERE id = $2', [data.name, id]);
  await redis.del(`user:${id}`); // invalidate cache
}

// Express middleware for response caching
function cacheMiddleware(duration) {
  return async (req, res, next) => {
    const key = `cache:${req.originalUrl}`;
    const cached = await redis.get(key);

    if (cached) {
      return res.json(JSON.parse(cached));
    }

    const originalJson = res.json.bind(res);
    res.json = (data) => {
      redis.set(key, JSON.stringify(data), 'EX', duration);
      return originalJson(data);
    };
    next();
  };
}

app.get('/api/products', cacheMiddleware(300), getProducts);
```

---

## 24. What is WebSocket communication in Node.js?

WebSockets provide full-duplex, persistent connections between client and server.

```js
// Using ws library
const { WebSocketServer } = require('ws');
const wss = new WebSocketServer({ port: 8080 });

wss.on('connection', (ws, req) => {
  console.log('Client connected');

  ws.on('message', (data) => {
    const message = JSON.parse(data);
    console.log('Received:', message);

    // Echo back
    ws.send(JSON.stringify({ echo: message }));

    // Broadcast to all clients
    wss.clients.forEach((client) => {
      if (client !== ws && client.readyState === 1) {
        client.send(JSON.stringify(message));
      }
    });
  });

  ws.on('close', () => {
    console.log('Client disconnected');
  });

  // Send welcome message
  ws.send(JSON.stringify({ type: 'welcome', message: 'Connected!' }));
});

// Using Socket.IO (more features)
const { Server } = require('socket.io');
const io = new Server(httpServer, {
  cors: { origin: 'http://localhost:3000' },
});

io.on('connection', (socket) => {
  // Rooms
  socket.join('room-1');

  // Emit to specific room
  io.to('room-1').emit('message', 'Hello room!');

  // Namespaces
  const adminIo = io.of('/admin');
  adminIo.on('connection', (socket) => { /* ... */ });
});
```

---

## 25. What is the `os` module?

The `os` module provides operating system-related utility methods.

```js
const os = require('os');

os.platform();     // 'darwin', 'linux', 'win32'
os.arch();         // 'x64', 'arm64'
os.cpus();         // CPU info array
os.cpus().length;  // number of CPU cores
os.totalmem();     // total memory in bytes
os.freemem();      // free memory in bytes
os.homedir();      // user home directory
os.tmpdir();       // temp directory
os.hostname();     // machine hostname
os.uptime();       // system uptime in seconds
os.networkInterfaces(); // network interface info
os.type();         // 'Darwin', 'Linux', 'Windows_NT'
os.EOL;            // end-of-line character for the OS
```

---

## 26. What is the `process` object?

`process` is a global object that provides info and control over the current Node.js process.

```js
// Process info
process.pid;           // process ID
process.ppid;          // parent process ID
process.version;       // Node.js version
process.versions;      // versions of Node and dependencies
process.platform;      // 'darwin', 'linux', 'win32'
process.arch;          // 'x64', 'arm64'
process.cwd();         // current working directory
process.memoryUsage(); // memory stats

// Command line arguments
// node app.js --port 3000
process.argv;
// ['node', '/path/to/app.js', '--port', '3000']

// Standard I/O
process.stdin;   // readable stream
process.stdout;  // writable stream
process.stderr;  // writable stream

// Exit
process.exit(0);       // success
process.exit(1);       // failure
process.exitCode = 1;  // set exit code without forcing immediate exit

// Signals
process.on('SIGTERM', () => {
  console.log('Graceful shutdown');
  server.close(() => process.exit(0));
});

process.on('SIGINT', () => {
  console.log('Interrupted');
  process.exit(0);
});

// Timers
process.nextTick(() => { /* runs before I/O */ });

// Resource usage
process.cpuUsage();
process.resourceUsage();
```

---

## 27. What is graceful shutdown?

Graceful shutdown means properly closing all connections and finishing pending requests before the process exits.

```js
const http = require('http');

const server = http.createServer(app);
const connections = new Set();

server.on('connection', (conn) => {
  connections.add(conn);
  conn.on('close', () => connections.delete(conn));
});

async function shutdown(signal) {
  console.log(`${signal} received. Starting graceful shutdown...`);

  // Stop accepting new connections
  server.close(async () => {
    console.log('HTTP server closed');

    // Close database connections
    await db.disconnect();
    console.log('Database disconnected');

    // Close Redis
    await redis.quit();
    console.log('Redis disconnected');

    process.exit(0);
  });

  // Force close connections after timeout
  setTimeout(() => {
    console.error('Forcing shutdown');
    connections.forEach(conn => conn.destroy());
    process.exit(1);
  }, 10000);
}

process.on('SIGTERM', () => shutdown('SIGTERM'));
process.on('SIGINT', () => shutdown('SIGINT'));
```

---

## 28. What are design patterns commonly used in Node.js?

**Singleton (database connection):**
```js
class Database {
  static instance;

  static getInstance() {
    if (!Database.instance) {
      Database.instance = new Database();
    }
    return Database.instance;
  }

  async connect(url) {
    this.client = await MongoClient.connect(url);
  }
}
```

**Repository pattern:**
```js
class UserRepository {
  constructor(db) { this.db = db; }

  async findById(id) {
    return this.db.query('SELECT * FROM users WHERE id = $1', [id]);
  }

  async create(data) {
    return this.db.query(
      'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
      [data.name, data.email]
    );
  }
}
```

**Factory pattern:**
```js
function createLogger(type) {
  switch (type) {
    case 'file': return new FileLogger();
    case 'console': return new ConsoleLogger();
    case 'remote': return new RemoteLogger();
    default: throw new Error(`Unknown logger type: ${type}`);
  }
}
```

**Middleware/Chain of Responsibility:**
```js
// Express middleware pattern
const pipeline = [authenticate, validate, authorize, handler];
app.post('/api/orders', ...pipeline);
```

**Observer (EventEmitter):**
```js
const emitter = new EventEmitter();
emitter.on('userCreated', sendWelcomeEmail);
emitter.on('userCreated', createDefaultSettings);
emitter.emit('userCreated', user);
```

---

## 29. What is testing in Node.js?

```js
// Jest example
describe('UserService', () => {
  // Setup and teardown
  beforeAll(async () => { await db.connect(); });
  afterAll(async () => { await db.disconnect(); });
  beforeEach(async () => { await db.clear(); });

  test('should create a user', async () => {
    const user = await userService.create({
      name: 'Alice',
      email: 'alice@example.com',
    });

    expect(user).toBeDefined();
    expect(user.name).toBe('Alice');
    expect(user.email).toBe('alice@example.com');
    expect(user.id).toBeDefined();
  });

  test('should throw on duplicate email', async () => {
    await userService.create({ name: 'Alice', email: 'a@b.com' });

    await expect(
      userService.create({ name: 'Bob', email: 'a@b.com' })
    ).rejects.toThrow('Email already exists');
  });
});

// Mocking
jest.mock('./emailService');
const emailService = require('./emailService');

test('sends welcome email on signup', async () => {
  emailService.send.mockResolvedValue(true);

  await userService.signup({ name: 'Alice', email: 'a@b.com' });

  expect(emailService.send).toHaveBeenCalledWith(
    'a@b.com',
    expect.stringContaining('Welcome')
  );
});

// Supertest for HTTP endpoints
const request = require('supertest');

test('GET /api/users returns 200', async () => {
  const res = await request(app)
    .get('/api/users')
    .set('Authorization', `Bearer ${token}`)
    .expect(200);

  expect(res.body.data).toHaveLength(3);
});
```

Testing tools:
- **Jest** — test runner, assertions, mocking
- **Vitest** — fast, Vite-native test runner
- **Supertest** — HTTP endpoint testing
- **Sinon** — spies, stubs, mocks
- **Node's built-in test runner** — `node:test` (Node 18+)

---

## 30. What is logging best practice in Node.js?

```js
// Using winston
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'user-service' },
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' }),
  ],
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple(),
  }));
}

// Usage
logger.info('User created', { userId: user.id, email: user.email });
logger.error('Database error', { error: err.message, stack: err.stack });
logger.warn('Rate limit approaching', { ip: req.ip, count: 95 });

// Using pino (faster, structured logging)
const pino = require('pino');
const logger = pino({ level: 'info' });

logger.info({ userId: 123 }, 'User logged in');
logger.error({ err }, 'Request failed');
```

Best practices:
- Use structured logging (JSON) for machine parsing
- Include request IDs for tracing
- Use appropriate log levels (error, warn, info, debug)
- Never log sensitive data (passwords, tokens, PII)
- Use a logging library (not `console.log` in production)

---

## 31. What is the `url` module?

```js
const { URL } = require('url');

const myUrl = new URL('https://user:pass@example.com:8080/path?query=1#hash');

myUrl.protocol;   // 'https:'
myUrl.username;   // 'user'
myUrl.password;   // 'pass'
myUrl.hostname;   // 'example.com'
myUrl.port;       // '8080'
myUrl.pathname;   // '/path'
myUrl.search;     // '?query=1'
myUrl.hash;       // '#hash'
myUrl.origin;     // 'https://example.com:8080'

// URLSearchParams
const params = new URLSearchParams('page=1&limit=10&sort=name');
params.get('page');       // '1'
params.has('limit');      // true
params.set('page', '2');
params.append('filter', 'active');
params.delete('sort');
params.toString();        // 'page=2&limit=10&filter=active'

// Building URLs
const url = new URL('/api/users', 'https://example.com');
url.searchParams.set('page', '1');
url.toString(); // 'https://example.com/api/users?page=1'
```

---

## 32. What is the difference between `spawn`, `exec`, `execFile`, and `fork`?

```js
const { spawn, exec, execFile, fork } = require('child_process');

// exec — shell command, buffered output
// Use for: short commands where you need the full output
exec('ls -la | grep node', (err, stdout) => {
  console.log(stdout); // full output as string
});

// execFile — run executable, no shell, buffered output
// Use for: running binaries safely (no shell injection risk)
execFile('/usr/bin/git', ['status'], (err, stdout) => {
  console.log(stdout);
});

// spawn — shell command, streamed output
// Use for: long-running processes, large output
const child = spawn('ffmpeg', ['-i', 'input.mp4', 'output.avi']);
child.stdout.on('data', (data) => process.stdout.write(data));

// fork — spawn Node.js process with IPC
// Use for: running Node.js scripts with message passing
const worker = fork('./heavy-computation.js');
worker.send({ task: 'process', data: largeDataSet });
worker.on('message', (result) => console.log(result));
```

---

## 33. What is the `util` module?

The `util` module provides utility functions.

```js
const util = require('util');

// Promisify — convert callback-based functions to promises
const fs = require('fs');
const readFile = util.promisify(fs.readFile);
const data = await readFile('file.txt', 'utf8');

// Callbackify — convert promise-based to callback-based
const asyncFn = async () => 'hello';
const callbackFn = util.callbackify(asyncFn);

// inspect — convert object to string with formatting
console.log(util.inspect(deepObject, {
  depth: null,     // show all levels
  colors: true,    // colorize output
  showHidden: true // show non-enumerable properties
}));

// types — type checking
util.types.isPromise(Promise.resolve());  // true
util.types.isDate(new Date());            // true
util.types.isRegExp(/abc/);               // true
util.types.isAsyncFunction(async () => {}); // true

// format — printf-style string formatting
util.format('%s has %d items', 'Cart', 5); // 'Cart has 5 items'

// deprecate — mark function as deprecated
const oldFn = util.deprecate(
  () => { /* ... */ },
  'oldFn() is deprecated. Use newFn() instead.'
);
```

---

## 34. What is the `zlib` module?

The `zlib` module provides compression and decompression using Gzip, Deflate, and Brotli.

```js
const zlib = require('zlib');
const { promisify } = require('util');

const gzip = promisify(zlib.gzip);
const gunzip = promisify(zlib.gunzip);

// Compress data
const input = 'Hello World! '.repeat(1000);
const compressed = await gzip(Buffer.from(input));
console.log(`Original: ${input.length}, Compressed: ${compressed.length}`);

// Decompress data
const decompressed = await gunzip(compressed);
console.log(decompressed.toString());

// Stream compression (for files)
const fs = require('fs');
fs.createReadStream('input.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('input.txt.gz'));

// HTTP compression middleware (Express)
const compression = require('compression');
app.use(compression()); // automatically gzip responses
```

---

## 35. What is PM2?

PM2 is a production process manager for Node.js applications.

```bash
# Install
npm install -g pm2

# Start application
pm2 start app.js
pm2 start app.js -i max      # cluster mode (all CPUs)
pm2 start app.js --name api   # named process

# Management
pm2 list                      # list processes
pm2 logs                      # view logs
pm2 monit                     # real-time monitoring
pm2 restart api               # restart process
pm2 stop api                  # stop process
pm2 delete api                # remove process

# Ecosystem file (ecosystem.config.js)
module.exports = {
  apps: [{
    name: 'api',
    script: './app.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'development',
      PORT: 3000,
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 8080,
    },
  }],
};

# Start with ecosystem file
pm2 start ecosystem.config.js --env production

# Startup script (auto-restart on reboot)
pm2 startup
pm2 save
```

---

## 36. What is Docker with Node.js?

```dockerfile
# Dockerfile
FROM node:20-alpine

WORKDIR /app

# Copy package files first (better layer caching)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Don't run as root
USER node

EXPOSE 3000

CMD ["node", "app.js"]
```

```yaml
# docker-compose.yml
services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://db:5432/myapp
    depends_on:
      - db
      - redis

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine

volumes:
  pgdata:
```

Best practices:
- Use multi-stage builds for smaller images
- Use `npm ci` instead of `npm install` for deterministic installs
- Copy `package.json` before source code for better caching
- Use `.dockerignore` to exclude `node_modules`, `.git`, etc.
- Run as non-root user
- Use Alpine images for smaller size

---

## 37. What is the `net` module?

The `net` module provides low-level TCP/IPC networking.

```js
const net = require('net');

// TCP Server
const server = net.createServer((socket) => {
  console.log('Client connected');

  socket.on('data', (data) => {
    console.log('Received:', data.toString());
    socket.write('Echo: ' + data.toString());
  });

  socket.on('end', () => {
    console.log('Client disconnected');
  });

  socket.on('error', (err) => {
    console.error('Socket error:', err);
  });
});

server.listen(9000, () => {
  console.log('TCP server on port 9000');
});

// TCP Client
const client = net.createConnection({ port: 9000 }, () => {
  client.write('Hello, server!');
});

client.on('data', (data) => {
  console.log('Server says:', data.toString());
  client.end();
});
```

---

## 38. What are Node.js performance optimization techniques?

1. **Use async/await** — never block the event loop with synchronous operations

```js
// Bad
const data = fs.readFileSync('large-file.txt');
// Good
const data = await fsPromises.readFile('large-file.txt');
```

2. **Use streams for large data** — avoid loading everything into memory

```js
// Bad
const data = await fsPromises.readFile('1gb-file.txt');
// Good
const stream = fs.createReadStream('1gb-file.txt');
stream.pipe(transformStream).pipe(outputStream);
```

3. **Use worker threads for CPU-intensive tasks**

4. **Implement caching** — Redis for distributed, in-memory for local

5. **Use connection pooling** — for database connections

```js
const pool = new Pool({ max: 20, min: 5 });
```

6. **Enable HTTP compression** — gzip/brotli responses

7. **Use clustering** — utilize all CPU cores

8. **Profile and monitor**
```js
// Built-in profiler
node --prof app.js
node --prof-process isolate-*.log

// Heap snapshot
const v8 = require('v8');
v8.writeHeapSnapshot();
```

9. **Avoid memory leaks** — clean up event listeners, close connections, use WeakMap/WeakRef

10. **Use connection keep-alive**
```js
const agent = new http.Agent({ keepAlive: true, maxSockets: 50 });
```

---

## 39. What is the difference between `require` and `import`?

```js
// require (CommonJS) — synchronous, dynamic
const fs = require('fs');
const { readFile } = require('fs');

// Conditional require
if (condition) {
  const module = require('./optional-module');
}

// import (ES Modules) — asynchronous, static
import fs from 'fs';
import { readFile } from 'fs';

// Dynamic import (works in both CJS and ESM)
const module = await import('./module.js');

// import.meta (ESM only)
import.meta.url;    // file URL of current module
import.meta.resolve('./file.js'); // resolve relative path
```

| Feature | `require` (CJS) | `import` (ESM) |
|---|---|---|
| Loading | Synchronous | Asynchronous |
| Caching | `require.cache` | Module cache |
| Conditional | Yes | Only via dynamic `import()` |
| Tree shaking | No | Yes |
| Top-level await | No | Yes |
| `__dirname` / `__filename` | Available | Not available (use `import.meta.url`) |
| JSON import | Built-in | Needs assertion or `createRequire` |

---

## 40. What are Node.js LTS versions and release schedule?

Node.js follows a predictable release schedule:

- **Even-numbered versions** (18, 20, 22) become **LTS (Long Term Support)**
- **Odd-numbered versions** (19, 21, 23) are **Current** (short-lived, experimental)
- LTS versions receive **30 months** of support (18 months Active + 12 months Maintenance)
- A new major version is released every **6 months** (April and October)

For production, always use the current **Active LTS** version.

---

## 41. What is the `dns` module?

```js
const dns = require('dns');
const dnsPromises = require('dns/promises');

// Resolve hostname to IP
const addresses = await dnsPromises.resolve4('example.com');
// ['93.184.216.34']

// Lookup (uses OS resolver, respects /etc/hosts)
const { address, family } = await dnsPromises.lookup('localhost');
// { address: '127.0.0.1', family: 4 }

// Reverse DNS
const hostnames = await dnsPromises.reverse('8.8.8.8');
// ['dns.google']

// MX records
const mx = await dnsPromises.resolveMx('gmail.com');
// [{ exchange: 'gmail-smtp-in.l.google.com', priority: 5 }, ...]

// All record types
await dnsPromises.resolveAny('example.com');
await dnsPromises.resolveTxt('example.com');
await dnsPromises.resolveCname('www.example.com');
await dnsPromises.resolveNs('example.com');
```

---

## 42. What is the `assert` module?

The `assert` module provides assertion functions for testing and validation.

```js
const assert = require('assert/strict');

// Equality
assert.equal(1, 1);                       // OK
assert.notEqual(1, 2);                    // OK
assert.deepEqual({ a: 1 }, { a: 1 });    // OK (deep comparison)
assert.strictEqual(1, 1);                 // OK (=== comparison)

// Truthiness
assert.ok(true);                          // OK
assert.ok(1);                             // OK
assert.ok('');                            // AssertionError

// Errors
assert.throws(() => { throw new Error('fail'); }, Error);
await assert.rejects(async () => { throw new Error('fail'); }, Error);

// Custom messages
assert.equal(result, expected, 'Result did not match expected value');

// Match (partial object matching)
assert.match('hello world', /world/);     // OK
assert.doesNotMatch('hello', /world/);    // OK
```

For production testing, use Jest, Vitest, or Node's built-in test runner (`node:test`) which provides a richer testing API.

---

## 43. What are signals and how does Node.js handle them?

Signals are notifications sent to a process by the OS or other processes.

```js
// SIGINT — user pressed Ctrl+C
process.on('SIGINT', () => {
  console.log('Interrupted');
  cleanup();
  process.exit(0);
});

// SIGTERM — termination request (sent by kill, Docker, K8s)
process.on('SIGTERM', () => {
  console.log('Termination requested');
  gracefulShutdown();
});

// SIGHUP — terminal closed
process.on('SIGHUP', () => {
  reloadConfig();
});

// SIGUSR1 — used by Node.js for debugger
// SIGUSR2 — user-defined (e.g., pm2 uses for restart)

// Sending signals
process.kill(pid, 'SIGTERM'); // send signal to another process

// Cannot catch: SIGKILL (kill -9), SIGSTOP
```

---

## 44. What is a microservice architecture with Node.js?

Microservices split an application into small, independently deployable services.

```
┌─────────┐     ┌──────────────┐
│  Client  │────▶│  API Gateway │
└─────────┘     └──────┬───────┘
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
  ┌───────────┐  ┌───────────┐  ┌───────────┐
  │   User    │  │   Order   │  │  Payment  │
  │  Service  │  │  Service  │  │  Service  │
  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘
        ▼              ▼              ▼
    [User DB]      [Order DB]    [Payment DB]
```

**Communication patterns:**
- **HTTP/REST** — synchronous, request/response
- **Message queues** — async (RabbitMQ, AWS SQS, BullMQ)
- **Event streaming** — async (Kafka, Redis Streams)
- **gRPC** — high-performance RPC

```js
// BullMQ example (job queue)
const { Queue, Worker } = require('bullmq');

// Producer
const emailQueue = new Queue('emails');
await emailQueue.add('welcome', { to: 'user@example.com', name: 'Alice' });

// Consumer (separate service)
const worker = new Worker('emails', async (job) => {
  await sendEmail(job.data.to, `Welcome ${job.data.name}!`);
});
```

---

## 45. What is the `Intl` API in Node.js?

Node.js includes full ICU support for internationalization.

```js
// Number formatting
new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
}).format(1234.56);
// '$1,234.56'

new Intl.NumberFormat('de-DE').format(1234.56);
// '1.234,56'

// Date formatting
new Intl.DateTimeFormat('en-US', {
  dateStyle: 'full',
  timeStyle: 'short',
}).format(new Date());
// 'Thursday, January 30, 2026 at 2:30 PM'

// Relative time
new Intl.RelativeTimeFormat('en', { numeric: 'auto' }).format(-1, 'day');
// 'yesterday'

// Collation (sorting)
const collator = new Intl.Collator('de');
['ä', 'a', 'z'].sort(collator.compare);
// ['a', 'ä', 'z']

// Plural rules
const pr = new Intl.PluralRules('en');
pr.select(0); // 'other'
pr.select(1); // 'one'
pr.select(2); // 'other'

// List formatting
new Intl.ListFormat('en', { type: 'conjunction' }).format(['Alice', 'Bob', 'Charlie']);
// 'Alice, Bob, and Charlie'
```

---

## 46. What are Node.js native test runner features?

Node.js 18+ includes a built-in test runner (`node:test`).

```js
const { describe, it, before, after, mock } = require('node:test');
const assert = require('node:assert/strict');

describe('UserService', () => {
  before(() => { /* setup */ });
  after(() => { /* cleanup */ });

  it('should create a user', async () => {
    const user = await userService.create({ name: 'Alice' });
    assert.equal(user.name, 'Alice');
    assert.ok(user.id);
  });

  it('should throw for invalid input', async () => {
    await assert.rejects(
      () => userService.create({}),
      { message: 'Name is required' }
    );
  });

  // Mocking
  it('should call email service', async () => {
    const sendMock = mock.fn(() => Promise.resolve(true));
    const service = new UserService({ send: sendMock });

    await service.signup({ name: 'Alice', email: 'a@b.com' });

    assert.equal(sendMock.mock.calls.length, 1);
  });

  // Subtests
  it('validation', async (t) => {
    await t.test('rejects empty name', async () => {
      await assert.rejects(() => userService.create({ name: '' }));
    });

    await t.test('rejects invalid email', async () => {
      await assert.rejects(() => userService.create({ email: 'bad' }));
    });
  });
});

// Run: node --test
// Or: node --test --watch (watch mode)
```

---

## 47. What is the `AbortController` in Node.js?

`AbortController` provides a way to cancel asynchronous operations.

```js
// Cancel a fetch request
const controller = new AbortController();
const { signal } = controller;

setTimeout(() => controller.abort(), 5000); // cancel after 5s

try {
  const response = await fetch('https://api.example.com/data', { signal });
  const data = await response.json();
} catch (err) {
  if (err.name === 'AbortError') {
    console.log('Request was cancelled');
  }
}

// Cancel a stream operation
const stream = fs.createReadStream('large-file.txt', { signal });

// Cancel with setTimeout
const data = await fsPromises.readFile('file.txt', {
  signal: AbortSignal.timeout(3000) // built-in timeout signal
});

// Cancel EventEmitter listeners
const ee = new EventEmitter();
ee.on('data', handler, { signal });
controller.abort(); // automatically removes the listener

// Custom cancellable operations
async function cancellableTask(signal) {
  for (const item of items) {
    if (signal.aborted) throw new Error('Cancelled');
    await processItem(item);
  }
}
```

---

## 48. What is the difference between Node.js frameworks?

| Framework | Style | Use Case |
|---|---|---|
| **Express** | Minimal, unopinionated | General-purpose APIs, traditional apps |
| **Fastify** | Performance-focused | High-throughput APIs |
| **Koa** | Minimal, modern (async/await) | Middleware-heavy apps |
| **NestJS** | Opinionated, Angular-inspired | Enterprise apps, microservices |
| **Hono** | Ultra-fast, edge-compatible | Edge/serverless, small APIs |
| **Hapi** | Configuration-driven | Enterprise apps with strict config |
| **Adonis** | Full-featured MVC | Laravel-like full-stack apps |

```js
// Express
app.get('/hello', (req, res) => res.json({ hello: 'world' }));

// Fastify
fastify.get('/hello', async () => ({ hello: 'world' }));

// Koa
router.get('/hello', (ctx) => { ctx.body = { hello: 'world' }; });

// NestJS
@Controller()
export class AppController {
  @Get('/hello')
  getHello() { return { hello: 'world' }; }
}

// Hono
app.get('/hello', (c) => c.json({ hello: 'world' }));
```

---

## 49. What are common Node.js anti-patterns?

1. **Blocking the event loop** — using sync methods (`readFileSync`) in request handlers
2. **Not handling errors** — unhandled rejections and uncaught exceptions crash the process
3. **Callback hell** — deeply nested callbacks instead of using async/await
4. **Memory leaks** — not removing event listeners, accumulating data in closures
5. **Not using environment variables** — hardcoding secrets and configuration
6. **Using `console.log` in production** — use a structured logging library
7. **Not implementing graceful shutdown** — killing processes abruptly loses in-flight requests
8. **Ignoring backpressure** — writing to streams faster than they can be consumed
9. **Not validating input** — trusting user input leads to injection attacks
10. **Monolithic `node_modules` copy in Docker** — not using `.dockerignore` and multi-stage builds

---

## 50. What are the key differences between Node.js versions?

| Version | Key Features |
|---|---|
| **Node 14** | Optional chaining, nullish coalescing, `AsyncLocalStorage` |
| **Node 16** | Apple Silicon support, `timersPromises`, V8 9.0 |
| **Node 18 (LTS)** | Built-in `fetch`, test runner (`node:test`), `--watch` mode |
| **Node 20 (LTS)** | Stable test runner, stable `import.meta.resolve`, Permission Model |
| **Node 22 (LTS)** | `require()` for ESM, WebSocket client, `glob`/`globSync` in `fs` |

Key trends:
- Built-in APIs replacing third-party packages (fetch, test runner, watch mode)
- Better ESM support and CJS/ESM interop
- Performance improvements with each V8 update
- Security features (Permission Model)
- Web-standard APIs being added (fetch, WebSocket, Blob, FormData)
