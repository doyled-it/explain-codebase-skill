# ASCII Diagram Toolkit

Copy and adapt these patterns. Keep everything monospace; count characters per row so borders align.

## Boxes and flow
```
┌─────────────┐      ┌─────────────┐
│  Component  │ ───▶ │  Component  │
└─────────────┘      └─────────────┘
```

## Hierarchy / tree
```
App
├── Auth
│   ├── login()
│   └── logout()
├── Users
│   ├── getUser()
│   └── updateUser()
└── Utils
```

## Data flow
```
Request ──▶ Validate ──▶ Process ──▶ Respond
                │
                ▼
             Log Error
```

## Sequence
```
Client          Server          Database
  │                │                │
  │── GET /users ─▶│                │
  │                │── SELECT * ───▶│
  │                │◀── rows ───────│
  │◀── JSON ───────│                │
  │                │                │
```

## Layers
```
┌─────────────────────────────────────┐
│            Controllers              │
├─────────────────────────────────────┤
│             Services                │
├─────────────────────────────────────┤
│              Models                 │
├─────────────────────────────────────┤
│             Database                │
└─────────────────────────────────────┘
```

## State machine
```
          ┌─────────────────┐
          ▼                 │
┌──────┐     ┌─────────┐    │    ┌──────────┐
│ Idle │ ──▶ │ Loading │ ───┴──▶ │ Complete │
└──────┘     └─────────┘         └──────────┘
    ▲                                  │
    └──────────── reset ───────────────┘
```

## Comparison table
```
┌────────────┬───────────┬───────────┐
│  Approach  │   Pros    │   Cons    │
├────────────┼───────────┼───────────┤
│ Option A   │ Fast      │ Complex   │
│ Option B   │ Simple    │ Slow      │
└────────────┴───────────┴───────────┘
```

---

# Worked example interactions

These show the target output style: one-line answer, a diagram, cited files, follow-ups.

## "How does authentication work?"

```
Authentication uses JWT tokens with a refresh token flow.

Here's the big picture:

  Login                           Authenticated Request
  =====                           =====================

  Client                          Client
    │                               │
    │── POST /login ──────▶         │── GET /api/data ──────▶
    │   {email, password}           │   Header: Bearer <token>
    │                               │
    │◀── {accessToken, ────         │◀── {data} ─────────────
    │     refreshToken}             │
    │                               │
    │   (store tokens)              │   (middleware validates)


The key files:

  src/
  ├── middleware/
  │   └── auth.middleware.ts    ← Validates tokens on each request
  ├── services/
  │   └── auth.service.ts       ← Login logic, token generation
  └── routes/
      └── auth.routes.ts        ← POST /login, POST /refresh

Token lifecycle:
  • Access token expires in 15 minutes
  • Refresh token expires in 7 days
  • Refresh tokens are rotated on use (one-time use)

Want me to dig into any of these? For example:
  • How does the middleware validate tokens?
  • What happens when the access token expires?
  • How are passwords hashed?
```

## "What happens when the access token expires?"

```
When the access token expires, the client silently refreshes it.

Timeline:
─────────────────────────────────────────────────────────────▶ time

  Token issued      Token expires     Refresh happens
       │                  │                  │
       ▼                  ▼                  ▼
  [============ valid ============]
                          │
                          │  Request fails with 401
                          │
                          └──▶ Client catches 401
                               │
                               ├──▶ POST /auth/refresh
                               │    {refreshToken}
                               │
                               ◀── {newAccessToken, newRefreshToken}
                               │
                               └──▶ Retry original request

This happens in `src/lib/api-client.ts:34-52`:

  • Axios interceptor catches 401 responses
  • Queues failed requests while refreshing
  • Retries all queued requests with new token
  • If refresh fails → redirect to /login

Gotcha: The refresh token is single-use. Once used, it's
invalidated and a new one is issued. This prevents replay attacks.
```

## High-level overview (no specific question)

```
Project: my-app (Node.js/Express API)

Architecture:
┌──────────────────────────────────────────────────────────┐
│                        Client                            │
└────────────────────────────┬─────────────────────────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────────┐
│                     Express Server                       │
│  ┌─────────┐  ┌────────────┐  ┌───────────┐             │
│  │ Routes  │─▶│ Middleware │─▶│ Controllers│             │
│  └─────────┘  └────────────┘  └─────┬─────┘             │
│                                     │                    │
│                               ┌─────▼─────┐              │
│                               │ Services  │              │
│                               └─────┬─────┘              │
│                                     │                    │
│                               ┌─────▼─────┐              │
│                               │  Prisma   │              │
│                               └─────┬─────┘              │
└─────────────────────────────────────┼────────────────────┘
                                      │
                                      ▼
                             ┌────────────────┐
                             │   PostgreSQL   │
                             └────────────────┘

Directory Guide:
src/
├── routes/        → URL → Controller mapping
├── middleware/    → Auth, validation, logging
├── controllers/   → Request handling, response formatting
├── services/      → Business logic (the interesting stuff)
├── models/        → Prisma schema and generated types
└── utils/         → Shared helpers

Start here:
1. src/app.ts — Server bootstrap
2. src/routes/index.ts — All available endpoints
3. src/services/auth.service.ts — Auth is always important
4. prisma/schema.prisma — Data model
5. src/middleware/error.middleware.ts — How errors are handled

Questions you might ask:
• "How does authentication work?"
• "How do I add a new endpoint?"
• "What's the database schema?"
• "How is error handling done?"
```
