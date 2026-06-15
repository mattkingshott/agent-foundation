# Examples

This file contains examples of Laravel route files and can be used as a guide.

## Complete File Structure

```php
<?php

declare(strict_types=1);

use Illuminate\Support\Facades\Route;
use App\Controllers\Operations\TaskController;
use App\Controllers\Operations\ContactController;
use App\Controllers\Operations\ListingController;

/**
 * Routes.
 *
 */
Route::get('/operations/{organization}/contacts', [ContactController::class, 'index'])->name('operations.contacts');
Route::post('/operations/{organization}/contacts', [ContactController::class, 'store'])->name('operations.contacts.store');
Route::get('/operations/{organization}/contacts/create', [ContactController::class, 'create'])->name('operations.contacts.create');
Route::get('/operations/{organization}/contacts/{contact}', [ContactController::class, 'edit'])->name('operations.contacts.edit');
Route::patch('/operations/{organization}/contacts/{contact}', [ContactController::class, 'update'])->name('operations.contacts.update');
Route::delete('/operations/{organization}/contacts/{contact}', [ContactController::class, 'delete'])->name('operations.contacts.delete');
```

## Standard Resource Routes (Organization-Scoped)

```php
// Index - Note: No .index suffix!
Route::get('/operations/{organization}/contacts', [ContactController::class, 'index'])->name('operations.contacts');

// Create form
Route::get('/operations/{organization}/contacts/create', [ContactController::class, 'create'])->name('operations.contacts.create');

// Store
Route::post('/operations/{organization}/contacts', [ContactController::class, 'store'])->name('operations.contacts.store');

// Edit form
Route::get('/operations/{organization}/contacts/{contact}', [ContactController::class, 'edit'])->name('operations.contacts.edit');

// Update
Route::patch('/operations/{organization}/contacts/{contact}', [ContactController::class, 'update'])->name('operations.contacts.update');

// Delete (not destroy!)
Route::delete('/operations/{organization}/contacts/{contact}', [ContactController::class, 'delete'])->name('operations.contacts.delete');
```

## Custom Actions

```php
// Custom GET action
Route::get('/operations/{organization}/contacts/{contact}/match', [ContactController::class, 'match'])->name('operations.contacts.match');

// Custom POST action
Route::post('/operations/{organization}/contacts/parse', [ContactController::class, 'parse'])->name('operations.contacts.parse');

// Custom PATCH action
Route::patch('/operations/{organization}/tasks/{task}/complete', [TaskController::class, 'complete'])->name('operations.tasks.complete');
```

## Nested Resources

```php
// Nested resource - parent listing, child photos
Route::get('/operations/{organization}/listings/{listing}/photos', [PhotoController::class, 'show'])->name('operations.photos.show');
Route::post('/operations/{organization}/listings/{listing}/photos', [PhotoController::class, 'store'])->name('operations.photos.store');
Route::patch('/operations/{organization}/listings/{listing}/photos', [PhotoController::class, 'sort'])->name('operations.photos.sort');
Route::patch('/operations/{organization}/listings/{listing}/photos/{photo}', [PhotoController::class, 'update'])->name('operations.photos.update');
Route::delete('/operations/{organization}/listings/{listing}/photos/{photo}', [PhotoController::class, 'delete'])->name('operations.photos.delete');

// Nested resource - parent listing, child offers
Route::get('/operations/{organization}/listings/{listing}/offers', [OfferController::class, 'show'])->name('operations.offers.show');
Route::get('/operations/{organization}/listings/{listing}/offers/create', [OfferController::class, 'create'])->name('operations.offers.create');
Route::post('/operations/{organization}/listings/{listing}/offers', [OfferController::class, 'store'])->name('operations.offers.store');
Route::get('/operations/{organization}/listings/{listing}/offers/{offer}', [OfferController::class, 'edit'])->name('operations.offers.edit');
Route::patch('/operations/{organization}/listings/{listing}/offers/{offer}', [OfferController::class, 'update'])->name('operations.offers.update');
Route::delete('/operations/{organization}/listings/{listing}/offers/{offer}', [OfferController::class, 'delete'])->name('operations.offers.delete');
```

## Account Routes (User-Scoped, No Organization)

```php
Route::get('/account/profile', [ProfileController::class, 'show'])->name('account.profile.show');
Route::patch('/account/profile', [ProfileController::class, 'update'])->name('account.profile.update');
```

## Routes with Middleware Modifiers

```php
Route::get('/organizations/register', [RegisterController::class, 'show'])->name('organizations.register.show')->withoutMiddleware('member:manager');
Route::post('/organizations/register', [RegisterController::class, 'store'])->name('organizations.register.store')->withoutMiddleware('member:manager');
```

## Report Routes (Non-RESTful)

```php
Route::get('/operations/{organization}/reports/activity', [ReportController::class, 'activity'])->name('operations.reports.activity');
Route::get('/operations/{organization}/reports/performance', [ReportController::class, 'performance'])->name('operations.reports.performance');
```

## Import Ordering (Alphabetically by Length)

```php
use Illuminate\Support\Facades\Route;
use App\Controllers\Operations\TaskController;
use App\Controllers\Operations\EventController;
use App\Controllers\Operations\OfferController;
use App\Controllers\Operations\PhotoController;
use App\Controllers\Operations\ReportController;
use App\Controllers\Operations\SearchController;
use App\Controllers\Operations\ContactController;
use App\Controllers\Operations\ListingController;
use App\Controllers\Operations\DashboardController;
```
