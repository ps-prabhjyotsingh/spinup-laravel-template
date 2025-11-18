# Testing Guide

This guide provides comprehensive testing instructions for all services included in the Spin Laravel template.

## Prerequisites

- Docker and Docker Compose installed
- Spin CLI installed
- A Laravel project initialized with this template

## Quick Validation

Run these commands to verify all services are working:

```bash
# Start all services
spin up

# Check service status
docker compose ps

# Expected services:
# - traefik (running)
# - mariadb (healthy)
# - redis (healthy)
# - php (running)
# - queue (running)
# - reverb (running)
# - mailpit (running)
# - node (running - may exit if no npm run dev configured)
```

## Service-by-Service Testing

### 1. MariaDB 12

**Health Check:**
```bash
docker compose ps mariadb
# Should show "(healthy)"
```

**Manual Connection Test:**
```bash
# Connect to MariaDB
spin exec php php -r "new PDO('mysql:host=mariadb;dbname=laravel', 'laravel', 'secret'); echo 'Connected successfully\n';"

# Or use mysql client
docker exec -it $(docker compose ps -q mariadb) mariadb -ularavel -psecret laravel
```

**Run Migrations:**
```bash
spin exec php php artisan migrate
```

**Expected Result:** All migrations should run successfully.

---

### 2. Redis 7

**Health Check:**
```bash
docker compose ps redis
# Should show "(healthy)"
```

**Connection Test:**
```bash
# Test Redis connection
spin exec php php -r "\$redis = new Redis(); \$redis->connect('redis', 6379); \$redis->set('test', 'value'); echo \$redis->get('test') . PHP_EOL;"
# Expected output: value
```

**Cache Test:**
```bash
spin exec php php artisan tinker
# In tinker:
>>> Cache::put('test-key', 'test-value', 60)
>>> Cache::get('test-key')
# Expected: "test-value"
```

---

### 3. Laravel Queue Workers

**Service Status:**
```bash
docker compose logs queue
# Should show: "Processing jobs from the [default] queue."
```

**Dispatch Test Job:**

1. Create a test job:
```bash
spin exec php php artisan make:job TestQueueJob
```

2. Edit `app/Jobs/TestQueueJob.php`:
```php
<?php

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\Log;

class TestQueueJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function handle(): void
    {
        Log::info('Test queue job executed successfully!');
    }
}
```

3. Dispatch the job:
```bash
spin exec php php artisan tinker
# In tinker:
>>> dispatch(new \App\Jobs\TestQueueJob())
```

4. Check logs:
```bash
docker compose logs -f queue
# Should show: "Test queue job executed successfully!"
```

---

### 4. Laravel Reverb (WebSocket Server)

**Service Status:**
```bash
docker compose logs reverb
# Should show: "Reverb server started successfully."
```

**Connection Test:**

1. Install Reverb package:
```bash
spin exec php composer require laravel/reverb
spin exec php php artisan reverb:install
```

2. Create a test event:
```bash
spin exec php php artisan make:event TestBroadcast
```

3. Edit `app/Events/TestBroadcast.php`:
```php
<?php

namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class TestBroadcast implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public string $message;

    public function __construct(string $message)
    {
        $this->message = $message;
    }

    public function broadcastOn(): array
    {
        return [
            new Channel('test-channel'),
        ];
    }
}
```

4. Broadcast the event:
```bash
spin exec php php artisan tinker
# In tinker:
>>> event(new \App\Events\TestBroadcast('Hello from Reverb!'))
```

5. Check Reverb logs:
```bash
docker compose logs -f reverb
# Should show the broadcast activity
```

**Browser Test:**

Open the browser console and test the WebSocket connection:

```javascript
// Load these in your app first:
// npm install --save-dev laravel-echo pusher-js

import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

window.Pusher = Pusher;

const echo = new Echo({
    broadcaster: 'reverb',
    key: 'my-app-key', // From .env REVERB_APP_KEY
    wsHost: 'reverb.localhost',
    wsPort: 80,
    wssPort: 80,
    forceTLS: false,
    enabledTransports: ['ws', 'wss'],
});

echo.channel('test-channel')
    .listen('TestBroadcast', (e) => {
        console.log('Received:', e);
    });
```

---

### 5. Mailpit (SMTP Trap)

**Access Web UI:**
Open http://mailpit.localhost or http://localhost:8025

**Send Test Email:**

1. Using Artisan:
```bash
spin exec php php artisan tinker
# In tinker:
>>> Mail::raw('Test email from Laravel', function($msg) { $msg->to('test@example.com')->subject('Test Email'); })
```

2. Using a notification:
```bash
spin exec php php artisan make:notification TestNotification
```

Edit `app/Notifications/TestNotification.php`:
```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Notifications\Messages\MailMessage;
use Illuminate\Notifications\Notification;

class TestNotification extends Notification
{
    use Queueable;

    public function via(object $notifiable): array
    {
        return ['mail'];
    }

    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
            ->subject('Test Notification')
            ->line('This is a test notification from Laravel.')
            ->action('Test Action', url('/'))
            ->line('Thank you!');
    }
}
```

Send it:
```bash
spin exec php php artisan tinker
# In tinker:
>>> $user = User::first() ?? User::factory()->create()
>>> $user->notify(new \App\Notifications\TestNotification())
```

3. Check Mailpit:
- Open http://mailpit.localhost
- You should see the test email(s)

