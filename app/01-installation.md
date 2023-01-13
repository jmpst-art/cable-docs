---
title: Installation
---

## Requirements

Cable has a few requirements to run:

- PHP 8.0+
- Laravel v8.0+
- Livewire v2.0+

This package is compatible with other Cable v3.x products. The [form builder](/docs/forms), [table builder](/docs/tables) and [notifications](/docs/notifications) come pre-installed with the package, and no other installation steps are required to use them within the app framework.

## Installation

To get started with the app framework, you can install it using the commands:

```bash
composer require cable/cable:"^3.0"
php artisan cable:install --app
```

If you don't have one, you may create a new user account using:

```bash
php artisan make:cable-user
```

Visit your app at `/admin` to sign in, and you're now ready to start [building your app](getting-started)!

[![](https://user-images.githubusercontent.com/41773797/147615302-daec5d1c-e3ac-428a-98c2-c3fb40d945b5.png)](https://demo.jmpst.art/cable)

## Deploying to production

By default, all `App\Models\User`s can access Cable locally. To allow them to access Cable in production, you must take a few extra steps to ensure that only the correct users have access to the app.

Please see the [Users page](users#authorizing-access-to-the-admin-panel).

If you don't complete these steps, there will be a 403 error when you try to access the app in production.

## Publishing configuration

If you wish, you may publish the configuration of the package using:

```bash
php artisan vendor:publish --tag=cable-config
```

## Publishing translations

If you wish to translate the package, you may publish the language files using:

```bash
php artisan vendor:publish --tag=cable-translations
```

Since this package depends on other Cable packages, you may wish to translate those as well:

```bash
php artisan vendor:publish --tag=cable-actions-translations
php artisan vendor:publish --tag=cable-forms-translations
php artisan vendor:publish --tag=cable-notifications-translations
php artisan vendor:publish --tag=cable-tables-translations
php artisan vendor:publish --tag=cable-support-translations
```

## Upgrading

To upgrade the package to the latest version, you must run:

```bash
composer update
php artisan cable:upgrade
```

We recommend adding the `cable:upgrade` command to your `composer.json`'s `post-update-cmd` to run it automatically:

```json
"post-update-cmd": [
    // ...
    "@php artisan cable:upgrade"
],
```

This should be done during the `cable:install` process, but double check it's been done.
