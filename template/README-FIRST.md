# 🚀 Quick Setup Guide

Welcome to your new Laravel project with the Spin template! Follow these steps to get started.

## 1️⃣ Start Services

```bash
spin up
```

This will start:
- ✅ Traefik (reverse proxy)
- ✅ MariaDB 12 (database)
- ✅ Redis 7 (cache & queues)
- ✅ PHP/Nginx (your Laravel app)
- ✅ Queue worker
- ✅ Mailpit (email testing)

**Note:** Reverb and Node services are disabled by default. See "Optional Services" below.

## 2️⃣ Run Database Migrations

```bash
spin exec php php artisan migrate
```

This creates the necessary database tables including `cache`, `sessions`, `jobs`, and `users`.

## 3️⃣ Access Your Application

- **App**: http://localhost
- **Mailpit**: http://mailpit.localhost or http://localhost:8025
- **Database**: localhost:3306 (user: `laravel`, password: `secret`)

## 📋 Configuration

Your `.env` file is already configured with:
- ✅ MariaDB connection
- ✅ Redis for cache and queues
- ✅ Mailpit for email testing

## 🔧 Optional Services

### Laravel Reverb (Real-time Broadcasting)

1. **Install the package:**
   ```bash
   spin exec php composer require laravel/reverb
   spin exec php php artisan reverb:install
   ```

2. **Update `.env`:**
   ```bash
   BROADCAST_CONNECTION=reverb
   ```

3. **Enable the service:**
   Remove the `reverb` section from profiles in `docker-compose.dev.yml` or start with:
   ```bash
   spin up --profile reverb
   ```

### Node.js / Vite (Frontend Development)

1. **Install dependencies:**
   ```bash
   spin exec node npm install
   # or
   spin exec node sh -c "corepack enable && yarn install"
   ```

2. **Start dev server:**
   ```bash
   spin exec node npm run dev
   # or
   spin exec node yarn dev
   ```

3. **Enable auto-start:**
   Remove the `node` section from profiles in `docker-compose.dev.yml`

## 🎯 Common Commands

```bash
# Run Artisan commands
spin exec php php artisan migrate
spin exec php php artisan tinker
spin exec php php artisan make:model Post

# Run Composer
spin exec php composer require package/name
spin exec php composer update

# View logs
docker compose logs -f php
docker compose logs -f queue
docker compose logs -f mariadb

# Stop services
spin down

# Restart a specific service
docker compose restart queue
```

## 🔍 Troubleshooting

### "Table 'cache' not found"
Run migrations: `spin exec php php artisan migrate`

### "Reverb command not found"
Install Laravel Reverb: `spin exec php composer require laravel/reverb`

### Node service keeps restarting
This is normal - Node is disabled by default. See "Optional Services" above.

### Queue worker not processing jobs
Check `.env` has `QUEUE_CONNECTION=redis` and restart: `docker compose restart queue`

## 📚 Next Steps

1. ✅ Run migrations
2. ✅ Create your first model
3. ✅ Set up authentication (if needed)
4. ✅ Configure Reverb (if using real-time features)
5. ✅ Install frontend dependencies (if building a SPA)

For complete documentation, see the main [README.md](../README.md) and [TESTING.md](../TESTING.md).

---

**Need help?** Check the [Spin documentation](https://serversideup.net/open-source/spin/docs) or the [template README](../README.md).
