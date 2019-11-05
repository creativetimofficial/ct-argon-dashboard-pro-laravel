# Argon Pro Theme for Laravel Framework 5.5 and Up

![version](https://img.shields.io/badge/version-1.2.1-blue.svg) ![license](https://img.shields.io/badge/license-MIT-blue.svg) [![GitHub issues open](https://img.shields.io/github/issues/creativetimofficial/ct-argon-dashboard-pro-laravel.svg?maxAge=2592000)](https://github.com/creativetimofficial/ct-argon-dashboard-pro-laravel/issues?q=is%3Aopen+is%3Aissue) [![GitHub issues closed](https://img.shields.io/github/issues-closed-raw/creativetimofficial/ct-argon-dashboard-pro-laravel/ct-argon-dashboard-pro-laravel.svg?maxAge=2592000)](https://github.com/creativetimofficial/ct-argon-dashboard-pro-laravel/issues?q=is%3Aissue+is%3Aclosed)

*Frontend version*: Argon Dashboard v1.0.0. More info at https://www.creative-tim.com/product/argon-dashboard-pro

![Product Image](https://s3.amazonaws.com/creativetim_bucket/products/146/original/opt_adp_laravel_thumbnail.jpg?1550851795)

## Prerequisites

If you don't already have an Apache local environment with PHP and MySQL, use one of the following links:

- Windows: https://updivision.com/blog/post/beginner-s-guide-to-setting-up-your-local-development-environment-on-windows
- Linux: https://howtoubuntu.org/how-to-install-lamp-on-ubuntu
- Mac: https://wpshout.com/quick-guides/how-to-install-mamp-on-your-mac

You will also need to install Composer: https://getcomposer.org/doc/00-intro.md


## Installation

1. Unzip the downloaded archive
2. Copy and paste **argon-master** folder in your **projects** folder. Rename the folder to your project's name
3. In your terminal run `composer install`
4. Copy `.env.example` to `.env` and updated the configurations (mainly the database configuration)
5. In your terminal run `php artisan key:generate`
6. Run `php artisan migrate --seed` to create the database tables and seed the roles and users tables
7. Run `php artisan storage:link` to create the storage symlink (if you are using **Vagrant** with **Homestead** for development, remember to ssh into your virtual machine and run the command from there).

## Usage

To start testing the Pro theme, register as a user or log in using one of the default users: 

- admin type - **admin@argon.com** with the password **secret**
- creator type - **creator@argon.com** with the password **secret** 
- member type - **member@argon.com** with the password **secret** 

Make sure to run the migrations and seeders for the above credentials to be available.

In addition to the features included in the free preset, the Pro theme also has a role management example with an updated user management, as well as tag management, category management and item management examples. All the necessary files (controllers, requests, views) are installed out of the box and all the needed routes are added to `routes/web.php`. Keep in mind that all the features can be viewed once you log in using the credentials provided above or by registering your own user.

Each role has a different privilege level and can perform a certain number of actions according to this level.  

A **member type** user can log in, update his profile and view a list of added items.  
A **creator type** user can log in, update his profile and perform actions on categories, tags and items.  
A **admin type** user can log in, update his profile and perform actions on categories, tags, items, roles and users.  

### Dashboard

You can access the dashboard either by using the "**Dashboards/Dashboard**" link in the left sidebar or by adding **/home** in the URL.

### Profile edit

You have the option to edit the current logged in user's profile information (name, email, profile picture) and password. To access this page, just click the "**Examples/Profile**" link in the left sidebar or add **/profile** in the URL.

The `App\Http\Controllers\ProfileController` handles the update of the user information and password.

```
public function update(ProfileRequest $request)
{
    auth()->user()->update(
        $request->merge(['picture' => $request->photo ? $request->photo->store('profile', 'public') : null])
            ->except([$request->hasFile('photo') ? '' : 'picture'])
    );

    return back()->withStatus(__('Profile successfully updated.'));
}

/**
* Change the password
*
* @param  \App\Http\Requests\PasswordRequest  $request
* @return \Illuminate\Http\RedirectResponse
*/
public function password(PasswordRequest $request)
{
   auth()->user()->update(['password' => Hash::make($request->get('password'))]);

   return back()->withPasswordStatus(__('Password successfully updated.'));
}
```

If you input the wrong data when editing the profile, don`t worry. Validation rules have been added to prevent this (see `App\Http\Requests\ProfileRequest`). If you try to change the password, you will see that additional validation rules have been added in `App\Http\Requests\PasswordRequest`. You also have  a custom validation rule that can be found in `App\Rules\CurrentPasswordCheckRule`.

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

The Pro theme allows you to add user roles. By default, the theme comes with **Admin**, **Creator** and **Member** roles. To access the role management example click the "**Examples/Role Management**" link in the left sidebar or add **/role** to the URL. Here you can add/edit new roles. Deletion is not allowed in this section.
To add a new role, click the "**Add role**" button. To edit an existing role, click the dotted menu (available on every table row) and then click "**Edit**". In both cases, you will be directed to a form which allows you to modify the name and description of a role.

The policy which authorizes the user to access the role management option is implemented in `App\Policies\RolePolicy.php`.``

You can find this functionality in `App\Http\RoleController.php`.

Also, validation rules were added so you will know exactly what to input in the form fields (see `App\Http\Requests\RoleRequest`). Note that these validation rules also apply for the role edit option.
```
public function rules()
{
    return [
        'name' => [
            'required', 'min:3', Rule::unique((new Role)->getTable())->ignore($this->route()->role->id ?? null)
        ],
        'description' => [
            'nullable', 'min:5'
        ]
    ];
}
```

### User management

The theme comes with an out of the box user management option. To access this option ,click the "**Examples/User Management**" link in the left sidebar or add **/user** to the URL.
The first thing you will see is a list of existing users. You can add new ones by clicking the "**Add user**" button (above the table on the right). On the Add user page, you will find a form which allows you to fill out the user`s name, email, role and password. All pages are generated using blade templates:

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

Validation rules were added to prevent errors in the form fields (see `App\Http\Requests\UserRequest`). Note that these validation rules also apply for the user edit option.

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

/**
* Get the validation attributes that apply to the request.
*
* @return array
*/
public function attributes()
{
    return [
        'role_id' => 'role',
    ];
}
```

The policy which authorizes the user to access the user management pages is implemented in `App\Policies\UserPolicy.php`.``

Once you add more users, the list will grow and for every user you will have edit and delete options (access these options by clicking the three dotted menu that appears at the end of every row).

All the sample code for the user management can be found in `App\Http\Controllers\UserController`. See store method example bellow:

```
public function store(UserRequest $request, User $model)
{
    $model->create($request->merge([
        'picture' => $request->photo ? $request->photo->store('profile', 'public') : null,
        'password' => Hash::make($request->get('password'))
    ])->all());

    return redirect()->route('user.index')->withStatus(__('User successfully created.'));
}
```

### Tag management

The Pro theme also comes with a tag management option. To access this example, click the "**Examples/Tag Management**" link in the left sidebar or add **/tag** to the URL.
In this section you can add and edit tags, but you can only delete them if they are not attached to any items. You can add new ones by clicking the "**Add tag**" button (above the table on the right). You will then be directed to a form which allows you to add new tags.
Although you can add in the form only the name and color of a tag, you can write your own migrations to extend this functionality.

```
public function destroy(Tag $tag)
{
    if (!$tag->items->isEmpty()) {
        return redirect()->route('tag.index')->withErrors(__('This tag has items attached and can\'t be deleted.'));
    }

    $tag->delete();

    return redirect()->route('tag.index')->withStatus(__('Tag successfully deleted.'));
}
```

The policy which authorizes the user to access tags management pages is implemented in `App\Policies\TagPolicy.php`.

### Category management

Out of the box you will have an example of category management (for the cases in which you are developing a blog or a shop). To access this example, click the "**Examples/Category Management**" link in the left sidebar or add **/category** to the URL.
You can add and edit categories here, but you can only delete them if they are not attached to any items.

```
@if ($category->items->isEmpty() && auth()->user()->can('delete', $category))
    <form action="{{ route('category.destroy', $category) }}" method="post">
        @csrf
        @method('delete')
        <button type="button" class="dropdown-item" onclick="confirm('{{ __("Are you sure you want to delete this category?") }}') ? this.parentElement.submit() : ''">
            {{ __('Delete') }}
        </button>
    </form>
@endif
```

The policy which authorizes the user on categories management pages is implemented in `App\Policies\CategoryPolicy.php`.

### Item management

Item management is the most advanced example included in the Pro theme, because every item has a picture, belongs to a category and has multiple tags. To access this example click the "**Examples/Item Management**" link in the left sidebar or add **/item** to the URL.
Here you can manage the items. A list of items will appear once you start adding them (to access the add page click "**Add item**").
On the add page, besides the Name and Description fields (which are present in most of the CRUD examples) you can see a category dropdown, which contains the categories you added, a file input and a tag multi select. If you did not add any categories or tags, please go to the corresponding sections (category management, tag management) and add some.  


The code for saving a new item is a bit different than before (see snippet bellow):

```
public function store(ItemRequest $request, Item $model)
{
    $item = $model->create($request->merge([
        'picture' => $request->photo->store('pictures', 'public'),
        'show_on_homepage' => $request->show_on_homepage ? 1 : 0,
        'options' => $request->options ? $request->options : null,
        'date' => $request->date ? Carbon::parse($request->date)->format('Y-m-d') : null
    ])->all());

    $item->tags()->sync($request->get('tags'));

    return redirect()->route('item.index')->withStatus(__('Item successfully created.'));
}
```

Notice how the picture and tags are easily saved in the database and on the disk.

Similar to all the examples included in the theme, this one also has validation rules in place. Note that the picture is mandatory only on the item creation. On the edit page, if no new picture is added, the old picture will not be modified.

```
public function rules()
{
    return [
        'name' => [
            'required', 'min:3', Rule::unique((new Item)->getTable())->ignore($this->route()->item->id ?? null)
        ],
        'category_id' => [
            'required', 'exists:'.(new Category)->getTable().',id'
        ],
        'excerpt' => [
            'required'
        ],
        'description' => [
            'required'
        ],
        'photo' => [
            $this->route()->item ? 'nullable' : 'required', 'image'
        ],
        'tags' => [
            'required'
        ],
        'tags.*' => [
            'exists:'.(new Tag)->getTable().',id'
        ],
        'status' => [
            'required',
            'in:published,draft,archive'
        ],
        'date' => [
            'required',
            'date_format:d-m-Y'
        ]
    ];
}
```

In the item management we have an **observer** (`app\Observers\ItemObserver`) example. This observer handles the deletion of the picture from the disk when the item is deleted or when the picture is changed via the edit form. The **observer** also removes the association between the item and the tags.

```
public function deleting(Item  $item)
{
    File::delete(storage_path("/app/public/{$item->picture}"));
    
    $item->tags()->detach();
}
```

The policy which authorizes the user on item management pages is implemented in `App\Policies\ItemPolicy.php`.

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
