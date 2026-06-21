# Microservices E-commerce Platform

A full-stack e-commerce monorepo built with **Turborepo** and **pnpm workspaces**, featuring a customer storefront, admin dashboard, and six backend microservices communicating over Kafka.

## 1. Project Overview

This repository is organized as a Turborepo monorepo with clearly defined service boundaries and shared packages.

### What this project includes

- Customer storefront and admin dashboard using Next.js
- Clerk authentication with protected APIs and UIs
- Product service using PostgreSQL and Prisma
- Order service using MongoDB
- Payment service with Stripe session creation and webhook handling
- Email service for transactional email notifications
- Kafka event-driven communication across services
- Cloudinary support for product images
- TanStack React Query and React Table for admin UX

## 2. Monorepo Structure

This project uses **Turborepo** with **pnpm workspaces** for high-performance builds, shared caching, and coordinated task pipelines across all apps and packages.

### Apps

- `apps/client` — customer storefront application
- `apps/admin` — admin dashboard application
- `apps/auth-service` — authentication and user management service
- `apps/product-service` — product API service
- `apps/order-service` — order API service
- `apps/payment-service` — payment and webhook service
- `apps/email-service` — email worker service

### Packages

- `packages/kafka` — shared Kafka helpers and topic tooling
- `packages/order-db` — shared MongoDB helper
- `packages/product-db` — shared Prisma client and PostgreSQL schema
- `packages/types` — shared TypeScript types
- `packages/eslint-config` — shared ESLint config
- `packages/typescript-config` — shared TS config

## 3. Key Concepts

### Microservices

Each service runs independently and communicates via HTTP APIs and Kafka events. Service boundaries are enforced through the monorepo workspace structure.

### Turborepo

