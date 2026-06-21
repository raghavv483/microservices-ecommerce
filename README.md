# Microservices E-commerce Platform

A full-stack e-commerce monorepo with a customer storefront, admin dashboard, auth service, product service, order service, payment service, email service, and shared Kafka/database packages.

## 1. Project Overview

This repository is built as a microservices architecture with separate service boundaries and shared packages.

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

## 2. Repository Structure

### Apps

- `apps/client` � customer storefront application
- `apps/admin` � admin dashboard application
- `apps/auth-service` � authentication and user management service
- `apps/product-service` � product API service
- `apps/order-service` � order API service
- `apps/payment-service` � payment and webhook service
- `apps/email-service` � email worker service

### Packages

- `packages/kafka` � shared Kafka helpers and topic tooling
- `packages/order-db` � shared MongoDB helper
- `packages/product-db` � shared Prisma client and PostgreSQL schema
- `packages/types` � shared TypeScript types
- `packages/eslint-config` � shared ESLint config
- `packages/typescript-config` � shared TS config

## 3. Key Concepts

### Microservices

Each service runs independently and communicates via HTTP APIs and Kafka events.

### Kafka

Kafka is used to decouple services and support event-driven workflows. This repo includes topic setup and producer/consumer wrappers.

### Databases

- PostgreSQL for product and category data
- MongoDB for order data

### Third-party integrations

- Clerk for authentication
- Stripe for payments
- Cloudinary for image handling
- Google OAuth2 for email delivery

## 4. Architecture Diagram

```mermaid
flowchart LR
    UI_CLIENT[Client UI]\n    UI_ADMIN[Admin UI]\n    CLIENT -->|API requests| PRODUCT_SERVICE[Product Service]
    CLIENT -->|API requests| ORDER_SERVICE[Order Service]
    CLIENT -->|Stripe session| PAYMENT_SERVICE[Payment Service]
    ADMIN -->|API requests| PRODUCT_SERVICE
    ADMIN -->|API requests| AUTH_SERVICE[Auth Service]
    AUTH_SERVICE -->|publishes| KAFKA[Kafka Cluster]
    PRODUCT_SERVICE -->|publishes| KAFKA
    ORDER_SERVICE -->|publishes| KAFKA
    PAYMENT_SERVICE -->|publishes| KAFKA
    EMAIL_SERVICE -->|consumes| KAFKA
    PRODUCT_SERVICE -->|reads/writes| POSTGRESQL[(PostgreSQL)]
    ORDER_SERVICE -->|reads/writes| MONGODB[(MongoDB)]
    PAYMENT_SERVICE -->|verifies webhooks| STRIPE[Stripe]
    PRODUCT_SERVICE -->|uploads/images| CLOUDINARY[Cloudinary]
```

> **Images:** I did not find the attached screenshot files in the repository workspace. To embed them, please add the image files to the repo (for example `docs/images/`) and I can update the README with those exact image paths.

## 5. Requirements

- Node.js >= 18
- `pnpm` installed globally
- Docker Desktop / Docker Engine for Kafka
- PostgreSQL database
- MongoDB database
- Stripe account
- Clerk account
- Cloudinary account
- Google OAuth credentials for Gmail sending

## 5. Setup and Installation

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

## 6. Service Ports

| Service | URL |
|---|---|
| apps/client | http://localhost:3002 |
| apps/admin | http://localhost:3003 |
| apps/product-service | http://localhost:8000 |
| apps/order-service | http://localhost:8001 |
| apps/payment-service | http://localhost:8002 |
| apps/auth-service | http://localhost:8003 |

## 7. Kafka Setup

### Start Kafka

```bash
pnpm kafka:up
```

### Create required topics

```bash
pnpm --filter @repo/kafka create-topics
```

### Kafka topics used

- `user.created`
- `order.created`
- `payment.successful`
- `product.created`
- `product.deleted`

## 8. Database Configuration

### PostgreSQL

The product service uses `packages/product-db` and Prisma. Set `DATABASE_URL` in `packages/product-db/.env`.

### MongoDB

The order service uses `packages/order-db`. Set `MONGO_URL` in `packages/order-db/.env`.

## 9. Visual Overview

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

### Order panel

![Order panel screenshot](docs/images/screenshot-6.png)

## 10. Environment Variable Examples

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

## 10. Service Summaries

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

## 11. Shared Packages

### packages/kafka

Shared Kafka client, producer, consumer, and topic creation scripts.

### packages/product-db

Prisma client and PostgreSQL schema for product data.

### packages/order-db

MongoDB connection helper used by the order service.

## 12. Helpful Commands

```bash
pnpm install
pnpm dev
pnpm lint
pnpm format
pnpm check-types
pnpm kafka:up
pnpm --filter @repo/kafka create-topics
```

## 13. Notes

- `.env` files are excluded by `.gitignore`
- Use `.env.example` templates to document required variables without exposing secrets
- Each service may still require additional config for Clerk, Stripe, Cloudinary, and Google OAuth
