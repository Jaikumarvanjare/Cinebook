# CineBook Database Schema

The backend uses **MongoDB** with **Prisma ORM**. The notification service maintains a separate MongoDB database via Mongoose.

---

## Entity Relationship Overview

```
User ──────────────► Theatre (owner)
  │                      │
  │                      ▼
  │                    Show ◄──── Movie
  │                      │
  ▼                      │
Booking ◄────────────────┘
  │
  ▼
Payment
```

---

## Backend Collections (Prisma)

### User

| Field | Type | Notes |
|-------|------|-------|
| id | ObjectId | Primary key |
| name | String | Display name |
| email | String | Unique login |
| password | String | bcrypt hashed |
| about | String? | Profile bio |
| profilePhoto | String? | Image URL |
| resetOtp | String? | Password reset OTP |
| resetOtpExpiry | DateTime? | OTP expiration |
| role | Enum | `CUSTOMER`, `CLIENT`, `ADMIN` |
| status | Enum | `APPROVED`, `PENDING`, `REJECTED` |

### Movie

| Field | Type | Notes |
|-------|------|-------|
| id | ObjectId | Primary key |
| name | String | Movie title |
| description | String | Synopsis |
| casts | String[] | Cast list |
| trailer | String? | Trailer URL |
| language | String | e.g. English, Hindi |
| releaseDate | DateTime | Release date |
| director | String | Director name |
| releaseStatus | String | e.g. NOW_SHOWING |
| poster | String? | Poster image URL |

### Theatre

| Field | Type | Notes |
|-------|------|-------|
| id | ObjectId | Primary key |
| name | String | Theatre name |
| city | String | City location |
| pincode | String | Postal code |
| address | String | Full address |
| owner | ObjectId | Reference to User (CLIENT) |
| movieIds | ObjectId[] | Movies playing at this theatre |

### Show

| Field | Type | Notes |
|-------|------|-------|
| id | ObjectId | Primary key |
| theatreId | ObjectId | Reference to Theatre |
| movieId | ObjectId | Reference to Movie |
| timing | DateTime | Show start time |
| seats | Int | Available seat count |
| seatConfig | Json | Seat layout and availability map |
| price | Float | Ticket price |
| format | String | e.g. 2D, IMAX |

### Booking

| Field | Type | Notes |
|-------|------|-------|
| id | ObjectId | Primary key |
| userId | ObjectId | Reference to User |
| theatreId | ObjectId | Reference to Theatre |
| movieId | ObjectId | Reference to Movie |
| timing | DateTime | Show timing |
| seatCount | Int | Number of seats booked |
| totalCost | Float | Total booking amount |
| status | String | `PROCESSING`, `SUCCESSFULL`, `EXPIRED` |
| selectedSeats | Json? | Seat numbers/labels selected |

### Payment

| Field | Type | Notes |
|-------|------|-------|
| id | ObjectId | Primary key |
| bookingId | ObjectId | Reference to Booking |
| amount | Float | Payment amount |
| status | String | `PENDING`, `SUCCESS`, `FAILED` |
| gateway | String | e.g. `RAZORPAY`, `DEMO` |
| orderId | String? | Razorpay order ID |
| paymentId | String? | Razorpay payment ID |
| currency | String | e.g. `INR` |

---

## Notification Service (Mongoose)

The notification service stores email job history in its own MongoDB database, separate from the backend.

| Collection | Purpose |
|------------|---------|
| Notifications | Email delivery records (recipient, subject, status, timestamps) |

---

## Booking Status Flow

```
PROCESSING  ──►  SUCCESSFULL   (payment completed)
     │
     └──►  EXPIRED            (cron job — unpaid booking timeout)
```

---

## Related Documentation

| Document | Description |
|----------|-------------|
| [Architecture](architecture.md) | System components and design |
| [API Flow](api-flow.md) | Request sequence for bookings |
