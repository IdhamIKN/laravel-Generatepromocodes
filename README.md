# laravel-Generatepromocodes
Generate promo codes using Laravel

[![Packagist](https://img.shields.io/packagist/v/zgabievi/promocodes.svg)](https://packagist.org/packages/zgabievi/promocodes)
[![Packagist](https://img.shields.io/packagist/dt/zgabievi/promocodes.svg)](https://packagist.org/packages/zgabievi/promocodes)
[![license](https://img.shields.io/github/license/zgabievi/promocodes.svg)](https://packagist.org/packages/zgabievi/promocodes)

Coupons and promotional codes generator for [Laravel](https://laravel.com). Current release is only
for [Laravel 9.x](https://laravel.com/docs/9.x) and [PHP 8.1](https://www.php.net/releases/8.1/en.php). It's completely
rewritten, and if you are using previous version, you should change your code accordingly. Code is simplified now and it
should take you several minutes to completely rewrite usage.


## Installation

You can install the package via composer:

```bash
composer require zgabievi/laravel-promocodes
```

## Configuration

```bash
php artisan vendor:publish --provider="Zorb\Promocodes\PromocodesServiceProvider"
```

Now you can change configurations as you need:

```php
return [
    'models' => [
        'promocodes' => [
            'model' => \Zorb\Promocodes\Models\Promocode::class,
            'table_name' => 'promocodes',
            'foreign_id' => 'promocode_id',
        ],

        'users' => [
            'model' => \App\Models\User::class,
            'table_name' => 'users',
            'foreign_id' => 'user_id',
        ],

        'pivot' => [
            'model' => \Zorb\Promocodes\Models\PromocodeUser::class,
            'table_name' => 'promocode_user',
        ],
    ],
    'code_mask' => '****-****',
    'allowed_symbols' => 'ABCDEFGHIJKLMNOPQRSTUVWXYZ123456789',
];
```

After you configure this file, run migrations:

```bash
php artisan migrate
```

Now you will need to use AppliesPromocode on your user model.

```php
namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Zorb\Promocodes\Traits\AppliesPromocode;

class User extends Authenticatable {
    use AppliesPromocode;

    //
}```

### Using class

Combine methods as you need. You can skip any method that you don't need, most of them already have default values.

```php
use Zorb\Promocodes\Facades\Promocodes;

Promocodes::mask('AA-***-BB') // default: config('promocodes.code_mask')
          ->characters('ABCDE12345') // default: config('promocodes.allowed_symbols')
          ->multiUse() // default: false
          ->unlimited() // default: false
          ->boundToUser() // default: false
          ->user(User::find(1)) // default: null
          ->count(5) // default: 1
          ->usages(5) // default: 1
          ->expiration(now()->addYear()) // default: null
          ->details([ 'discount' => 50 ]) // default: []
          ->create();
```
### Using helper

There is a global helper function which will do the same as promocodes class. You can use named arguments magic from php
8.1.

```php
createPromocodes(
    mask: 'AA-***-BB', // default: config('promocodes.code_mask')
    characters: 'ABCDE12345', // default: config('promocodes.allowed_symbols')
    multiUse: true, // default: false
    unlimited: true, // default: false
    boundToUser: true, // default: false
    user: User::find(1), // default: null
    count: 5, // default: 1
    usages: 5, // default: 1
    expiration: now()->addYear(), // default: null
    details: [ 'discount' => 50 ] // default: []
);
```
### Using command

There is also the command for creating promocodes. Parameters are optional here too.

```bash
php artisan promocodes:create\
  --mask="AA-***-BB"\
  --characters="ABCDE12345"\
  --multi-use\
  --unlimited\
  --bound-to-user\
  --user=1\
  --count=5\
  --usages=5\
  --expiration="2022-01-01 00:00:00"
```

### Generating Promocodes

If you want to output promocodes and not save them to database, you can call generate method instead of create.

```php
use Zorb\Promocodes\Facades\Promocodes;

Promocodes::mask('AA-***-BB') // default: config('promocodes.code_mask')
          ->characters('ABCDE12345') // default: config('promocodes.allowed_symbols')
          ->multiUse() // default: false
          ->unlimited() // default: false
          ->boundToUser() // default: false
          ->user(User::find(1)) // default: null
          ->count(5) // default: 1
          ->usages(5) // default: 1
          ->expiration(now()->addYear()) // default: null
          ->details([ 'discount' => 50 ]) // default: []
          ->generate();
```

### Using class

Combine methods as you need. You can skip any method that you don't need.

```php
use Zorb\Promocodes\Facades\Promocodes;

Promocodes::code('ABC-DEF')
          ->user(User::find(1)) // default: null
          ->apply();
```

### Using helper

There is a global helper function which will do the same as promocodes class.

```php
applyPomocode(
    'ABC-DEF',
    User::find(1) // default: null
);
```

### Using command

There is also the command for applying promocode.

```bash
php artisan promocodes:apply ABC-DEF --user=1
```

#### Exceptions

While trying to apply promocode, you should be aware of exceptions. Most part of the code throws exceptions, when there is a problem:

```php
// Zorb\Promocodes\Exceptions\*

PromocodeAlreadyUsedByUserException - "The given code `ABC-DEF` is already used by user with id 1."
PromocodeBoundToOtherUserException - "The given code `ABC-DEF` is bound to other user, not user with id 1."
PromocodeDoesNotExistException - "The given code `ABC-DEF` doesn't exist." | "The code was not event provided."
PromocodeExpiredException - "The given code `ABC-DEF` already expired."
PromocodeNoUsagesLeftException - "The given code `ABC-DEF` has no usages left."
UserHasNoAppliesPromocodeTrait - "The given user model doesn't have AppliesPromocode trait."
UserRequiredToAcceptPromocode - "The given code `ABC-DEF` requires to be used by user, not by guest."
```

#### Events

There are two events which are fired upon applying.

```php
// Zorb\Promocodes\Events\*

GuestAppliedPromocode // Fired when guest applies promocode
    // It has public variable: promocode

UserAppliedPromocode // Fired when user applies promocode
    // It has public variable: promocode
    // It has public variable: user
```


### Using helper

There is a global helper function which will expire promocode.

```php
expirePromocode('ABC-DEF');
```

### Using command

There is also the command for expiring promocode.

```bash
php artisan promocodes:expire ABC-DEF
```

## Trait Methods

If you added AppliesPromocode trait to your user model, you will have some additional methods on user.

```php
$user = User::find(1);

$user->appliedPromocodes // Returns promocodes applied by user
$user->boundPromocodes // Returns promocodes bound to user
$user->applyPromocode('ABC-DEF') // Applies promocode to user
```

## Additional Methods

```php
Promocodes::all(); // To retrieve all (available/not available) promocodes
Promocodes::available(); // To retrieve valid (available) promocodes
Promocodes::notAvailable(); // To retrieve invalid (not available) promocodes
```
## Testing

```bash
composer test
```

<h2 id="[dukungan](https://saweria.co/idhamIKN)">💌 [Support Me]</h2>

<p>
You can support me on the Saweria.co platform! Your support will mean a lot. However, with you giving a <i>star</i> to this <i>project</i>, it's really enough~!
</p>

<a href="https://saweria.co/idhamIKN" target="_blank"><img id="wse-buttons-preview" src="💌 [Support Me]" height="40" style="border:0px;height:40px;" alt="💌 [Support Me]" ></a>

<h2 id="kontribusi">🤝 Contributing</h2>

<p>
<i>Contributions, issues and feature requests</i> really appreciated because this application is far from perfect. Feel free to make <i>pull requests</i> and make changes to this <i>project</i>, okay!
</p>

<h2 id="lisensi">📝 License</h2>

<p>laravel-Generatepromocodes is open-sourced software licensed under the MIT license.</p>

<h2 id="pembuat">🧍 Author</h2>

<p>laravel-Generatepromocodes  created by <a href="https://instagram.com/idhamikn?igshid=MmJiY2I4NDBkZg==">IdhamIKn</a> .</p>

