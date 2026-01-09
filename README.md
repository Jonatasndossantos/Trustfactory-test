# Simple E-commerce Shopping Cart

This repository contains the technical assessment for the **Laravel Developer** position at **Trustfactory**.

It is a full-stack e-commerce application built with **Laravel 11** and **Livewire 3**, featuring a robust, Dockerized environment that simulates a production-ready architecture with S3-compatible storage, Redis queues, and automated background workers.

## 🛠 Tech Stack

-   **Backend:** Laravel 12
-   **Frontend:** Livewire 3 + Tailwind CSS (Breeze)
-   **Database:** MySQL 8.0
-   **Queue/Cache:** Redis
-   **File Storage:** MinIO (Local AWS S3 simulation)
-   **Mail Testing:** Mailpit
-   **Monitoring:** Laravel Horizon

---

## 🚀 Option 1: Docker Installation (Recommended)

This is the preferred method as it guarantees an environment identical to development, including MinIO for images and Redis for Queues.

**Prerequisites:** Docker Desktop and Git.

### 1. Setup

Clone the repository and enter the directory:

```bash
git clone https://github.com/Jonatasndossantos/Trustfactory-test.git
cd Trustfactory-test
```

Setup the environment variables (Pre-configured for Docker):

```bash
cp .env.example .env
```

Install dependencies (using a temporary container ensures no local PHP version conflicts):

```bash
docker run --rm \
    -u "$(id -u):$(id -g)" \
    -v "$(pwd):/var/www/html" \
    -w /var/www/html \
    laravelsail/php84-composer:latest \
    composer install --ignore-platform-reqs

```

### 2. Start Environment

Start the containers in the background. The first run may take a few minutes to build the images.

```bash
./vendor/bin/sail up -d
```

### 3. Database & Seeding

Run migrations and seed the database with products and a test user:

```bash
./vendor/bin/sail artisan migrate --seed
```

### 4. Access the Application

