# CineBook System Architecture

CineBook is a multi-repository movie ticket booking platform. The presentation layer, core API, persistence, and notification worker are split into separate deployable services.

---

## High-Level Overview

```
              React Web + Flutter Mobile
                         │
                         │ REST API (JWT)
                         ▼
              Core Backend API (Express + Prisma)
                         │
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
      MongoDB      Notification API    Razorpay
                         │
                         ▼
                   Redis + BullMQ
                         │
                         ▼
                  Email Worker (SMTP)
```

---

## Repositories


| Service              | Repository                                                                        | Responsibility                                    |
| -------------------- | --------------------------------------------------------------------------------- | ------------------------------------------------- |
| Web Frontend         | [cinebook-frontend](https://github.com/JaikumarVanjare/cinebook-frontend)         | Customer booking UI and admin dashboard           |
| Mobile App           | [cinebook-mobile](https://github.com/JaikumarVanjare/cinebook-mobile)             | Cross-platform booking and admin shell            |
| Backend API          | [cinebook-backend](https://github.com/JaikumarVanjare/cinebook-backend)           | Auth, movies, theatres, shows, bookings, payments |
| Notification Service | [cinebook-notification](https://github.com/JaikumarVanjare/cinebook-notification) | Async email delivery via BullMQ                   |


---

## Components

### Presentation Layer

**React Web App**

- Browse movies, select shows, pick seats, and complete payments
- Admin tools for movies, theatres, shows, users, and bookings
- Communicates with the backend over REST using JWT tokens

**Flutter Mobile App**

- Same core flows as the web app on iOS and Android
- Uses the same backend API (`/mba/api/v1`)

### Core Backend API

**Stack:** Node.js, Express, Prisma, MongoDB

**Base path:** `/mba/api/v1`

**Responsibilities:**

- User registration, login, and role-based authorization
- Movie catalogue and theatre management
- Show scheduling and seat configuration
- Booking creation and payment finalization
- Razorpay order creation and signature verification
- Booking expiration via cron jobs
- HTTP calls to the notification service after successful bookings

### Notification Service

**Stack:** Node.js, Express, TypeScript, Redis, BullMQ, Nodemailer

**Base path:** `/notiservice/api/v1`

**Responsibilities:**

- Accept notification requests from the backend
- Enqueue email jobs in Redis (BullMQ)
- Process jobs in a separate worker process
- Send emails via SMTP (booking confirmations, password reset OTPs, etc.)
- Store notification history in its own MongoDB database

### External Services


| Service      | Purpose                                                                     |
| ------------ | --------------------------------------------------------------------------- |
| **MongoDB**  | Primary data store for backend (Prisma) and notification history (Mongoose) |
| **Redis**    | Job queue for async email processing                                        |
| **Razorpay** | Payment gateway for ticket checkout                                         |
| **SMTP**     | Outbound email delivery                                                     |


---

## Request Flow

### Booking Flow

```
Client                    Backend API              MongoDB        Notification        Redis/Worker
  │                            │                      │                │                  │
  │── POST /bookings ─────────►│                      │                │                  │
  │                            │── create booking ───►│                │                  │
  │◄── booking created ────────│                      │                │                  │
  │                            │                      │                │                  │
  │── POST /payment ──────────►│                      │                │                  │
  │                            │── verify Razorpay ──►│                │                  │
  │                            │── update seats ─────►│                │                  │
  │                            │── POST notify  ──────────────────────►│                  │
  │                            │                      │                │── enqueue job ──►│
  │◄── booking confirmed ──────│                      │                │                  │
  │                            │                      │                │                  │── send email
```

### Authentication

1. Client sends credentials to `/mba/api/v1/auth/login`
2. Backend validates and returns a JWT
3. Client attaches the token in the `Authorization` header on subsequent requests
4. Backend middleware verifies the token and enforces role checks

### Roles


| Role       | Access                                                 |
| ---------- | ------------------------------------------------------ |
| `CUSTOMER` | Browse movies, book tickets, view own bookings         |
| `CLIENT`   | Manage owned theatres, shows, and related bookings     |
| `ADMIN`    | Full access — users, movies, theatres, shows, bookings |


---

## API Endpoints


| Service      | Base URL                                   | Docs                                      |
| ------------ | ------------------------------------------ | ----------------------------------------- |
| Backend      | `http://localhost:3000/mba/api/v1`         | [Swagger](http://localhost:3000/api-docs) |
| Notification | `http://localhost:3001/notiservice/api/v1` | See notification repo README              |


---

## Design Decisions

**Multi-repository layout** — Each service has its own repo, dependencies, and deployment lifecycle. This showcase hub documents the system without containing application code.

**Async notifications** — Email sending is offloaded to a worker so booking responses are not blocked by SMTP latency.

**Prisma + MongoDB** — The backend uses Prisma ORM with MongoDB for flexible document-style schemas (seat configs, embedded arrays).

**Shared admin in web and mobile** — Admin features live inside the frontend and mobile apps rather than a separate admin repository.

---

## Related Documentation


| Document                              | Description                           |
| ------------------------------------- | ------------------------------------- |
| [Database Schema](database-schema.md) | MongoDB collections and relationships |
| [API Flow](api-flow.md)               | Booking and notification sequence     |


