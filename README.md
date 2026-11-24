# Sweets by Efrat — Boutique Bakery Sales & Management Platform

A full-stack e-commerce and back-office system for a boutique bakery, **live in production** and used by a real business.

🔗 **Live site:** [sweetsbyefrat.com](https://sweetsbyefrat.com)

Customers browse the catalog, manage a cart and place orders; the owner manages products, categories, orders and customers from a dedicated admin dashboard.

---

## Tech Stack

| Layer | Technologies |
|---|---|
| **Backend** | Java 21, Spring Boot 3, Spring Security, Spring Data JPA / Hibernate, Bean Validation, Maven, Lombok |
| **Frontend** | React, TypeScript, TanStack Router & React Query, Tailwind CSS, React Hook Form + Zod, Axios |
| **Database** | PostgreSQL (production), H2 (local development) |
| **Auth** | JWT (access + refresh tokens), Google Sign-In, BCrypt |
| **Integrations** | Cloudinary (image storage), Brevo (transactional email), Bucket4j (rate limiting) |
| **Deployment** | Backend on Railway, frontend on Cloudflare Workers |

---

## Features

### For customers
- Product catalog with categories and search
- Shopping cart and order placement
- Order history and profile management
- Product recommendations / reviews
- Contact form
- Registration, login and **Sign in with Google**
- Password reset via a one-time email link

### For the business owner (admin dashboard)
- Overview dashboard
- Full management of products and categories, including **image upload**
- Order management: view all orders, filter by date or customer, update order status
- Customer management
- Email notification on every new order

---

## Architecture

```
┌──────────────────────┐      REST / JSON       ┌───────────────────────────────┐
│  React + TypeScript  │  ───────────────────▶  │  Spring Boot REST API         │
│  (Cloudflare)        │  ◀───────────────────  │  (Railway)                    │
└──────────────────────┘   JWT + HttpOnly       │                               │
                            refresh cookie      │  Controllers → Services →     │
                                                │  Repositories (JPA/Hibernate) │
                                                └──────┬───────────┬────────┬───┘
                                                       │           │        │
                                                PostgreSQL   Cloudinary   Brevo
```

The backend follows a layered architecture:
- **Controllers** — REST endpoints and request validation
- **Services** — business logic, split into customer (`ClientService`) and admin (`AdminService`) concerns
- **Repositories** — data access with Spring Data JPA
- **Global exception handler** — consistent error responses

---

## Security

Security was a central focus of this project:

- **JWT with access + refresh tokens** — short-lived access token (15 min); refresh token (7 days) stored in an `HttpOnly`, `Secure`, `SameSite=Strict` cookie to mitigate XSS and CSRF.
- **Role-based authorization** — `USER` / `ADMIN` roles with a role hierarchy, enforced in `SecurityConfig`.
- **Ownership checks** — users can only update their own profile.
- **Secure logout** — revoked access tokens are blacklisted.
- **Google Sign-In** — Google ID tokens are verified server-side before issuing the app's own tokens.
- **Password security** — BCrypt hashing; password reset via a one-time token that expires after 15 minutes, without revealing whether an email is registered.
- **Rate limiting** — Bucket4j limits order and contact-form submissions to prevent abuse.
- **Upload validation** — only JPG / PNG / WEBP images up to 8 MB are accepted.
- **Secrets management** — all credentials are provided via environment variables; nothing sensitive is committed to the repository.

---

## Implementation Highlights

- **Image pipeline** — uploaded images are validated, resized (max 800×600) and auto-compressed in Cloudinary; only the secure URL is stored in the database.
- **Asynchronous email** — emails are sent via the Brevo API with `@Async`, so user requests are never blocked by email delivery.
- **Environment profiles** — separate `dev` (H2) and `prod` (PostgreSQL) Spring profiles; in production the schema is validated rather than auto-modified and the H2 console is disabled.
- **Performance** — Hibernate batch fetching to avoid the N+1 query problem; tuned connection pool for the hosting environment.

---

## Running Locally

### Prerequisites
- Java 21
- Node.js 20+

### Backend
```bash
cd server
./mvnw spring-boot:run
```
Runs with the `dev` profile and a local H2 database, seeded with sample categories and products.

### Frontend
```bash
cd client
npm install
npm run dev
```
Set `VITE_API_BASE_URL` in `client/.env` to point to the backend (e.g. `http://localhost:8080`).

### Production environment variables (backend)

| Variable | Purpose |
|---|---|
| `DB_URL`, `DB_USERNAME`, `DB_PASSWORD` | PostgreSQL connection |
| `JWT_SECRET` | JWT signing key |
| `ALLOWED_ORIGINS` | CORS allowed origins |
| `BREVO_API_KEY`, `MAIL_FROM` | Transactional email |
| `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET` | Image storage |

---

## Roadmap

- Move the token blacklist to Redis so it survives restarts and scales across instances
- Refresh-token rotation
- Automated test coverage

---

## Author

**Ruti Shrem** — Full Stack Developer
📧 Rutishrem0224@gmail.com · [GitHub](https://github.com/ruti0224)
