<div align="center">

# Safar

![Node.js](https://img.shields.io/badge/Node.js-22-339933?logo=nodedotjs&logoColor=white)
![Express](https://img.shields.io/badge/Express-5-000000?logo=express&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?logo=postgresql&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-blue)

A backend API for tracking my job hunt: every application, every status change,
and which job boards actually get replies.

**[Live API](https://safar-dyq2.onrender.com/health)** (free tier, the first request can take about a minute to wake up)

</div>

## Why it's useful

A spreadsheet only shows where each application is now. Safar keeps the whole history.

- **Every status change is saved:** saved → applied → interview → offer / rejected / ghosted
- **Honest stats:** an application that went interview → rejected still counts as an interview
- **Which job boards work:** reply rate for each source (Naukri, LinkedIn, referrals...)
- **Notes on each application**
- **Your data only:** sign up and log in with a JWT; nobody else can see your applications

## API

| Method | Route | What it does |
|---|---|---|
| POST | `/auth/register` | Create an account |
| POST | `/auth/login` | Get a token |
| GET | `/applications` | List your applications |
| POST | `/applications` | Add one |
| GET / PATCH / DELETE | `/applications/:id` | Read, edit or delete one |
| PATCH | `/applications/:id/status` | Change its status (saved in the history) |
| POST | `/applications/:id/notes` | Add a note |
| GET | `/stats` | Totals, counts by status, reply rate by source |

## Getting started

Needs Node.js 22+ and PostgreSQL.

```bash
git clone https://github.com/air01aditya/safar.git
cd safar
npm install
cp .env.example .env        # fill in your Postgres password and a JWT secret
createdb safar && createdb safar_test
npm run migrate
npm run dev                 # http://localhost:3000
```

Run the tests:

```bash
npm test
```

## Maintainer

Built and maintained by [Aditya Chandra Prajapati](https://github.com/air01aditya).
Released under the [MIT License](LICENSE).
