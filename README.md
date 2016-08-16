# laravel-jwt-authentication
A PHP project using the Laravel framework and JWT (JSON Web Tokens) to create an authentication API with E-mail verification for mobile apps.

The steps below are a compilation of a series of tutorials.

<ul>  
  <li><a href="#tutorial1">Tutorial 1: Install jwt-auth</a></li>
  <li><a href="#tutorial2">Tutorial 2: API Routes Setup</a></li>
  <li><a href="#tutorial3">Tutorial 3: Prepare Database</a></li>
  <li><a href="#tutorial4">Tutorial 4: Authenticating Users</a></li>
  <li><a href="#tutorial5">Tutorial 5: Email Templates</a></li>
  <li><a href="#tutorial6">Testing</a></li>
</ul>

<a name="tutorial1"></a> 
<h1>Tutorial 1: Install jwt-auth</h1>

Open composer.json and update the require object to include jwt-auth: 
```php
"require": { 
    "php": ">=5.5.9", 
    "laravel/framework": "5.2.*", 
    "tymon/jwt-auth": "0.5.*" 
}, 
```
 
Then, run 
```bash
composer update 
```
 
 
<h2>What are JWT (JSON Web Tokens)?</h2>
 
JWT works by providing client application(consumer) with a token to store, when client wants to get or post something to the server, it needs to send this token. Token can be regenerated from time to time. This token exchanging is better than sending user login credentials to authenticate during requests and storing it on client side. 

You would not need to touch any core code of creating JWT, because tymon/jwt-auth package took care of that, we just need to authenticate users and use this package to return a token for future requests authentication. 
We’ll now need to update the providers array in <b>config/app.php</b> with the jwt-auth provider. Open up <b>config/app.php</b>, find the <b>providers</b> array located on line 111 and add this to it: 
 
```php
Tymon\JWTAuth\Providers\JWTAuthServiceProvider::class, 

```
Add in the jwt-auth facades which we can do in config/app.php. Find the aliases array and add these facades to it: 
 
```php
'JWTAuth'   => Tymon\JWTAuth\Facades\JWTAuth::class, 
'JWTFactory' => Tymon\JWTAuth\Facades\JWTFactory::class 
 
```
We also need to publish the assets for this package. From the command line: 
 
```bash
php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\JWTAuthServiceProvider" 
 
```
 
After you run this command you will see a new file in the <b>config</b> folder called <b>jwt.php</b>. This file contains settings for jwt-auth, one of which we need to change right away. We need to generate a secret key which we can do from the command line: 
 
```bash
php artisan jwt:generate 
 
```

You’ll see that after running this command we get a new value next to'secret' where “changeme” was before. 
 

<a name="tutorial2"></a> 
<h1>Tutorial 2: API Routes Setup</h1>
```php
Route::group(['prefix' => 'api'], function() { 
     //anything goes here will be under api/ 
 
        Route::post('login', 'Api\AuthController@login'); 
        Route::post('register', 'Api\AuthController@register'); 
        Route::get('verify/{confirmationCode}', 'Api\AuthController@confirm'); 
        Route::post('resend', 'Api\AuthController@resendVerification'); 
        Route::post('recover', 'Api\AuthController@recoverPassword'); 
 
 
        Route::group(['middleware' => ['jwt.auth']], function() { 
                  Route::post('logout', 'Api\AuthController@logout'); 
 
                  Route::get('test', function(){ 
              return response()->json(['foo'=>'bar']); 
          }); 
      }); 
}); 
```

<a name="tutorial3"></a> 
<h1>
<a href="http://esandigital.com/blog/2016/08/16/tutorial-3-json-web-token-jwt-authentication-using-laravel-5-2/">
Tutorial 3: Database Setup
</a>
</h1>
Since we are going to allow users to create their accounts within the application, we will need a table to store all of our users. Thankfully, Laravel already ships with a migration to create a basic users table, so we do not need to manually generate one. The default migration for the users table is located in the database/migrations directory.

We need to create just two fields in addition to the fields that are standard in most users tables (username, email, password, etc.).

<a href="http://esandigital.com/blog/2016/08/16/tutorial-3-json-web-token-jwt-authentication-using-laravel-5-2/">
Continue reading...
</a>
 

<a name="tutorial4"></a> 
<h1>
<a href="http://esandigital.com/blog/2016/08/16/tutorial-4-json-web-token-jwt-authentication-using-laravel-5-2/">
Tutorial 4: Authenticating Users
</a>
</h1>

