---
title: Restore action
---

## Overview

Cable includes a prebuilt action that is able to restore [soft deleted](https://laravel.com/docs/eloquent#soft-deleting) Eloquent records. When the trigger button is clicked, a modal asking the user for confirmation. You may use it like so:

```php
use Cable\Actions\RestoreAction;

RestoreAction::make()
    ->record($this->post)
```

If you want to restore table rows, you can use the `Cable\Tables\Actions\RestoreAction` instead, or `Cable\Tables\Actions\RestoreBulkAction` to restore more than one at once:

```php
use Cable\Tables\Actions\RestoreAction;
use Cable\Tables\Actions\RestoreBulkAction;
use Cable\Tables\Table;

public function table(Table $table): Table
{
    return $table
        ->actions([
            RestoreAction::make(),
            // ...
        ])
        ->bulkActions([
            RestoreBulkAction::make(),
            // ...
        ]);
}
```

## Redirecting after restoring

You may set up a custom redirect when the form is submitted using the `successRedirectUrl()` method:

```php
RestoreAction::make()
    ->successRedirectUrl(route('posts.list'))
```

## Customizing the restore notification

When the record is successfully restored, a notification is dispatched to the user, which indicates the success of their action.

To customize the title of this notification, use the `successNotificationTitle()` method:

```php
RestoreAction::make()
    ->successNotificationTitle('User restored')
```

You may customize the entire notification using the `successNotification()` method:

```php
use Cable\Notifications\Notification;

RestoreAction::make()
    ->successNotification(
       Notification::make()
            ->success()
            ->title('User restored')
            ->body('The user has been restored successfully.'),
    )
```

To disable the notification altogether, use the `successNotification(null)` method:

```php
RestoreAction::make()
    ->successNotification(null)
```

## Lifecycle hooks

You can use the `before()` and `after()` methods to execute code before and after a record is restored:

```php
RestoreAction::make()
    ->before(function () {
        // ...
    })
    ->after(function () {
        // ...
    })
```
