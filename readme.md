# Create REST API in Laravel 6.0 with authentication using Passport #

## Steps to get following web services ##

S.N  | API
---- | -------------
1.   | Login API
2.   | Register API
3.   | Details API

### Step 1: Install Laravel ###
composer create project laravel/laravel project_name --prefer-dist

### Step 2: Install Package ###
composer require laravel/passport

### Step 3: Run Migration and Install
Run migration command to get new tables in database:
        php artisan migrate

using passport:install command, it will create token keys for security:
        php artisan passport:install

### Step 4: Passport Configuration ###
We need to configure files on three places:
* In Model, we need to add HasApiTokens class of Passport
* In AuthServiceProvider, we add "Passport:routes" in boot method
* In auth.php, we change the driver to passport

### Step 5: Create API route ###
Laravel provides api.php for web services route. 

#### routes/api.php ###
` 
Route::post('login', 'API\UserController@login');
Route::post('register', 'API\UserController@register');
Route::group(['middleware' => 'auth:api'], function() {
    Route::post('details', 'API\UserController@details');
});
`

### Step 6: Create the controller ###
At last we create a new controller and three API method in UserController

`
<?php
namespace App\Http\Controllers\API;
use Illuminate\Http\Request; 
use App\Http\Controllers\Controller; 
use App\User; 
use Illuminate\Support\Facades\Auth; 
use Validator;
class UserController extends Controller 
{
public $successStatus = 200;
/** 
     * login api 
     * 
     * @return \Illuminate\Http\Response 
     */ 
    public function login(){ 
        if(Auth::attempt(['email' => request('email'), 'password' => request('password')])){ 
            $user = Auth::user(); 
            $success['token'] =  $user->createToken('MyApp')-> accessToken; 
            return response()->json(['success' => $success], $this-> successStatus); 
        } 
        else{ 
            return response()->json(['error'=>'Unauthorised'], 401); 
        } 
    }
/** 
     * Register api 
     * 
     * @return \Illuminate\Http\Response 
     */ 
    public function register(Request $request) 
    { 
        $validator = Validator::make($request->all(), [ 
            'name' => 'required', 
            'email' => 'required|email', 
            'password' => 'required', 
            'c_password' => 'required|same:password', 
        ]);
if ($validator->fails()) { 
            return response()->json(['error'=>$validator->errors()], 401);            
        }
$input = $request->all(); 
        $input['password'] = bcrypt($input['password']); 
        $user = User::create($input); 
        $success['token'] =  $user->createToken('MyApp')-> accessToken; 
        $success['name'] =  $user->name;
return response()->json(['success'=>$success], $this-> successStatus); 
    }
/** 
     * details api 
     * 
     * @return \Illuminate\Http\Response 
     */ 
    public function details() 
    { 
        $user = Auth::user(); 
        return response()->json(['success' => $user], $this-> successStatus); 
    } 
}
`
To run code :
`php artisan serve`

#### Test result ####
We can perform simple test using rest client tools. For example, Postman.

Register API:
![Register API](https://github.com/BipinMhzn/rest_api_authentication_using_passport/blob/master/register%20api.png)

Login API:
![Login API](https://github.com/BipinMhzn/rest_api_authentication_using_passport/blob/master/login%20api.png)

Details API:
![Details API](https://github.com/BipinMhzn/rest_api_authentication_using_passport/blob/master/details%20api.png)
