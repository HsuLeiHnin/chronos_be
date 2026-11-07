# Chronos Backend

Backend service for **Chronos**, a mobile app that combines a calendar with 
booking and an AI assistant. 

## What it does

- **Authentication:** registration with email or Google, login with hashed passwords and session tokens, reset password.
- **Schedule:** stores each user's calendar events.
- **Booking:** mock restaurant inventory with time-slot availability.
  Bookings return a confirmation ID, or a "sold out" result when no slot is left.
- **AI proxy:** Supports these actions: `get_schedule`,
  `create_event`, `cancel_event`, `search_and_book_restaurant` and `cancel_booking`.

## Tech stack

| Part | Choice |
|---|---|
| Runtime | Node.js |
| Language | TypeScript |
| Framework | Express |
| Database | PostgreSQL |
| Auth | bcrypt (password hashing) + JWT |
| AI | LLM API with tool calling |

## Data model

| Table | Purpose |
|---|---|
| `users` | Email, hashed password (empty for Google-only accounts), Google ID |
| `password_reset_tokens` | Hashed single-use reset tokens with expiry |
| `events` | User calendar events (linked to `user_id`) |
| `restaurants` | Seeded mock restaurant data |
| `time_slots` | Availability count per restaurant and time |
| `bookings` | Confirmed bookings (linked to `user_id`, slot and event) |

## API endpoints

### Authentication

| Method | Endpoint | Description |
|---|---|---|
| POST | `/auth/register` | Create an account with email and password |
| POST | `/auth/login` | Log in with email and password, returns a token |
| POST | `/auth/google` | Log in or register with a Google ID token from the app |
| POST | `/auth/logout` | End the current session |
| POST | `/auth/forgot-password` | Send a reset link or code to the email |
| POST | `/auth/reset-password` | Set a new password using the reset token |

### User

| Method | Endpoint | Description |
|---|---|---|
| GET | `/users/me` | Get the current user's profile |
| DELETE | `/users/me` | Delete the account and all its data |

### Events

| Method | Endpoint | Description |
|---|---|---|
| GET | `/events?from=&to=` | List events in a date range |
| POST | `/events` | Create an event |
| DELETE | `/events/:id` | Cancel (delete) an event |

### Restaurants

| Method | Endpoint | Description |
|---|---|---|
| GET | `/restaurants?date=&time=&partySize=&price=` | Search restaurants with available slots |
| GET | `/restaurants/:id/slots?date=` | List time slots for one restaurant |

### Bookings

| Method | Endpoint | Description |
|---|---|---|
| GET | `/bookings` | List the user's bookings |
| POST | `/bookings` | Book a slot; adds a calendar event on success, or returns "sold out" |
| DELETE | `/bookings/:id` | Cancel a booking; releases the slot and removes its event |

### AI assistant

| Method | Endpoint | Description |
|---|---|---|
| POST | `/ai/chat` | Send a message; returns a reply and, if needed, a pending action |
| POST | `/ai/confirm` | Confirm or reject a pending action proposed by the AI |

Actions that change data (`create_event`, `cancel_event`,
`search_and_book_restaurant`, `cancel_booking`) are never executed until the
user confirms them. `get_schedule` is read-only and runs immediately.

| AI action | Uses |
|---|---|
| `get_schedule` | `GET /events` |
| `create_event` | `POST /events` |
| `cancel_event` | `DELETE /events/:id` |
| `search_and_book_restaurant` | `GET /restaurants`, `POST /bookings` |
| `cancel_booking` | `DELETE /bookings/:id` |

## Security

- Passwords are hashed with bcrypt and never stored as plain text.
- Google login verifies the ID token on the server before issuing a session.
- Reset tokens are random, stored hashed, single-use and expire after a short
  time. `/auth/forgot-password` returns the same response whether or not the
  email exists.
- Every query on `events` and `bookings` filters by the authenticated user's
  ID, so users can only read and change their own data.
- Booking and cancelling run inside database transactions, so two users cannot
  take the last slot and cancelled slots are released correctly.
- Inputs are validated, login and reset requests are rate-limited, and HTTPS
  is required in production.
- Secrets are environment variables and are never committed.
- Passwords, tokens and message content are not logged.
- Only the minimum data needed is sent to the AI service.
  
## Limitations

- Restaurant data is mock; there are no real suppliers or payments.
- Free hosting tiers may sleep or limit the database when idle.
- Not professionally security-audited.
  
## Getting started

1. Clone the repository and run `npm install`.
2. Copy `.env.example` to `.env` and fill in your values:
   - `DATABASE_URL`
   - `JWT_SECRET`
   - `LLM_API_KEY`
   - `GOOGLE_CLIENT_ID`
   - `SMTP_HOST`, `SMTP_USER`, `SMTP_PASS` (or your email service key)
3. Create the tables and seed data:
   `npm run db:setup`
4. Start the server:
   `npm run dev`

## Related

- Mobile app repository: `[link]`

## Authors

`Nang Lao Morn`, `Hsu Lei Hnin`