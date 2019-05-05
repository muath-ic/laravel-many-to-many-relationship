# laravel-many-to-many-relationship

**1. Run this artisan commands**

> php artisan make:model Roles -m
> php artisan make:model Permissions -m
> php artisan make:model Roles_permissions -m

**2. Edit the following files as showing below**
1. 2019_05_04_111645_create_roles_table.php ```database\migrations\2019_05_04_111645_create_roles_table.php```
```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateRolesTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('roles', function (Blueprint $table) {
            $table->engine = 'InnoDB';
            $table->increments('id');
            $table->string('name', 191)->comment('This role contains amount of permissions');
            $table->timestamps();
            $table->softDeletes();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('roles');
    }
}
```
2. 2019_05_04_111728_create_permissions_table.php ```database\migrations\2019_05_04_111728_create_permissions_table.php```
```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreatePermissionsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('permissions', function (Blueprint $table) {
            $table->engine = 'InnoDB';
            $table->increments('id');
            $table->string('name', 191)->comment('displayed name for the user');
            $table->string('p_name', 191)->comment('name used on programming');
            $table->tinyInteger('status')->comment('one if the permission is always active ZERO if the permission can be set or not like admin');
            $table->timestamps();
            $table->softDeletes();  
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('permissions');
    }
}
```
3. 2019_05_04_112957_create_role_permissions_table.php ```database\migrations\2019_05_04_112957_create_role_permissions_table.php```
```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateRolePermissionsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('role_permissions', function (Blueprint $table) {
            $table->engine = 'InnoDB';
            $table->increments('id');
            $table->unsignedInteger('role_id')->unsigned();
            $table->unsignedInteger('permission_id')->unsigned();
            $table->timestamps();
            $table->softDeletes();
        });
        
        /**
         * for adding relationship
         * 
         * @return void
         */
        Schema::table('role_permissions', function($table) {
            $table->foreign('role_id','fk_permission_role_id')->references('id')->on('roles');//->onUpdate('CASCADE')->onDelete('CASCADE');
            $table->foreign('permission_id','fk_role_permission_id')->references('id')->on('permissions');//->onUpdate('CASCADE')->onDelete('CASCADE');
        });
    }
    

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('role_permissions');
    }
}
```
4. Roles.php ```App\Roles.php```
```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Roles extends Model
{
    /**
     * The Permissions that belong to the Roles.
     */
    public function Permissions()
    {

        return $this->belongsToMany(Permissions::class, 'role_permissions');

    }
}
```
5. Permissions.php ```App\Permissions.php```
```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Permissions extends Model
{
    /**
     * The Roles that belong to the Permissions.
     */
    public function Roles()
    {

        return $this->belongsToMany(Roles::class, 'role_permissions');

    }
}
```
6. Role_permissions.php ```App\Role_permissions.php```
```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Role_permissions extends Model
{
    //
}
```

#Put All Models files on App\Models
1. create `Models` directory on `App`
2. move all models to `Models`
3. Modify all namespace for all models into the new directory as following:
   form `namespace App;` to `namespace App\Models;`
4. Modify `config\auth.php` as following:
   ```php
   'providers' => [
        'users' => [
            'driver' => 'eloquent',
            // 'model' => App\User::class, // original line
            'model' => App\Models\User::class, // change to this
        ],
   ```
5. Modify `config\service.php` as following
   ```php
   'stripe' => [
        // 'model' => App\User::class, // original
        'model' => App\Models\User::class, // change to this
        'key' => env('STRIPE_KEY'),
        'secret' => env('STRIPE_SECRET'),
    ],
   ```
6. So if you want to use User Model you need to change the namespace from `app\User` to `app\Models\User`
7. In this case we need to change `app\Http\Controllers\Auth\RegisterController.php`
8. This is a Stack Overflow link:
   [[https://stackoverflow.com/questions/33975235/how-to-move-the-laravel-5-1-user-model-to-app-models-user]]

# Then Generate Authentication Scaffolding with this Composer Command
```composer
php artisan make:auth
```
### Controller
The controller which is used for the authentication process is **HomeController**.

```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests;
use Illuminate\Http\Request;

class HomeController extends Controller{
   /**
      * Create a new controller instance.
      *
      * @return void
   */
   
   public function __construct() {
      $this->middleware('auth');
   }
   
   /**
      * Show the application dashboard.
      *
      * @return \Illuminate\Http\Response
   */
   
   public function index() {
      return view('home');
   }
}
```
As a result, the scaffold application generated creates the login page and the registration page for performing authentication. They are on 
*resources\views\auth\\**

### Manually Authenticating Users

Laravel uses the Auth façade which helps in manually authenticating the users. It includes the attempt method to verify their email and password.

Consider the following lines of code for **LoginController** which includes all the functions for authentication

```php
<?php

// Authentication mechanism
namespace App\Http\Controllers;

use Illuminate\Support\Facades\Auth;

class LoginController extends Controller{
   /**
      * Handling authentication request
      *
      * @return Response
   */
   
   public function authenticate() {
      if (Auth::attempt(['email' => $email, 'password' => $password])) {
      
         // Authentication passed...
         return redirect()->intended('dashboard');
      }
   }
}
```

## Then Generating Policies
