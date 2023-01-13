---
title: Placeholder
---

Placeholders can be used to render text-only "fields" within your forms. Each placeholder has `content()`, which cannot be changed by the user.

```php
use Cable\Forms\Components\Placeholder;

Placeholder::make('Label')
    ->content('Content, displayed underneath the label')
```

## Rendering HTML inside the placeholder

You may even render custom HTML within placeholder content:

```php
use Cable\Forms\Components\Placeholder;
use Illuminate\Support\HtmlString;

Placeholder::make('Documentation')
    ->content(new HtmlString('<a href="https://jmpst.art/cable/docs">jmpst.art/cable</a>'))
```
