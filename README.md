# Argon Pro Theme For Laravel Framework 5.5 and Up

Argon Pro Theme For Laravel Framework 5.5 and Up

*Current version*: Argon Pro v1.0.0. More info at https://demos.creative-tim.com/argon-dashboard-pro/

## Installation

1. [Download the archive](https://github.com/laravel-frontend-presets/argon/archive/master.zip) of the repo and unzip it
2. Copy and paste **argon-master** folder in your **projects** folder. Rename the folder to your project's name
3. In your terminal run `composer install`
4. Copy `.env.example` to `.env` and updated the configurations (mainly the database configuration)
5. In your terminal run `php artisan key:generate`
6. Run `php artisan migrate --seed` to create the database tables and seed the roles and users tables
7. Run `php artisan storage:link` to create the storage symlink (if you are using **Vagrant** with **Homestead** for development remember to ssh in your virtual machine and run the command from there).

## Usage

Register a user or login using **admin@argon.com** and **secret** to start testing the pro theme (make sure to run the migrations and seeders for these credentials to be available).

Besides the features included in the free preset the pro theme also has a role management example with an updated user management, tag management, category management and an item management example. All the necessary files (controllers, requests, views) are installed out of the box and all the needed routes are added to `routes/web.php`. Keep in mind that all of the features can be viewed once you login using the credentials provided above or by registering your own user. 

### Dashboard

You can access the dashboard either by using the "**Dashboards/Dashboard**" link in the left sidebar or by adding **/home** in the URL. 

### Profile edit

You have the option to edit the current logged in user's profile (change name, email and password). To access this page just click the "**Examples/Profile**" link in the left sidebar or by adding **/profile** in the URL.

The `App\Htttp\Controlers\ProfileController` handles the update of the user information. 

```
public function update(ProfileRequest $request)
{
    auth()->user()->update($request->all());

    return back()->withStatus(__('Profile successfully updated.'));
}
```

Also you shouldn't worry about entering wrong data in the inputs when editing the profile, validation rules were added to prevent this (see `App\Http\Requests\ProfileRequest`). If you try to change the password you will see that other validation rules were added in `App\Http\Requests\PasswordRequest`. Notice that in this file you have a custom validation rule that can be found in `App\Rules\CurrentPasswordCheckRule`.

```
public function rules()
{
    return [
        'old_password' => ['required', 'min:6', new CurrentPasswordCheckRule],
        'password' => ['required', 'min:6', 'confirmed', 'different:old_password'],
        'password_confirmation' => ['required', 'min:6'],
    ];
}
```

### Role management

You have the ability to add user roles. By default the theme comes with **Admin** and **Employee** roles. To access the role management click the "**Examples/Role Management**" link in the left sidebar or add **/role** to the URL. Here you can add/edit new roles. Deletion is not allowed in this section. 
To add a new role click the "**Add role**" button and to edit an existing role click the dotted menu (available on every table line) and then click "**Edit**". In both cases you will be taken to a page that contains a form that allows you to modify the name and description of a role.

### User management

The theme comes with a user management option out of the box. To access this click the "**Examples/User Management**" link in the left sidebar or add **/user** to the URL.
The first thing you will see is the listing of the existing users. You can add new ones by clicking the "**Add user**" button (above the table on the right). On the Add user page you will see the form that allows you to do this. All pages are generate using blade templates:

```
<div class="form-group{{ $errors->has('name') ? ' has-danger' : '' }}">
    <label class="form-control-label" for="input-name">{{ __('Name') }}</label>
    <input type="text" name="name" id="input-name" class="form-control form-control-alternative{{ $errors->has('name') ? ' is-invalid' : '' }}" placeholder="{{ __('Name') }}" value="{{ old('name') }}" required autofocus>

    @if ($errors->has('name'))
        <span class="invalid-feedback" role="alert">
            <strong>{{ $errors->first('name') }}</strong>
        </span>
    @endif
</div>
```

Also validation rules were added so you will know exactly what to enter in the form fields (see `App\Http\Requests\UserRequest`). Note that these validation rules also apply for the user edit option.

```
public function rules()
{
    return [
        'name' => [
            'required', 'min:3'
        ],
        'email' => [
            'required', 'email', Rule::unique((new User)->getTable())->ignore($this->route()->user->id ?? null)
        ],
        'role_id' => [
            'required', 'exists:'.(new Role)->getTable().',id'
        ],
        'password' => [
            $this->route()->user ? 'nullable' : 'required', 'confirmed', 'min:6'
        ]
    ];
}
```

Once you add more users, the list will get bigger and for every user you will have edit and delete options (access these options by clicking the three dotted menu that appears at the end of every line). 

All the sample code for the user management can be found in `App\Http\Controllers\UserController`. See store method example bellow:

```
public function store(UserRequest $request, User $model)
{
    $model->create($request->merge(['password' => Hash::make($request->get('password'))])->all());

    return redirect()->route('user.index')->withStatus(__('User successfully created.'));
}
```

### Tag management

The pro theme also has tag management. To access this click the "**Examples/Tag Management**" link in the left sidebar or add **/tag** to the URL.
You can add and edit tags here, but you can only delete them if they are not attached to any items. You can add new ones by clicking the "**Add tag**" button (above the table on the right). On the Add tag page you will see the form that allows you to do this.
You can only add a tag name, but you can write your own migrations and extend this functionality.

```
public function destroy(Tag  $tag)
{
    if (!$tag->items->isEmpty()) {
        return redirect()->route('tag.index')->withErrors(__('This tag can\'t be deleted.'));
    }

    $tag->delete();

    return redirect()->route('tag.index')->withStatus(__('Tag successfully deleted.'));
}
```

### Category management

Out of the box you will have an example of category management (for the cases in which you are developing a blog or a shop). To access this click the "**Examples/Category Management**" link in the left sidebar or add **/category** to the URL.
You can add and edit categories here, but you can only delete them if they are not attached to any items. 

```
@if ($category->items->isEmpty())
    <form action="{{ route('category.destroy', $category) }}" method="post">
        @csrf
        @method('delete')
                                                            
        <button type="button" class="dropdown-item" onclick="confirm('{{ __("Are you sure you want to delete this category?") }}') ? this.parentElement.submit() : ''">
            {{ __('Delete') }}
        </button>
    </form>    
@endif
```

### Item management

The most advanced example that the pro theme has is the item management, because every item has a picture, belongs to a category and has multiple tags. To access this click the "**Examples/Item Management**" link in the left sidebar or add **/item** to the URL.
Here you can manage the items. A list of items will appear once you start adding them (to access the add page click "**Add item**").
On the add page, besides the name and description fields (which are present in most of the CRUD examples of the theme) you can see a category dropdown, which contains the categories that you added (if you did not add any categories please go to the category management and add some), a file input and a tag multi select (if you did not add any tags please go to the tag management and add some).

```
TODO: all blade code once the pro theme html is added
```

The code for saving a new item is a bit different than before (see snippet bellow):

```
public function store(ItemRequest $request, Item $model)
{
    $item = $model->create($request->merge(['picture' => $request->photo->store('pictures', 'public')])->all());

    $item->tags()->sync($request->get('tags'));
    
    return redirect()->route('item.index')->withStatus(__('Item successfully created.'));
}
```

Notice how the picture and tags are easily saved in the database and on the disk.

As all the presented examples, this one also has validation rules in place. Note that the picture is mandatory only on the item creation. On the edit page, if no new picture is added the old picture will not be modified.

```
public function rules()
{
    return [
        'category_id' => [
            'required', 'exists:'.(new Category)->getTable().',id'
        ],
        'name' => [
            'required', 'min:3', Rule::unique((new Item)->getTable())->ignore($this->route()->item->id ?? null)
        ],
        'description' => [
            'nullable', 'min:5'
        ],
        'photo' => [
            $this->route()->item ? 'nullable' : 'required', 'image'
        ],
        'tags' => [
            'required'
        ],
        'tags.*' => [
            'exists:'.(new Tag)->getTable().',id'
        ]
    ];
}
```

In the item management we have an **observer** (`app\Observers\ItemObserver`) example. This observer handles the deletion of the picture from the disk when the item is deleted or when the picture is changed via the edit form and also it removes the association between the item and the tags.

```
public function deleting(Item  $item)
{
    File::delete(storage_path("/app/public/{$item->picture}"));
    
    $item->tags()->detach();
}
```

## Credits

- [Creative Tim](https://creative-tim.com/)
- [Updivision](https://updivision.com)

## Screen shots

![Argon Login](/screens/Login.png)

![Argon Register ](/screens/Register.png)

![Argon Dashboard](/screens/Dashboard.png)

![Argon Roles](/screens/RoleManagement.png)

![Argon Users](/screens/UserManagement.png)

![Argon Profile](/screens/Profile.png)

![Argon Items](/screens/ItemManagement.png)

![Argon Item Create](/screens/ItemCreate.png)
