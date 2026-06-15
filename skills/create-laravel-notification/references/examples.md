# Examples

This file contains examples of Laravel notifications and can be used as a guide.

## Simple Notification (No Constructor)

For notifications that don't need contextual data:

```php
<?php

declare(strict_types=1);

namespace App\Notifications;

use App\Types\Notification;
use Illuminate\Notifications\Messages\MailMessage;

final class NotificationName extends Notification
{
    /**
     * Retrieve the mail representation of the notification.
     *
     */
    public function toMail(object $notifiable) : MailMessage
    {
        return $this->email('Subject Line');
    }
}
```

## Notification with Dependencies

For notifications that need models or data:

```php
<?php

declare(strict_types=1);

namespace App\Notifications;

use App\Models\Organization;
use App\Types\Notification;
use Illuminate\Support\Facades\URL;
use Illuminate\Notifications\Messages\MailMessage;

final class MembershipInvitation extends Notification
{
    /**
     * Constructor.
     *
     */
    public function __construct(
        public Organization $organization,
        public string $role,
    ) {}

    /**
     * Create the content to be passed to the mailable.
     *
     */
    protected function preparePayload(object $notifiable) : array
    {
        return [
            'organization_name' => $this->organization->name,
            'role'              => $this->role,
            'url'               => URL::signedRoute('organizations.memberships.accept', [
                'organization' => $this->organization,
                'email'        => $notifiable->email,
            ]),
        ];
    }

    /**
     * Retrieve the mail representation of the notification.
     *
     */
    public function toMail(object $notifiable) : MailMessage
    {
        return $this->email(
            subject : "Invitation - {$this->organization->name}",
            data    : $this->preparePayload($notifiable),
        );
    }
}
```

## Notification with Token

For notifications requiring secure tokens:

```php
<?php

declare(strict_types=1);

namespace App\Notifications;

use App\Types\Notification;
use Illuminate\Support\Facades\URL;
use Illuminate\Notifications\Messages\MailMessage;

final class ResetPassword extends Notification
{
    /**
     * Constructor.
     *
     */
    public function __construct(
        public string $token,
    ) {}

    /**
     * Create the content to be passed to the mailable.
     *
     */
    protected function preparePayload(object $notifiable) : array
    {
        return [
            'url' => URL::route('authentication.password-reset.edit', [
                'token' => $this->token,
                'email' => $notifiable->email,
            ]),
        ];
    }

    /**
     * Retrieve the mail representation of the notification.
     *
     */
    public function toMail(object $notifiable) : MailMessage
    {
        return $this->email(
            subject : 'Password Reset Request',
            data    : $this->preparePayload($notifiable),
        );
    }
}
```

## Minimal Notification (No Payload)

For simple informational notifications:

```php
<?php

declare(strict_types=1);

namespace App\Notifications;

use App\Types\Notification;
use Illuminate\Notifications\Messages\MailMessage;

final class ProcessedImportedContacts extends Notification
{
    /**
     * Retrieve the mail representation of the notification.
     *
     */
    public function toMail(object $notifiable) : MailMessage
    {
        return $this->email('Your contacts have been imported');
    }
}
```

## Email Templates

### Simple Template (No Button)

```blade
@component('mail::message')
Your contacts have been successfully imported and are now available in your account.

You can view them in your contacts list.
@endcomponent
```

### Template with Action Button

```blade
@component('mail::message')
You have requested a password reset for your account.

Click the button below to reset your password. This link will expire in 60 minutes.

{{-- Action --}}
@component('mail::button', ['url' => $url])
Reset Password
@endcomponent

If you did not request a password reset, you can safely ignore this email.

{{-- Subcopy --}}
@slot('subcopy')
If you are unable to click the above button, then please either
click the following link, or copy and paste it into your browser:
[{{ $url }}]({{ $url }})
@endslot
@endcomponent
```

### Template with Multiple Variables

```blade
@component('mail::message')
You have been invited to {{ $organization_name }}.

Your role will be: {{ $role }}

{{-- Action --}}
@component('mail::button', ['url' => $url])
Accept Invitation
@endcomponent

If you were not expecting this invitation, you can safely ignore this email.

{{-- Subcopy --}}
@slot('subcopy')
If you are unable to click the above button, then please either
click the following link, or copy and paste it into your browser:
[{{ $url }}]({{ $url }})
@endslot
@endcomponent
```

## URL Generation Patterns

```php
// Simple route
'url' => URL::route('account.home.show'),

// Route with parameters
'url' => URL::route('operations.offers.show', [
    'organization' => $this->organization,
    'offer'        => $this->offer,
]),

// Signed URL (for guest access)
'url' => URL::signedRoute('public.resource.show', [
    'token' => $this->token,
]),

// Route with email parameter
'url' => URL::route('authentication.password-reset.edit', [
    'token' => $this->token,
    'email' => $notifiable->email,
]),
```

## Subject Line Patterns

```php
// Simple subject
$this->email('Action Required')

// With organization context
$this->email("Invitation - {$this->organization->name}")

// With named parameters
$this->email(
    subject : 'Password Reset Request',
    data    : $this->preparePayload($notifiable),
)
```

## Sending Notifications

### From Controllers

```php
use App\Notifications\MembershipInvitation;

// In controller method
$user->notify(new MembershipInvitation($organization, 'Owner'));
```

### From Actions

```php
final class Store
{
    public function __invoke() : Model
    {
        $model = $this->organization->models()->create($this->payload);

        $this->organization->owner->notify(
            new ModelCreated($model)
        );

        return $model;
    }
}
```

### From Jobs

```php
final class ProcessJob extends Job
{
    public function handle() : void
    {
        // Process work...

        $this->organization->owner->notify(
            new ProcessingComplete($this->organization)
        );
    }
}
```
