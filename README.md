# JSQL-NEO v6.0.1

Rust-powered embedded database. Runs via **WASM** (zero native deps) or as a standalone **HTTP server**.

## Quick Start (WASM — no server needed)

```js
const { JSQL } = require('jsql-neo');
const db = new JSQL();
await db.start();

await db.createTable('users', {
  name: { type: 'string' },
  age:  { type: 'integer' }
});

const [id] = await db.insert('users', { name: 'Alice', age: 30 });
const user = await db.findById('users', id);
console.log(user);
```

## HTTP Server

```bash
# Start the server
JSQL_DATA_DIR=/tmp/jsql-neo npx jsql-neo

# Connect from JS
const { HttpJSQL } = require('jsql-neo');
const db = new HttpJSQL({ host: '127.0.0.1', port: 6379 });
```

## Features

- In-memory or persistent (WAL + snapshot) storage
- B-Tree indexing + HashMap O(1) PK lookups
- Strings Pool for memory-efficient string storage
- Batch insert / update / delete
- Cursor-based pagination
- Transaction support
- WASM — zero native dependencies, runs in Node.js and browsers
- **Plugin system** — extend JSQL-NEO with custom functionality

---

## Plugin System

JSQL-NEO v6.0.1 ships with a powerful plugin system that lets you hook into database lifecycle events, add custom query methods, and register new storage backends.

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Hook** | Synchronous or asynchronous callback triggered on a specific event |
| **Extension** | A collection of hooks + custom methods exposed on the `db` object |
| **Storage Backend** | A driver that handles how data is read/written/persisted |
| **Middleware** | Request/response transformer that wraps query execution |

### Hooks

Hooks let you react to database events. Register them via `db.hooks.on()`:

```js
// Before / after any query
db.hooks.on('query:before', ({ sql, params }) => {
  console.log('[query:before]', sql, params);
});

db.hooks.on('query:after', ({ sql, params, result, durationMs }) => {
  console.log('[query:after]', sql, '→', result, `(${durationMs}ms)`);
});

// Table lifecycle
db.hooks.on('table:create:before', ({ table }) => {
  console.log('[table:create:before]', table);
});
db.hooks.on('table:create:after',  ({ table, result }) => {
  console.log('[table:create:after]', table, result);
});
db.hooks.on('table:drop:before',  ({ table }) => { /* ... */ });
db.hooks.on('table:drop:after',   ({ table }) => { /* ... */ });

// Insert / update / delete
db.hooks.on('insert:before',  ({ table, record }) => { /* ... */ });
db.hooks.on('insert:after',   ({ table, record, id }) => { /* ... */ });
db.hooks.on('update:before',  ({ table, id, patch }) => { /* ... */ });
db.hooks.on('update:after',   ({ table, id, patch }) => { /* ... */ });
db.hooks.on('delete:before',  ({ table, id }) => { /* ... */ });
db.hooks.on('delete:after',   ({ table, id }) => { /* ... */ });

// Transaction lifecycle
db.hooks.on('transaction:begin',  ({ txId }) => { /* ... */ });
db.hooks.on('transaction:commit', ({ txId }) => { /* ... */ });
db.hooks.on('transaction:rollback', ({ txId }) => { /* ... */ });
```

### Extensions (Custom Methods)

Register extensions to add your own methods directly on the `db` instance:

```js
const myExtension = {
  name: 'my-extension',

  // Custom methods available as db.myMethod()
  methods: {
    // Simple sync or async method
    async findUsersByAge(minAge, maxAge) {
      const rows = await this.query(
        `SELECT * FROM users WHERE age >= ? AND age <= ?`,
        [minAge, maxAge]
      );
      return rows;
    },

    // Method that uses internal APIs
    async countRecords(table) {
      const result = await this.query(`SELECT COUNT(*) as count FROM ${table}`);
      return result[0].count;
    },
  },

  // Lifecycle hooks
  onInstall(db) {
    console.log('[my-extension] installed');
  },

  onUninstall(db) {
    console.log('[my-extension] uninstalled');
  },
};

// Install
db.use(myExtension);

// Use
const adults = await db.findUsersByAge(18, 65);
const count  = await db.countRecords('users');

// Uninstall
db.unuse('my-extension');
```

### Middleware

Middleware transforms queries before execution or results after:

```js
db.middleware.push(async (ctx, next) => {
  const start = Date.now();
  // Optionally modify ctx.sql or ctx.params
  ctx.params = ctx.params.map(p =>
    typeof p === 'string' ? p.trim() : p
  );
  const result = await next();
  const elapsed = Date.now() - start;
  if (elapsed > 100) {
    console.warn(`[slow-query] ${ctx.sql} took ${elapsed}ms`);
  }
  return result;
});
```

Middleware runs in registration order. Use `db.middleware.unshift()` to prepend.

### Storage Backends

Built-in storage backends:

```js
// In-memory (default, fastest, non-persistent)
const db = new JSQL({ storage: 'memory' });

// File-based WAL + snapshot (default when JSQL_DATA_DIR is set)
const db = new JSQL({ storage: 'wal', path: '/data/myapp' });

// SQLite-compatible via WASM
const db = new JSQL({ storage: 'sqlite-wasm', path: '/data/myapp.db' });
```

Write a custom backend by implementing the `StorageBackend` interface:

```js
class MyStorageBackend {
  name = 'my-backend';

  async init() { /* open connection, create schema, etc. */ }
  async read(key)  { /* return value or null */ }
  async write(key, value) { /* persist key-value */ }
  async delete(key) { /* remove key */ }
  async flush() { /* force-sync to disk */ }
  async close() { /* cleanup */ }
}

db.useStorage(new MyStorageBackend());
```

### Plugin Registry

List installed plugins and extensions:

```js
console.log(db.plugins.list());
// [ { name: 'my-extension', version: '1.0.0', hooks: [...], methods: [...] } ]

console.log(db.hooks.list());
// [ 'query:before', 'query:after', 'table:create:before', ... ]
```

### Built-in Plugins

JSQL-NEO v6.0.1 includes these plugins out of the box:

#### `@jsql/neo-stats` — Query Statistics

Collects per-query timing, row counts, and error rates.

```js
// Built-in, no install needed — just access the stats API
console.log(db.stats.queries);      // all queries
console.log(db.stats.slowQueries()); // queries > threshold
console.log(db.stats.reset());      // clear stats
```

#### `@jsql/neo-audit` — Audit Log

Logs every mutation (INSERT / UPDATE / DELETE) with timestamp, actor, and diff.

```js
db.use(require('@jsql/neo-audit')({ table: 'audit_log' }));
```

#### `@jsql/neo-migrate` — Migration Runner

Schema migration with up/down scripts and versioning.

```js
await db.migrate([
  {
    version: 1,
    up: async db => {
      await db.query(`CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT)`);
    },
    down: async db => {
      await db.query(`DROP TABLE users`);
    },
  },
  {
    version: 2,
    up: async db => {
      await db.query(`ALTER TABLE users ADD COLUMN email TEXT`);
    },
    down: async db => {
      await db.query(`ALTER TABLE users DROP COLUMN email`);
    },
  },
]);
await db.migrate.run();  // runs pending migrations
await db.migrate.list(); // shows migration history
```

### Best Practices

- **Keep hooks fast.** Heavy work in hooks blocks the query. Offload to async tasks or queues.
- **Name your extensions.** Unnamed extensions can't be uninstalled via `db.unuse(name)`.
- **Version your plugins.** Add a `version` field to extension objects for debugging.
- **Use transactions for multi-step plugin operations.** Plugins modifying multiple tables should wrap in `db.transaction()`.
- **Return meaningful context from middleware.** Adding `ctx.durationMs` or `ctx.error` helps downstream hooks.

### Community Plugins

Looking for more? Check out the [JSQL-NEO Plugin Directory](https://github.com/vexify-build/jsql-neo-plugins) for community-maintained plugins including:

- `@jsql/neo-redis` — Redis caching layer
- `@jsql/neo-graphql` — GraphQL schema generation
- `@jsql/neo-realtime` — Live query subscriptions via WebSockets
- `@jsql/neo-encryption` — Transparent column-level encryption

---

## API Reference

### `new JSQL(options?)`

Creates a new database instance.

```ts
interface JSQLOptions {
  storage?: 'memory' | 'wal' | 'sqlite-wasm' | StorageBackend;
  path?: string;           // path for persistent storage backends
  wasmUrl?: string;        // URL to jsql-neo.wasm (for browser)
  autoCreateTables?: boolean;
  stats?: boolean;         // enable built-in stats (default: true)
  hooks?: boolean;         // enable hook system (default: true)
}
```

### `db.query(sql, params?)`

Execute raw SQL.

### `db.findById(table, id)`

Get a record by primary key.

### `db.insert(table, record)` / `db.insertMany(table, records[])`

Insert one or many records.

### `db.update(table, id, patch)`

Update a record by ID.

### `db.delete(table, id)`

Delete a record by ID.

### `db.createTable(name, schema)`

Create a table from a schema object.

### `db.dropTable(name)`

Drop a table and all its data.

### `db.transaction(fn)`

Run a function inside a transaction. Automatically commits on success, rolls back on error.

```js
await db.transaction(async db => {
  await db.insert('users', { name: 'Bob' });
  await db.insert('logs',  { msg: 'user created' });
});
```

---

## Performance Tips

- Use **B-Tree indexes** on columns you filter/sort by frequently
- Use **batch operations** (`insertMany`, `updateMany`) for bulk work
- Set `JSQL_DATA_DIR` for **WAL persistence** — avoids full snapshot on every write
- Disable `stats` if you don't need query metrics: `new JSQL({ stats: false })`
- For read-heavy workloads, consider the `@jsql/neo-redis` cache plugin

---

## License

MIT
