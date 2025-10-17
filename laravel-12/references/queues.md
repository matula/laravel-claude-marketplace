# Laravel 12 Queues & Jobs Reference

## Creating Jobs

Always use the Artisan command to create jobs:

```bash
php artisan make:job ProcessPodcast
php artisan make:job SendEmailNotification
```

## Basic Job Structure

```php
namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ProcessPodcast implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public Podcast $podcast,
    ) {}

    public function handle(): void
    {
        // Process the podcast
        $this->podcast->process();
    }
}
```

## Dispatching Jobs

### Immediate Dispatch

```php
// Dispatch to queue
ProcessPodcast::dispatch($podcast);

// Dispatch to specific queue
ProcessPodcast::dispatch($podcast)->onQueue('processing');

// Dispatch to specific connection
ProcessPodcast::dispatch($podcast)->onConnection('redis');

// Delay dispatch
ProcessPodcast::dispatch($podcast)->delay(now()->addMinutes(5));

// Dispatch after response sent to user
ProcessPodcast::dispatch($podcast)->afterResponse();
```

### Conditional Dispatch

```php
ProcessPodcast::dispatchIf($condition, $podcast);
ProcessPodcast::dispatchUnless($condition, $podcast);
```

### Synchronous Dispatch

```php
// Dispatch immediately without queue
ProcessPodcast::dispatchSync($podcast);
```

## Job Chains

Execute jobs in sequence:

```php
use Illuminate\Support\Facades\Bus;

Bus::chain([
    new ProcessPodcast($podcast),
    new OptimizePodcast($podcast),
    new ReleasePodcast($podcast),
])->dispatch();

// With error handling
Bus::chain([
    new ProcessPodcast($podcast),
    new OptimizePodcast($podcast),
])->catch(function (Throwable $e) {
    // Handle chain failure
})->dispatch();
```

## Job Batches

Execute multiple jobs as a batch:

```php
use Illuminate\Bus\Batch;
use Illuminate\Support\Facades\Bus;

$batch = Bus::batch([
    new ProcessPodcast($podcast1),
    new ProcessPodcast($podcast2),
    new ProcessPodcast($podcast3),
])->then(function (Batch $batch) {
    // All jobs completed successfully
})->catch(function (Batch $batch, Throwable $e) {
    // First batch job failure detected
})->finally(function (Batch $batch) {
    // The batch has finished executing
})->dispatch();

// Check batch status
$batch = Bus::findBatch($batchId);
$batch->finished(); // All jobs finished
$batch->cancelled(); // Batch was cancelled
```

## Job Middleware

Add middleware to jobs for rate limiting, authentication, etc.:

```php
use Illuminate\Queue\Middleware\RateLimited;
use Illuminate\Queue\Middleware\WithoutOverlapping;

class ProcessPodcast implements ShouldQueue
{
    public function middleware(): array
    {
        return [
            new RateLimited('podcasts'),
            new WithoutOverlapping($this->podcast->id),
        ];
    }
}
```

## Failed Jobs

### Handling Failed Jobs

```php
class ProcessPodcast implements ShouldQueue
{
    // Maximum number of attempts
    public int $tries = 3;
    
    // Maximum seconds job can run
    public int $timeout = 120;
    
    // Maximum exceptions before failing
    public int $maxExceptions = 3;
    
    public function failed(?Throwable $exception): void
    {
        // Send notification, log, etc.
        Log::error('Podcast processing failed', [
            'podcast_id' => $this->podcast->id,
            'error' => $exception?->getMessage(),
        ]);
        
        // Notify user
        $this->podcast->user->notify(
            new PodcastProcessingFailed($this->podcast)
        );
    }
}
```

### Retry Failed Jobs

```bash
# Retry all failed jobs
php artisan queue:retry all

# Retry specific job
php artisan queue:retry 5

# Forget failed job
php artisan queue:forget 5

# Flush all failed jobs
php artisan queue:flush
```

## Job Events

Listen to job lifecycle events:

```php
use Illuminate\Support\Facades\Queue;
use Illuminate\Queue\Events\JobProcessing;
use Illuminate\Queue\Events\JobProcessed;
use Illuminate\Queue\Events\JobFailed;

Queue::before(function (JobProcessing $event) {
    // Before job is processed
});

Queue::after(function (JobProcessed $event) {
    // After job is processed
});

Queue::failing(function (JobFailed $event) {
    // Job has failed
});
```

## Job Prioritization

### Queue Priority

```php
// High priority queue
ProcessPodcast::dispatch($podcast)->onQueue('high');

// Default queue
ProcessPodcast::dispatch($podcast)->onQueue('default');

// Low priority queue
ProcessPodcast::dispatch($podcast)->onQueue('low');
```

### Worker Priority

Start worker with queue priority:

```bash
php artisan queue:work --queue=high,default,low
```

## Rate Limiting

```php
use Illuminate\Queue\Middleware\RateLimited;

public function middleware(): array
{
    return [
        new RateLimited('podcasts'),
    ];
}

// In a service provider
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

RateLimiter::for('podcasts', function ($job) {
    return Limit::perMinute(10);
});
```

## Job Uniqueness

Prevent duplicate jobs from being queued:

```php
use Illuminate\Contracts\Queue\ShouldBeUnique;

class ProcessPodcast implements ShouldQueue, ShouldBeUnique
{
    public Podcast $podcast;
    
    // Unique for 1 hour
    public int $uniqueFor = 3600;
    
    public function uniqueId(): string
    {
        return $this->podcast->id;
    }
}
```

## Job Timeout

```php
class ProcessPodcast implements ShouldQueue
{
    // Timeout after 2 minutes
    public int $timeout = 120;
    
    // Or dynamic timeout
    public function timeout(): int
    {
        return $this->podcast->duration > 60 ? 300 : 120;
    }
}
```

## Queue Workers

### Starting Workers

```bash
# Start worker (development)
php artisan queue:work

# Start worker with specific queue
php artisan queue:work --queue=high,default

# Start worker with try count
php artisan queue:work --tries=3

# Start worker with timeout
php artisan queue:work --timeout=60

# Listen for new jobs (development)
php artisan queue:listen

# Process specific number of jobs then exit
php artisan queue:work --max-jobs=100

# Process for specific time then exit
php artisan queue:work --max-time=3600
```

### Production Workers

Use a process monitor like Supervisor:

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /path/to/artisan queue:work --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=forge
numprocs=8
redirect_stderr=true
stdout_logfile=/path/to/worker.log
stopwaitsecs=3600
```

## Queue Configuration

Configure queues in `config/queue.php`:

```php
'connections' => [
    'sync' => [
        'driver' => 'sync',
    ],

    'database' => [
        'driver' => 'database',
        'table' => 'jobs',
        'queue' => 'default',
        'retry_after' => 90,
        'after_commit' => false,
    ],

    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => env('REDIS_QUEUE', 'default'),
        'retry_after' => 90,
        'block_for' => null,
        'after_commit' => false,
    ],
],
```

## Best Practices Summary

1. **Use `ShouldQueue` interface** for all queued jobs
2. **Set appropriate timeouts** to prevent hanging jobs
3. **Implement `failed()` method** for error handling
4. **Use job chains** for sequential processing
5. **Use job batches** for parallel processing with completion tracking
6. **Add rate limiting** to prevent overwhelming external services
7. **Use `ShouldBeUnique`** to prevent duplicate jobs
8. **Configure proper retries** with `$tries` property
9. **Use job middleware** for cross-cutting concerns
10. **Monitor failed jobs** and set up alerts
11. **Use queue priorities** for important jobs
12. **Use Supervisor** or similar in production for worker management
