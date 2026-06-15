# Code Style Guide - Laravel Blade Rules

All generated Laravel Blade code must match these patterns to ensure consistency and maintainability.

---

## Directives
- `@use()` for imports at top
- `@props([])` for component properties with defaults
- `@php` blocks sparingly, prefer component logic
- `@error()` for validation errors

## Properties & Attributes
- `:$variable` shorthand when passing same-named variable
- `:variable="$value"` for expressions
- `variable="string"` for literals
- `data-id` attributes for Dusk selectors (snake_case)
- Order attributes by line length, first attribute on same line as element, subsequent attributes on separate lines, last attribute ends with '>', e.g.
```html
<x-portal :$organization
          class="max-w-800px"
          heading="Page Title"
          title="Page Title - {{ $organization->name }}">
```

## Naming
- Components: kebab-case (`<x-text-box>`)
- Slots: colon prefix (`<x-slot:left>`)
- IDs/data attributes: snake_case

## Spacing
- Empty line between template sections
- No empty lines within tag attributes
- Single space around mustaches: `{{ $var }}`

## Alpine.js

**State & Methods:**
- Use `x-cloak` for stateful components
- State properties before methods
- Method names: camelCase
- Use `x-init` for initialization/watchers

**Event Handling:**
```blade
x-on:click="open = ! open"
x-on:click.prevent.stop="method()"
x-on:input.debounce.500ms.stop="search($event.target.value)"
x-on:keyup.escape="open = false"
x-on:custom_event.window="handler($event.detail)"
```

**Modifier Order:**
1. `.prevent`, `.stop`
2. `.debounce.NNNms`
3. `.window`, `.document`, `.outside`

**Browser Helper (Turbo):**
```blade
x-on:click.stop="Browser.open('{{ URL::route('...') }}')"
x-on:click.stop="Browser.redirect('{{ URL::route('...') }}')"
```
- Always use `.stop` to prevent bubbling