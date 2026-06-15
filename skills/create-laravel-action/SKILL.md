---
name: create-laravel-action
description: This skill creates Laravel action classes in accordance with the patterns used in this codebase.
---

You are responsible for creating action classes that match the exact patterns used in the codebase.

## Rules

- **Extend `App\Types\Action`** - NOT a generic service class or Laravel job.
- **Store actions in `app/Actions/`** - Use the `App\Actions` namespace.
- **Dispatch via `ActionName::dispatch(...)`** - The base action instantiates the class and calls `handle()`.
- **Use constructor promotion** - Prefer `private` promoted properties; use `public` only when existing call sites need property access.
- **Use `#[AtomicLock]` only for concurrency-sensitive workflows** - Implement `uniqueId()` whenever the action uses the attribute.
- **Keep controllers thin** - Put multi-step business logic, external service calls, storage cleanup, and transactional workflows in actions.
- **Abort through `$this->abort()`** - Use it for action-level failure responses instead of calling `App::abort()` directly.

## File Organization

Actions are stored in:
```text
app/Actions/
├── DeleteOrganization.php
├── DeleteUser.php
└── PurchaseCredits.php
```

## Implementation

1. **Reference project rules** - Read `.agents/rules/general.md`, `.agents/rules/php.md`, and `.agents/rules/laravel.md` before editing PHP.
2. **Inspect existing actions** - Match the closest action in `app/Actions/` before creating a new pattern.
3. **Create the action file** - Use a singular, verb-led class name such as `DeleteUser` or `PurchaseCredits`.
4. **Extend `App\Types\Action`** - Import `App\Types\Action` and declare the class `final`.
5. **Add a documented constructor** - Use promoted properties for all required inputs.
6. **Add private helper methods when useful** - Keep helper method names in alphabetical order among methods you add, without reordering existing methods.
7. **Implement `handle()`** - Put the main workflow here and return the narrowest useful type, often `void`.
8. **Add `uniqueId()` when locked** - Return a stable string that scopes the lock to the relevant model or business operation.
9. **Wire the action from the caller** - Replace inline workflow logic with `ActionName::dispatch(...)`.

## Structure

Use this base shape:
```php
<?php

declare(strict_types=1);

namespace App\Actions;

use App\Types\Action;

final class ActionName extends Action
{
    /**
     * Constructor.
     *
     */
    public function __construct(
        private Model $model,
    ) {}

    /**
     * Execute the action.
     *
     */
    public function handle() : void
    {
        //
    }
}
```

For locked workflows:
```php
#[AtomicLock(lockFor : 45, waitFor : 30)]
final class PurchaseCredits extends Action
{
    /**
     * Generate a unique identifier for the action.
     *
     */
    public function uniqueId() : string
    {
        return "organizations-{$this->organization->id}-purchase-credits";
    }
}
```

## Output Format

After completing your work:
1. Show the action file created or changed
2. Note each caller updated to dispatch the action
3. Note whether `#[AtomicLock]` was used and why
4. Suggest focused tests for the workflow

## Additional Resources

- For examples, see [examples.md](references/examples.md)
