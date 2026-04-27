# Marketplace Platform

Production-style full-stack e-commerce platform — Angular, GraphQL, Node.js, MongoDB, Redis, AWS S3

> **This project is currently under active development and the source code is private. If you'd like to view the code, please reach out to me.**

---

## Overview

Marketplace Platform is a production-style, full-stack e-commerce system designed for scalability and extensibility. Built with a modern Angular frontend and a GraphQL API layer over Node.js, the platform supports the full commerce lifecycle — from product browsing and cart management to checkout, order tracking, and admin oversight.

The architecture emphasizes clean separation of concerns, environment-based infrastructure switching, and a layered backend (services, middleware, resolvers) that makes the system easy to extend without touching core logic.

---

## Tech stack

| Layer | Local Development | Production |
|---|---|---|
| Frontend | Angular 17 (standalone, Signals, OnPush) | Same |
| API | GraphQL via Apollo Server 4 | Same |
| Backend | Node.js + Express | Same |
| Database | MongoDB + Mongoose | MongoDB Atlas |
| File Storage | MongoDB GridFS | AWS S3 (ca-central-1) |
| Caching | In-memory Map | Redis (ioredis) |
| Auth | JWT Bearer + OAuth2 Google | Same |
| Email | Nodemailer (console fallback) | SMTP |
| Payments | Stripe + PayPal + Interac | Same |
| Testing (BE) | Jest + Supertest + MongoMemoryServer | CI: Jest |
| Testing (FE) | Karma + Jasmine | CI: Karma headless |
| E2E | Playwright | CI: Playwright headless |
| Linting | ESLint + Prettier | Husky pre-commit |

---

## Architecture highlights

### Layered backend architecture

The backend is structured into distinct layers — resolvers, services, middleware, and models — keeping business logic decoupled from transport and persistence concerns. This makes each layer independently testable and easy to extend.

```
backend/src/
├── cache/ # CacheInterface, MemoryCache, RedisCache
├── storage/ # StorageInterface, GridFSDriver, S3Driver
├── config/ # Database connection and environment config
├── graphql/ # Typedefs + resolvers (8 resolver modules)
├── middleware/ # JWT auth, rate limiter, activity logger
├── models/ # 10 Mongoose models
├── routes/ # File serving route
├── services/ # email service
└── scripts/ # seed scripts
```

### Cache abstraction layer

A cache interface (`CacheInterface`) abstracts over both in-memory and Redis backends. Switching between them requires no code changes — only an environment variable:

```
CACHE_DRIVER=memory # local development
CACHE_DRIVER=redis # production
```

The cache-aside pattern is used throughout: data is read from cache first, with a cache miss triggering a database fetch and subsequent cache population. Rate limiting also runs through the cache abstraction, so it works identically in both environments.

### Storage abstraction layer

File storage follows the same pattern. A `StorageInterface` is implemented by both a GridFS driver (for local dev) and an S3 driver (for production). Switching is controlled by a single env var:

```
STORAGE_DRIVER=gridfs # local dev
STORAGE_DRIVER=s3 # production (ca-central-1)
```

This means the development environment faithfully mirrors production behaviour without requiring cloud credentials locally.

### GraphQL API

The API layer is built with Apollo Server 4 and organized into 8 resolver modules covering users, products, orders, reviews, carts, wishlists, sellers, and admin operations. All mutations from admin and seller roles are audit-logged.

### Angular frontend

The frontend uses Angular 17's standalone component model with Signals for local component state and BehaviorSubject + Observable for shared service-level state. All routes are lazy-loaded with functional guards. CSS is strictly flat (no `&` nesting) for maintainability.

---

## Features by role

### Guest
- browse & search products
- view product detail pages and reviews
- add items to cart

### Customer
- full purchase flow with Stripe, PayPal, or Interac
- order history and real-time order tracking
- write and manage reviews
- wishlist management

### Seller
- product listing and inventory management
- sales analytics dashboard
- audit trail of all mutations

### Admin
- full content management via GUI (users, products, orders)
- activity logging across all seller and admin actions
- user role management

---

## CI/CD Pipeline

GitHub Actions runs on every push to `main` and `develop`:

1. **Backend** — lint, format check, Jest tests with coverage report
2. **Frontend** — lint, format check, Karma tests with coverage report
3. **E2E** — Playwright tests against a seeded MongoDB instance with both servers running
4. **Security** — `npm audit` on both packages

Coverage targets: Backend >= 50%, Frontend >= 50% (targets will be raised to 80% / 75% as the codebase matures).

---

## Testing

| Layer | Tool | What's Covered |
|---|---|---|
| Backend unit + integration | Jest + Supertest + MongoMemoryServer | Resolvers, services, middleware |
| Frontend unit | Karma + Jasmine | Components, services, guards |
| E2E | Playwright + Page Object Models | Full user flows across all roles |

---

## Project structure

```
marketplace-platform/
├── .github/workflows/ci.yml # GitHub Actions CI pipeline
├── .husky/pre-commit # Pre-commit lint hook
├── .prettierrc # Shared Prettier config
├── backend/
│   ├── src/
│   │   ├── cache/
│   │   ├── storage/
│   │   ├── config/
│   │   ├── graphql/
│   │   ├── middleware/
│   │   ├── models/
│   │   ├── routes/
│   │   ├── services/
│   │   └── scripts/
│   ├── jest.config.js
│   └── .env.example
└── frontend/
    ├── src/app/
    │   ├── core/ # Services, guards, interceptors
    │   ├── layout/ # Navbar, Footer
    │   ├── shared/ # reusable components, models
    │   └── features/ # Page components (home, shop, admin, seller)
    ├── e2e/ # playwright E2E tests + Page Object Models
    ├── angular.json
    ├── karma.conf.js
    └── playwright.config.ts
```

---

## Environment configuration

All environment switching is handled via a single `.env` file. See `backend/.env.example` for the full reference. Key variables:

| Variable | Local | Production |
|---|---|---|
| `STORAGE_DRIVER` | `gridfs` | `s3` |
| `CACHE_DRIVER` | `memory` | `redis` |
| `MONGO_URI` | Local MongoDB | MongoDB Atlas URI |
| `JWT_SECRET` | Any string | Secure secret |
| `STRIPE_SECRET_KEY` | Stripe test key | Stripe live key |

---

## Status

This project is under active development. Some features are still in progress and the architecture may evolve.

---

## Viewing the code

The repository is currently private. If you are a recruiter, collaborator, or are otherwise interested in reviewing the source code, please get in touch: **https://www.linkedin.com/in/anjalicpatidar/**