This is an important part of the API, you will be authenticating users so that they can access the information they have access to. To use authentication, you need to register it in your <b>http/Kernel.php</b>
 
By default, Laravel has CSRF token verification turned on, but since we’re using JWTs in a stateless manner now, we don’t really need CSRF tokens. We can turn this default behavior off by commenting out the <b>VerifyCsrfToken</b> middleware in Kernel.php. 

<a href="http://esandigital.com/blog/2016/08/16/tutorial-4-json-web-token-jwt-authentication-using-laravel-5-2/">
Continue reading...
</a>
 
 
 
<a name="tutorial5"></a> 
<h1>Tutorial 5: E-Mail Templates</h1>

Create a file blade on resources /views/email/<b>verify.blade.php</b> as content for email delivery and input the code like this. 

```html
<!DOCTYPE html> 
<html lang="en-US"> 
<head> 
    <meta charset="utf-8"> 
</head> 
<body> 
<h2>Verify Your Email Address</h2> 
 
<div> 
    Thanks for creating an account with Test API. 
    Please follow the link below to verify your email address 
    <a href="{{ URL::to('api/verify/' . $confirmation_code) }}">Verify your account</a>. 
 
</div> 
 
</body> 
</html>

```

/views/email/<b>welcome.blade.php</b>
```html
<!DOCTYPE html> 
<html lang="en-US"> 
<head> 
    <meta charset="utf-8"> 
</head> 
<body> 
<h2>Welcome To Test API</h2> 
 
<div> 
    Thanks for joining Test API. 
 
    <br/> 
 
</div> 
 
</body> 
</html>
```
/views/email/<b>reset.blade.php</b>

```html
<!DOCTYPE html> 
<html lang="en-US"> 
<head> 
    <meta charset="utf-8"> 
</head> 
<body> 
<h2>Reset Your Password</h2>> 
 
<div> 
 
Please follow the link below to 
<a href="{{ URL::to('api/reset/' . $confirmation_code) }}">reset your password</a>. 
 
</div> 
 
</body> 
</html>
```

<a name="tutorial6"></a> 
<h1>Testing</h1>

Postman Chrome App is a great tool for sending post/get requests with an UI.

<h2>Register</h2>
http://localhost:8888/laravel-jwt-login-api/public/api/register [POST] 
{"success":true,"message":"Thanks for signing up! Please check your email."} 
 
http://localhost:8888/laravel-jwt-login-api/public/api/verify/[v_code] [GET] 

```json
{ 
success: true, 
message: "You have successfully verified your account." 
} 
```
 
 
<h2>Log In </h2>
http://localhost:8888/laravel-jwt-login-api/public/api/login [POST] 

```json
{ 
token:"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOjYsImlzcyI6Imh0dHA6XC9cL2xvY2FsaG9zdDo4ODg4XC9sYXJhdmVsLWp3dC1sb2dpbi1hcGlcL3B1YmxpY1wvYXBpXC9sb2dpbiIsImlhdCI6MTQ3MDQxNDk5NiwiZXhwIjoxNDcwNDE4NTk2LCJuYmYiOjE0NzA0MTQ5OTYsImp0aSI6ImMyYmU0YjM0MzIzZWQxYjgxOTgzMGU1N2NiNTE2NjkyIn0.6I9wobL-21o2ESRWvZUuuQPl6EKR52FfBC9BbFVnho0" 
} 
```
 
 
<h2>Access Other Routes </h2>
http://localhost:8888/laravel-jwt-login-api/public/api/test [GET] 

```json
{ 
error: "token_not_provided" 
} 
```

http://localhost:8888/laravel-jwt-login-api/public/api/test?token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOjYsImlzcyI6Imh0dHA6XC9cL2xvY2FsaG9zdDo4ODg4XC9sYXJhdmVsLWp3dC1sb2dpbi1hcGlcL3B1YmxpY1wvYXBpXC9sb2dpbiIsImlhdCI6MTQ3MDQxNTMwNSwiZXhwIjoxNDcwNDE4OTA1LCJuYmYiOjE0NzA0MTUzMDUsImp0aSI6IjZmNmYyYTMxOTAwNTdiMzk5NmEyYTYzOGMyOGM1ZTZhIn0.bwrVFEY1uPlNdm-AG5FHW1RbnFnl2npocxpCODBgu7k 

```json
{ 
foo: "bar" 
} 
 ```


