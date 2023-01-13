---
title: Adding an action to a Livewire component
---

## Setting up the Livewire component

You must use the `InteractsWithActions` trait and implement the `HasActions` interface on your Livewire component class:

```php
use Cable\Actions\Concerns\InteractsWithActions;
use Cable\Actions\Contracts\HasActions;
use Livewire\Component;

class ManagePost extends Component implements HasActions
{
    use InteractsWithActions;

    // ...
}
```

## Adding the action

Add a method that returns your action:

```php
use App\Models\Post;
use Cable\Actions\Action;
use Cable\Actions\Concerns\InteractsWithActions;
use Cable\Actions\Contracts\HasActions;
use Livewire\Component;

class ManagePost extends Component implements HasActions
{
    use InteractsWithActions;
    
    public Post $post;
    
    public function delete(): Action
    {
        return Action::make('delete')
            ->requiresConfirmation()
            ->action(fn () => $this->post->delete());
    }

    // ...
}
```

Finally, you need to render the action in your view. To do this, you can use `{{ $this->delete }}`, where you replace `delete` with the name of your action method:

```blade
<div>
    {{ $this->delete }}
    
    <x-cable-actions::modals />
</div>
```

You also need `<x-cable-actions::modals />` which injects the HTML required to render action modals. This only needs to be included within the Livewire component once, regardless of how many actions you have for that component.

## Action arguments

Sometimes, you may wish to pass arguments to your action. For example, if you're rendering the same action multiple times in the same view, but each time for a different model, you could pass the model ID as an argument, and then retrieve it later. To do this, you can invoke the action in your view and pass in the arguments as an array:

```php
<div>
    @foreach ($posts as $post)
        <h2>{{ $post->title }}</h2>
        
        {{ ($this->delete)(['post' => $post->id]) }}
    @endforeach
    
    <x-cable-actions::modals />
</div>
```

Now, you can access the post ID in your action method:

```php
use App\Models\Post;
use Cable\Actions\Action;

public function delete(): Action
{
    return Action::make('delete')
        ->requiresConfirmation()
        ->action(function (array $arguments) {
            $post = Post::find($arguments['post']);
            
            $post?->delete();
        });
}
```

## Grouping actions

You may group actions together into a dropdown menu by using the `<x-cable-actions::group>` Blade component, passing in the `actions` array as an attribute:

```blade
<div>
    <x-cable-actions::group :actions="[
        $this->edit,
        $this->view,
        $this->delete,
    ]" />
    
    <x-cable-actions::modals />
</div>
```

You can also pass in a `label`, `icon`, `color`, `size`, `tooltip`, and `dropdown-placement` to the group, as attributes:

```blade
<div>
    <x-cable-actions::group
        :actions="[
            $this->edit,
            $this->view,
            $this->delete,
        ]"
        label="Actions"
        icon="heroicon-m-ellipsis-vertical"
        color="primary"
        size="md"
        tooltip="More actions"
        dropdown-placement="bottom-end"
    />
    
    <x-cable-actions::modals />
</div>
```
