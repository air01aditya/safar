# Safar

A backend API for tracking my job hunt: every application, every status change, and which
job boards actually lead to interviews.

**[Live API](https://safar-dyq2.onrender.com/health)** (free tier, the first request can take about a minute to wake up)

Node.js · Express · PostgreSQL · JWT auth · 24 API tests

## Try it

```bash
curl -X POST https://safar-dyq2.onrender.com/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"you@example.com","password":"a-long-password"}'
```

## Run it yourself

Needs Node 22+ and PostgreSQL.

```bash
npm install
cp .env.example .env        # fill in your Postgres password and a JWT secret
createdb safar && createdb safar_test
npm run migrate
npm run dev                 # http://localhost:3000
npm test
```
