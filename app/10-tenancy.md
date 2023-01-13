---
title: Multi-tenancy
---

Multi-tenancy is a concept where a single instance of an application serves multiple customers. Each customer has their own data and access rules that prevent them from viewing or modifying each other's data. This is a common pattern in SaaS applications.

There are two main approaches to "ownership" in a multi-tenant application:

- Users own records they create. This is by far the simplest approach.
- Users belong to groups of users (often called teams or organizations). Records are owned by the group, and users can be members of multiple groups. This is a more complex approach, but is suitable for applications where users need to collaborate on data. More UI is required to support this approach, which is used to [register tenants](#tenant-registration). Often, you'll also want a page where users can manage the tenant and its members.

Cable supports both of these approaches. If users own records then the `App\Models\User` model is the "tenant model". If users belong to teams then the `App\Models\Team` model is the "tenant model". When reading this guide, we will refer to this as the "tenant model".

## Setting up tenancy

To set up tenancy, you'll need to specify the tenant model in the [configuration](configuration):

```php
use App\Models\Team;
use Cable\Context;

public function context(Context $context): Context
{
    return $context
        // ...
        ->tenant(Team::class);
}
```

If you're using the `App\Models\User` model as the tenant model, then you can pass this instead. Now, users will only be able to see and edit records in [resources](resources) that they own. No additional setup is required.

If you're using a different tenant model, you'll need to tell Cable which tenants a user belongs to. You can do this by implementing the `HasTenants` interface on the `App\Models\User` model:

```php
<?php

namespace App\Models;

use Cable\Context;
use Cable\Models\Contracts\CableUser;
use Cable\Models\Contracts\HasTenants;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Support\Collection;

class User extends Authenticatable implements CableUser, HasTenants
{
    // ...

    public function getTenants(Context $context): Collection
    {
        return $this->teams;
    }
    
    public function teams(): BelongsToMany
    {
        return $this->belongsToMany(Team::class);
    }
}
```

In this example, users belong to many teams, so there is a `teams()` relationship. The `getTenants()` method returns the teams that the user belongs to. Cable uses this to list the tenants that the user has access to.

You'll also want users to be able to [register new teams](#registration).

## Registration

A registration page will allow users to create a new tenant. This is useful if you're using a tenant model other than `App\Models\User`.

When visiting your app after logging in, users will be redirected to this page if they don't already have a tenant.

To set up a registration page, you'll need to create a new page class that extends `Cable\Pages\Tenancy\RegisterTenant`. This is a full-page Livewire component. You can put this anywhere you want, such as `app/Cable/Pages/Tenancy/RegisterTenant.php`:

```php
namespace App\Cable\Pages\Tenancy;

use Cable\Forms\Components\TextInput;
use Cable\Forms\Form;
use Cable\Pages\Tenancy\RegisterTenant;
use Illuminate\Database\Eloquent\Model;

class RegisterTeam extends RegisterTenant
{
    public static function getLabel(): string
    {
        return 'Register team';
    }
    
    public function form(Form $form): Form
    {
        return $form
            ->schema([
                TextInput::make('name'),
                // ...
            ]);
    }
    
    protected function handleRegistration(array $data): Team
    {
        $team = Team::create($data);
        
        $team->members()->attach(auth()->user());
        
        return $team;
    }
}
```

You may add any [form components](../forms/getting-started) to the `form()` method, and create the team inside the `handleRegistration()` method.

Now, we need to tell Cable to use this page. We can do this in the [configuration](configuration):

```php
use App\Cable\Pages\Tenancy\RegisterTeam;
use Cable\Context;

public function context(Context $context): Context
{
    return $context
        // ...
        ->tenantRegistration(RegisterTeam::class);
}
```

### Customizing the registration page

You can override anything you want on the registration page class to make it act as you want. Even the `$view` property can be overridden to use a custom view.

Alternatively, you can pass any route action to the `tenantRegistration()` method. That could be a callback function that gets executed when you visit the page, or the name of a controller, or a Livewire component - anything that works when using `Route::get()` in Laravel normally.

## Accessing the current tenant

Anywhere in the app, you can access the tenant model for the current request using `Cable::getTenant()`:

```php
$tenant = Cable::getTenant();
```

## Billing

### Using Laravel Spark

Cable provides a billing integration with [Laravel Spark](https://spark.laravel.com). Your users can start subscriptions and manage their billing information.

To install the integration, first [install Spark](https://spark.laravel.com/docs/2.x/installation.html) and configure it for your tenant model.

Now, you can install the Cable billing provider for Spark using Composer:

```bash
composer require cable/spark-billing-provider
```

In the [configuration](configuration), set Spark as the `tenantBillingProvider()`:

```php
use Cable\Context;
use Cable\Billing\Providers\SparkBillingProvider;

public function context(Context $context): Context
{
    return $context
        // ...
        ->tenantBillingProvider(SparkBillingProvider::class);
}
```

Now, you're all good to go! Users can manage their billing by clicking a link in the tenant menu.

### Requiring a subscription

To require a subscription to use any part of the app, you can use the `requiresTenantSubscription()` configuration method:

```php
use Cable\Context;

public function context(Context $context): Context
{
    return $context
        // ...
        ->requiresTenantSubscription();
}
```

Now, users will be redirected to the billing page if they don't have an active subscription.

#### Requiring a subscription for specific resources and pages

Sometimes, you may wish to only require a subscription for certain [resources](resources) and [pages](pages) in your app. You can do this by returning `true` from the `isTenantSubscriptionRequired()` method on the resource or page class:

```php
public static function isTenantSubscriptionRequired(Context $context): bool
{
    return true;
}
```

If you're using the `requiresTenantSubscription()` configuration method, then you can return `false` from this method to allow access to the resource or page as an exception.

### Writing a custom billing integration

Billing integrations are quite simple to write. You just need a class that implements the `Cable\Billing\Contracts\Provider` interface. This interface has two methods.

`getRouteAction()` is used to get the route action that should be run when the user visits the billing page. This could be a callback function, or the name of a controller, or a Livewire component - anything that works when using `Route::get()` in Laravel normally. For example, you could put in a simple redirect to your own billing page using a callback function.

`getSubscribedMiddleware()` returns the name of a middleware that should be used to check if the tenant has an active subscription. This middleware should redirect the user to the billing page if they don't have an active subscription.

Here's an example billing provider that uses a callback function for the route action and a middleware for the subscribed middleware:

```php
use App\Http\Middleware\RedirectIfUserNotSubscribed;
use Cable\Billing\Contracts\Provider;
use Illuminate\Http\RedirectResponse;

class ExampleBillingProvider implements Provider
{
    public function getRouteAction(): string
    {
        return function (): RedirectResponse {
            return redirect('https://billing.example.com');
        };
    }
    
    public function getSubscribedMiddleware(): string
    {
        return RedirectIfUserNotSubscribed::class;
    }
}
```

## Customizing the tenant menu

The tenant-switching menu is featured in the sidebar of the admin layout. It's fully customizable.

To register new items to the tenant menu, you can use the [configuration](configuration):

```php
use Cable\Context;
use Cable\Navigation\MenuItem;

public function context(Context $context): Context
{
    return $context
        // ...
        ->tenantMenuItems([
            MenuItem::make()
                ->label('Settings')
                ->url(route('cable.pages.settings'))
                ->icon('heroicon-m-cog-6-tooth'),
            // ...
        ]);
}
```

### Customizing the registration link

To customize the registration link on the tenant menu, register a new item with the `register` array key:

```php
use Cable\Context;
use Cable\Navigation\MenuItem;

public function context(Context $context): Context
{
    return $context
        // ...
        ->userMenuItems([
            'register' => MenuItem::make()->label('Register new team'),
            // ...
        ]);
}
```

### Customizing the billing link

To customize the billing link on the tenant menu, register a new item with the `billing` array key:

```php
use Cable\Context;
use Cable\Navigation\MenuItem;

public function context(Context $context): Context
{
    return $context
        // ...
        ->userMenuItems([
            'billing' => MenuItem::make()->label('Manage subscription'),
            // ...
        ]);
}
```

## Setting up avatars

Out of the box, Cable uses [ui-avatars.com](https://ui-avatars.com) to generate avatars based on a user's name. To provide your own avatar URLs, you can implement the `HasAvatar` contract:

```php
<?php

namespace App\Models;

use Cable\Models\Contracts\CableUser;
use Cable\Models\Contracts\HasAvatar;
use Illuminate\Database\Eloquent\Model;

class Team extends Model implements HasAvatar
{
    // ...

    public function getCableAvatarUrl(): ?string
    {
        return $this->avatar_url;
    }
}
```

The `getCableAvatarUrl()` method is used to retrieve the avatar of the current user. If `null` is returned from this method, Cable will fall back to [ui-avatars.com](https://ui-avatars.com).

You can easily swap out [ui-avatars.com](https://ui-avatars.com) for a different service, by creating a new avatar provider. [You can learn how to do this here.](users#using-a-different-avatar-provider)

## Configuring the ownership relationship

When creating a new [resource record](resources/creating-records), it will attempt to associate it with the current tenant. It will use the tenant's model to guess which relationship exists on the resource model. For example, if the tenant model is `App\Models\Team`, it will look for a `team()` relationship on the resource model. You can customize this relationship using the `ownershipRelationship` argument on the `tenant()` configuration method:

```php
use Cable\Context;

public function context(Context $context): Context
{
    return $context
        // ...
        ->tenant(Team::class, ownershipRelationship: 'owner');
}
```

Alternatively, you can override the `associateRecordWithTenant()` method on the [Create page class](resources/creating-records), which you can then use to customize how the record is associated with the tenant:

```php
use Cable\Pages\CreateRecord;

class CreatePost extends CreateRecord
{
    // ...
    
    protected function associateRecordWithTenant(Model $record, Model $tenant): void
    {
        $record->owner()->associate($tenant);
    }
}
```

## Configuring the slug attribute

When using a tenant like a team, you might want to add a slug field to the URL rather than the team's ID. You can do that with the `slugAttribute` argument on the `tenant()` configuration method:

```php
use Cable\Context;

public function context(Context $context): Context
{
    return $context
        // ...
        ->tenant(Team::class, slugAttribute: 'slug');
}
```

## Configuring the name attribute

By default, Cable will use Cable\Resources\Pages\CreateRecord;use the `name` attribute of the tenant to display its name in the app. To change this, you can implement the `HasName` contract:

```php
<?php

namespace App\Models;

use Cable\Models\Contracts\HasName;
use Illuminate\Database\Eloquent\Model;

class Team extends Model implements HasName
{
    // ...

    public function getCableName(): string
    {
        return "{$this->name} {$this->subscription_plan}";
    }
}
```

The `getCableName()` method is used to retrieve the name of the current user.

## Setting the current tenant label

Inside the tenant switcher on the sidebar, you may wish to add a small label like "Active team" above the name of the current team. You can do this by implementing the `HasCurrentTenantLabel` method on the tenant model:

```php
<?php

namespace App\Models;

use Cable\Models\Contracts\HasCurrentTenantLabel;
use Illuminate\Database\Eloquent\Model;

class Team extends Model implements HasCurrentTenantLabel
{
    // ...
    
    public function getCurrentTenantLabel(): string
    {
        return 'Active team';
    }
}
```

## Setting the default tenant

When signing in, Cable will redirect the user to the first tenant returned from the `getTenants()` method.

Sometimes, you might wish to change this. For example, you might store which team was last active, and redirect the user to that team instead.

To customize this, you can implement the `HasDefaultTenant` contract on the user:

```php
<?php

namespace App\Models;

use Cable\Models\Contracts\CableUser;
use Cable\Models\Contracts\HasDefaultTenant;
use Cable\Models\Contracts\HasTenants;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class User extends Model implements CableUser, HasDefaultTenant, HasTenants
{
    // ...
    
    public function getDefaultTenant(): ?Model
    {
        return $this->latestTeam;
    }
    
    public function latestTeam(): BelongsTo
    {
        return $this->belongsTo(Team::class, 'latest_team_id');
    }
}
```
