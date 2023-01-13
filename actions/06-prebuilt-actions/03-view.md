---
title: View action
---

## Overview

Cable includes a prebuilt action that is able to view Eloquent records. When the trigger button is clicked, a modal will open with information inside. Cable uses form fields to structure this information. All form fields are disabled, so they are not editable by the user. You may use it like so:

```php
use Cable\Actions\ViewAction;
use Cable\Forms\Components\TextInput;

ViewAction::make()
    ->record($this->post)
    ->form([
        TextInput::make('title')
            ->required()
            ->maxLength(255),
        // ...
    ])
```

If you want to view table rows, you can use the `Cable\Tables\Actions\ViewAction` instead:

```php
use Cable\Forms\Components\TextInput;
use Cable\Tables\Actions\ViewAction;
use Cable\Tables\Table;

public function table(Table $table): Table
{
    return $table
        ->actions([
            ViewAction::make()
                ->form([
                    TextInput::make('title')
                        ->required()
                        ->maxLength(255),
                    // ...
                ]),
        ]);
}
```

## Customizing data before filling the form

You may wish to modify the data from a record before it is filled into the form. To do this, you may use the `mutateRecordDataUsing()` method to modify the `$data` array, and return the modified version before it is filled into the form:

```php
ViewAction::make()
    ->mutateRecordDataUsing(function (array $data): array {
        $data['user_id'] = auth()->id();

        return $data;
    })
```
