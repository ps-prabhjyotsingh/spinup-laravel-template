<p align="center">
		<a href="https://serversideup.net/open-source/spin/"><img src=".github/images/header.png" width="1200" alt="Spin Header" /></a>
</p>

# 🏆 Official "Laravel Basic" Template by Spin
This is the official Spin template for Laravel that helps you get up and running with:

- Latest version of Laravel
- SQLite

## Looking for more features?
We have a "[Spin Pro Laravel Template](https://getspin.pro)" that includes more features for Laravel Pros:

| Feature | Spin Basic Laravel Template | Spin Pro Laravel Template |
|---------|---------------------------|-----------|
| Price | Free | $199/once (lifetime access) |
| Automated Deployments with GitHub Actions | ❌ | ✅ |
| Local Development SSL | ❌ | ✅ (Trusted) |
| Tunnel Support | ❌ | ✅ |
| SMTP Trapping | ✅ (Mailpit) | ✅ (Mailpit) |
| Vite over HTTPS | ❌ | ✅ |
| Databases | ✅ MariaDB 12, SQLite | ✅ MariaDB, MySQL, PostgreSQL, SQLite |
| Redis | ✅ | ✅ |
| Meilisearch |  ❌ | ✅ |
| Laravel Horizon | ❌ | ✅ |
| Laravel Reverb | ✅ | ✅ |
| Laravel Queues | ✅ | ✅ |
| Mailpit over HTTPS | ❌ | ✅ |
| Node Package Manager | ✅ `npm` and `yarn` | ✅ `yarn` or `npm` |
| Support | ✅ Discord, GitHub Discussions | ✅ Private Community Support |

If you're interested in the Pro version, you visit [https://getspin.pro](https://getspin.pro) for more information.

