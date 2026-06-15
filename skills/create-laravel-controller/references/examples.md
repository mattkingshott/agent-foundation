# Examples

This file contains examples of Laravel controllers and can be used as a guide. Make sure to include docblocks for each method.

## Full Resource Controller

```php
<?php

declare(strict_types=1);

namespace App\Controllers\Operations;

use App\Models\Contact;
use App\Resources\ContactResource;
use App\Requests\Operations\Contact\IndexRequest;
use App\Requests\Operations\Contact\StoreRequest;
use App\Requests\Operations\Contact\UpdateRequest;
use App\Requests\Operations\Contact\DeleteRequest;
use App\Types\Controller;
use Illuminate\Http\Resources\Json\AnonymousResourceCollection;
use Illuminate\Http\JsonResponse;

final class ContactController extends Controller
{
    /**
     * Delete the existing contact.
     *
     */
    public function delete(DeleteRequest $request, Contact $contact) : RedirectResponse
    {
        $contact->delete();

        return $this->json(status : 204);
    }

    /**
     * Retrieve the organization's contacts.
     *
     */
    public function index(IndexRequest $request) : AnonymousResourceCollection
    {
        $contacts = $request->organization()->contacts()
            ->orderBy('created_at', 'desc')
            ->paginate(20);

        return ContactResource::collection($contacts);
    }

    /**
     * Store the new contact.
     *
     */
    public function store(StoreRequest $request) : ContactResource
    {
        $contact = $request->organization()->contacts()->create($request->validated()));

        return new ContactResource($contact);
    }

    /**
     * Revise the existing contact.
     *
     */
    public function update(UpdateRequest $request, Contact $contact) : JsonResponse
    {
        $contact->update($request->validated());

        return $this->json(status : 204);
    }
}
```

## Import Order Template

```php
<?php

declare(strict_types=1);

namespace App\Controllers\{Domain};

// Model imports (ordered by length)
use App\Models\{Model};
use App\Models\Organization;

// Request imports (ordered by length)
use App\Requests\{Domain}\{Resource}\IndexRequest;
use App\Requests\{Domain}\{Resource}\StoreRequest;
use App\Requests\{Domain}\{Resource}\UpdateRequest;
use App\Requests\{Domain}\{Resource}\DeleteRequest;

// Illuminate facades
use Illuminate\Support\Facades\App;

// Illuminate types
use Illuminate\Http\JsonResponse;

// Base controller
use App\Types\Controller;

final class {Resource}Controller extends Controller
{
    // Methods...
}
```
