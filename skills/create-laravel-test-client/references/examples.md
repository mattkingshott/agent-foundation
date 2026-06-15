# Examples

This file contains examples of Laravel browser tests and can be used as a guide.

## Basic Structure

```php
<?php

declare(strict_types=1);

namespace Tests\Browser\Operations;

use App\Models\User;
use App\Models\Organization;
use App\Models\Contact;
use App\Support\Browser;
use App\Types\DuskTestCase;
use PHPUnit\Framework\Attributes\Test;

final class ResourceTest extends DuskTestCase
{
    #[Test]
    public function a_user_can_view_the_page() : void
    {
        $this->browse(function(Browser $browser) : void {
            $organization = Organization::factory()->create();

            $user = User::factory()
                ->for($organization)
                ->create();

            $resource = Resource::factory()
                ->for($organization)
                ->create();

            $browser->start($user);

            $browser->visitRoute('operations.resources')
                ->assertTitle('Resources')
                ->assertSee('Resources');

            $browser->assertSee($resource->name);
        });
    }
}
```

## Form Interaction

```php
#[Test]
public function a_user_can_create_a_resource() : void
{
    $this->browse(function(Browser $browser) : void {
        $organization = Organization::factory()->create();

        $user = User::factory()
            ->for($organization)
            ->create();

        $payload = Resource::factory()->make();

        $browser->start($user);

        $browser->visitRoute('operations.resources.create')
            ->type('name', $payload->name)
            ->type('description', $payload->description)
            ->select('coordinator_id', $membership->id)
            ->select('interest', Interest::HIGH->value)
            ->check('active')
            ->date('connect_at', $payload->connect_at)
            ->push('submit');

        $browser->assertRouteIs('operations.resources', ['resource' => Resource::first()])
            ->assertSee('Resource created successfully');

        $browser->assertSee($payload->name);
    });
}
```

## HTTP Mocking

```php
#[Test]
public function a_user_can_create_a_contact_with_ai_requirements() : void
{
    $this->browse(function(Browser $browser) : void {
        $browser->fake(Http::class, [
            'https://api.openai.com/v1/embeddings' => [
                'data' => [[
                    'embedding' => $this->createVector(),
                ]],
            ],
        ]);

        $organization = Organization::factory()->create([
            'credits' => 100,
        ]);

        $user = User::factory()
            ->for($organization)
            ->create();

        $payload = Contact::factory()->make();

        $browser->start($user);

        $browser->visitRoute('operations.contacts.create')
            ->type('name', $payload->name)
            ->type('requirements', 'Looking for a 3-bedroom house')
            ->push('submit');

        $browser->assertRouteIs('operations.contacts', ['contact' => Contact::first()])
            ->assertSee('Contact created successfully');
    });
}
```

## Modal Interaction

```php
#[Test]
public function a_user_can_delete_a_resource() : void
{
    $this->browse(function(Browser $browser) : void {
        $organization = Organization::factory()->create();

        $user = User::factory()
            ->for($organization)
            ->create();

        $resource = Resource::factory()
            ->for($organization)
            ->create();

        $browser->start($user);

        $browser->visitRoute('operations.resources', ['resource' => $resource])
            ->push('delete');

        $browser->menu($resource->id, 'delete_resource');

        $browser->assertRouteIs('operations.resources')
            ->assertSee('Resource deleted successfully');

        $this->assertDatabaseCount('resources', 0);
    });
}
```

## Table Item Menu

```php
#[Test]
public function a_user_can_edit_a_resource_from_table() : void
{
    $this->browse(function(Browser $browser) : void {
        $organization = Organization::factory()->create();

        $user = User::factory()
            ->for($organization)
            ->create();

        $resource = Resource::factory()
            ->for($organization)
            ->create();

        $browser->start($user);

        $browser->visitRoute('operations.resources.index');

        $browser->menu($resource->id, 'edit_resource');

        $browser->assertRouteIs('operations.resources.edit', ['resource' => $resource])
            ->assertTitle("Edit Resource");
    });
}
```

## Multi-Select

```php
#[Test]
public function a_user_can_select_multiple_professions() : void
{
    $this->browse(function(Browser $browser) : void {
        $organization = Organization::factory()->create();

        $user = User::factory()
            ->for($organization)
            ->create();

        $contact = Contact::factory()
            ->for($organization)
            ->create();

        $browser->start($user);

        $browser->visitRoute('operations.contacts.edit', ['contact' => $contact])
            ->selectMany('professions', Profession::ACCOUNTANT->value)
            ->selectMany('professions', Profession::LAWYER->value)
            ->push('submit');

        $browser->assertRouteIs('operations.contacts', ['contact' => $contact]);

        $this->assertTrue($contact->fresh()->professions->contains(Profession::ACCOUNTANT));
        $this->assertTrue($contact->fresh()->professions->contains(Profession::LAWYER));
    });
}
```