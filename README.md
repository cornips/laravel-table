# Laravel Tables
[![Made for Laravel 5](https://img.shields.io/badge/laravel-5.0-red.svg)](http://laravel.com/)
[![Latest Tag](https://img.shields.io/github/tag/gbrock/laravel-table.svg)](https://github.com/gbrock/laravel-table/releases)
<!--[![Build Status](https://img.shields.io/travis/gbrock/laravel-table.svg)](https://travis-ci.org/gbrock/laravel-table)-->

This package contains flexible ways of rendering Eloquent collections as dynamic HTML tables.  This includes
techniques for sortable columns, automatic pagination, ~~user-definable amount of rows shown per page, batch action
handling, and extensible filtering~~ (coming soon).


## Installation

Require the package in your `composer.json`:

```
"gbrock/laravel-table": "dev-master"
```

(Optional) Publish the views and config:

```
php artisan vendor:publish
```

## Usage

**In order to render an HTML table of Eloquent models into a view**, first create a Table object, passing in your
 model collection (this could be done in your controller, repository, or any service class):
 
 ```php
 $rows = User::get(); // Get all users from the database
 $table = Table::create($rows); // Generate a Table based on these "rows"
 ```
 
 Then pass that object to your view:

```php
return view('users.index', ['table' => $table]);
```

In your view, the table object can be rendered using its `render` function:

```php
{!! $table->render() !!}
```

Which would render something like this:

![Basic example](https://raw.githubusercontent.com/gbrock/laravel-table/master/examples/images/basic_initialization.png)

### Sorting

To add links in your headers which sort the indicated column, add the `Sortable` trait to the your model.  Since no 
fields are allowed to be sorted by default (for security reasons), also add a `sortable` array containing allowed fields.

```php
use Gbrock\Table\Traits\Sortable;

class User extends Model {

	use Sortable;
	
    /**
     * The attributes which may be used for sorting dynamically.
     *
     * @var array
     */
    protected $sortable = ['username', 'email', 'created_at'];
```

This adds the `sortable` scope to your model, which you should use when retrieving rows.  Altering our first example,
the database call / Table creation:

 ```php
 $rows = User::sorted()->get(); // Get all users from the database, but listen to the user Request and sort accordingly
```

Now, our table will be rendered with links in the header:

![Sortable example](https://raw.githubusercontent.com/gbrock/laravel-table/master/examples/images/sortable_initialization.png)

The links will contain query strings like `?sort=username&direction=asc`.

### Pagination

If you paginate your Eloquent collection, it will automatically be rendered below the table:

 ```php
 $rows = User::sorted()->paginate(); // Get all users from the database, sort, and paginate
```

### Customizing the Columns

For a bit more control, pass in a second argument to your database call / Table creation, **columns**:

```php
 $table = Table::create($rows, ['username', 'created_at']); // Generate a Table based on these "rows"
```

Since our view is accessing our model's attributes, we can add or modify any column key we'd like by using
[accessors](http://laravel.com/docs/5.0/eloquent#accessors-and-mutators):

```php
    protected function getRenderedCreatedAtAttribute()
    {
        // We access the following diff string with "$model->rendered_created_at"
        return $this->created_at->diffForHumans();
    }
```

### Customizing the View

The default view favors the `rendered_foo_bar` attribute, if present, else it uses the `foo_bar` attribute.  A copy of
the view file is located in `/resources/vendor/gbrock/tables/` after you've run `php artisan vendor:publish`.  You
can copy this file wherever you'd like and alter it, then tell your table to use the new view:

```php
$table->setView('users.table');
```
