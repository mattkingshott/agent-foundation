# Examples

This file contains examples of Laravel feature tests and can be used as a guide.

## Basic Structure (Nuxt SPA Frontend)

```php
<?php

declare(strict_types=1);

namespace Tests\Controllers\Api;

use App\Models\Contact;
use App\Models\Listing;
use Illuminate\Support\Facades\URL;
use App\Types\TestCase;
use PHPUnit\Framework\Attributes\Test;

final class ResourceTest extends TestCase
{
    #[Test]
    public function a_user_can_view_the_page() : void
    {
        $organization = Organization::factory()->create();

        $user = User::factory()
            ->for($organization)
            ->create();

        $response = $this->actingAs($user)
            ->getJson(URL::route('api.resources'))
            ->assertSuccessful();

        // Response assertions...
    }

    #[Test]
    public function a_user_can_create_a_resource() : void
    {
        $organization = Organization::factory()->create();

        $user = User::factory()
            ->for($organization)
            ->create();

        $payload = Arr::merge(Resource::factory()->make()->toArray(), [
            'user_id' => $user->id,
        ]);

        $response = $this->actingAs($user)
            ->postJson(URL::route('api.resources.store'), $payload)
            ->assertCreated();

        // Response assertions...

        $this->assertMatch(Resource::first()->name, $payload['name']);
        $this->assertTrue(Resource::first()->user->is($user));
    }

    #[Test]
    public function a_user_cannot_access_when_resource_does_not_belong_to_organization() : void
    {
        $organization = Organization::factory()->create();

        $user = User::factory()
            ->for($organization)
            ->create();

        $resource = Resource::factory()->create();

        $this->actingAs($user)
            ->getJson(URL::route('api.resources', ['resource' => $resource]))
            ->assertForbidden();
    }
}
```

## Basic Structure (Blade Frontend)

```php
<?php

declare(strict_types=1);

namespace Tests\Feature\Operations;

use App\Models\Contact;
use App\Models\Listing;
use Illuminate\Support\Facades\URL;
use App\Types\FeatureTest;
use PHPUnit\Framework\Attributes\Test;

final class ResourceTest extends FeatureTest
{
    #[Test]
    public function a_user_can_view_the_page() : void
    {
        $organization = Organization::factory()->create();

        $user = User::factory()
            ->for($organization)
            ->create();

        $this->actingAs($user)
            ->get(URL::route('operations.resources'))
            ->assertSuccessful()
            ->assertViewIs('pages.operations.resources.index');
    }

    #[Test]
    public function a_user_can_create_a_resource() : void
    {
        $organization = Organization::factory()->create();

        $user = User::factory()
            ->for($organization)
            ->create();

        $payload = Arr::merge(Resource::factory()->make()->toArray(), [
            'user_id' => $user->id,
        ]);

        $this->actingAs($user)
            ->post(URL::route('operations.resources.store'), $payload)
            ->assertRedirectToRoute('operations.resources', ['resource' => Resource::first()])
            ->assertSessionHasNotification('Resource created successfully');

        $this->assertMatch(Resource::first()->name, $payload['name']);
        $this->assertTrue(Resource::first()->user->is($user));
    }

    #[Test]
    public function a_user_cannot_access_when_resource_does_not_belong_to_organization() : void
    {
        $organization = Organization::factory()->create();

        $user = User::factory()
            ->for($organization)
            ->create();

        $resource = Resource::factory()->create();

        $this->actingAs($user)
            ->getJson(URL::route('operations.resources', ['resource' => $resource]))
            ->assertForbidden();
    }
}
```

## Using assertMatch()

Pass the model/current value as the first argument, and the payload or expected value as the second argument.

```php
#[Test]
public function a_user_can_update_a_contact() : void
{
    $organization = Organization::factory()->create();

    $user = User::factory()
        ->for($organization)
        ->create();

    $contact = Contact::factory()
        ->for($organization)
        ->create();

    $payload = Arr::merge(Contact::factory()->make()->toArray(), [
        'user_id' => $user->id,
        'interest'       => Interest::HIGH,
    ]);

    $this->actingAs($user)
        ->patchJson(URL::route('api.contacts.update', ['contact' => $contact]), $payload)
        ->assertNoContent();

    $this->assertMatch($contact->fresh()->name, $payload['name']);
    $this->assertMatch($contact->fresh()->interest, Interest::HIGH);
    $this->assertMatch($contact->fresh()->connect_at, $payload['connect_at']);
    $this->assertTrue($contact->fresh()->user->is($user));
}
```

## HTTP Mocking

```php
#[Test]
public function a_user_can_create_a_contact_with_vector() : void
{
    $organization = Organization::factory()->create([
        'credits' => 100,
    ]);

    $user = User::factory()
        ->for($organization)
        ->create();

    $payload = Arr::merge(Contact::factory()->make()->toArray(), [
        'user_id' => $user->id,
        'requirements'   => 'Looking for a 3-bedroom house',
        'vector'         => $this->createVector(),
    ]);

    $url = 'https://api.openai.com/v1/embeddings';

    Http::fake([$url => [
        'data' => [[
            'embedding' => $payload['vector'],
        ]],
    ]]);

    $response = $this->actingAs($user)
        ->postJson(URL::route('api.contacts.store'), $payload)
        ->assertSuccessful();

    // Response assertions...

    $this->assertHttpRequestSent($url, [
        'input'      => $payload['requirements'],
        'model'      => 'text-embedding-3-small',
        'dimensions' => 1536,
        'encoding'   => 'float',
        'user'       => $organization->id,
    ]);

    $this->assertVector(Contact::first()->vector, $payload['vector']);
}
```

## Queue Testing

```php
#[Test]
public function a_user_can_import_contacts() : void
{
    $organization = Organization::factory()->create();

    $user = User::factory()
        ->for($organization)
        ->create();

    Queue::fake();

    $file = UploadedFile::fake()->create('contacts.csv', 100);

    $response = $this->actingAs($user)
        ->postJson(URL::route('api.contacts.import'), ['file' => $file])
        ->assertSuccessful();

    // Response assertions...

    Queue::assertPushed(function(ImportContacts $job) use ($organization, $file) : bool {
        return $job->organization->is($organization) && $job->file === $file;
    });
}
```

## Validation Testing

```php
#[Test]
public function a_user_cannot_create_a_contact_when_credits_are_insufficient() : void
{
    $organization = Organization::factory()->create([
        'credits' => 0,
    ]);

    $user = User::factory()
        ->for($organization)
        ->create();

    $payload = Contact::factory()->make()->toArray();

    $this->actingAs($user)
        ->postJson(URL::route('api.contacts.store'), $payload)
        ->assertInvalid(['error' => 'You do not have enough AI credits']);
}
```

## Notification Testing

```php
use App\Notifications\LowCreditCount;
use Illuminate\Support\Facades\Notification;

#[Test]
public function it_sends_notification_when_credits_are_low() : void
{
    $organization = Organization::factory()->create([
        'credits' => 10,
    ]);

    $user = User::factory()
        ->for($organization)
        ->create();

    Notification::fake();

    Notification::assertSent($organization, LowCreditCount::class)
        ->assertSubject('Your organization is running low on credits')
        ->assertBodyContains($organization->name)
        ->assertAction('Manage Billing', URL::route('organizations.billing'));
}
```
