# Facebook Provider for OAuth 2.0 Client

[![Build Status](https://travis-ci.org/thephpleague/oauth2-facebook.png?branch=master)](https://travis-ci.org/thephpleague/oauth2-facebook)
[![Latest Stable Version](https://poser.pugx.org/league/oauth2-facebook/v/stable.png)](https://packagist.org/packages/league/oauth2-facebook)

This package provides Facebook OAuth 2.0 support for the PHP League's [OAuth 2.0 Client](https://github.com/thephpleague/oauth2-client).

This package is compliant with [PSR-1][], [PSR-2][] and [PSR-4][]. If you notice compliance oversights, please send
a patch via pull request.

[PSR-1]: https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-1-basic-coding-standard.md
[PSR-2]: https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md
[PSR-4]: https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md


## Requirements

The following versions of PHP are supported.

* PHP 5.4
* PHP 5.5
* PHP 5.6
* HHVM

## Installation

Add the following to your `composer.json` file.

> **Note:** Once version 1.0 of the [OAuth 2.0 Client](https://github.com/thephpleague/oauth2-client) is released, you'll be able to install from composer without the `@dev` minimum stability flag.

```json
{
    "require": {
        "league/oauth2-facebook": "~0.0@dev"
    }
}
```

## Usage

### Authorization Code Flow

```php
$provider = new League\OAuth2\Client\Provider\Facebook([
    'clientId'          => '{facebook-app-id}',
    'clientSecret'      => '{facebook-app-secret}',
    'redirectUri'       => 'https://example.com/callback-url',
    'scopes'            => ['email', '...', '...'],
    'graphApiVersion'   => 'v2.2',
]);

if (!isset($_GET['code'])) {

    // If we don't have an authorization code then get one
    $authUrl = $provider->getAuthorizationUrl();
    $_SESSION['oauth2state'] = $provider->state;
    header('Location: '.$authUrl);
    exit;

// Check given state against previously stored one to mitigate CSRF attack
} elseif (empty($_GET['state']) || ($_GET['state'] !== $_SESSION['oauth2state'])) {

    unset($_SESSION['oauth2state']);
    exit('Invalid state');

} else {

    // Try to get an access token (using the authorization code grant)
    $token = $provider->getAccessToken('authorization_code', [
        'code' => $_GET['code']
    ]);

    // Optional: Now you have a token you can look up a users profile data
    try {

        // We got an access token, let's now get the user's details
        $userDetails = $provider->getUserDetails($token);

        // Use these details to create a new profile
        printf('Hello %s!', $userDetails->firstName);

    } catch (Exception $e) {

        // Failed to get user details
        exit('Oh dear...');
    }

    // Use this to interact with an API on the users behalf
    echo $token->accessToken;

    // Number of seconds until the access token will expire, and need refreshing
    echo $token->expires;
}
```

### Graph API Version

You should not rely on the default fallback Graph version defined in this package. You should specify a Graph version when you instantiate the `Facebook` provider.

```php
$provider = new League\OAuth2\Client\Provider\Facebook([
    /* . . . */
    'graphApiVersion'   => 'v2.2',
]);
```

This is best so that you have full control over when your app upgrades to the next version of Graph. Otherwise your app might break when the fallback Graph version is updated in this package.

See the [Graph API version schedule](https://developers.facebook.com/docs/apps/changelog) for more info.

### Refreshing a Token

Facebook does not support refreshing tokens. In order to get a new "refreshed" token, you must send the user through the login-with-Facebook process again.

From the [Facebook documentation](https://developers.facebook.com/docs/facebook-login/access-tokens#extending):

> Once [the access tokens] expire, your app must send the user through the login flow again to generate a new short-lived token.

The following code will throw a `League\OAuth2\Client\Exception\FacebookProviderException`.

```php
$grant = new \League\OAuth2\Client\Grant\RefreshToken();
$token = $provider->getAccessToken($grant, ['refresh_token' => $refreshToken]);
```

## Testing

``` bash
$ ./vendor/bin/phpunit
```

## Contributing

Please see [CONTRIBUTING](https://github.com/thephpleague/oauth2-facebook/blob/master/CONTRIBUTING.md) for details.


## Credits

- [Sammy Kaye Powers](https://github.com/SammyK)
- [All Contributors](https://github.com/thephpleague/oauth2-facebook/contributors)


## License

The MIT License (MIT). Please see [License File](https://github.com/thephpleague/oauth2-facebook/blob/master/LICENSE) for more information.
