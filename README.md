# Laravel 5.3 install with Facebook login #
This is a clean Laravel 5.3 login installation with Facebook login

## Project description ##
This started as a project to explore the PHP Framework Laravel. I wanted to build an application with login and registration possibilities in Laravel 5. After playing around for a bit I also get interested in using the Facebook login possibility. So this documentation is a step-by-step explanation what I have done to accomplish this. 

## Composer ##
For this project, Composer is needed. To install this, you can find the instructions here: https://getcomposer.org/

## Install Laravel 5.3 ##
To install a clean installation of Laravel I use the following Composer command in the Terminal:

`composer create-project --prefer-dist laravel/laravel app`

## Setup Local Environment ##
After installing Laravel I set up a local environment. I used VirualHostX, this is a very easy to use application where you can set up your Local Environment.

## Make Authentication ##
Because a lot of application need a user login, Laravel made it easy to achieve this. I used the following command in the Terminal to add some extra files that handle the authentication for me:

`php artisan make:auth`

This command installs a layout, registration and login view. It also added all the routes to the authentication endpoints and a `HomeContoller` to handle post-login requests to the application's dashboard.

## Set up Database ##
To store the user information I had to setup a database. For database management I use Querious and I created a new database. After that is in the `.env` file I added the database credentials to connect Laravel to the database.  

To create the right tables in the database Laravel uses migration files. When I used the authentication command Laravel created a migration file for a user's table with the following code:

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateUsersTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->string('email')->unique();
            $table->string('password');
            $table->rememberToken();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('users');
    }
}
```

For now, I leave it like this and create tables by running the following command:

`php artisan migrate`

Now the first part of the application is ready, and I can register and login in the application. 

## Install Socialite ##
Now I want to add the Facebook login functionality to the application. To achieve that I use the Socialite package, this makes building authentication with social networks simple.

I use composer to install the Socialite package by running te following command:

`composer require laravel/socialite`

After that, I have to configure the Socialite package in the application. The configuration files for the application are stored in the `config` folder in the root of the application. For this configuration, I need to add the provider to the `app.php` file.

```php
'providers' => [
    // Other service providers...

    Laravel\Socialite\SocialiteServiceProvider::class,
],
```

In this same file I also have to and an alias. 

```php
'Socialite' => Laravel\Socialite\Facades\Socialite::class,
```

Now the Socialite package is added to the application and is ready for use.

## Create Facebook App 
For the login functionality of Facebook, I need to create a Facebook App. To create this app I need to be a Facebook Developer. By following these steps you can easily become a Facebook developer(http://www.wikihow.com/Become-a-Facebook-Developer).

To create a Facebook App I go to the Facebook developers page(developers.facebook.com). In the right top of the page, I can create an app by clicking on `Add a New App`. The next step is to fill in a name for the app and select a category.

On the next page, there is a menu on the right. On the dashboard, I can find all the data to configure Socialite. 

To configure the Facebook app to the Socialite package I need add some credentials to `service.php` in the `config` folder of the root of the application:

```php
'facebook' => [
        'client_id' => env('FACEBOOK_CLIENT_ID'),
        'client_secret' => env('FACEBOOK_CLIENT_SECRET'),
        'redirect' => env('CALLBACK_URL'),
    ],
```

For security resons it is better to store the Facebook app credentials in the `.env`. So i add the credentials to my local .env file:

```php
FACEBOOK_CLIENT_ID=1838479676323487
FACEBOOK_CLIENT_SECRET=246abd8fae9034c27abbbcb34df2f234
CALLBACK_URL=
```

Now the Facebook app is added to my Laravel application. 

## Create a SocialController ##
If I want to use Facebook login I have to make sure that my application can do the following things:
* Redirect my users to Facebook
* Handle the callback from Facebook

To achieve that I need to use a controller. I can create a controller by using the following command in the terminal:

`php artisan make:controller SocialAuthController`

This controller can be found here: `app/Http/Controllers/SocialAuthController.php`

I added two methods to the controller:
* Redirect
* Callback

In the head of the file, I also added to Socialite package because I'm gonna use that for handling redirects and callbacks. These results in de following code:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

use App\Http\Requests;
use App\Http\Controllers\Controller;
use Socialite;

class SocialAuthController extends Controller
{
    public function redirect()
    {
        return Socialite::driver('facebook')->redirect();   
    }   

    public function callback()
    {
        // facebook callback with token   
    }
}
```

