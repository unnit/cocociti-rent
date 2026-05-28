# Cocociti Rent

![Ruby](https://img.shields.io/badge/Ruby-3.3.0-CC342D?style=flat&logo=ruby&logoColor=white)
![Rails](https://img.shields.io/badge/Rails-7.1-CC0000?style=flat&logo=rubyonrails&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-14+-4169E1?style=flat&logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-cache%20%2F%20pub--sub-DC382D?style=flat&logo=redis&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-compose-2496ED?style=flat&logo=docker&logoColor=white)
![Bootstrap](https://img.shields.io/badge/Bootstrap-3-7952B3?style=flat&logo=bootstrap&logoColor=white)
![Cloudinary](https://img.shields.io/badge/Cloudinary-image%20storage-3448C5?style=flat&logo=cloudinary&logoColor=white)
![Firebase](https://img.shields.io/badge/Firebase-notifications-FFCA28?style=flat&logo=firebase&logoColor=black)
![AWS](https://img.shields.io/badge/AWS-Lightsail-FF9900?style=flat&logo=amazonaws&logoColor=white)
![CI/CD](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-2088FF?style=flat&logo=githubactions&logoColor=white)

A peer-to-peer rental marketplace where users can list items for rent, discover listings, and manage bookings end-to-end. Built with Ruby on Rails 7.1, backed by PostgreSQL, and deployed via Docker on AWS Lightsail.

---

## Table of Contents

- [Features](#features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Environment Variables](#environment-variables)
- [Docker Setup](#docker-setup)
- [Database Setup](#database-setup)
- [Running Locally (Without Docker)](#running-locally-without-docker)
- [Deployment](#deployment)
- [Project Structure](#project-structure)
- [License](#license)

---

## Features

- **Product Listings** — Users can list items for rent with photos (via Cloudinary), pricing (daily/hourly), categories, and location.
- **Booking & Transactions** — End-to-end rental flow with status tracking: Requested → Waiting Payment → Paid → Accepted/Denied.
- **Payments** — Citrus payment gateway integration for secure payments and refunds.
- **Search & Discovery** — Category, location, and price-based product search.
- **User Profiles** — Individual and business accounts with availability schedules, seasonal pricing, and banking/withdrawal setup.
- **Real-time Chat** — ActionCable-powered chat between renters and owners.
- **Notifications** — In-app and Firebase push notifications for bookings, messages, and activity.
- **Admin Panel** — ForestLiana dashboard for user management, product moderation, and transaction oversight.
- **Giveaways** — Users can create and participate in item giveaways.
- **Authentication** — Devise with email confirmation, Facebook OAuth, and phone OTP verification (Plivo SMS).

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Browser / Mobile                     │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTP / WebSocket
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              Rails 7.1 App  (Puma, port 3000)               │
│                                                             │
│  ┌──────────────┐  ┌─────────────────┐  ┌───────────────┐   │
│  │  Controllers │  │  ActionCable    │  │  Background   │   │
│  │  (MVC)       │  │  (WebSockets)   │  │  Jobs         │   │
│  └──────┬───────┘  └────────┬────────┘  └───────┬───────┘   │
│         │                   │                   │           │
│  ┌──────▼───────────────────▼───────────────────▼───────┐   │
│  │                      Models / ORM                    │   │
│  └──────┬──────────────────────────────────┬────────────┘   │
│         │                                  │                │
│  ┌──────▼──────┐                  ┌────────▼────────┐       │
│  │ PostgreSQL  │                  │     Redis       │       │
│  │ (primary DB)│                  │ (cache/pub-sub) │       │
│  └─────────────┘                  └─────────────────┘       │
└─────────────────────────────────────────────────────────────┘
         │
         ├── Cloudinary       (image storage & CDN)
         ├── Citrus            (payment gateway)
         ├── Firebase          (push notifications)
         ├── Plivo             (SMS / OTP)
         └── Facebook OAuth    (social login)
```

**Key design choices:**

- **MVC with Rails conventions** — controllers, HAML views, and ActiveRecord models.
- **ActionCable channels** — `ChatChannel`, `UserChannel`, and `GlobalChannel` for real-time features.
- **Background jobs** — ActiveJob (async queue) handles notification broadcasts, chat messages, and scheduled transaction cleanup.
- **Asset pipeline** — Sprockets with Sass and CoffeeScript; Bootstrap 3 for UI components.
- **ForestLiana** — mounted at `/forest` for admin database management without a custom admin UI.

---

## Tech Stack

| Layer              | Technology                                    |
| ------------------ | --------------------------------------------- |
| Language           | Ruby 3.3.0                                    |
| Framework          | Rails 7.1                                     |
| Database           | PostgreSQL                                    |
| Cache / Pub-Sub    | Redis                                         |
| Web Server         | Puma                                          |
| Frontend           | HAML, Bootstrap 3, jQuery, Sass, CoffeeScript |
| Real-time          | ActionCable (WebSockets)                      |
| File Uploads       | CarrierWave + Cloudinary                      |
| Authentication     | Devise + OmniAuth (Facebook)                  |
| Payments           | Citrus gateway                                |
| SMS                | Plivo                                         |
| Push Notifications | Firebase                                      |
| PDF Generation     | WickedPDF + wkhtmltopdf                       |
| Search             | SearchKick / Algolia                          |
| Admin Panel        | ForestLiana                                   |
| Containerization   | Docker + Docker Compose                       |
| CI/CD              | GitHub Actions → AWS Lightsail                |

---

## Prerequisites

**With Docker (recommended):**

- Docker Engine 20+
- Docker Compose v2
- An external Docker network named `shared-net`
- An external Docker volume named `wishboard_pgdata` (shared PostgreSQL data)

**Without Docker:**

- Ruby 3.3.0
- PostgreSQL 14+
- Redis
- Node.js + Yarn
- ImageMagick, wkhtmltopdf, libvips

---

## Environment Variables

Copy `.env.example` to `.env` and fill in the values. The required variables are:

```bash
# Rails
RAILS_ENV=development
SECRET_KEY_BASE=your_secret_key_base
DEVISE_SECRET_KEY=your_devise_secret_key

# Database (used by Docker Compose)
POSTGRES_USER=cocociti
POSTGRES_PASSWORD=your_db_password
POSTGRES_DB=cocociti_development
DATABASE_URL=postgres://cocociti:your_db_password@db:5432/cocociti_development

# Redis
REDIS_URL=redis://redis:6379/0

# Cloudinary
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret

# Firebase (push notifications)
FIREBASE_SERVER_KEY=your_server_key
FIREBASE_API_KEY=your_api_key
FIREBASE_AUTHDOMAIN=your_project.firebaseapp.com
FIREBASE_DB_URL=https://your_project.firebaseio.com
FIREBASE_PROJECT_ID=your_project_id
FIREBASE_MESSAGE_SENDER_ID=your_sender_id

# Plivo (SMS / OTP)
PLIVO_AUTH_ID=your_auth_id
PLIVO_AUTH_TOKEN=your_auth_token

# Citrus (payments)
CITRUS_ACCESS_KEY=your_access_key
CITRUS_SECRET_KEY=your_secret_key

# Facebook OAuth
FB_APP_ID=your_app_id
FB_API_KEY=your_api_key
```

---

## Docker Setup

The app is designed to run alongside a shared PostgreSQL container on a Docker network. The `web` service mounts the app directory and connects to the external network and volume.

### 1. Create the shared network and volume (once per machine)

```bash
docker network create shared-net
docker volume create wishboard_pgdata
```

### 2. Start a PostgreSQL container (if not already running)

```bash
docker run -d \
  --name postgres \
  --network shared-net \
  -e POSTGRES_USER=cocociti \
  -e POSTGRES_PASSWORD=your_db_password \
  -v wishboard_pgdata:/var/lib/postgresql/data \
  postgres:14
```

### 3. Build and start the app

```bash
docker compose up --build
```

The app will be available at `http://localhost:3001`.

### 4. Set up the database (first time only)

```bash
docker compose run --rm web bundle exec rails db:create db:migrate db:seed
```

### Common Docker commands

```bash
# Start in background
docker compose up -d

# View logs
docker compose logs -f web

# Run a Rails console
docker compose run --rm web bundle exec rails console

# Run migrations
docker compose run --rm web bundle exec rails db:migrate

# Stop containers
docker compose down

# Rebuild after Gemfile changes
docker compose up --build
```

---

## Database Setup

The schema has 200+ migrations tracking full feature evolution. After running migrations, seed data populates categories and initial chat rooms:

```bash
rails db:create db:migrate db:seed
```

---

## Running Locally (Without Docker)

```bash
# Install dependencies
bundle install

# Set up environment variables
cp .env.example .env   # then edit .env

# Set up database
rails db:create db:migrate db:seed

# Start the server
bundle exec puma -C config/puma.rb
```

The app runs on `http://localhost:3000`.

---

## Deployment

Deployments are automated via GitHub Actions on every push to `master`. The workflow:

1. SSH into the AWS Lightsail instance.
2. Pull the latest code (`git pull origin master`).
3. Rebuild and restart containers (`docker compose up --build -d`).

**Required GitHub secrets:**

| Secret     | Description                                |
| ---------- | ------------------------------------------ |
| `EC2_KEY`  | SSH private key for the Lightsail instance |
| `EC2_USER` | SSH username (e.g. `ubuntu`)               |
| `EC2_HOST` | Lightsail instance IP or hostname          |

---

## Project Structure

```
cocociti-rent/
├── app/
│   ├── controllers/        # Request handling (products, transactions, profiles, ...)
│   ├── models/             # ActiveRecord models (~42 models)
│   ├── views/              # HAML templates
│   ├── jobs/               # Background jobs (notifications, chat broadcasts)
│   ├── channels/           # ActionCable channels (chat, user, global)
│   ├── mailers/            # Email templates
│   └── uploaders/          # CarrierWave + Cloudinary uploaders
├── config/
│   ├── routes.rb           # Full route definitions
│   ├── database.yml        # PostgreSQL config
│   └── initializers/       # Third-party service setup (Cloudinary, Firebase, Plivo, ...)
├── db/
│   ├── migrate/            # 200+ migration files
│   └── seeds.rb            # Seed data (categories, chat rooms)
├── lib/
│   ├── firebase_service.rb # Firebase push notification helper
│   └── sms_service.rb      # Plivo SMS integration
├── .github/workflows/      # GitHub Actions CI/CD
├── Dockerfile
├── docker-compose.yml
└── entrypoint.sh
```

---

## License

This project is open-source and available under the [MIT License](LICENSE).
