# Laravel Action Examples

Use these examples to match the local action style.

## DeleteUser

Use for simple model cleanup workflows.

```php
final class DeleteUser extends Action
{
    /**
     * Constructor.
     *
     */
    public function __construct(
        public User $user,
    ) {}

    /**
     * Execute the action.
     *
     */
    public function handle() : void
    {
        Storage::disk('public')->delete($this->user->avatar());

        $this->user->tokens()->delete();

        $this->user->memberships()->delete();

        $this->user->delete();
    }
}
```

## DeleteOrganization

Use for external API failure checks before destructive local cleanup.

```php
public function handle() : void
{
    $response = new DeleteStripeCustomer($this->organization)();

    if ($response->failed()) {
        $this->abort(424, 'Unable to delete the organization.');
    }

    Storage::disk('public')->deleteDirectory("organizations/{$this->organization->id}");

    Storage::disk('private')->deleteDirectory("organizations/{$this->organization->id}");

    $this->organization->delete();
}
```

## PurchaseCredits

Use for multi-step workflows that need locking, helper methods, and transactional persistence.

```php
#[AtomicLock(lockFor : 45, waitFor : 30)]
final class PurchaseCredits extends Action
{
    /**
     * Execute the action.
     *
     */
    public function handle() : void
    {
        $invoice = $this->createStripeInvoice();

        $this->createStripeInvoiceItem($invoice);

        $response = $this->payStripeInvoice($invoice);

        DB::transaction(fn() : null => $this->finalizePurchase($response));
    }

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
