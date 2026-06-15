# Examples

This file contains examples of Laravel jobs and can be used as a guide.

## Simple Cleanup Job

```php
<?php

declare(strict_types=1);

namespace App\Jobs;

use App\Types\Job;
use App\Support\Clock;
use App\Models\Dismissal;
use Illuminate\Contracts\Queue\ShouldBeUnique;
use Illuminate\Queue\Attributes\Tries;
use Illuminate\Queue\Attributes\Timeout;

#[Tries(1)]
#[Timeout(600)]
final class PurgeExpiredDismissals extends Job implements ShouldBeUnique
{
    /**
     * Execute the job.
     *
     */
    public function handle() : void
    {
        Dismissal::query()
            ->where('expires_at', '<=', Clock::now())
            ->delete();
    }
}
```

## File Processing Job with Cleanup

```php
<?php

declare(strict_types=1);

namespace App\Jobs;

use Throwable;
use App\Types\Job;
use App\Models\Organization;
use Illuminate\Support\Facades\Storage;
use Illuminate\Contracts\Queue\ShouldBeUnique;
use App\Actions\Operations\Contact\Import as ImportContactAction;
use Illuminate\Queue\Attributes\Tries;
use Illuminate\Queue\Attributes\Timeout;

#[Tries(1)]
#[Timeout(600)]
final class ImportContacts extends Job implements ShouldBeUnique
{
    /**
     * Constructor.
     *
     */
    public function __construct(
        public Organization $organization,
        public string $file,
    ) {}

    /**
     * Perform any cleanup after the job has run.
     *
     */
    public function clear() : void
    {
        Storage::disk('private')->delete($this->file);
    }

    /**
     * Handle a job failure.
     *
     */
    public function failed(?Throwable $exception) : void
    {
        $this->clear();
    }

    /**
     * Execute the job.
     *
     */
    public function handle() : void
    {
        // Logic...

        $this->clear();
    }

    /**
     * Retrieve the unique ID for the job.
     *
     */
    public function uniqueId() : string
    {
        return "import-contacts-job-{$this->organization->id}-{$this->file}";
    }
}
```

## Complex Job with Private Methods

```php
<?php

declare(strict_types=1);

namespace App\Jobs;

use App\Types\Job;
use App\Support\Clock;
use App\Models\Organization;
use App\Notifications\ExpiringOrganization;
use Illuminate\Contracts\Queue\ShouldBeUnique;
use App\Actions\Organizations\Organization\Delete as DeleteOrganizationAction;
use Illuminate\Queue\Attributes\Tries;
use Illuminate\Queue\Attributes\Timeout;

#[Tries(1)]
#[Timeout(300)]
final class PurgeExpiredOrganizations extends Job implements ShouldBeUnique
{
    /**
     * Execute the job.
     *
     */
    public function handle() : void
    {
        $this->deleteExpiredOrganizations();
        $this->warnExpiringOrganizations();
    }

    /**
     * Remove organizations that are due to be purged.
     *
     */
    private function deleteExpiredOrganizations() : void
    {
        Organization::query()
            ->whereNull('stripe_subscription_id')
            ->where('purge_at', '<=', Clock::now())
            ->lazyById()
            ->each(function(Organization $organization) : void {
                // Logic...
            });
    }

    /**
     * Warn organizations that will expire in seven days.
     *
     */
    private function warnExpiringOrganizations() : void
    {
        Organization::query()
            ->whereNull('stripe_subscription_id')
            ->whereBetween('purge_at', [
                Clock::now()->plus(days : 7)->startOfDay(),
                Clock::now()->plus(days : 7)->endOfDay(),
            ])
            ->lazyById()
            ->each(function(Organization $organization) : void {
                // Logic...
            });
    }
}
```

## Key Patterns

### Import Organization
```php
use Throwable;
use App\Types\Job;
use App\Support\Clock;
use App\Models\Organization;
use Illuminate\Support\Facades\Storage;
use Illuminate\Contracts\Queue\ShouldBeUnique;
```
- Ordered by length (shortest first)
- Grouped: external classes, then App namespace

### Constructor
```php
/**
 * Constructor.
 *
 */
public function __construct(
    public Organization $organization,
    public string $file,
) {}
```
- Use `public` properties (NOT `readonly`) for serialization
- Required for queued jobs to serialize properly

### Cleanup Pattern
```php
public function clear() : void
{
    Storage::disk('private')->delete($this->file);
}

public function failed(?Throwable $exception) : void
{
    $this->clear();
}

public function handle() : void
{
    // Work...

    $this->clear();
}
```
- Create private `clear()` method for cleanup
- Call from both `handle()` (success) and `failed()` (failure)
- Ensures cleanup always happens

### Time Handling
```php
use App\Support\Clock;

Organization::query()
    ->where('purge_at', '<=', Clock::now())
    ->delete();
```
- NEVER use `now()`, `Carbon::now()`, or `new DateTime()`
- ALWAYS use `Clock::now()` for testability
- Use `Clock::now()->plus(days : 7)` for date math

### Method Order
2. Constructor
3. Private helper methods (`clear()`, `deleteExpiredOrganizations()`, etc.)
4. `failed()` method
5. `handle()` method
6. `uniqueId()` method
