<div align="center">

# Safar

**A job-application tracker API that remembers every status change.**

![Node.js](https://img.shields.io/badge/Node.js-22-339933?logo=nodedotjs&logoColor=white)
![Express](https://img.shields.io/badge/Express-5-000000?logo=express&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?logo=postgresql&logoColor=white)
![Tests](https://img.shields.io/badge/tests-24%20passing-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)

[**Live API**](https://safar-dyq2.onrender.com/health) · free tier, first request takes ~1 min

</div>

`GET /applications/12`

```json
{
  "company": "Globex",
  "status": "rejected",
  "history": [
    { "from": "saved",     "to": "applied" },
    { "from": "applied",   "to": "interview" },
    { "from": "interview", "to": "rejected" }
  ]
}
```

Rejected, but it still counts as an interview. `/stats` uses the history, so you see which job boards actually work.

## Highlights

- **Atomic history:** status and history are written in one transaction, with [`FOR UPDATE`](https://www.postgresql.org/docs/current/sql-select.html#SQL-FOR-UPDATE-SHARE) row locks
- **Private by default:** every query is scoped to the JWT user, and other users' data returns `404`
- **Plain SQL:** no ORM, [`COUNT FILTER`](https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-AGGREGATES) stats in a single pass
- **Validated input:** [Zod](https://zod.dev/) errors come back per field
- **Real tests:** 24 integration tests on Postgres with [`node:test`](https://nodejs.org/api/test.html) and Supertest, no mocks

## API

| | Route | |
|---|---|---|
| `POST` | `/auth/register` · `/auth/login` | Sign up, get a JWT |
| `GET` `POST` | `/applications` | List (filters + paging) / add |
| `GET` `PATCH` `DELETE` | `/applications/:id` | Read with history and notes / edit / delete |
| `PATCH` | `/applications/:id/status` | Move status, saved to history |
| `POST` | `/applications/:id/notes` | Add a note |
| `GET` | `/stats` | Totals, response rate, interviews per source |

---

<div align="center">

Built by [Aditya Chandra Prajapati](https://github.com/air01aditya) · [MIT](LICENSE)

</div>
