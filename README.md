# Safar

![Node.js](https://img.shields.io/badge/Node.js-22-339933?logo=nodedotjs&logoColor=white)
![Express](https://img.shields.io/badge/Express-5-000000?logo=express&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?logo=postgresql&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-blue)

Safar is a REST API for tracking job applications. It keeps every status change, not just the latest one. An application that went from interview to rejected still counts as an interview, so the stats show which job boards actually lead somewhere.

**[Live API](https://safar-dyq2.onrender.com/health)**: it runs on Render's free tier, so the first request can take about a minute.

```json
GET /applications/12
{
  "company": "Globex",
  "role": "Backend Developer",
  "status": "rejected",
  "history": [
    { "from": null,        "to": "saved" },
    { "from": "saved",     "to": "applied" },
    { "from": "applied",   "to": "interview" },
    { "from": "interview", "to": "rejected" }
  ]
}
```

## API

| Method | Route | What it does |
|---|---|---|
| POST | `/auth/register` | Create an account |
| POST | `/auth/login` | Get a JWT |
| GET | `/applications` | List yours, with filters for status, company and date range, and pagination |
| POST | `/applications` | Add one |
| GET / PATCH / DELETE | `/applications/:id` | Read (with history and notes), edit or delete one |
| PATCH | `/applications/:id/status` | Change status and record it in the history |
| POST | `/applications/:id/notes` | Add a note |
| GET | `/stats` | Totals, counts by status, response rate, and interviews per source |

## Interesting techniques

**Data integrity**

- **Atomic status changes.** A status change and its history row are written in one transaction. The row is locked with [`SELECT … FOR UPDATE`](https://www.postgresql.org/docs/current/sql-select.html#SQL-FOR-UPDATE-SHARE), so two requests at once can't record the wrong previous status.
- **One way to change status.** The general update schema is a strict object without `status`, so sending it returns [`400`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/400). Status only changes through the route that writes history. Setting the same status twice returns [`409 Conflict`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/409).
- **The database enforces the rules too.** Foreign keys with `ON DELETE CASCADE`, a `CHECK` constraint on status, and a `UNIQUE` email back up the app's validation. A duplicate signup is caught as Postgres error `23505` rather than with a check-then-insert, which would have a race.

**Security**

- **Per-user scoping.** Every query filters on the user id from the [`Authorization`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization) Bearer token. Another user's application returns [`404`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404), not 403, so ids aren't confirmed.
- **Dynamic SQL without injection.** Filters and partial updates generate `$n` placeholders. Column names come only from a fixed allow-list.
- **Errors stay generic.** Unknown errors are logged on the server, and the client only sees `internal server error`.

**Correctness**

- **Dates stay dates.** `DATE` columns are kept as `YYYY-MM-DD` strings instead of JavaScript [`Date`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date) objects. A `Date` set to local midnight shows up as the previous day in UTC.
- **Counts are real numbers.** `COUNT(*)` is cast to `int` because `pg` returns `bigint` as a string, since it can exceed [`Number.MAX_SAFE_INTEGER`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_SAFE_INTEGER).
- **Bad ids return 404.** Non-numeric ids and ids beyond Postgres's `INTEGER` range return 404 before any SQL runs.

**Performance**

- **One pass for the stats.** [`COUNT(*) FILTER (WHERE …)`](https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-AGGREGATES) with `EXISTS` over the history table produces several counts from a single scan.
- **Parallel reads.** [`Promise.all`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) loads history and notes together, and runs the three stats queries together.
- **Indexes match the queries.** Composite indexes like `(user_id, status)` and `(user_id, applied_on DESC)` follow the real query patterns.

## Technologies and libraries

- [Express 5](https://expressjs.com/en/guide/migrating-5.html) passes rejected promises from `async` handlers to the error middleware, so routes have no `try...catch`.
- [node-postgres (`pg`)](https://node-postgres.com/) runs plain SQL with no ORM. A [small migration runner](src/db/migrate.js) applies numbered `.sql` files once each, inside a transaction.
- [Zod 4](https://zod.dev/) validates and transforms input, for example trimming, lowercasing emails, and turning query strings into numbers. It returns errors per field.
- [bcryptjs](https://github.com/dcodeIO/bcrypt.js) hashes passwords in pure JavaScript. Passwords are capped at 72 bytes, which is bcrypt's input limit.
- [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken) handles stateless auth.
- The [Node.js test runner](https://nodejs.org/api/test.html) and [Supertest](https://github.com/ladjs/supertest) run 24 integration tests against a real Postgres database, with no mocks.
- Node's built-in [`--env-file`](https://nodejs.org/api/cli.html#--env-filefile) and [`--watch`](https://nodejs.org/api/cli.html#--watch) flags replace `dotenv` and `nodemon`.
- It's hosted on [Render](https://render.com/), with the database on [Neon](https://neon.tech/).

## Project structure

```
safar/
├── src/
│   ├── db/
│   │   └── migrations/
│   ├── middleware/
│   ├── modules/
│   │   ├── applications/
│   │   ├── auth/
│   │   └── stats/
│   └── utils/
├── tests/
├── .env.example
├── LICENSE
├── package.json
└── README.md
```

- [`src/db`](src/db) holds the connection pool, a `withTransaction` helper, and the migration runner.
- [`src/modules`](src/modules) has one folder per feature, each with the same four layers: routes for HTTP, a schema for validation, a service for the rules, and a repository for the SQL.
- [`src/middleware`](src/middleware) holds JWT checking and the central error handler.
- [`tests`](tests) holds integration tests that run against a separate `safar_test` database.

## Known limitations

- Login has no rate limiting.
- Tokens last 7 days and can't be revoked before they expire.
- Pagination uses `LIMIT`/`OFFSET`, which is fine for one person's data but not for very large tables.

Built by [Aditya Chandra Prajapati](https://github.com/air01aditya). Released under the [MIT License](LICENSE).