| Service           | URL                                                                                  | Credentials / Note                             |
| ----------------- | ------------------------------------------------------------------------------------ | ---------------------------------------------- |
| **Storefront**    | [http://localhost](https://www.google.com/search?q=http://localhost)                 | User: `test@example.com` / Pass: `password`    |
| **Mailpit**       | [http://localhost:8025](https://www.google.com/search?q=http://localhost:8025)       | View "Low Stock" & "Daily Report" emails here. |
| **MinIO Console** | [http://localhost:8900](https://www.google.com/search?q=http://localhost:8900)       | User: `sail` / Pass: `password`                |
| **Horizon**       | [http://localhost/horizon](https://www.google.com/search?q=http://localhost/horizon) | Monitor Queues and Jobs.                       |

---

## 🐢 Option 2: Manual Installation (Lite Mode)

If you cannot run Docker, you can run the project using a standard PHP setup with SQLite.

**Prerequisites:** PHP 8.2+, Composer, Node.js.

1. **Configure Environment:**

```bash
cp .env.example .env
```

Open `.env` and **change/uncomment** these lines to use local drivers:

```ini
DB_CONNECTION=sqlite
# DB_HOST, DB_PORT... -> Comment these out

QUEUE_CONNECTION=sync
FILESYSTEM_DISK=public
MAIL_MAILER=log

```

2. **Prepare Database:**

```bash
touch database/database.sqlite
```

3. **Install & Build:**

```bash
composer install
npm install && npm run build
```

4. **Migrate & Seed:**

```bash
php artisan migrate --seed
php artisan storage:link
```

5. **Run:**

```bash
php artisan serve
```

_Note: In this mode, check `storage/logs/laravel.log` for emails._

---

## 🏗️ Architectural Highlights

### 1. Persistence & Cart Logic

-   **Requirement:** Sessions or local storage are **NOT** used for cart data.
-   **Solution:** A relational database approach (`carts` and `cart_items` tables) linked to the `users` table. This ensures the cart persists across devices and login sessions.

### 2. Mock Payment & Checkout

-   Since external gateways (Stripe) were out of scope, a **Mock Checkout** with Database Transactions was implemented.
-   It validates stock, creates an `Order`, moves items from `Cart`, and decrements stock atomically.

### 3. Automation (Queues & Scheduler)

-   **Low Stock Notification:** Implemented via Model Observers. When stock drops below threshold, a Job is dispatched to Redis.
-   **Daily Report:** Scheduled via Laravel Scheduler.
-   **Docker Worker:** A dedicated `worker` container runs `php artisan schedule:work` automatically. You do not need to manually run queue listeners.

### 4. Connection Guide (For External Tools)

If you want to connect tools like **TablePlus** or **DBeaver** to the running Docker services:

| Service   | Host        | Port   | User   | Password   | DB/Bucket      |
| --------- | ----------- | ------ | ------ | ---------- | -------------- |
| **MySQL** | `127.0.0.1` | `3306` | `sail` | `password` | `laravel`      |
| **Redis** | `127.0.0.1` | `6379` | -      | -          | -              |
| **MinIO** | `127.0.0.1` | `9000` | `sail` | `password` | `local-bucket` |

<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400" alt="Laravel Logo"></a></p>

<p align="center">
<a href="https://github.com/laravel/framework/actions"><img src="https://github.com/laravel/framework/workflows/tests/badge.svg" alt="Build Status"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/dt/laravel/framework" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/v/laravel/framework" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/l/laravel/framework" alt="License"></a>
</p>

## About Laravel

Laravel is a web application framework with expressive, elegant syntax. We believe development must be an enjoyable and creative experience to be truly fulfilling. Laravel takes the pain out of development by easing common tasks used in many web projects, such as:

-   [Simple, fast routing engine](https://laravel.com/docs/routing).
-   [Powerful dependency injection container](https://laravel.com/docs/container).
-   Multiple back-ends for [session](https://laravel.com/docs/session) and [cache](https://laravel.com/docs/cache) storage.
-   Expressive, intuitive [database ORM](https://laravel.com/docs/eloquent).
-   Database agnostic [schema migrations](https://laravel.com/docs/migrations).
-   [Robust background job processing](https://laravel.com/docs/queues).
-   [Real-time event broadcasting](https://laravel.com/docs/broadcasting).

Laravel is accessible, powerful, and provides tools required for large, robust applications.

## Learning Laravel

Laravel has the most extensive and thorough [documentation](https://laravel.com/docs) and video tutorial library of all modern web application frameworks, making it a breeze to get started with the framework. You can also check out [Laravel Learn](https://laravel.com/learn), where you will be guided through building a modern Laravel application.

If you don't feel like reading, [Laracasts](https://laracasts.com) can help. Laracasts contains thousands of video tutorials on a range of topics including Laravel, modern PHP, unit testing, and JavaScript. Boost your skills by digging into our comprehensive video library.

## Laravel Sponsors

We would like to extend our thanks to the following sponsors for funding Laravel development. If you are interested in becoming a sponsor, please visit the [Laravel Partners program](https://partners.laravel.com).

### Premium Partners

-   **[Vehikl](https://vehikl.com)**
-   **[Tighten Co.](https://tighten.co)**
-   **[Kirschbaum Development Group](https://kirschbaumdevelopment.com)**
-   **[64 Robots](https://64robots.com)**
-   **[Curotec](https://www.curotec.com/services/technologies/laravel)**
-   **[DevSquad](https://devsquad.com/hire-laravel-developers)**
-   **[Redberry](https://redberry.international/laravel-development)**
-   **[Active Logic](https://activelogic.com)**

## Contributing

Thank you for considering contributing to the Laravel framework! The contribution guide can be found in the [Laravel documentation](https://laravel.com/docs/contributions).

## Code of Conduct

In order to ensure that the Laravel community is welcoming to all, please review and abide by the [Code of Conduct](https://laravel.com/docs/contributions#code-of-conduct).

## Security Vulnerabilities

If you discover a security vulnerability within Laravel, please send an e-mail to Taylor Otwell via [taylor@laravel.com](mailto:taylor@laravel.com). All security vulnerabilities will be promptly addressed.

## License

The Laravel framework is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).

```

```