[Turborepo](https://turbo.build/repo) orchestrates task execution across the monorepo — `pnpm dev`, `pnpm build`, and `pnpm lint` run only what has changed, with remote caching support via Vercel.

### Kafka

Kafka decouples services and supports event-driven workflows. The shared `packages/kafka` package provides topic setup and producer/consumer wrappers used across all services.

### Databases

- PostgreSQL for product and category data (via Prisma in `packages/product-db`)
- MongoDB for order data (via `packages/order-db`)

### Third-party integrations

- Clerk for authentication
- Stripe for payments
- Cloudinary for image handling
- Google OAuth2 for email delivery

## 4. Architecture Diagram

```mermaid
flowchart LR
    CLIENT[apps/client\nCustomer Storefront]
    ADMIN[apps/admin\nAdmin Dashboard]
    AUTH[apps/auth-service]
    PRODUCT[apps/product-service]
    ORDER[apps/order-service]
    PAYMENT[apps/payment-service]
    EMAIL[apps/email-service]
    KAFKA[(Kafka Cluster)]
    PG[(PostgreSQL)]
    MONGO[(MongoDB)]
    STRIPE[Stripe]
    CLOUDINARY[Cloudinary]
    CLERK[Clerk Auth]

    CLIENT -->|product & order APIs| PRODUCT
    CLIENT -->|order APIs| ORDER
    CLIENT -->|Stripe session| PAYMENT
    ADMIN -->|product & category APIs| PRODUCT
    ADMIN -->|user & auth APIs| AUTH

    AUTH -->|user.created| KAFKA
    PRODUCT -->|product.created / product.deleted| KAFKA
    ORDER -->|order.created| KAFKA
    PAYMENT -->|payment.successful| KAFKA
    KAFKA -->|consumes events| EMAIL

    PRODUCT -->|reads / writes| PG
    ORDER -->|reads / writes| MONGO
    PAYMENT -->|verifies webhooks| STRIPE
    PRODUCT -->|image uploads| CLOUDINARY
    AUTH -->|token validation| CLERK
    CLIENT -->|token validation| CLERK
```

## 5. Requirements

- Node.js >= 18
- `pnpm` installed globally
- `turbo` (installed as a dev dependency via pnpm)
- Docker Desktop / Docker Engine for Kafka
- PostgreSQL database
- MongoDB database
- Stripe account
- Clerk account
- Cloudinary account
- Google OAuth credentials for Gmail sending

## 6. Setup and Installation

### Clone repository

```bash
git clone <your-repo-url>
cd microservices-ecommerce
pnpm install
```

### Environment files

Copy each service `.env.example` to `.env` in the same folder and fill in values.

### Start development

```bash
pnpm dev
```

Turborepo will start all apps and services in parallel, respecting the dependency graph.

## 7. Service Ports

| Service | URL |
|---|---|
| apps/client | http://localhost:3002 |
| apps/admin | http://localhost:3003 |
| apps/product-service | http://localhost:8000 |
| apps/order-service | http://localhost:8001 |
| apps/payment-service | http://localhost:8002 |
| apps/auth-service | http://localhost:8003 |

## 8. Kafka Setup

### Start Kafka

```bash
pnpm kafka:up
```

### Create required topics

```bash
pnpm --filter @repo/kafka create-topics
```

### Kafka topics used

| Topic | Published by | Consumed by |
|---|---|---|
| `user.created` | auth-service | email-service |
| `order.created` | order-service | email-service |
| `payment.successful` | payment-service | email-service |
| `product.created` | product-service | email-service |
| `product.deleted` | product-service | email-service |

## 9. Database Configuration

### PostgreSQL

The product service uses `packages/product-db` and Prisma. Set `DATABASE_URL` in `packages/product-db/.env`.

```bash
pnpm --filter @repo/product-db prisma migrate dev
pnpm --filter @repo/product-db prisma generate
```

### MongoDB

The order service uses `packages/order-db`. Set `MONGO_URL` in `packages/order-db/.env`.

## 10. Visual Overview

### Admin dashboard

![Admin dashboard screenshot](docs/images/screenshot-1.png)

### Client storefront

![Client storefront screenshot](docs/images/screenshot-2.png)

### Cart and checkout

![Cart and checkout screenshot](docs/images/screenshot-3.png)

### Payment method

![Payment method screenshot](docs/images/screenshot-4.png)

### Metrics dashboard

![Metrics dashboard screenshot](docs/images/screenshot-5.png)

### Admin Panel

![Order panel screenshot](docs/images/screenshot-6.png)

## 11. Environment Variable Examples

### `apps/admin/.env.example`

```env
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=your-clerk-publishable-key
CLERK_SECRET_KEY=your-clerk-secret-key
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_FALLBACK_REDIRECT_URL=/
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_SIGN_IN_FALLBACK_REDIRECT_URL=/
NEXT_PUBLIC_PRODUCT_SERVICE_URL=http://localhost:8000
NEXT_PUBLIC_ORDER_SERVICE_URL=http://localhost:8001
NEXT_PUBLIC_PAYMENT_SERVICE_URL=http://localhost:8002
NEXT_PUBLIC_AUTH_SERVICE_URL=http://localhost:8003
NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME=your-cloudinary-cloud-name
```

### `apps/client/.env.example`

```env
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=your-clerk-publishable-key
CLERK_SECRET_KEY=your-clerk-secret-key
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_FALLBACK_REDIRECT_URL=/
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_SIGN_IN_FALLBACK_REDIRECT_URL=/
NEXT_PUBLIC_PRODUCT_SERVICE_URL=http://localhost:8000
NEXT_PUBLIC_ORDER_SERVICE_URL=http://localhost:8001
NEXT_PUBLIC_PAYMENT_SERVICE_URL=http://localhost:8002
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=your-stripe-publishable-key
```

### `apps/auth-service/.env.example`

```env
CLERK_PUBLISHABLE_KEY=your-clerk-publishable-key
CLERK_SECRET_KEY=your-clerk-secret-key
```

### `apps/email-service/.env.example`

```env
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
GOOGLE_REFRESH_TOKEN=your-google-refresh-token
```

### `apps/order-service/.env.example`

```env
CLERK_PUBLISHABLE_KEY=your-clerk-publishable-key
CLERK_SECRET_KEY=your-clerk-secret-key
MONGO_URL=your-mongodb-connection-string
```

### `apps/payment-service/.env.example`

```env
STRIPE_SECRET_KEY=your-stripe-secret-key
STRIPE_WEBHOOK_SECRET=your-stripe-webhook-secret
CLERK_PUBLISHABLE_KEY=your-clerk-publishable-key
CLERK_SECRET_KEY=your-clerk-secret-key
```

### `apps/product-service/.env.example`

```env
CLERK_PUBLISHABLE_KEY=your-clerk-publishable-key
CLERK_SECRET_KEY=your-clerk-secret-key
```

### `packages/order-db/.env.example`

```env
MONGO_URL=your-mongodb-connection-string
```

### `packages/product-db/.env.example`

```env
DATABASE_URL="postgresql://username:password@localhost:5432/products?schema=public"
```

## 12. Service Summaries

### apps/client

Customer storefront with product browsing, search, cart, and Stripe checkout.

### apps/admin

Admin dashboard for product, category, user, order management, and analytics.

### apps/auth-service

Clerk-authenticated user service with protected admin routes and Kafka event publishing.

### apps/product-service

Product API service with Express, PostgreSQL, Prisma, and Kafka event publishing.

### apps/order-service

Order service with Fastify, MongoDB, and Kafka subscriptions for payments.

### apps/payment-service

Stripe session and webhook handling service with published payment events.

### apps/email-service

Email worker service that consumes Kafka topics and sends transactional emails.

## 13. Shared Packages

### packages/kafka

Shared Kafka client, producer, consumer, and topic creation scripts.

### packages/product-db

Prisma client and PostgreSQL schema for product data.

### packages/order-db

MongoDB connection helper used by the order service.

## 14. Helpful Commands

```bash
# Install all workspace dependencies
pnpm install

# Start all apps and services in parallel (via Turborepo)
pnpm dev

# Lint all packages
pnpm lint

# Format all packages
pnpm format

# Type-check all packages
pnpm check-types

# Start Kafka via Docker
pnpm kafka:up

# Create all required Kafka topics
pnpm --filter @repo/kafka create-topics
```

## 15. Notes

- `.env` files are excluded by `.gitignore`
- Use `.env.example` templates to document required variables without exposing secrets
- Turborepo caches build artifacts in `.turbo/` — run `pnpm turbo run build --force` to bypass cache
- Each service may still require additional config for Clerk, Stripe, Cloudinary, and Google OAuth
