# Argon Pro Theme for Laravel Framework 8.x and Up

Argon Pro Theme for Laravel Framework 8.x and Up

*Current version*: Argon Pro v1.2.0. More info at http://argon-dashboard-pro-laravel.creative-tim.com/

![Product Image](https://raw.githubusercontent.com/creativetimofficial/public-assets/master/argon-dashboard-pro-laravel/intro.gif)

## Prerequisites

If you don't already have an Apache local environment with PHP and MySQL, use one of the following links:

- Windows: https://updivision.com/blog/post/beginner-s-guide-to-setting-up-your-local-development-environment-on-windows
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

## Documentation
The documentation for the Argon Dashboard Laravel is hosted at our [website](https://argon-dashboard-pro-laravel.creative-tim.com/docs/getting-started/overview.html?_ga=2.124985738.1023638542.1605859071-1600268259.1587021768).

## File Structure
```

├── app
│   ├── Category.php
│   ├── Console
│   │   └── Kernel.php
│   ├── Exceptions
│   │   └── Handler.php
│   ├── Http
│   │   ├── Controllers
│   │   │   ├── Auth
│   │   │   │   ├── ForgotPasswordController.php
│   │   │   │   ├── LoginController.php
│   │   │   │   ├── RegisterController.php
│   │   │   │   ├── ResetPasswordController.php
│   │   │   │   └── VerificationController.php
│   │   │   ├── CategoryController.php
│   │   │   ├── Controller.php
│   │   │   ├── HomeController.php
│   │   │   ├── ItemController.php
│   │   │   ├── PageController.php
│   │   │   ├── ProfileController.php
│   │   │   ├── RoleController.php
│   │   │   ├── TagController.php
│   │   │   └── UserController.php
│   │   ├── Kernel.php
│   │   ├── Middleware
│   │   │   ├── Authenticate.php
│   │   │   ├── CheckForMaintenanceMode.php
│   │   │   ├── EncryptCookies.php
│   │   │   ├── RedirectIfAuthenticated.php
│   │   │   ├── TrimStrings.php
│   │   │   ├── TrustProxies.php
│   │   │   └── VerifyCsrfToken.php
│   │   └── Requests
│   │       ├── CategoryRequest.php
│   │       ├── ItemRequest.php
│   │       ├── PasswordRequest.php
│   │       ├── ProfileRequest.php
│   │       ├── RoleRequest.php
│   │       ├── TagRequest.php
│   │       └── UserRequest.php
│   ├── Item.php
│   ├── Observers
│   │   ├── ItemObserver.php
│   │   └── UserObserver.php
│   ├── Policies
│   │   ├── CategoryPolicy.php
│   │   ├── ItemPolicy.php
│   │   ├── RolePolicy.php
│   │   ├── TagPolicy.php
│   │   └── UserPolicy.php
│   ├── Providers
│   │   ├── AppServiceProvider.php
│   │   ├── AuthServiceProvider.php
│   │   ├── BroadcastServiceProvider.php
│   │   ├── EventServiceProvider.php
│   │   └── RouteServiceProvider.php
│   ├── Role.php
│   ├── Rules
│   │   └── CurrentPasswordCheckRule.php
│   ├── Tag.php
│   └── User.php
├── artisan
├── bootstrap
│   ├── app.php
│   └── cache
│       ├── packages.php
│       └── services.php
├── CHANGELOG.md
├── composer.json
├── composer.lock
├── config
│   ├── app.php
│   ├── auth.php
│   ├── broadcasting.php
│   ├── cache.php
│   ├── database.php
│   ├── filesystems.php
│   ├── hashing.php
│   ├── items.php
│   ├── logging.php
│   ├── mail.php
│   ├── queue.php
│   ├── services.php
│   ├── session.php
│   └── view.php
├── database
│   ├── factories
│   │   └── UserFactory.php
│   ├── migrations
│   │   ├── 2014_10_12_100000_create_password_resets_table.php
│   │   ├── 2019_01_15_100000_create_roles_table.php
│   │   ├── 2019_01_15_110000_create_users_table.php
│   │   ├── 2019_01_17_121504_create_categories_table.php
│   │   ├── 2019_01_21_130422_create_tags_table.php
│   │   ├── 2019_01_21_163402_create_items_table.php
│   │   ├── 2019_01_21_163414_create_item_tag_table.php
│   │   ├── 2019_03_06_132557_add_photo_column_to_users_table.php
│   │   ├── 2019_03_06_143255_add_fields_to_items_table.php
│   │   └── 2019_03_20_090438_add_color_tags_table.php
│   └── seeds
│       ├── CategoriesTableSeeder.php
│       ├── DatabaseSeeder.php
│       ├── ItemsTableSeeder.php
│       ├── RolesTableSeeder.php
│       ├── TagsTableSeeder.php
│       └── UsersTableSeeder.php
├── newrelic.ini
├── package.json
├── phpunit.xml
├── Procfile
├── public
│   ├── argon
│   │   ├── css
│   │   │   ├── argon.css
│   │   │   └── argon.min.css
│   │   ├── fonts
│   │   │   └── nucleo
│   │   │       ├── nucleo-icons.eot
│   │   │       ├── nucleo-icons.svg
│   │   │       ├── nucleo-icons.ttf
│   │   │       ├── nucleo-icons.woff
│   │   │       └── nucleo-icons.woff2
│   │   ├── img
│   │   │   ├── brand
│   │   │   │   ├── blue.png
│   │   │   │   ├── favicon.png
│   │   │   │   └── white.png
│   │   │   ├── icons
│   │   │   │   ├── cards
│   │   │   │   │   ├── bitcoin.png
│   │   │   │   │   ├── mastercard.png
│   │   │   │   │   ├── paypal.png
│   │   │   │   │   └── visa.png
│   │   │   │   ├── common
│   │   │   │   │   ├── github.svg
│   │   │   │   │   └── google.svg
│   │   │   │   └── flags
│   │   │   │       ├── DE.png
│   │   │   │       ├── GB.png
│   │   │   │       └── US.png
│   │   │   └── theme
│   │   │       ├── angular.jpg
│   │   │       ├── bootstrap.jpg
│   │   │       ├── img-1-1000x600.jpg
│   │   │       ├── img-1-1000x900.jpg
│   │   │       ├── landing-1.png
│   │   │       ├── landing-2.png
│   │   │       ├── landing-3.png
│   │   │       ├── profile-cover.jpg
│   │   │       ├── react.jpg
│   │   │       ├── sketch.jpg
│   │   │       ├── team-1-800x800.jpg
│   │   │       ├── team-1.jpg
│   │   │       ├── team-2-800x800.jpg
│   │   │       ├── team-2.jpg
│   │   │       ├── team-3-800x800.jpg
│   │   │       ├── team-3.jpg
│   │   │       ├── team-4-800x800.jpg
│   │   │       ├── team-4.jpg
│   │   │       ├── team-5.jpg
│   │   │       └── vue.jpg
│   │   ├── js
│   │   │   ├── argon.js
│   │   │   ├── argon.min.js
│   │   │   ├── components
│   │   │   │   ├── charts
│   │   │   │   │   ├── chart-bars.js
│   │   │   │   │   ├── chart-bars.min.js
│   │   │   │   │   ├── chart-bar-stacked.js
│   │   │   │   │   ├── chart-bar-stacked.min.js
│   │   │   │   │   ├── chart-doughnut.js
│   │   │   │   │   ├── chart-doughnut.min.js
│   │   │   │   │   ├── chart-line.js
│   │   │   │   │   ├── chart-line.min.js
│   │   │   │   │   ├── chart-pie.js
│   │   │   │   │   ├── chart-pie.min.js
│   │   │   │   │   ├── chart-points.js
│   │   │   │   │   ├── chart-points.min.js
│   │   │   │   │   ├── chart-sales-dark.js
│   │   │   │   │   ├── chart-sales-dark.min.js
│   │   │   │   │   ├── chart-sales.js
│   │   │   │   │   └── chart-sales.min.js
│   │   │   │   ├── custom
│   │   │   │   │   ├── checklist.js
│   │   │   │   │   ├── checklist.min.js
│   │   │   │   │   ├── form-control.js
│   │   │   │   │   └── form-control.min.js
│   │   │   │   ├── init
│   │   │   │   │   ├── chart-init.js
│   │   │   │   │   ├── chart-init.min.js
│   │   │   │   │   ├── copy-icon.js
│   │   │   │   │   ├── copy-icon.min.js
│   │   │   │   │   ├── navbar.js
│   │   │   │   │   ├── navbar.min.js
│   │   │   │   │   ├── popover.js
│   │   │   │   │   ├── popover.min.js
│   │   │   │   │   ├── scroll-to.js
│   │   │   │   │   ├── scroll-to.min.js
│   │   │   │   │   ├── tooltip.js
│   │   │   │   │   └── tooltip.min.js
│   │   │   │   ├── layout.js
│   │   │   │   ├── layout.min.js
│   │   │   │   ├── license.js
│   │   │   │   ├── license.min.js
│   │   │   │   ├── maps
│   │   │   │   │   ├── maps-custom.js
│   │   │   │   │   ├── maps-custom.min.js
│   │   │   │   │   ├── maps-default.js
│   │   │   │   │   └── maps-default.min.js
│   │   │   │   └── vendor
│   │   │   │       ├── bootstrap-datepicker.js
│   │   │   │       ├── bootstrap-datepicker.min.js
│   │   │   │       ├── calendar.js
│   │   │   │       ├── calendar.min.js
│   │   │   │       ├── datatable.js
│   │   │   │       ├── datatable.min.js
│   │   │   │       ├── dropzone.js
│   │   │   │       ├── dropzone.min.js
│   │   │   │       ├── fullcalendar.js
│   │   │   │       ├── fullcalendar.min.js
│   │   │   │       ├── jvectormap.js
│   │   │   │       ├── jvectormap.min.js
│   │   │   │       ├── lavalamp.js
│   │   │   │       ├── lavalamp.min.js
│   │   │   │       ├── list.js
│   │   │   │       ├── list.min.js
│   │   │   │       ├── notify.js
│   │   │   │       ├── notify.min.js
│   │   │   │       ├── nouislider.js
│   │   │   │       ├── nouislider.min.js
│   │   │   │       ├── onscreen.js
│   │   │   │       ├── onscreen.min.js
│   │   │   │       ├── quill.js
│   │   │   │       ├── quill.min.js
│   │   │   │       ├── scrollbar.js
│   │   │   │       ├── scrollbar.min.js
│   │   │   │       ├── select2.js
│   │   │   │       ├── select2.min.js
│   │   │   │       ├── tags.js
│   │   │   │       └── tags.min.js
│   │   │   ├── demo.js
│   │   │   ├── demo.min.js
│   │   │   ├── items.js
│   │   │   └── vendor
│   │           └── jvectormap
│   │               ├── jquery-jvectormap-world-mill.js
│   │               ├── jquery-jvectormap-world-mill.min.js
│   │               └── README.md
│   ├── css
│   │   └── app.css
│   ├── docs
│   │   ├── assets
│   │   │   ├── css
│   │   │   │   └── argon.min.css
│   │   │   ├── fonts
│   │   │   │   └── nucleo
│   │   │   │       ├── nucleo-icons.eot
│   │   │   │       ├── nucleo-icons.svg
│   │   │   │       ├── nucleo-icons.ttf
│   │   │   │       ├── nucleo-icons.woff
│   │   │   │       └── nucleo-icons.woff2
│   │   │   ├── img
│   │   │   │   ├── brand
│   │   │   │   │   ├── blue.png
│   │   │   │   │   ├── favicon.png
│   │   │   │   │   └── white.png
│   │   │   │   ├── docs
│   │   │   │   │   └── getting-started
│   │   │   │   │       └── overview.svg
│   │   │   │   ├── icons
│   │   │   │   │   ├── cards
│   │   │   │   │   │   ├── bitcoin.png
│   │   │   │   │   │   ├── mastercard.png
│   │   │   │   │   │   ├── paypal.png
│   │   │   │   │   │   └── visa.png
│   │   │   │   │   ├── common
│   │   │   │   │   │   ├── github.svg
│   │   │   │   │   │   └── google.svg
│   │   │   │   │   └── flags
│   │   │   │   │       ├── DE.png
│   │   │   │   │       ├── FR.png
│   │   │   │   │       ├── GB.png
│   │   │   │   │       └── US.png
│   │   │   │   └── theme
│   │   │   │       ├── angular.jpg
│   │   │   │       ├── bootstrap.jpg
│   │   │   │       ├── img-1-1000x600.jpg
│   │   │   │       ├── img-1-1000x900.jpg
│   │   │   │       ├── img-1-1200x1000.jpg
│   │   │   │       ├── img-2-1200x1000.jpg
│   │   │   │       ├── landing-1.png
│   │   │   │       ├── landing-2.png
│   │   │   │       ├── landing-3.png
│   │   │   │       ├── profile-cover.jpg
│   │   │   │       ├── react.jpg
│   │   │   │       ├── sketch.jpg
│   │   │   │       ├── team-1.jpg
│   │   │   │       ├── team-2.jpg
│   │   │   │       ├── team-3.jpg
│   │   │   │       ├── team-4.jpg
│   │   │   │       ├── team-5.jpg
│   │   │   │       └── vue.jpg
│   │   │   ├── js
│   │   │   │   ├── argon.min.js
│   │   │   │   ├── demo.min.js
│   │   │   │   └── vendor
│   │   │   │       └── jvectormap
│   │   │   │           ├── jquery-jvectormap-world-mill.js
│   │   │   │           ├── jquery-jvectormap-world-mill.min.js
│   │   │   │           └── README.md
│   │   ├── components
│   │   │   ├── alerts.html
│   │   │   ├── avatar.html
│   │   │   ├── badge.html
│   │   │   ├── breadcrumb.html
│   │   │   ├── buttons.html
│   │   │   ├── card.html
│   │   │   ├── carousel.html
│   │   │   ├── collapse.html
│   │   │   ├── dropdowns.html
│   │   │   ├── forms.html
│   │   │   ├── input-group.html
│   │   │   ├── list-group.html
│   │   │   ├── modal.html
│   │   │   ├── navbar.html
│   │   │   ├── navs.html
│   │   │   ├── pagination.html
│   │   │   ├── popovers.html
│   │   │   ├── progress.html
│   │   │   ├── social-buttons.html
│   │   │   ├── tables.html
│   │   │   └── tooltips.html
│   │   ├── foundation
│   │   │   ├── colors.html
│   │   │   ├── grid.html
│   │   │   ├── icons.html
│   │   │   └── typography.html
│   │   ├── getting-started
│   │   │   ├── contents.html
│   │   │   ├── download.html
│   │   │   ├── installation.html
│   │   │   ├── license.html
│   │   │   └── overview.html
│   │   ├── laravel
│   │   │   ├── category-management.html
│   │   │   ├── item-management.html
│   │   │   ├── profile-edit.html
│   │   │   ├── role-management.html
│   │   │   ├── tag-management.html
│   │   │   └── user-management.html
│   │   └── plugins
│   │       ├── charts.html
│   │       ├── datatables.html
│   │       ├── datepicker.html
│   │       ├── dropdown.html
│   │       ├── dropzone.html
│   │       ├── fullcalendar.html
│   │       ├── headroom.html
│   │       ├── jvectormap.html
│   │       ├── list-js.html
│   │       ├── maps.html
│   │       ├── notify.html
│   │       ├── quill.html
│   │       ├── sliders.html
│   │       ├── sweet-alerts.html
│   │       └── tags.html
│   ├── favicon.ico
│   ├── index.php
│   ├── js
│   │   └── app.js
│   ├── pictures
│   │   ├── img1.jpg
│   │   ├── img2.jpg
│   │   └── img3.jpg
│   ├── robots.txt
│   └── svg
│       ├── 403.svg
│       ├── 404.svg
│       ├── 500.svg
│       └── 503.svg
├── README.md
├── resources
│   ├── assets
│   │   └── scss
│   │       ├── argon.scss
│   │       ├── bootstrap
│   │       │   ├── _alert.scss
│   │       │   ├── _badge.scss
│   │       │   ├── bootstrap-grid.scss
│   │       │   ├── bootstrap-reboot.scss
│   │       │   ├── bootstrap.scss
│   │       │   ├── _breadcrumb.scss
│   │       │   ├── _button-group.scss
│   │       │   ├── _buttons.scss
│   │       │   ├── _card.scss
│   │       │   ├── _carousel.scss
│   │       │   ├── _close.scss
│   │       │   ├── _code.scss
│   │       │   ├── _custom-forms.scss
│   │       │   ├── _dropdown.scss
│   │       │   ├── _forms.scss
│   │       │   ├── _functions.scss
│   │       │   ├── _grid.scss
│   │       │   ├── _images.scss
│   │       │   ├── _input-group.scss
│   │       │   ├── _jumbotron.scss
│   │       │   ├── _list-group.scss
│   │       │   ├── _media.scss
│   │       │   ├── mixins
│   │       │   │   ├── _alert.scss
│   │       │   │   ├── _background-variant.scss
│   │       │   │   ├── _badge.scss
│   │       │   │   ├── _border-radius.scss
│   │       │   │   ├── _box-shadow.scss
│   │       │   │   ├── _breakpoints.scss
│   │       │   │   ├── _buttons.scss
│   │       │   │   ├── _caret.scss
│   │       │   │   ├── _clearfix.scss
│   │       │   │   ├── _float.scss
│   │       │   │   ├── _forms.scss
│   │       │   │   ├── _gradients.scss
│   │       │   │   ├── _grid-framework.scss
│   │       │   │   ├── _grid.scss
│   │       │   │   ├── _hover.scss
│   │       │   │   ├── _image.scss
│   │       │   │   ├── _list-group.scss
│   │       │   │   ├── _lists.scss
│   │       │   │   ├── _nav-divider.scss
│   │       │   │   ├── _pagination.scss
│   │       │   │   ├── _reset-text.scss
│   │       │   │   ├── _resize.scss
│   │       │   │   ├── _screen-reader.scss
│   │       │   │   ├── _size.scss
│   │       │   │   ├── _table-row.scss
│   │       │   │   ├── _text-emphasis.scss
│   │       │   │   ├── _text-hide.scss
│   │       │   │   ├── _text-truncate.scss
│   │       │   │   ├── _transition.scss
│   │       │   │   └── _visibility.scss
│   │       │   ├── _mixins.scss
│   │       │   ├── _modal.scss
│   │       │   ├── _navbar.scss
│   │       │   ├── _nav.scss
│   │       │   ├── _pagination.scss
│   │       │   ├── _popover.scss
│   │       │   ├── _print.scss
│   │       │   ├── _progress.scss
│   │       │   ├── _reboot.scss
│   │       │   ├── _root.scss
│   │       │   ├── _tables.scss
│   │       │   ├── _tooltip.scss
│   │       │   ├── _transitions.scss
│   │       │   ├── _type.scss
│   │       │   ├── utilities
│   │       │   │   ├── _align.scss
│   │       │   │   ├── _background.scss
│   │       │   │   ├── _borders.scss
│   │       │   │   ├── _clearfix.scss
│   │       │   │   ├── _display.scss
│   │       │   │   ├── _embed.scss
│   │       │   │   ├── _flex.scss
│   │       │   │   ├── _float.scss
│   │       │   │   ├── _position.scss
│   │       │   │   ├── _screenreaders.scss
│   │       │   │   ├── _shadows.scss
│   │       │   │   ├── _sizing.scss
│   │       │   │   ├── _spacing.scss
│   │       │   │   ├── _text.scss
│   │       │   │   └── _visibility.scss
│   │       │   ├── _utilities.scss
│   │       │   └── _variables.scss
│   │       ├── core
│   │       │   ├── alerts
│   │       │   │   ├── _alert-dismissible.scss
│   │       │   │   ├── _alert-notify.scss
│   │       │   │   └── _alert.scss
│   │       │   ├── avatars
│   │       │   │   ├── _avatar-group.scss
│   │       │   │   └── _avatar.scss
│   │       │   ├── badges
│   │       │   │   ├── _badge-circle.scss
│   │       │   │   ├── _badge-dot.scss
│   │       │   │   ├── _badge-floating.scss
│   │       │   │   └── _badge.scss
│   │       │   ├── breadcrumbs
│   │       │   │   └── _breadcrumb.scss
│   │       │   ├── buttons
│   │       │   │   ├── _button-brand.scss
│   │       │   │   ├── _button-group.scss
│   │       │   │   ├── _button-icon.scss
│   │       │   │   └── _button.scss
│   │       │   ├── cards
│   │       │   │   ├── _card-animations.scss
│   │       │   │   ├── _card-blockquote.scss
│   │       │   │   ├── _card-money.scss
│   │       │   │   ├── _card-pricing.scss
│   │       │   │   ├── _card-profile.scss
│   │       │   │   ├── _card.scss
│   │       │   │   └── _card-stats.scss
│   │       │   ├── charts
│   │       │   │   └── _chart.scss
│   │       │   ├── close
│   │       │   │   └── _close.scss
│   │       │   ├── collapse
│   │       │   │   └── _accordion.scss
│   │       │   ├── content
│   │       │   │   └── _main-content.scss
│   │       │   ├── custom-forms
│   │       │   │   ├── _custom-checkbox.scss
│   │       │   │   ├── _custom-control.scss
│   │       │   │   ├── _custom-form.scss
│   │       │   │   ├── _custom-radio.scss
│   │       │   │   └── _custom-toggle.scss
│   │       │   ├── dropdowns
│   │       │   │   └── _dropdown.scss
│   │       │   ├── footers
│   │       │   │   └── _footer.scss
│   │       │   ├── forms
│   │       │   │   ├── _form-extend.scss
│   │       │   │   ├── _form.scss
│   │       │   │   ├── _form-validation.scss
│   │       │   │   └── _input-group.scss
│   │       │   ├── grid
│   │       │   │   └── _grid.scss
│   │       │   ├── headers
│   │       │   │   └── _header.scss
│   │       │   ├── icons
│   │       │   │   ├── _icon-actions.scss
│   │       │   │   ├── _icon.scss
│   │       │   │   └── _icon-shape.scss
│   │       │   ├── list-groups
│   │       │   │   ├── _list-check.scss
│   │       │   │   └── _list-group.scss
│   │       │   ├── maps
│   │       │   │   └── _map.scss
│   │       │   ├── masks
│   │       │   │   └── _mask.scss
│   │       │   ├── medias
│   │       │   │   ├── _media-comment.scss
│   │       │   │   └── _media.scss
│   │       │   ├── mixins
│   │       │   │   ├── _alert.scss
│   │       │   │   ├── _background-variant.scss
│   │       │   │   ├── _badge.scss
│   │       │   │   ├── _buttons.scss
│   │       │   │   ├── _custom-forms.scss
│   │       │   │   ├── _forms.scss
│   │       │   │   ├── _icon.scss
│   │       │   │   ├── _modals.scss
│   │       │   │   └── _popover.scss
│   │       │   ├── modals
│   │       │   │   └── _modal.scss
│   │       │   ├── navbars
│   │       │   │   ├── _navbar-collapse.scss
│   │       │   │   ├── _navbar-dropdown.scss
│   │       │   │   ├── _navbar-floating.scss
│   │       │   │   ├── _navbar.scss
│   │       │   │   ├── _navbar-search.scss
│   │       │   │   ├── _navbar-top.scss
│   │       │   │   └── _navbar-vertical.scss
│   │       │   ├── navs
│   │       │   │   ├── _nav-pills.scss
│   │       │   │   └── _nav.scss
│   │       │   ├── paginations
│   │       │   │   └── _pagination.scss
│   │       │   ├── popovers
│   │       │   │   └── _popover.scss
│   │       │   ├── progresses
│   │       │   │   └── _progress.scss
│   │       │   ├── reboot
│   │       │   │   └── _reboot.scss
│   │       │   ├── sections
│   │       │   │   └── _nucleo-icons.scss
│   │       │   ├── separators
│   │       │   │   └── _separator.scss
│   │       │   ├── shortcuts
│   │       │   │   └── _shortcut.scss
│   │       │   ├── tables
│   │       │   │   ├── _table-actions.scss
│   │       │   │   ├── _table.scss
│   │       │   │   └── _table-sortable.scss
│   │       │   ├── timeline
│   │       │   │   └── _timeline.scss
│   │       │   ├── type
│   │       │   │   ├── _article.scss
│   │       │   │   ├── _display.scss
│   │       │   │   ├── _heading.scss
│   │       │   │   └── _type.scss
│   │       │   ├── utilities
│   │       │   │   ├── _backgrounds.scss
│   │       │   │   ├── _blurable.scss
│   │       │   │   ├── _floating.scss
│   │       │   │   ├── _helper.scss
│   │       │   │   ├── _image.scss
│   │       │   │   ├── _opacity.scss
│   │       │   │   ├── _overflow.scss
│   │       │   │   ├── _position.scss
│   │       │   │   ├── _shadows.scss
│   │       │   │   ├── _sizing.scss
│   │       │   │   ├── _spacing.scss
│   │       │   │   ├── _text.scss
│   │       │   │   └── _transform.scss
│   │       │   └── vendors
│   │       │       ├── _bootstrap-datepicker.scss
│   │       │       ├── _bootstrap-tagsinput.scss
│   │       │       ├── _chartjs.scss
│   │       │       ├── _datatables.scss
│   │       │       ├── _dropzone.scss
│   │       │       ├── _fullcalendar.scss
│   │       │       ├── _headroom.scss
│   │       │       ├── _jvectormap.scss
│   │       │       ├── _lavalamp.scss
│   │       │       ├── _nouislider.scss
│   │       │       ├── _quill.scss
│   │       │       ├── _scrollbar.scss
│   │       │       ├── _select2.scss
│   │       │       └── _sweet-alert-2.scss
│   │       └── custom
│   │           ├── _components.scss
│   │           ├── _functions.scss
│   │           ├── _mixins.scss
│   │           ├── _utilities.scss
│   │           ├── _variables.scss
│   │           └── _vendors.scss
│   ├── js
│   │   ├── app.js
│   │   └── bootstrap.js
│   ├── lang
│   │   └── en
│   │       ├── auth.php
│   │       ├── pagination.php
│   │       ├── passwords.php
│   │       └── validation.php
│   └── views
│       ├── alerts
│       │   ├── errors.blade.php
│       │   ├── error_self_update.blade.php
│       │   ├── feedback.blade.php
│       │   ├── migrations_check.blade.php
│       │   └── success.blade.php
│       ├── auth
│       │   ├── login.blade.php
│       │   ├── passwords
│       │   │   ├── email.blade.php
│       │   │   └── reset.blade.php
│       │   ├── register.blade.php
│       │   └── verify.blade.php
│       ├── categories
│       │   ├── create.blade.php
│       │   ├── edit.blade.php
│       │   └── index.blade.php
│       ├── forms
│       │   └── header.blade.php
│       ├── items
│       │   ├── create.blade.php
│       │   ├── edit.blade.php
│       │   └── index.blade.php
│       ├── layouts
│       │   ├── app.blade.php
│       │   ├── footers
│       │   │   ├── auth.blade.php
│       │   │   ├── guest.blade.php
│       │   │   └── nav.blade.php
│       │   ├── headers
│       │   │   ├── auth.blade.php
│       │   │   ├── breadcrumbs.blade.php
│       │   │   ├── cards.blade.php
│       │   │   └── guest.blade.php
│       │   └── navbars
│       │       ├── navbar.blade.php
│       │       ├── navs
│       │       │   ├── auth.blade.php
│       │       │   └── guest.blade.php
│       │       └── sidebar.blade.php
│       ├── pages
│       │   ├── buttons.blade.php
│       │   ├── calendar.blade.php
│       │   ├── cards.blade.php
│       │   ├── charts.blade.php
│       │   ├── components.blade.php
│       │   ├── dashboard-alternative.blade.php
│       │   ├── dashboard.blade.php
│       │   ├── datatables.blade.php
│       │   ├── elements.blade.php
│       │   ├── googlemaps.blade.php
│       │   ├── grid.blade.php
│       │   ├── icons.blade.php
│       │   ├── lock.blade.php
│       │   ├── notifications.blade.php
│       │   ├── pricing.blade.php
│       │   ├── sortable.blade.php
│       │   ├── tables.blade.php
│       │   ├── timeline.blade.php
│       │   ├── typography.blade.php
│       │   ├── validation.blade.php
│       │   ├── vectormaps.blade.php
│       │   ├── welcome.blade.php
│       │   └── widgets.blade.php
│       ├── profile
│       │   └── edit.blade.php
│       ├── roles
│       │   ├── create.blade.php
│       │   ├── edit.blade.php
│       │   └── index.blade.php
│       ├── tags
│       │   ├── create.blade.php
│       │   ├── edit.blade.php
│       │   └── index.blade.php
│       └── users
│           ├── create.blade.php
│           ├── edit.blade.php
│           └── index.blade.php
├── routes
│   ├── api.php
│   ├── channels.php
│   ├── console.php
│   └── web.php
├── screens
├── server.php
├── storage
│   ├── app
│   │   └── public
│   ├── framework
│   │   ├── cache
│   │   │   └── data
│   │   ├── sessions
│   │   ├── testing
│   │   └── views
│   └── logs
├── tests
│   ├── CreatesApplication.php
│   ├── Feature
│   │   └── ExampleTest.php
│   ├── TestCase.php
│   └── Unit
│       └── ExampleTest.php
└── webpack.mix.js
```

## Browser Support

At present, we officially aim to support the last two versions of the following browsers:

<img src="https://github.com/creativetimofficial/public-assets/blob/master/logos/chrome-logo.png?raw=true" width="64" height="64"> <img src="https://raw.githubusercontent.com/creativetimofficial/public-assets/master/logos/firefox-logo.png" width="64" height="64"> <img src="https://raw.githubusercontent.com/creativetimofficial/public-assets/master/logos/edge-logo.png" width="64" height="64"> <img src="https://raw.githubusercontent.com/creativetimofficial/public-assets/master/logos/safari-logo.png" width="64" height="64"> <img src="https://raw.githubusercontent.com/creativetimofficial/public-assets/master/logos/opera-logo.png" width="64" height="64">

## Resources
- Demo: <https://argon-dashboard-pro-laravel.creative-tim.com/?ref=adl-readme>
- Download Page: <https://www.creative-tim.com/product/argon-dashboard-pro-laravel?ref=adl-readme>
- Documentation: <https://argon-dashboard-pro-laravel.creative-tim.com/docs/getting-started/overview.html?_ga=2.124985738.1023638542.1605859071-1600268259.1587021768>
- License Agreement: <https://www.creative-tim.com/license>
- Support: <https://www.creative-tim.com/contact-us>
- Issues: [Github Issues Page](https://github.com/creativetimofficial/ct-argon-dashboard-pro-laravel/issues)

## Change log

Please see the [changelog](CHANGELOG.md) for more information on what has changed recently.

## Credits

- [Creative Tim](https://creative-tim.com/)
- [Updivision](https://updivision.com)

## Reporting Issues

We use GitHub Issues as the official bug tracker for the Material Dashboard Laravel. Here are some advices for our users that want to report an issue:

1. Make sure that you are using the latest version of the Material Dashboard Laravel. Check the CHANGELOG from your dashboard on our [website](https://www.creative-tim.com/?ref=adl-readme).
2. Providing us reproducible steps for the issue will shorten the time it takes for it to be fixed.
3. Some issues may be browser specific, so specifying in what browser you encountered the issue might help.

## Licensing

- Copyright Creative Tim (https://www.creative-tim.com/?ref=adl-readme)
- Creative Tim License (https://www.creative-tim.com/license).

## Screen shots

![Argon Login](/screens/Login.png)

![Argon Register ](/screens/Register.png)

![Argon Dashboard](/screens/Dashboard.png)

![Argon Roles](/screens/RoleManagement.png)

![Argon Users](/screens/UserManagement.png)

![Argon Profile](/screens/Profile.png)

![Argon Items](/screens/ItemManagement.png)

![Argon Item Create](/screens/ItemCreate.png)

## Social Media

### Creative Tim:

Twitter: <https://twitter.com/CreativeTim?ref=adl-readme>

Facebook: <https://www.facebook.com/CreativeTim?ref=adl-readme>

Dribbble: <https://dribbble.com/creativetim?ref=adl-readme>

Instagram: <https://www.instagram.com/CreativeTimOfficial?ref=adl-readme>


### Updivision:

Twitter: <https://twitter.com/updivision?ref=adl-readme>

Facebook: <https://www.facebook.com/updivision?ref=adl-readme>

Linkedin: <https://www.linkedin.com/company/updivision?ref=adl-readme>

Updivision Blog: <https://updivision.com/blog/?ref=adl-readm
