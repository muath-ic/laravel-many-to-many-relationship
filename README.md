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
