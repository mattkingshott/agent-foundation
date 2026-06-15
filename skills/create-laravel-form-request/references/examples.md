# Examples

This file contains examples of Laravel form request classes and can be used as a guide.

## Simple Store Request (with Credits Check)

```php
<?php

declare(strict_types=1);

namespace App\Requests\Operations\Contact;

use App\Types\FormRequest;
use App\Enumerations\Channel;
use App\Enumerations\Country;
use App\Enumerations\Interest;
use Illuminate\Validation\Rule;
use App\Enumerations\Profession;
use App\Rules\DistinctArray;

final class StoreRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     *
     */
    public function authorize() : bool
    {
        return ! $this->organization()->credits
            ? $this->abort('You do not have enough AI credits')
            : true;
    }

    /**
     * Retrieve the validation rules that apply to the request.
     *
     */
    public function rules() : array
    {
        $exists = Rule::exists('memberships', 'id')->where(
            column : 'organization_id',
            value  : $this->organization()->id,
        );

        $unique = Rule::unique('contacts', 'email')->where(
            column : 'organization_id',
            value  : $this->organization()->id,
        );

        return [
            'coordinator_id' => ['bail', 'required', 'string', 'uuid', $exists],
            'title'          => 'bail|required|string|min:2|max:4',
            'name'           => 'bail|required|string|min:2|max:50',
            'email'          => ['bail', 'required', 'string', 'min:6', 'max:100', 'email:rfc', $unique],
            'phone'          => 'bail|nullable|string|min:10|max:15',
            'company'        => 'bail|nullable|string|min:2|max:50',
            'website'        => 'bail|nullable|string|min:11|max:100|url',
            'line_1'         => 'bail|nullable|string|min:5|max:30',
            'line_2'         => 'bail|nullable|string|min:5|max:30',
            'city'           => 'bail|nullable|string|min:2|max:30',
            'region'         => 'bail|nullable|string|min:2|max:30',
            'post_code'      => 'bail|nullable|string|min:2|max:16',
            'country'        => ['bail', 'required', 'string', Rule::enum(Country::class)],
            'channel'        => ['bail', 'required', 'string', Rule::enum(Channel::class)],
            'interest'       => ['bail', 'required', 'string', Rule::enum(Interest::class)],
            'professions'    => 'bail|nullable|array',
            'professions.*'  => ['bail', 'nullable', new DistinctArray('professions'), 'string', Rule::enum(Profession::class)],
            'requirements'   => 'bail|nullable|string|min:50|max:500',
            'notes'          => 'bail|nullable|string|min:5|max:5000',
            'connect_at'     => 'bail|nullable|date',
        ];
    }
}
```

## Update Request (with Ownership Check + ignoreModel)

```php
<?php

declare(strict_types=1);

namespace App\Requests\Operations\Contact;

use App\Types\FormRequest;
use App\Enumerations\Channel;
use App\Enumerations\Country;
use App\Enumerations\Interest;
use Illuminate\Validation\Rule;
use App\Enumerations\Profession;
use App\Rules\DistinctArray;

final class UpdateRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     *
     */
    public function authorize() : bool
    {
        return ! $this->organization()->credits
            ? $this->abort('You do not have enough AI credits')
            : $this->organization()->owns($this->route('contact'));
    }

    /**
     * Retrieve the validation rules that apply to the request.
     *
     */
    public function rules() : array
    {
        $exists = Rule::exists('memberships', 'id')->where(
            column : 'organization_id',
            value  : $this->organization()->id,
        );

        $unique = Rule::unique('contacts', 'email')->where(
            column : 'organization_id',
            value  : $this->organization()->id,
        )->ignoreModel(
            $this->route('contact'),
        );

        return [
            'coordinator_id' => ['bail', 'required', 'string', 'uuid', $exists],
            'title'          => 'bail|required|string|min:2|max:4',
            'name'           => 'bail|required|string|min:2|max:50',
            'email'          => ['bail', 'required', 'string', 'min:6', 'max:100', 'email:rfc', $unique],
            'phone'          => 'bail|nullable|string|min:10|max:15',
            'company'        => 'bail|nullable|string|min:2|max:50',
            'website'        => 'bail|nullable|string|min:11|max:100|url',
            'line_1'         => 'bail|nullable|string|min:5|max:30',
            'line_2'         => 'bail|nullable|string|min:5|max:30',
            'city'           => 'bail|nullable|string|min:2|max:30',
            'region'         => 'bail|nullable|string|min:2|max:30',
            'post_code'      => 'bail|nullable|string|min:2|max:16',
            'country'        => ['bail', 'required', 'string', Rule::enum(Country::class)],
            'channel'        => ['bail', 'required', 'string', Rule::enum(Channel::class)],
            'interest'       => ['bail', 'required', 'string', Rule::enum(Interest::class)],
            'professions'    => 'bail|nullable|array',
            'professions.*'  => ['bail', 'nullable', new DistinctArray('professions'), 'string', Rule::enum(Profession::class)],
            'requirements'   => 'bail|nullable|string|min:50|max:500',
            'notes'          => 'bail|nullable|string|min:5|max:5000',
            'connect_at'     => 'bail|nullable|date',
        ];
    }
}
```

## Common Validation Patterns

### Pipe Format (Simple Rules)
```php
'title'      => 'bail|required|string|min:2|max:50',
'phone'      => 'bail|nullable|string|min:10|max:15',
'notes'      => 'bail|nullable|string|min:5|max:5000',
'due_at'     => 'bail|required|date',
'published'  => 'bail|required|boolean',
'bedrooms'   => 'bail|required|integer|min:0|max:1000',
'commission' => 'bail|required|decimal:0,2|min:0|max:100',
'website'    => 'bail|nullable|string|min:11|max:100|url',
```

### Array Format (Complex Rules or Alignment)
```php
'coordinator_id' => ['bail', 'required', 'string', 'uuid', $exists],
'email'          => ['bail', 'required', 'string', 'min:6', 'max:100', 'email:rfc', $unique],
'country'        => ['bail', 'required', 'string', Rule::enum(Country::class)],
'role'           => ['bail', 'required', 'string', Rule::enum(Role::class)],
```

### Array Validation
```php
'professions'   => 'bail|nullable|array',
'professions.*' => ['bail', 'nullable', new DistinctArray('professions'), 'string', Rule::enum(Profession::class)],

'parking'    => 'bail|required|array|min:1|max:13',
'parking.*'  => ['bail', 'required', new DistinctArray('parking'), 'string', Rule::enum(Parking::class)],
```

### Rule Variables (Before Return Statement)
```php
// Exists with organization scoping
$exists = Rule::exists('memberships', 'id')->where(
    column : 'organization_id',
    value  : $this->organization()->id,
);

// Unique with organization scoping
$unique = Rule::unique('contacts', 'email')->where(
    column : 'organization_id',
    value  : $this->organization()->id,
);

// Unique with ignoreModel (UpdateRequest only)
$unique = Rule::unique('contacts', 'email')->where(
    column : 'organization_id',
    value  : $this->organization()->id,
)->ignoreModel(
    $this->route('contact'),
);
```