---

### 6. Node.js & Vite

**Install Dependencies:**
```bash
spin exec node npm install
# Or with Yarn:
spin exec node sh -c "corepack enable && yarn install"
```

**Start Dev Server:**
```bash
# Option 1: Via docker compose (if configured)
docker compose up node

# Option 2: Manually
spin exec node npm run dev

# Option 3: With Yarn
spin exec node sh -c "corepack enable && yarn dev"
```

**Test HMR:**
1. Access your Laravel app: http://localhost
2. Access Vite dev server: http://localhost:5173
3. Edit a Vue/React component or CSS file
4. Changes should hot-reload without full page refresh

**Build Production Assets:**
```bash
spin exec node npm run build
# Or with Yarn:
spin exec node sh -c "corepack enable && yarn build"
```

---

## Integration Testing

### Full Stack Test

1. **Start all services:**
```bash
spin up
```

2. **Run migrations:**
```bash
spin exec php php artisan migrate:fresh --seed
```

3. **Create a user:**
```bash
spin exec php php artisan tinker
>>> $user = User::factory()->create(['email' => 'test@example.com'])
>>> $user->email
```

4. **Send welcome email (should appear in Mailpit):**
```bash
>>> $user->notify(new \App\Notifications\TestNotification())
```

5. **Dispatch a queued job:**
```bash
>>> dispatch(new \App\Jobs\TestQueueJob())
# Check queue logs:
docker compose logs -f queue
```

6. **Test cache:**
```bash
>>> Cache::remember('test', 60, fn() => 'cached-value')
>>> Cache::get('test')
```

7. **Broadcast an event (if Reverb configured):**
```bash
>>> event(new \App\Events\TestBroadcast('Integration test!'))
# Check Reverb logs:
docker compose logs -f reverb
```

---

## Production Testing (Docker Swarm)

### Prerequisites
- Docker Swarm initialized: `docker swarm init`
- `.env.production` configured with production credentials

### Deploy Stack

```bash
# Set environment variables
export SPIN_IMAGE_DOCKERFILE="your-registry/your-app:latest"
export SPIN_DEPLOYMENT_ENVIRONMENT="production"
export DB_ROOT_PASSWORD="strong-password"
export DB_PASSWORD="strong-password"
export REDIS_PASSWORD="strong-redis-password"
export REVERB_HOST="reverb.yourdomain.com"

# Deploy
docker stack deploy -c template/docker-compose.yml \
                     -c template/docker-compose.prod.yml \
                     myapp
```

### Verify Services

```bash
# List services
docker service ls

# Expected services:
# myapp_traefik    (1/1 replicas)
# myapp_mariadb    (1/1 replicas)
# myapp_redis      (1/1 replicas)
# myapp_php        (1/1 replicas)
# myapp_queue      (2/2 replicas)
# myapp_reverb     (2/2 replicas)
```

### Check Service Health

```bash
# Check specific service
docker service ps myapp_mariadb

# View logs
docker service logs -f myapp_queue
docker service logs -f myapp_reverb
```

### Test Scaling

```bash
# Scale queue workers
docker service scale myapp_queue=4

# Scale Reverb servers
docker service scale myapp_reverb=4

# Verify
docker service ls
```

---

## Troubleshooting

### MariaDB Connection Issues

```bash
# Check if MariaDB is healthy
docker compose ps mariadb

# View logs
docker compose logs mariadb

# Reset database
docker compose down -v
docker compose up -d mariadb
spin exec php php artisan migrate:fresh
```

### Redis Connection Issues

```bash
# Test Redis connectivity
spin exec php php -r "\$redis = new Redis(); \$redis->connect('redis', 6379); echo 'OK';"

# Clear Redis cache
spin exec php php artisan cache:clear
```

### Queue Not Processing

```bash
# Check queue worker logs
docker compose logs -f queue

# Restart queue worker
docker compose restart queue

# Manual processing
spin exec php php artisan queue:work --once
```

### Reverb Not Broadcasting

```bash
# Check Reverb logs
docker compose logs -f reverb

# Verify configuration
spin exec php php artisan config:show broadcasting

# Restart Reverb
docker compose restart reverb
```

### Mailpit Not Receiving Emails

```bash
# Check Mailpit logs
docker compose logs mailpit

# Verify SMTP settings
spin exec php php artisan config:show mail

# Test SMTP connection
telnet mailpit 1025
```

---

## Performance Testing

### Database Performance

```bash
# Run Laravel's built-in benchmarks
spin exec php php artisan tinker
>>> Benchmark::dd(fn() => User::all(), 100)
```

### Queue Performance

```bash
# Dispatch 1000 jobs
spin exec php php artisan tinker
>>> for($i = 0; $i < 1000; $i++) { dispatch(new \App\Jobs\TestQueueJob()); }

# Monitor processing
docker compose logs -f queue | grep "Processed"
```

### Redis Performance

```bash
docker exec -it $(docker compose ps -q redis) redis-benchmark -q
```

---

## Cleanup

```bash
# Stop all services
spin down

# Remove volumes (⚠️ deletes all data)
docker compose down -v

# Remove images
docker compose down --rmi all
```

---

## Next Steps

- Configure monitoring (e.g., Laravel Telescope, Horizon)
- Set up automated backups for MariaDB
- Configure Redis persistence strategy
- Implement health check endpoints
- Set up CI/CD pipelines
- Configure log aggregation