## Default configuration
To use this template, you must have [Spin](https://serversideup.net/open-source/spin/docs) installed.

```bash
spin new laravel my-laravel-app
```

By default, this template is configured to work with [`spin deploy`](https://serversideup.net/open-source/spin/docs/command-reference/deploy) out of the box. If you prefer to use CI/CD to deploy your files (which is a good idea for larger teams), you'll need to make additional changes (see the "Advanced Configuration" section below).

Before running `spin deploy`, ensure you've complete the following:

1. **ALL** steps from "Required Changes Before Using This Template" section have been completed
1. You've customized and have a valid [`.spin.yml`](https://serversideup.net/open-source/spin/docs/guide/preparing-your-servers-for-spin#configure-other-server-settings) file
1. You've customized and have a valid [`.spin-inventory.yml`](https://serversideup.net/open-source/spin/docs/guide/preparing-your-servers-for-spin#inventory)
1. Your server is online and has been provisioned with [`spin provision`](https://serversideup.net/open-source/spin/docs/command-reference/provision)


Once the steps above are complete, you can run `spin deploy` to deploy your application:

```bash
spin deploy <environment-name>
```

### 🌎 Default Development URLs
- **Laravel**: [http://localhost](http://localhost)
- **Mailpit Web UI**: [http://mailpit.localhost](http://mailpit.localhost) or [http://localhost:8025](http://localhost:8025)
- **Mailpit SMTP**: `mailpit:1025` (internal) or `localhost:1025` (external)
- **Laravel Reverb**: [http://reverb.localhost](http://reverb.localhost) (WebSocket server)
- **Vite Dev Server**: [http://localhost:5173](http://localhost:5173)
- **MariaDB**: `localhost:3306` (User: `laravel`, Password: `secret`, Database: `laravel`)
- **Redis**: `localhost:6379`

## 📦 Included Services

This template includes the following services configured and ready to use:

### MariaDB 12
- **Development**: Available at `localhost:3306`
- **Connection**: Configured via environment variables in `.env.example.spin`
- **Persistence**: Data stored in Docker volume `mariadb-data`
- **Health Checks**: Automatically ensures database is ready before starting dependent services
- **Production**: Runs with resource limits and automatic restarts

### Redis 7
- **Development**: Available at `localhost:6379`
- **Features**: AOF persistence enabled, automatic snapshots every 60 seconds
- **Use Cases**: Caching, sessions, queue backend, Reverb backend
- **Production**: Supports optional password authentication via `REDIS_PASSWORD`

### Laravel Queue Workers
- **Development**: Runs automatically as a dedicated service
- **Configuration**: Uses Redis as the queue connection by default
- **Command**: `php artisan queue:work --sleep=3 --timeout=120 --tries=3 --max-time=3600`
- **Production**: Scales to 2 replicas with resource limits (512MB memory, 0.5 CPU)
- **Restart Policy**: Automatically restarts on failure

### Laravel Reverb (WebSocket Server)
- **Development**: Available at [http://reverb.localhost](http://reverb.localhost)
- **Port**: Listens on 8080 internally, routed via Traefik
- **Features**: Real-time broadcasting with WebSocket support
- **Backend**: Uses Redis for scalability
- **Production**: Runs 2 replicas with automatic SSL via Traefik + Let's Encrypt
- **Configuration**: Set `REVERB_HOST` to your domain in production

### Mailpit (Development Only)
- **Web UI**: [http://mailpit.localhost](http://mailpit.localhost) or [http://localhost:8025](http://localhost:8025)
- **SMTP**: Port 1025 (no authentication required)
- **Features**: Catches all outgoing emails in development
- **Use**: Test password resets, notifications, etc. without sending real emails

### Node.js with npm & Yarn
- **Image**: Node 20 Alpine
- **Features**: Corepack enabled (supports npm, yarn, pnpm)
- **Vite**: Dev server runs on port 5173 with HMR support
- **Usage**: 
  - `spin exec node npm install`
  - `spin exec node npm run dev`
  - `spin exec node yarn install` (after enabling Corepack)
  - `spin exec node yarn dev`

## 📡 Setting Up Laravel Reverb

The template includes a Reverb service, but you need to install the Laravel Reverb package in your project:

### Installation

```bash
# Install Reverb package
spin exec php composer require laravel/reverb

# Publish Reverb configuration (if needed)
spin exec php php artisan reverb:install
```

### Configuration

Reverb is already configured in `.env.example.spin`. Ensure your Laravel project has these settings:

```env
BROADCAST_CONNECTION=reverb
REVERB_APP_ID=my-app-id
REVERB_APP_KEY=my-app-key
REVERB_APP_SECRET=my-app-secret
REVERB_HOST=reverb.localhost
REVERB_PORT=80
REVERB_SCHEME=http
```

### Broadcasting Configuration

Modern Laravel versions (11+) include Reverb support out-of-the-box. For older versions, add to `config/broadcasting.php`:

```php
'reverb' => [
    'driver' => 'reverb',
    'key' => env('REVERB_APP_KEY'),
    'secret' => env('REVERB_APP_SECRET'),
    'app_id' => env('REVERB_APP_ID'),
    'options' => [
        'host' => env('REVERB_HOST'),
        'port' => env('REVERB_PORT', 80),
        'scheme' => env('REVERB_SCHEME', 'http'),
    ],
],
```

### Frontend Setup

Install Laravel Echo and configure it to use Reverb:

```bash
spin exec node npm install --save-dev laravel-echo pusher-js
```

In your `resources/js/bootstrap.js`:

```javascript
import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'reverb',
    key: import.meta.env.VITE_REVERB_APP_KEY,
    wsHost: import.meta.env.VITE_REVERB_HOST,
    wsPort: import.meta.env.VITE_REVERB_PORT ?? 80,
    wssPort: import.meta.env.VITE_REVERB_PORT ?? 80,
    forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    enabledTransports: ['ws', 'wss'],
});
```

### Testing Reverb

```bash
# Restart the Reverb service
docker compose restart reverb

# View Reverb logs
docker compose logs -f reverb

# Access Reverb
# Development: http://reverb.localhost
# Production: https://your-reverb-domain.com
```

## 👉 Required Changes Before Using This Template
> [!CAUTION]
> You need to make changes before using this template.

### 🚀 Quick Start (Development)

```bash
# Start all services
spin up

# Run database migrations
spin exec php php artisan migrate

# Install Node dependencies (optional, for front-end development)
spin exec node npm install

# Start Vite dev server (optional, in separate terminal)
spin exec node npm run dev

# View queue worker logs
docker compose logs -f queue

# Access services:
# - App: http://localhost
# - Mailpit: http://mailpit.localhost or http://localhost:8025
# - Reverb: http://reverb.localhost
# - Vite: http://localhost:5173
```

### Create an `.env.production` file
By default, this template is configured to use `spin deploy` which defaults to the `production` environment. You need to create an `.env.production` file in the root of your project.

```bash
cp .env.example.spin .env.production
```

Configure your `.env.production` file with the appropriate values for your production environment:

**Critical Production Settings:**
- `APP_URL`: Your production domain
- `DB_CONNECTION=mysql`: MariaDB connection
- `DB_HOST=mariadb`
- `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`: Strong database credentials
- `DB_ROOT_PASSWORD`: Strong root password
- `REDIS_PASSWORD`: Strong Redis password (highly recommended)
- `REVERB_HOST`: Your Reverb WebSocket domain (e.g., `reverb.example.com`)
- `REVERB_SCHEME=https`: Use HTTPS in production
- `QUEUE_CONNECTION=redis`: Enable Redis-backed queues

### Set your production URL
Almost everyone wants to run HTTPS with a valid certificate in production for free and it's totally possible to do this with Let's Encrypt. You'll need to let Let's Encrypt which domain know what domain you are using.

> [!WARNING]
> **You must have your DNS configured correctly (with your provider like CloudFlare, NameCheap, etc) AND your server accessible to the outside world BEFORE running a deployment.** When Let's Encrypt validates you own the domain name, it will attempt to connect to your server over HTTP from the outside world using the [HTTP-01 challenge](https://letsencrypt.org/docs/challenge-types/). If your server is not accessible during this process, they will not issue a certificate.

```yaml
# File to update:
# docker-compose.prod.yml

      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.my-php-app.rule=Host(`${SPIN_APP_DOMAIN}`)"
```

By default, if you're using `spin deploy` it will use the `APP_URL` from your `.env.<environment>` file to generate the `SPIN_APP_DOMAIN`. You can also be explicit and change `${SPIN_APP_DOMAIN}` to `myapp.example.com`.

### Set your email contact for Let's Encrypt certificates
Let's encrypt requires an email address to issue certificates. You can set this in the Traefik configuration for production.

```yml
# File to update:
# .infrastructure/conf/traefik/prod/traefik.yml

certificatesResolvers:
  letsencryptresolver:
    acme:
      email: "changeme@example.com"
```

Change `changeme@example.com` to a valid email address. This email address will be used by Let's Encrypt to send you notifications about your certificates.

### Configure API IP Allowlist (Optional)
This template includes support for restricting API access by IP address. To enable this:

1. **Set allowed IPs in `.env`:**
   ```bash
   ALLOWED_API_IPS=203.0.113.0,198.51.100.0,::1,127.0.0.1
   ```

2. **Create middleware** (example implementation):
   ```php
   // app/Http/Middleware/RestrictApiByIp.php
   <?php

   namespace App\Http\Middleware;

   use Closure;
   use Illuminate\Http\Request;

   class RestrictApiByIp
   {
       public function handle(Request $request, Closure $next)
       {
           $allowedIps = array_map('trim', explode(',', env('ALLOWED_API_IPS', '')));
           
           if (!empty($allowedIps) && !in_array($request->ip(), $allowedIps, true)) {
               abort(403, 'Forbidden - IP not allowed');
           }

           return $next($request);
       }
   }
   ```

3. **Register middleware** in `bootstrap/app.php` or apply to specific routes.

### Determine if you want to use the "AUTORUN" feature
By default, we have [Laravel Automations](https://serversideup.net/open-source/docker-php/docs/laravel/laravel-automations) configured to run with the [serversideup/php](https://serversideup.net/open-source/docker-php/) Docker image.

If you do not want this behavior, you can remove the `AUTORUN_ENABLED` environment variable from the `php` service in the `docker-compose.prod.yml` file.

```yml
# File to update:
# docker-compose.prod.yml

  php:
    image: ${SPIN_IMAGE_DOCKERFILE}
    environment:
      - AUTORUN_ENABLED: "true" # 👈 Remove this line if you don't want Laravel Automations
```

## 🔄 Switching Between Databases

This template is configured by default to use **MariaDB 12**. If you want to use SQLite instead:

### Switch to SQLite

1. **Update `.env` file:**
   ```bash
   DB_CONNECTION=sqlite
   DB_DATABASE=/var/www/html/.infrastructure/volume_data/sqlite/database.sqlite
   # Comment out MariaDB settings
   # DB_HOST=mariadb
   # DB_PORT=3306
   # DB_USERNAME=laravel
   # DB_PASSWORD=secret
   ```

2. **Create the SQLite database directory:**
   ```bash
   mkdir -p .infrastructure/volume_data/sqlite
   touch .infrastructure/volume_data/sqlite/database.sqlite
   ```

3. **Run migrations:**
   ```bash
   spin exec php php artisan migrate
   ```

4. **Optional: Remove MariaDB service** from `docker-compose.dev.yml` to save resources.

### Switch to MariaDB (Default)

The template uses MariaDB by default. If you previously switched to SQLite:

1. **Update `.env` file:**
   ```bash
   DB_CONNECTION=mysql
   DB_HOST=mariadb
   DB_PORT=3306
   DB_DATABASE=laravel
   DB_USERNAME=laravel
   DB_PASSWORD=secret
   ```

2. **Ensure MariaDB service is running:**
   ```bash
   spin up
   ```

3. **Run migrations:**
   ```bash
   spin exec php php artisan migrate
   ```

## ⚡️ Initializing in an existing project
If you're using an existing project with SQLite, you will need to move your database to a volume, especially if you're deploying to production with these templates.

### Background
Laravel comes with the default path of `database/database.sqlite`. This path is not ideal for Docker deployments, because the parent directory of this file contains other database files used by Laravel.

For Docker Swarm (what is used in production), we need to place the SQLite in a dedicated folder so we can mount a Docker Volume to ensure the data persists.

### Laravel ENV Changes
To prepare the project, we automatically set the `DB_DATABASE` environment variable.

```bash
# Set absolute path to SQLite database from the container's perspective
DB_DATABASE=/var/www/html/.infrastructure/volume_data/sqlite/database.sqlite
```

**NOTE:** Notice how this is the ABSOLUTE path to the database file. The reason why we use `/var/www/html` is because that is the absolute path to the file **in the eyes of the CONTAINER** (not the host machine).

### Development Setup For SQLite
Your project folder is mounted as `/var/www/html` inside the container. You simply need to ensure the `.infrastructure/volume_data/sqlite/database.sqlite` file exists in your project folder on your host. Move your existing database file to this location if you want to migrate your data.

### Production Setup For SQLite
We automatically create a `database_sqlite` volume in production. This volume is mounted to `/var/www/html/.infrastructure/volume_data/sqlite/` to the `php` service.

## 🏗️ Building Assets for Production

For production deployments, you have two options for building front-end assets:

### Option A: Build in CI/CD (Recommended)
Build assets before creating your Docker image:

```bash
# Using npm
docker run --rm -v "$PWD":/app -w /app node:20-alpine sh -c "corepack enable && npm ci && npm run build"

# Using yarn
docker run --rm -v "$PWD":/app -w /app node:20-alpine sh -c "corepack enable && yarn install --frozen-lockfile && yarn build"
```

Then build your Docker image with the compiled assets included.

### Option B: Multi-stage Docker Build
Uncomment the `assets` stage in the `Dockerfile` to build assets during the Docker build process:

1. **Uncomment the assets stage** (lines 52-60 in Dockerfile):
   ```dockerfile
   FROM node:20-alpine AS assets
   WORKDIR /app
   COPY package*.json yarn.lock* ./
   RUN corepack enable && \
       if [ -f yarn.lock ]; then yarn install --frozen-lockfile; \
       else npm ci; fi
   COPY . .
   RUN if [ -f yarn.lock ]; then yarn build; \
       else npm run build; fi
   ```

2. **Uncomment the COPY command** in the deploy stage (line 69):
   ```dockerfile
   COPY --from=assets --chown=www-data:www-data /app/public/build /var/www/html/public/build
   ```

3. **Build your image**:
   ```bash
   docker build --target deploy -t myapp:latest .
   ```

**Note:** Do not add the Node service to `docker-compose.prod.yml` - it's only needed for development.

## 🚀 Running Laravel Specific Commands
In development, you may want to run artisan commands or composer commands. We can use [`spin run`](https://serversideup.net/open-source/spin/docs/command-reference/run) or [`spin exec`](https://serversideup.net/open-source/spin/docs/command-reference/exec) to run these commands.

```bash
spin run php php artisan migrate
```
The above command will create a new container to run the `php artisan migrate` command. You can change `run` for `exec` if you'd like to run the command in an existing, **running** container.

```bash
spin run php composer install
```

The above command will create a new container to run the `composer install` command. This is helpful if you need to install new packages or update your `composer.lock` file.

If you need to attach your terminal to the container's shell, you can use `spin exec`:

```bash
spin exec -it php sh
```

This will attach your terminal to the `php` container's shell. If you're using an Alpine image, `sh` is the default shell. If you're using a Debian image, you can use `bash` instead of `sh`.

Feel free to run any commands you'd like with `spin run` or `spin exec`. The above examples should help give you patterns what you need to do.

## 👨‍🔬 Advanced configuration
If you'd like to further customize your experience, here are some helpful tips:

### Trusted SSL certificates in development
We provide certificates by default. If you'd like to trust these certificates, you need to install the CA on your machine.

**Download the CA Certificate:**
- https://serversideup.net/ca/

You can create your own certificate trust if you'd like too. Just simply replace our certificates with your own.

### Change the deployment image name
If you're using CI/CD (and NOT using `spin deploy`), you'll likely want to change the image name in the `docker-compose.prod.yml` file.

```yaml
  php:
    image: ${SPIN_IMAGE_DOCKERFILE} # 👈 Change this if you're not using `spin deploy`
```

Set this value to the published image with your image repository.

### Set the Traefik configuration MD5 hash
When running `spin deploy`, we automatically grab the MD5 hash value of the Traefik configuration and set it to `SPIN_TRAEFIK_CONFIG_MD5_HASH`. This efficiently ensures your Docker Swarm Configuration is always up to date.

Be sure to change this value or set the MD5 hash so you can continue to receive the benefits of getting the best deployment strategy when updating Docker Swarm configurations with Traefik.

```yaml
configs:
  traefik:
    name: "traefik-${SPIN_TRAEFIK_CONFIG_MD5_HASH}.yml"
    file: ./.infrastructure/conf/traefik/prod/traefik.yml
```

## Resources
- **[Website](https://serversideup.net/open-source/spin/)** overview of the product.
- **[Docs](https://serversideup.net/open-source/spin/docs)** for a deep-dive on how to use the product.
- **[Discord](https://serversideup.net/discord)** for friendly support from the community and the team.
- **[GitHub](https://github.com/serversideup/spin)** for source code, bug reports, and project management.
- **[Get Professional Help](https://serversideup.net/professional-support)** - Get video + screen-sharing help directly from the core contributors.

## Contributing
As an open-source project, we strive for transparency and collaboration in our development process. We greatly appreciate any contributions members of our community can provide. Whether you're fixing bugs, proposing features, improving documentation, or spreading awareness - your involvement strengthens the project. Please review our [contribution guidelines](https://serversideup.net/open-source/spin/docs/community/contributing) and [code of conduct](./.github/code_of_conduct.md) to understand how we work together respectfully.

- **Bug Report**: If you're experiencing an issue while using this project, please [create an issue](https://github.com/serversideup/spin-template-laravel/issues/new).
- **Feature Request**: Make this project better by [submitting a feature request](https://github.com/serversideup/spin-template-laravel/issues/new).
- **Documentation**: Improve our documentation by contributing to this README
- **Community Support**: Help others on [GitHub Discussions](https://github.com/serversideup/spin/discussions) or [Discord](https://serversideup.net/discord).
- **Security Report**: Report critical security issues via [our responsible disclosure policy](https://www.notion.so/Responsible-Disclosure-Policy-421a6a3be1714d388ebbadba7eebbdc8).

Need help getting started? Join our Discord community and we'll help you out!

<a href="https://serversideup.net/discord"><img src="https://serversideup.net/wp-content/themes/serversideup/images/open-source/join-discord.svg" title="Join Discord"></a>

## Our Sponsors
All of our software is free an open to the world. None of this can be brought to you without the financial backing of our sponsors.

<p align="center"><a href="https://github.com/sponsors/serversideup"><img src="https://521public.s3.amazonaws.com/serversideup/sponsors/sponsor-box.png" alt="Sponsors"></a></p>

#### Individual Supporters
<!-- supporters --><a href="https://github.com/alexjustesen"><img src="https://github.com/alexjustesen.png" width="40px" alt="alexjustesen" /></a>&nbsp;&nbsp;<a href="https://github.com/GeekDougle"><img src="https://github.com/GeekDougle.png" width="40px" alt="GeekDougle" /></a>&nbsp;&nbsp;<!-- supporters -->

## About Us
We're [Dan](https://twitter.com/danpastori) and [Jay](https://twitter.com/jaydrogers) - a two person team with a passion for open source products. We created [Server Side Up](https://serversideup.net) to help share what we learn.

<div align="center">

| <div align="center">Dan Pastori</div>                  | <div align="center">Jay Rogers</div>                                 |
| ----------------------------- | ------------------------------------------ |
| <div align="center"><a href="https://twitter.com/danpastori"><img src="https://serversideup.net/wp-content/uploads/2023/08/dan.jpg" title="Dan Pastori" width="150px"></a><br /><a href="https://twitter.com/danpastori"><img src="https://serversideup.net/wp-content/themes/serversideup/images/open-source/twitter.svg" title="Twitter" width="24px"></a><a href="https://github.com/danpastori"><img src="https://serversideup.net/wp-content/themes/serversideup/images/open-source/github.svg" title="GitHub" width="24px"></a></div>                        | <div align="center"><a href="https://twitter.com/jaydrogers"><img src="https://serversideup.net/wp-content/uploads/2023/08/jay.jpg" title="Jay Rogers" width="150px"></a><br /><a href="https://twitter.com/jaydrogers"><img src="https://serversideup.net/wp-content/themes/serversideup/images/open-source/twitter.svg" title="Twitter" width="24px"></a><a href="https://github.com/jaydrogers"><img src="https://serversideup.net/wp-content/themes/serversideup/images/open-source/github.svg" title="GitHub" width="24px"></a></div>                                       |

</div>

### Find us at:

* **📖 [Blog](https://serversideup.net)** - Get the latest guides and free courses on all things web/mobile development.
* **🙋 [Community](https://community.serversideup.net)** - Get friendly help from our community members.
* **🤵‍♂️ [Get Professional Help](https://serversideup.net/professional-support)** - Get video + screen-sharing support from the core contributors.
* **💻 [GitHub](https://github.com/serversideup)** - Check out our other open source projects.
* **📫 [Newsletter](https://serversideup.net/subscribe)** - Skip the algorithms and get quality content right to your inbox.
* **🐥 [Twitter](https://twitter.com/serversideup)** - You can also follow [Dan](https://twitter.com/danpastori) and [Jay](https://twitter.com/jaydrogers).
* **❤️ [Sponsor Us](https://github.com/sponsors/serversideup)** - Please consider sponsoring us so we can create more helpful resources.

## Our products
If you appreciate this project, be sure to check out our other projects.

### 📚 Books
- **[The Ultimate Guide to Building APIs & SPAs](https://serversideup.net/ultimate-guide-to-building-apis-and-spas-with-laravel-and-nuxt3/)**: Build web & mobile apps from the same codebase.
- **[Building Multi-Platform Browser Extensions](https://serversideup.net/building-multi-platform-browser-extensions/)**: Ship extensions to all browsers from the same codebase.

### 🛠️ Software-as-a-Service
- **[Bugflow](https://bugflow.io/)**: Get visual bug reports directly in GitHub, GitLab, and more.
- **[SelfHost Pro](https://selfhostpro.com/)**: Connect Stripe or Lemonsqueezy to a private docker registry for self-hosted apps.

### 🌍 Open Source
- **[serversideup/php Docker Images](https://serversideup.net/open-source/docker-php/)**: PHP Docker images optimized for Laravel and running PHP applications in production.
- **[Financial Freedom](https://github.com/serversideup/financial-freedom)**: Open source alternative to Mint, YNAB, & Monarch Money.
- **[AmplitudeJS](https://521dimensions.com/open-source/amplitudejs)**: Open-source HTML5 & JavaScript Web Audio Library.