## Add routes to the application ##
The next step is to add two URL routes to the `web.php` file that is stored in the `routes` folder in the root of the application.

```php
Route::get('/facebook/redirect', 'SocialAuthController@redirect');
Route::get('/facebook/callback', 'SocialAuthController@callback');
```

After that, I have to add the callback URL in the `.env` file.

```php
CALLBACK_URL=http://login.dev/facebook/callback
```

Now I have to go back to the Facebook App page and add the application URL to the app. This can be done on the settings page by clicking `add platform` and select `website`.

The Facebook App is now successfully added to my Laravel application.

## Add the Facebook login to the login page ##
Now I need to add a link on the login page to give users the possibility login with Facebook. 

So I add some HTML code to `login.blade.php` that can be found in `resources/views/auth/`.

```html
<a href="{{ url('/facebook/redirect') }}">FB Login</a>
```

I placed this code right after the forgot password option in the code. By using the code between the curly brackets Laravel create the right link for me. 

So now the link is added to the login screen and can I test the login redirect. By clicking the link I get the familiar login screen from Facebook, this means the redirect is working right now!

## Database migration ##
Now the redirect is successful I want to save the user information in the database. For that, I have to change two things in the migration file of the user table. If the user login in the application with his Facebook account he doesn't have to fill an email or password. I have to set the database settings for `email` and `password` to nullable.

```php
$table->string('email')->unique()->nullable();
$table->string('password')->nullable();
```

The next step is to create a new table for the social account. I make a table for this so if I want to add another social login in the future this is easy to implement. I run the following command to create the migration file:

`php artisan make:migration create_social_accounts_table --create="social_accounts"`

In the migration file I added the following fields:

```php
Schema::create('social_accounts', function (Blueprint $table) {
        $table->integer('user_id');
        $table->string('provider_user_id');
        $table->string('provider');
        $table->timestamps();
    });
```

After that I run the migration refresh commando in the terminal:

`php artisan migrate:refresh`

## Create SocialAccount model
I also have to create a new model for the social accounts. So the database relations are set up correctly.
 
`php artisan make:model SocialAccount`

I added the following to `SocialAccount` model file in the `/app` directory:

```php 
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class SocialAccount extends Model
{
    protected $fillable = ['user_id', 'provider_user_id', 'provider'];

    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```

Now the database relations are set up correctly!

## Register and Login handling ##
Now I have to create some handling service that will try to register the user or login if the user already exists. I create a new file: `SocialAccountService.php` in the `app` folder. 

To achieve that I added the following code:

```php
<?php

namespace App;

use Laravel\Socialite\Contracts\User as ProviderUser;

class SocialAccountService
{
    public function createOrGetUser(ProviderUser $providerUser)
    {
        $account = SocialAccount::whereProvider('facebook')
            ->whereProviderUserId($providerUser->getId())
            ->first();

        if ($account) {
            return $account->user;
        } else {

            $account = new SocialAccount([
                'provider_user_id' => $providerUser->getId(),
                'provider' => 'facebook'
            ]);

            $user = User::whereEmail($providerUser->getEmail())->first();

            if (!$user) {

                $user = User::create([
                    'email' => $providerUser->getEmail(),
                    'name' => $providerUser->getName(),
                ]);
            }

            $account->user()->associate($user);
            $account->save();

            return $user;

        }

    }
}
```

This code will try to find the social account in the database and if it is not present it will create a new user. This method will also try to associate the social account with the email address in case the user already has an account. 

Now everything is ready to handle the Facebook callback to my app. 

## Facebook callback handling ##
Now everything is ready I only have to update the callback method in the `SocialAuthController`. I'm using Socialite to handle the Facebook callback: 

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

use App\Http\Requests;
use App\Http\Controllers\Controller;
use App\SocialAccountService;
use Socialite;

class SocialAuthController extends Controller
{
    public function redirect()
    {
        return Socialite::driver('facebook')->redirect();   
    }   

    public function callback(SocialAccountService $service)
    {
        $user = $service->createOrGetUser(Socialite::driver('facebook')->user());

        auth()->login($user);

        return redirect()->to('/home');
    }
}
```

Now the application is ready! When I click on FB Login on the login screen I can successfully login with my Facebook account!

## Sources ##

[Larvel 5.3 documentation](https://laravel.com/docs/5.3/installation) 

[Socialite documentation](https://github.com/laravel/socialite)




 




















