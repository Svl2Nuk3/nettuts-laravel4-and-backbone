## What we are going to build:

- Build a JSON API using Laravel 4
- Use Composer for all server-side dependencies
- Build a single page app using Backbone.js
- Use NPM to manage all client-side dependencies

> Note: This tutorial assumes that you have Composer, NPM, and a LESS compiler installed.




## Part 1: Architecture

### Git

Let's start by creating a git repository to work in.  For your reference this entire repo will be made publically available.

	mkdir project && cd project
	git init

### Laravel 4 Install

Laravel 4 uses Composer to install all of its dependencies, but first we will need an application structure to install into.  The develop branch on Laravel's Github repository is the home for this application structure.  However, since Laravel 4 is currently still in beta, we need to be prepared for this structure to change at any time.  By adding Laravel as a remote repository, we can pull in these changes whenever we need to.

	git remote add laravel https://github.com/laravel/laravel
	git fetch laravel
	git merge laravel/develop
	git add . && git commit -am "commit the laravel application structure"

Now we have the application structure, but all of the library files that Laravel needs are not yet installed.  You will notice that at the root of our application there is a file called `composer.json`.  This is the file that will keep track of all the dependencies that our application requires.  Before we tell Composer to download and install them, let's first add a few more dependencies that we are going to need.  We will be adding:

- Jeffrey Way's Generators:  Some very useful commands to greatly improve our workflow by automatically generating file stubs for us.
- Laravel 4 Mustache:  This will allow us to seemlessly use Mustache.php in our Laravel project
- Twitter Bootstrap:  We will use the LESS files from this project to speed up our front-end development.
- PHPUnit:  We will be doing some TDD for our JSON API, PHPUnit will be our testing engine
- Mockery:  Mockery will help us "mock" objects during our testing

PHPUnit and Mockery are only required in our development environment, so we will specify that in our composer.json file.

composer.json

	{
	    "require": {
	        "laravel/framework": "4.0.*",
	        "way/generators": "dev-master",
	        "twitter/bootstrap": "dev-master",
	        "conarwelsh/mustache-l4": "dev-master"
	    },
	    "require-dev": {
	        "phpunit/phpunit": "3.7.*",
	        "mockery/mockery": "0.7.*"
	    },
	    "autoload": {
	        "classmap": [
	            "app/commands",
	            "app/controllers",
	            "app/models",
	            "app/database/migrations",
	            "app/database/seeds",
	            "app/tests/TestCase.php"
	        ]
	    },
	    "scripts": {
	        "post-update-cmd": "php artisan optimize"
	    },
	    "minimum-stability": "dev"
	}

Now we just need to tell Composer to do all of our leg work!  Notice the --dev switch, we are telling composer that we are in our development evironment, and that it should also install all of our dependencies listed in `"require-dev"`.

	composer install --dev

After that finishes installing, we will need to inform Laravel of a few of our dependencies.  Laravel uses "service providers" for this purpose.  The basically just tell Laravel how their code is going to interact with the application, and run any necessary setup procedures.  Open up `app/config/app.php` and add the following 2 items to the "providers" array.

	...
	
	'Way\Generators\GeneratorsServiceProvider',
	'Conarwelsh\MustacheL4\MustacheL4ServiceProvider',

	...

Lastly, we just need to do some generic environment tweaks to complete our Laravel install.  First let's open up `bootstrap/start.php` and tell Laravel how to decide that it is in a development environment.

bootstrap/start.php

	/*
	|--------------------------------------------------------------------------
	| Detect The Application Environment
	|--------------------------------------------------------------------------
	|
	| Laravel takes a dead simple approach to your application environments
	| so you can just specify a machine name or HTTP host that matches a
	| given environment, then we will automatically detect it for you.
	|
	*/

	$env = $app->detectEnvironment(array(

		'local' => array('your-machine-name'),

	));

Replace "your-machine-name" with whatever the hostname for your machine is.  If you are unsure of what your exact machine name is, you can just type `hostname` at the command prompt (on Mac or Linux), whatever it returns is the value that belongs in this setting.

We want our views to be able to be served to our client from a web request.  Currently, our views are stored outside our public folder, which would means that they are not publically accessible.  Luckily Laravel makes it very easy to move, or add another views folder to our setup!  Open up app/config/view.php and change the `paths` setting to the following, which will change the location of our views path to inside the public folder:

	'paths' => array(__DIR__.'/../../public/views'),

Next you will need to configure your database.  Open up `app/config/database.php` and add in your database settings.

Finally, you just need to make sure that your storage folder can be written to.

	chmod -R 777 app/storage

Laravel is now installed, with all of its dependencies.  Now let's setup our Backbone install.  Just like our `composer.json` installed all of our server-side dependencies, we will create a `package.json` to install all of our client-side dependencies.

For our client-side dependencies we will use:

- Underscore.js: This is a dependency of Backbone.js
- Backbone.js: This is our client-side MVC that we will use to build out our application
- Mustache.js: The Javascript version of our templating library, by using the same templating language both on the client and the server, we can share views, as opposed to duplication logic.

package.json

	{
	    "name": "nettuts-laravel4-and-backbone",
	    "version": "0.0.1",
	    "private": true,
	    "dependencies": {
	        "underscore": "*",
	        "backbone": "*",
	        "mustache": "*"
	    }
	}

Now just switch into your public folder, and run `npm install`.  After that completes lets switch back to our application root so we are prepared for the rest of our commands.

	cd public
	npm install
	cd ..

Package managers save us from so much work, should you want to update any of these libraries, all you have to do is run `npm update` or `composer update`.  Also, should you want to lock any of these libraries in at a specific version, all you have to do is specify the version number, and the package manager will handle the rest.

To wrap up our setup process we will just add in all of the basic project files and folders that we will need, and then test that it all works OK.

We will need to add the following folders:

- public/views
- public/views/layouts
- public/js
- public/css

And the following files:

- public/css/styles.less
- public/js/app.js
- public/views/app.mustache

the one-liner:

	mkdir public/views public/views/layouts public/js public/css && touch public/css/styles.less public/js/app.js public/views/app.mustache

Twitter Bootstrap also has 2 Javascript dependencies that we will need, so let's just copy them from the vendor folder into our public folder.

- html5shiv: allows us to use HTML5 elements without fear of older browsers not supporting them
- bootstrap.js: the supporting Javascript libraries for Twitter Bootstrap

	cp vendor/twitter/bootstrap/docs/assets/js/html5shiv.js public/js/html5shiv.js
	cp vendor/twitter/bootstrap/docs/assets/js/bootstrap.min.js public/js/bootstrap.min.js

For our layout file, Twitter Bootstrap also provides us with some nice starter templates to work with, so let's copy one into our layouts folder for a good head start

	cp vendor/twitter/bootstrap/docs/examples/starter-template.html public/views/layouts/application.blade.php

> Note:: notice that I am using a blade extension here, this could just as easily be a mustache template, but I wanted to show you how easy it is to mix the templating engines.  In addition, we get some really useful helpers in our blade files, and since our layout will be rendered on page load, and will not need to be re-rendered by the client, we are safe to use PHP here exclusively.  If for some reason you found yourself needing to render this file on the client-side, you would want to switch this file to use the Mustache template engine instead.

Now that we have all of our basic files in place, let's add some starter content that we can use to test that everything is working as we would expect.  I am providing you with some basic stubs (nicely commented) to get you started.

public/css/styles.less

	/**
	 * Import Twitter Bootstrap Base File
	 ******************************************************************************************
	 */
	@import "../../vendor/twitter/bootstrap/less/bootstrap";


	/**
	 * Define App Styles
	 * I like to do this before the responsive include, so that it can override properly as needed.
	 ******************************************************************************************
	 */
	body {
	    padding-top: 60px; /* 60px to make the container go all the way to the bottom of the topbar */
	}

	/* this will be set the position of our alerts */
	#notifications {
		width: 300px;
		position: fixed;
		top: 50px;
		left: 50%;
		margin-left: -150px;
		text-align: center;
	}

	/**
	 * Import Bootstrap's Responsive Overrides
	 * now we allow bootstrap to set the overrides for a responsive layout
	 ******************************************************************************************
	 */
	@import "../../vendor/twitter/bootstrap/less/responsive";


	/**
	 * Define our variables last, any variable declared here will be used in the includes above
	 * which means that we can override any of the variables used in the bootstrap files easily
	 ******************************************************************************************
	 */

	// Scaffolding
	// -------------------------
	@bodyBackground:        #f2f2f2;
	@textColor:             #575757;

	// Links
	// -------------------------
	@linkColor:             #41a096;

	// Typography
	// -------------------------
	@sansFontFamily:        Arial, Helvetica, sans-serif;




public/js/app.js

	//alias the global object
	//alias jQuery so we can potentially use other libraries that utilize $
	//alias Backbone to save us on some typing
	(function(exports, $, bb){

	    //document ready
	    $(function(){

	        /**
	         ***************************************
	         * Cached Globals
	         ***************************************
	         */
	        var $window, $body, $document;

	        $window   = $(window);
	        $body     = $('body');
	        $document = $(document);


	    });//end document ready

	}(this, jQuery, Backbone));





public/views/layouts/application.blade.php

	<!DOCTYPE html>
	<html lang="en">
	<head>
	  <meta charset="utf-8">
	  <title>Laravel4 & Backbone | Nettuts</title>
	  <meta name="viewport" content="width=device-width, initial-scale=1.0">
	  <meta name="description" content="A single page blog built using Backbone.js, Laravel, and Twitter Bootstrap">
	  <meta name="author" content="Conar Welsh">

	  <link href="{{ asset('css/styles.css') }}" rel="stylesheet">

	  <!-- HTML5 shim, for IE6-8 support of HTML5 elements -->
	  <!--[if lt IE 9]>
	  <script src="{{ asset('js/html5shiv.js') }}"></script>
	  <![endif]-->
	</head>
	<body>

	  <div id="notifications">
	  </div>

	  <div class="navbar navbar-inverse navbar-fixed-top">
	    <div class="navbar-inner">
	      <div class="container">
	        <button type="button" class="btn btn-navbar" data-toggle="collapse" data-target=".nav-collapse">
	          <span class="icon-bar"></span>
	          <span class="icon-bar"></span>
	          <span class="icon-bar"></span>
	        </button>
	        <a class="brand" href="#">Nettuts Tutorial</a>
	        <div class="nav-collapse collapse">
	          <ul class="nav">
	            <li class="active"><a href="#">Blog</a></li>
	          </ul>
	        </div><!--/.nav-collapse -->
	      </div>
	    </div>
	  </div>

	  <div class="container" data-role="main">
	    {{ $content }}
	  </div> <!-- /container -->

	  <!-- Placed at the end of the document so the pages load faster -->
	  <script src="//ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script> <!-- use Google CDN for jQuery to hopefully get a cached copy -->
	  <script src="{{ asset('node_modules/underscore/underscore-min.js') }}"></script>
	  <script src="{{ asset('node_modules/backbone/backbone-min.js') }}"></script>
	  <script src="{{ asset('node_modules/mustache/mustache.js') }}"></script>
	  <script src="{{ asset('js/bootstrap.min.js') }}"></script>
	  <script src="{{ asset('js/app.js') }}"></script>
	  @yield('scripts')
	</body>
	</html>




public/views/app.mustache

	<dl>
	    <dt>Q. What did Biggie say when he watched inception?</dt>
	    <dd>A. "It was all a dream!"</dd>
	</dl>



app/routes.php
	
	<?php

	//backbone app route
	Route::get('/', function()
	{
	    //change our view name to the view we created in a previous step
	    //notice that we do not need to provide the .mustache extension
	    return View::make('layouts.application')->nest('content', 'app');
	});




With all of our basic files in place, we can test to ensure everything went OK.  Laravel 4 utilizes the new PHP web server to provide us with a great little web environment.  So long to the days of having a million virtual hosts setup on your development machine for every project that you work on!

> Note:: make sure that you compiled your LESS file!

	php artisan serve

If you followed along correctly, you should be laughing hysterically at my horrible sense of humor, and all of our assets should be properly included into the page.  



























## Part 2: Laravel 4 JSON API

Now we will build our API that will power our Backbone application.  Laravel 4 makes this process a breeze.

### API Rules
First let's go over a few general guidelines to keep in mind while we build an API:

- Status Codes:  Responses should reply with proper status codes, fight the temptation to just place an `{ error: "this is an error message" }` in the body of your response.  Use the HTTP protocol to it's fullest!
		
		- 200: success
		- 201: resource created
		- 204: success, but no content to return
		- 400: request not fulfilled //validation error
		- 401: not authenticated
		- 403: refusal to respond //wrong credentials, do not have permission (un-owned resource)
		- 404: not found
		- 500: other error

- Resource Methods:  Even though controllers will be serving different resources, they should still have very similar behavior.  The more predictable your API is, the easier it is to implement and adopt.

		- index: return a collection of resources
		- show: return a single resource
		- create: return a form, this form should detail out the required fields, validation, and labels as best as possible.  As well as anything else needed to properly create a resource
		- store: store a new resource, and return with the proper status code: 201
		- edit: return a form filled with the current state of a resource, this form should detail out the required fields, validation, and labels as best as possible.  As well as anything else needed to properly edit a resource
		- update: update an existing resource, and return with the proper status code
		- delete: delete an existing resource, and return with the proper status code: 204


### Routing & Versioning

API's are designed to be around for a while, this is not like your website where you can just change its functionality at the drop of a dime.  If you have programs that use your API, they are not going to be happy with you if you change things around in the API and everything breaks.  For this reason it is important that you version your API's.

Laravel provides us with route groups that are perfect for this, place the following code ABOVE our first route:

	//create a group of routes that will belong to APIv1
	Route::group(array('prefix' => 'v1'), function()
	{
		//... insert API routes here...
	});



### Generate Resources

We are going to use Jeffrey Way's generators to generate our resources.  When we generate a resource it will create us the following items:

- Controller
- Model
- Views (index.blade.php, show.blade.php, create.blade.php, edit.blade.php)
- Migration
- Seeds
- Tests

We are going to need only 2 resources for this app: a Post resource, and a Comment resource.

> Note: in a recent update to the generators, I have been receiving a permissions error due to the way my web servers are setup.  To remedy this problem, I just open up the permissions for the tmp folder that the generators use.  Normally I would not open permissions up this way, but this change will only be affected in our development environment so it is safe.
	
	sudo chmod -R 777 vendor/way/generators/src/Way/

	php artisan generate:resource post --fields="title:string, content:text, author_name:string"

	php artisan generate:resource comment --fields="content:text, author_name:string, post_id:integer"

Take a second to investigate all of the files that it created for us.




### Adjust the generated resources:

The resource generator saved us a lot of work, but due to our unique congiguration, we are still going to need to make some modifications.

First of all, the generator placed the views it created in the `app/views` folder, so we need to move them to the `public/views` folder

	mv app/views/posts public/views/posts
	mv app/views/comments public/views/comments

We decided that we wanted our API to be versioned, so we will need to move the routes the generator created for us into the version group.  We will want to namespace our controllers with the corresponding version, so that we can have a different set of controllers for each version we build, and we will also want to nest our comments resource under the posts resource.

app/routes.php

	<?php

	//create a group of routes that will belong to APIv1
	Route::group(array('prefix' => 'v1'), function()
	{
		//... insert API routes here...
		Route::resource('posts', 'V1\PostsController'); //notice the namespace
		Route::resource('posts.comments', 'V1\PostsCommentsController'); //notice the namespace, and the nesting
	});

	//backbone app route
	Route::get('/', function()
	{
	    //change our view name to the view we created in a previous step
	    //notice that we do not need to provide the .mustache extension
	    return View::make('layouts.application')->nest('content', 'app');
	});


Since we namespaced our controllers, we should move them into their own folder for organization, let's create a folder named `V1` and move our generated controllers into it.  Also, since we nested our comments controller under the posts controller, let's change the name of that controller to reflect the relationship.

	mkdir app/controllers/V1
	mv app/controllers/PostsController.php app/controllers/V1/
	mv app/controllers/CommentsController.php app/controllers/V1/PostsCommentsController.php

We need to make a few modifications to the controllers files now:

app/controllers/PostsController.php

	<?php
	//use our new namespace
	namespace V1;

	//import classes that are not in this new namespace
	use BaseController;

	class PostsController extends BaseController {

app/controllers/PostsCommentsController.php

	<?php
	//use our new namespace
	namespace V1;

	//import classes that are not in this new namespace
	use BaseController;

	//rename our controller class
	class PostsCommentsController extends BaseController {







### Repositories

By default, repositories are not part of Laravel.  Laravel is extremely flexible though, and makes it very easy to add them in.  We are going to use repositories to help us separate our logic for code re-usability, as well as for testing.  For now we will just get setup to use repositories, we will add in the proper logic later.

make a folder to store our repositories

	mkdir app/repositories

To let our auto-loader know about this new folder, we need to add it to our `composer.json` file.  Take a look at the updated "autoload" section of our file, and see that we added the repositories folder.

composer.json

	{
	    "require": {
	        "laravel/framework": "4.0.*",
	        "way/generators": "dev-master",
	        "twitter/bootstrap": "dev-master",
	        "conarwelsh/mustache-l4": "dev-master"
	    },
	    "require-dev": {
	        "phpunit/phpunit": "3.7.*",
	        "mockery/mockery": "0.7.*"
	    },
	    "autoload": {
	        "classmap": [
	            "app/commands",
	            "app/controllers",
	            "app/models",
	            "app/database/migrations",
	            "app/database/seeds",
	            "app/tests/TestCase.php",
	            "app/repositories"
	        ]
	    },
	    "scripts": {
	        "post-update-cmd": "php artisan optimize"
	    },
	    "minimum-stability": "dev"
	}








### Seeds

Database seeds are a useful tool, they provide us with an easy way to fill our database with some content.  The generators provided us with base files for seeding, we merely need to add in some actual seeds.

app/database/seeds/PostsTableSeeder.php

	<?php

	class PostsTableSeeder extends Seeder {

	    public function run()
	    {
	        $posts = array(
	        	array(
	                'title'       => 'Test Post',
	                'content'     => 'Lorem ipsum Reprehenderit velit est irure in enim in magna aute occaecat qui velit ad.',
	                'author_name' => 'Conar Welsh',
	                'created_at'  => date('Y-m-d H:i:s'),
	                'updated_at'  => date('Y-m-d H:i:s'),
	            ),
	            array(
	                'title'       => 'Another Test Post',
	                'content'     => 'Lorem ipsum Reprehenderit velit est irure in enim in magna aute occaecat qui velit ad.',
	                'author_name' => 'Conar Welsh',
	                'created_at'  => date('Y-m-d H:i:s'),
	                'updated_at'  => date('Y-m-d H:i:s'),
	            ),
	        );

	        // Uncomment the below to run the seeder
	        DB::table('posts')->insert($posts);
	    }

	}

app/database/seeds/CommentsTableSeeder.php

	<?php

	class CommentsTableSeeder extends Seeder {

	    public function run()
	    {
	        $comments = array(
	        	array(
	                'content'     => 'Lorem ipsum Nisi dolore ut incididunt mollit tempor proident eu velit cillum dolore sed',
	                'author_name' => 'Testy McTesterson',
	                'post_id'     => 1,
	                'created_at'  => date('Y-m-d H:i:s'),
	                'updated_at'  => date('Y-m-d H:i:s'),
	            ),
	            array(
	                'content'     => 'Lorem ipsum Nisi dolore ut incididunt mollit tempor proident eu velit cillum dolore sed',
	                'author_name' => 'Testy McTesterson',
	                'post_id'     => 1,
	                'created_at'  => date('Y-m-d H:i:s'),
	                'updated_at'  => date('Y-m-d H:i:s'),
	            ),
	            array(
	                'content'     => 'Lorem ipsum Nisi dolore ut incididunt mollit tempor proident eu velit cillum dolore sed',
	                'author_name' => 'Testy McTesterson',
	                'post_id'     => 2,
	                'created_at'  => date('Y-m-d H:i:s'),
	                'updated_at'  => date('Y-m-d H:i:s'),
	            ),
	        );

	        // Uncomment the below to run the seeder
	        DB::table('comments')->insert($comments);
	    }

	}

Do not forget to run `composer dump-autoload` to let the Composer auto loader know about the new migration files!

	composer dump-autoload

Now we can run our migrations, and seed the database. Laravel provides us with a single command to do both:

	php artisan migrate --seed
















### Tests

Testing is one of those topics in development that no one can argue the importance of, however most people tend to ignore due to the learning curve.  However testing is really not that difficult, and can really benefit your application.  For this tutorial, we will setup some basic tests to help us ensure that our API is functioning properly.  We will do this in a TDD style, however if I were to walk you through each test individually, this would prove to be a very long tutorial, so in the interest of brevity, I will just provide you with a few nicely commented set of tests to work from.

Before we write any tests though, we should first check the current status of our tests.  Since we installed PHPUnit via composer, we have the binaries available to us to use.  All you need to do is run:

	vendor/phpunit/phpunit/phpunit.php

Whoops! We already have a failure!  The test that is failing is actually an example test that comes pre-installed in our Laravel application structure, this tests against the default route that was also installed with the Laravel application structure.  Since we modified this route, we cannot be surprised that the test failed.  We can actually just delete this test altogether.

	rm -rf app/tests/ExampleTest.php

If you run the PHPUnit command again, you will see that no tests were executed, and we have a clean slate for testing.  The rules of TDD state that we are not allowed to write any production code, until we have failing tests that warrant it.

> Note: it is possible that if you have an older version of Jeffrey Way's generators that you will actually have a few tests in there that were created by those generators, and those tests are probably failing.  Just delete or overwrite those tests with the ones found below to proceed.

For this tutorial we will be testing our controllers and our repositories.  Let's create a few folders to store these tests in:

	mkdir app/tests/controllers app/tests/repositories

Now for the test files.  We are going to use Mockery to mock our repositories for our controller tests, for now just put the code in as is, we will explain/develop this functionality shortly.

app/tests/controllers/CommentsControllerTest.php

	<?php

	class CommentsControllerTest extends TestCase {

		/**
		 * Basic Route Tests
		 * notice that we can use our route() helper here!
		 */
		public function testIndex()
		{
			$response = $this->call('GET', route('v1.posts.comments.index', array(1)) );
			$this->assertTrue($response->isOk());
		}

		public function testShow()
		{
			$response = $this->call('GET', route('v1.posts.comments.show', array(1,1)) );
			$this->assertTrue($response->isOk());
		}

		public function testCreate()
		{
			$response = $this->call('GET', route('v1.posts.comments.create', array(1)) );
			$this->assertTrue($response->isOk());
		}

		public function testEdit()
		{
			$response = $this->call('GET', route('v1.posts.comments.edit', array(1,1)) );
			$this->assertTrue($response->isOk());
		}

		/**
		 * Test that the controller calls repo as we expect
		 * notice we are Mocking our repository
		 * also notice that we do not really care about the data or interactions
		 * we merely care that the controller is doing what we are going to want
		 * it to do, which is reach out to our repository for more information
		 */
		public function testIndexShouldCallFindAllMethod()
		{
			$mock = Mockery::mock('CommentRepositoryInterface');
			$mock->shouldReceive('findAll')->once()->andReturn('foo');
			App::instance('CommentRepositoryInterface', $mock);

			$response = $this->call('GET', route('v1.posts.comments.index', array(1)));
			$this->assertTrue(!! $response->original);
		}

		public function testShowShouldCallFindById()
		{
			$mock = Mockery::mock('CommentRepositoryInterface');
			$mock->shouldReceive('findById')->once()->andReturn('foo');
			App::instance('CommentRepositoryInterface', $mock);

			$response = $this->call('GET', route('v1.posts.comments.show', array(1,1)));
			$this->assertTrue(!! $response->original);
		}

		public function testCreateShouldCallInstanceMethod()
		{
			$mock = Mockery::mock('CommentRepositoryInterface');
			$mock->shouldReceive('instance')->once()->andReturn(array());
			App::instance('CommentRepositoryInterface', $mock);

			$response = $this->call('GET', route('v1.posts.comments.create', array(1)));
			$this->assertViewHas('comment');
		}

		public function testEditShouldCallFindByIdMethod()
		{
			$mock = Mockery::mock('CommentRepositoryInterface');
			$mock->shouldReceive('findById')->once()->andReturn(array());
			App::instance('CommentRepositoryInterface', $mock);

			$response = $this->call('GET', route('v1.posts.comments.edit', array(1,1)));
			$this->assertViewHas('comment');
		}

		public function testStoreShouldCallStoreMethod()
		{
			$mock = Mockery::mock('CommentRepositoryInterface');
			$mock->shouldReceive('store')->once()->andReturn('foo');
			App::instance('CommentRepositoryInterface', $mock);

			$response = $this->call('POST', route('v1.posts.comments.store', array(1)));
			$this->assertTrue(!! $response->original);
		}

		public function testUpdateShouldCallUpdateMethod()
		{
			$mock = Mockery::mock('CommentRepositoryInterface');
			$mock->shouldReceive('update')->once()->andReturn('foo');
			App::instance('CommentRepositoryInterface', $mock);

			$response = $this->call('PUT', route('v1.posts.comments.update', array(1,1)));
			$this->assertTrue(!! $response->original);
		}

		public function testDestroyShouldCallDestroyMethod()
		{
			$mock = Mockery::mock('CommentRepositoryInterface');
			$mock->shouldReceive('destroy')->once()->andReturn(true);
			App::instance('CommentRepositoryInterface', $mock);

			$response = $this->call('DELETE', route('v1.posts.comments.destroy', array(1,1)));
			$this->assertTrue( empty($response->original) );
		}


	}


app/tests/controllers/PostsControllerTest.php

	<?php

	class PostsControllerTest extends TestCase {

		/**
		 * Test Basic Route Responses
		 */
		public function testIndex()
		{
			$response = $this->call('GET', route('v1.posts.index'));
			$this->assertTrue($response->isOk());
		}

		public function testShow()
		{
			$response = $this->call('GET', route('v1.posts.show', array(1)));
			$this->assertTrue($response->isOk());
		}

		public function testCreate()
		{
			$response = $this->call('GET', route('v1.posts.create'));
			$this->assertTrue($response->isOk());
		}

		public function testEdit()
		{
			$response = $this->call('GET', route('v1.posts.edit', array(1)));
			$this->assertTrue($response->isOk());
		}

		/**
		 * Test that controller calls repo as we expect
		 */
		public function testIndexShouldCallFindAllMethod()
		{
			$mock = Mockery::mock('PostRepositoryInterface');
			$mock->shouldReceive('findAll')->once()->andReturn('foo');
			App::instance('PostRepositoryInterface', $mock);

			$response = $this->call('GET', route('v1.posts.index'));
			$this->assertTrue(!! $response->original);
		}

		public function testShowShouldCallFindById()
		{
			$mock = Mockery::mock('PostRepositoryInterface');
			$mock->shouldReceive('findById')->once()->andReturn('foo');
			App::instance('PostRepositoryInterface', $mock);

			$response = $this->call('GET', route('v1.posts.show', array(1)));
			$this->assertTrue(!! $response->original);
		}

		public function testCreateShouldCallInstanceMethod()
		{
			$mock = Mockery::mock('PostRepositoryInterface');
			$mock->shouldReceive('instance')->once()->andReturn(array());
			App::instance('PostRepositoryInterface', $mock);

			$response = $this->call('GET', route('v1.posts.create'));
			$this->assertViewHas('post');
		}

		public function testEditShouldCallFindByIdMethod()
		{
			$mock = Mockery::mock('PostRepositoryInterface');
			$mock->shouldReceive('findById')->once()->andReturn(array());
			App::instance('PostRepositoryInterface', $mock);

			$response = $this->call('GET', route('v1.posts.edit', array(1)));
			$this->assertViewHas('post');
		}

		public function testStoreShouldCallStoreMethod()
		{
			$mock = Mockery::mock('PostRepositoryInterface');
			$mock->shouldReceive('store')->once()->andReturn('foo');
			App::instance('PostRepositoryInterface', $mock);

			$response = $this->call('POST', route('v1.posts.store'));
			$this->assertTrue(!! $response->original);
		}

		public function testUpdateShouldCallUpdateMethod()
		{
			$mock = Mockery::mock('PostRepositoryInterface');
			$mock->shouldReceive('update')->once()->andReturn('foo');
			App::instance('PostRepositoryInterface', $mock);

			$response = $this->call('PUT', route('v1.posts.update', array(1)));
			$this->assertTrue(!! $response->original);
		}

		public function testDestroyShouldCallDestroyMethod()
		{
			$mock = Mockery::mock('PostRepositoryInterface');
			$mock->shouldReceive('destroy')->once()->andReturn(true);
			App::instance('PostRepositoryInterface', $mock);

			$response = $this->call('DELETE', route('v1.posts.destroy', array(1)));
			$this->assertTrue( empty($response->original) );
		}

	}


app/tests/repositories/EloquentCommentRepositoryTest.php

	<?php

	class EloquentCommentRepositoryTest extends TestCase {
		
		public function setUp()
		{
			parent::setUp();
			$this->repo = App::make('EloquentCommentRepository');
		}

		public function testFindByIdReturnsModel()
		{
			$comment = $this->repo->findById(1,1);
			$this->assertTrue($comment instanceof Illuminate\Database\Eloquent\Model);
		}

		public function testFindAllReturnsCollection()
		{
			$comments = $this->repo->findAll(1);
			$this->assertTrue($comments instanceof Illuminate\Database\Eloquent\Collection);
		}

		public function testValidatePasses()
		{
			$reply = $this->repo->validate(array(
				'post_id'     => 1,
				'content'     => 'Lorem ipsum Fugiat consectetur laborum Ut consequat aliqua.',
				'author_name' => 'Testy McTesterson'
			));

			$this->assertTrue($reply);
		}

		public function testValidateFailsWithoutContent()
		{
			try {
				$reply = $this->repo->validate(array(
					'post_id'     => 1,
					'author_name' => 'Testy McTesterson'
				));
			}
			catch(ValidationException $expected)
			{
				return;
			}

			$this->fail('ValidationException was not raised');
		}

		public function testValidateFailsWithoutAuthorName()
		{
			try {
				$reply = $this->repo->validate(array(
					'post_id'     => 1,
					'content'     => 'Lorem ipsum Fugiat consectetur laborum Ut consequat aliqua.'
				));
			}
			catch(ValidationException $expected)
			{
				return;
			}

			$this->fail('ValidationException was not raised');
		}

		public function testValidateFailsWithoutPostId()
		{
			try {
				$reply = $this->repo->validate(array(
					'author_name' => 'Testy McTesterson',
					'content'     => 'Lorem ipsum Fugiat consectetur laborum Ut consequat aliqua.'
				));
			}
			catch(ValidationException $expected)
			{
				return;
			}

			$this->fail('ValidationException was not raised');
		}

		public function testStoreReturnsModel()
		{
			$comment_data = array(
				'content'     => 'Lorem ipsum Fugiat consectetur laborum Ut consequat aliqua.',
				'author_name' => 'Testy McTesterson'
			);

			$comment = $this->repo->store(1, $comment_data);

			$this->assertTrue($comment instanceof Illuminate\Database\Eloquent\Model);
			$this->assertTrue($comment->content === $comment_data['content']);
			$this->assertTrue($comment->author_name === $comment_data['author_name']);
		}

		public function testUpdateSaves()
		{
			$comment_data = array(
				'content' => 'The Content Has Been Updated'
			);

			$comment = $this->repo->update(1, 1, $comment_data);

			$this->assertTrue($comment instanceof Illuminate\Database\Eloquent\Model);
			$this->assertTrue($comment->content === $comment_data['content']);
		}

		public function testDestroySaves()
		{
			$reply = $this->repo->destroy(1,1);
			$this->assertTrue($reply);

			try {
				$this->repo->findById(1,1);
			}
			catch(NotFoundException $expected)
			{
				return;
			}

			$this->fail('NotFoundException was not raised');
		}

		public function testInstanceReturnsModel()
		{
			$comment = $this->repo->instance();
			$this->assertTrue($comment instanceof Illuminate\Database\Eloquent\Model);
		}

		public function testInstanceReturnsModelWithData()
		{
			$comment_data = array(
				'title' => 'Un-validated title'
			);

			$comment = $this->repo->instance($comment_data);
			$this->assertTrue($comment instanceof Illuminate\Database\Eloquent\Model);
			$this->assertTrue($comment->title === $comment_data['title']);
		}

	}


app/tests/repositories/EloquentPostRepositoryTest.php

	<?php

	class EloquentPostRepositoryTest extends TestCase {
		
		public function setUp()
		{
			parent::setUp();
			$this->repo = App::make('EloquentPostRepository');
		}

		public function testFindByIdReturnsModel()
		{
			$post = $this->repo->findById(1);
			$this->assertTrue($post instanceof Illuminate\Database\Eloquent\Model);
		}

		public function testFindAllReturnsCollection()
		{
			$posts = $this->repo->findAll();
			$this->assertTrue($posts instanceof Illuminate\Database\Eloquent\Collection);
		}

		public function testValidatePasses()
		{
			$reply = $this->repo->validate(array(
				'title'       => 'This Should Pass',
				'content'     => 'Lorem ipsum Fugiat consectetur laborum Ut consequat aliqua.',
				'author_name' => 'Testy McTesterson'
			));

			$this->assertTrue($reply);
		}

		public function testValidateFailsWithoutTitle()
		{
			try {
				$reply = $this->repo->validate(array(
					'content'     => 'Lorem ipsum Fugiat consectetur laborum Ut consequat aliqua.',
					'author_name' => 'Testy McTesterson'
				));
			}
			catch(ValidationException $expected)
			{
				return;
			}

			$this->fail('ValidationException was not raised');
		}

		public function testValidateFailsWithoutAuthorName()
		{
			try {
				$reply = $this->repo->validate(array(
					'title'       => 'This Should Pass',
					'content'     => 'Lorem ipsum Fugiat consectetur laborum Ut consequat aliqua.'
				));
			}
			catch(ValidationException $expected)
			{
				return;
			}

			$this->fail('ValidationException was not raised');
		}

		public function testStoreReturnsModel()
		{
			$post_data = array(
				'title'       => 'This Should Pass',
				'content'     => 'Lorem ipsum Fugiat consectetur laborum Ut consequat aliqua.',
				'author_name' => 'Testy McTesterson'
			);

			$post = $this->repo->store($post_data);

			$this->assertTrue($post instanceof Illuminate\Database\Eloquent\Model);
			$this->assertTrue($post->title === $post_data['title']);
			$this->assertTrue($post->content === $post_data['content']);
			$this->assertTrue($post->author_name === $post_data['author_name']);
		}

		public function testUpdateSaves()
		{
			$post_data = array(
				'title' => 'The Title Has Been Updated'
			);

			$post = $this->repo->update(1, $post_data);

			$this->assertTrue($post instanceof Illuminate\Database\Eloquent\Model);
			$this->assertTrue($post->title === $post_data['title']);
		}

		public function testDestroySaves()
		{
			$reply = $this->repo->destroy(1);
			$this->assertTrue($reply);

			try {
				$this->repo->findById(1);
			}
			catch(NotFoundException $expected)
			{
				return;
			}

			$this->fail('NotFoundException was not raised');
		}

		public function testInstanceReturnsModel()
		{
			$post = $this->repo->instance();
			$this->assertTrue($post instanceof Illuminate\Database\Eloquent\Model);
		}

		public function testInstanceReturnsModelWithData()
		{
			$post_data = array(
				'title' => 'Un-validated title'
			);

			$post = $this->repo->instance($post_data);
			$this->assertTrue($post instanceof Illuminate\Database\Eloquent\Model);
			$this->assertTrue($post->title === $post_data['title']);
		}

	}



Now that we have all of our tests in place, let's run PHPUnit again to watch them fail.

	vendor/phpunit/phpunit/phpunit.php

You should have a whole ton of failures, and in fact, the test suite probably did not even finish testing before it crashed.  This is OK, that means we have followed the rules of TDD and wrote failing tests before production code, albeit that tpically these tests would be written on at a time, and you would not move on to the next test until you had code that allowed the previous test to pass.  Your terminal should probably look something like mine at the moment:

@@@TODO: SCREENSHOT@@@

What is actually failing is the assertViewHas method in our controller tests.  It is kind of intimidating to deal with this kind of an error when we have lumped together all of our tests without any production code at all.  This is why you should always write the tests one at a time, as you will find these errors in stride, as opposed to just a huge mess.  For now just follow my lead.

Let's break for a quick sidebar discussion on the true responsibilities of an MVC (my opinions of them at least).

- Model: represent the data layer, and handle translating application logic to data-storage logic
- View: display the data in the requested fashion.  HTTP protocol allows us to specify which type of content that we would like to receive, and a view is the layer that should be able to transform the content to suite this request.
- Controller: handle request input($_GET, $_POST, headers, etc), and return a response.  This is where the controversy comes...

In my opinion a controller should not contain any more logic than request and response, but the distance between that defined setup and data-layer abstraction (the model) is quite far.  There are many tasks that will need to be completed before just handing data to a view:

- filtering: you may need to adjust your response according to certain filters
- pagination: what if you only want to show certain segments of data at a time
- validation: checking that input data is valid
- authentication/authorization: ensuring that the requesting user is allowed access to this data
- and many more

If we place these types of logic in our controller, the usage of that functionality is only available to that route.  So where else can we put it... in the model?  If we place this logic in the model, what was a clean abstraction of our database is now becoming cluttered with all kinds of application logic... does not quite feel right.  What if 2 versions of our API have different methods of parsing input data, handling pagination, or validating... are we going to put just alternate versions of the same method all in the model?  Controllers cannot change without breaking routes, and models cannot change without breaking data manipulation.

This is where repositories come into the picture.  Repositories solve the gap between the model and the controller in beautiful fashion.  Our models are able to stay focused specifically on how to manage data coming in and out of the data-layer, and our controllers can focus solely on user input and responses.  We now will have re-usable, swap-able, test-able and version-able classes to work with to handle all of the above needs.  Repositories can now handle all of the growing-pains of our application, whilst leaving controllers and models to do what they do best!

Now that we have a general overview of what I consider to be a very functional application structure... our MVRC... let's start putting the pieces in place and getting our tests to pass.

If you take a read through the controller tests, you will see that all we really care about is how the controller is interacting with the repository.  So let's see how light that makes our PostsController.

app/controllers/V1/PostsController.php

	<?php
	namespace V1;

	use BaseController;
	use PostRepositoryInterface;
	use Input;
	use View;

	class PostsController extends BaseController {

		/**
		 * We will use Laravel's dependency injection to auto-magically
		 * "inject" our repository instance into our controller
		 */
		public function __construct(PostRepositoryInterface $posts)
		{
			$this->posts = $posts;
		}

		/**
		 * Display a listing of the resource.
		 *
		 * @return Response
		 */
		public function index()
		{
			return $this->posts->findAll();
		}

		/**
		 * Show the form for creating a new resource.
		 *
		 * @return Response
		 */
		public function create()
		{
			$post = $this->posts->instance();
			return View::make('posts._form', compact('post'));
		}

		/**
		 * Store a newly created resource in storage.
		 *
		 * @return Response
		 */
		public function store()
		{
			return $this->posts->store( Input::all() );
		}

		/**
		 * Display the specified resource.
		 *
		 * @param  int  $id
		 * @return Response
		 */
		public function show($id)
		{
			return $this->posts->findById($id);
		}

		/**
		 * Show the form for editing the specified resource.
		 *
		 * @param  int  $id
		 * @return Response
		 */
		public function edit($id)
		{
			$post = $this->posts->findById($id);
			return View::make('posts._form', compact('post'));
		}

		/**
		 * Update the specified resource in storage.
		 *
		 * @param  int  $id
		 * @return Response
		 */
		public function update($id)
		{
			return $this->posts->update($id, Input::all());
		}

		/**
		 * Remove the specified resource from storage.
		 *
		 * @param  int  $id
		 * @return Response
		 */
		public function destroy($id)
		{
			$this->posts->destroy($id);
			return '';
		}

	}

app/controllers/PostsCommentsController.php

	<?php
	namespace V1;

	use BaseController;
	use CommentRepositoryInterface;
	use Input;
	use View;

	class PostsCommentsController extends BaseController {

		/**
		 * We will use Laravel's dependency injection to auto-magically
		 * "inject" our repository instance into our controller
		 */
		public function __construct(CommentRepositoryInterface $comments)
		{
			$this->comments = $comments;
		}

		/**
		 * Display a listing of the resource.
		 *
		 * @return Response
		 */
		public function index($post_id)
		{
			return $this->comments->findAll($post_id);
		}

		/**
		 * Show the form for creating a new resource.
		 *
		 * @return Response
		 */
		public function create($post_id)
		{
			$comment = $this->comments->instance(array(
				'post_id' => $post_id
			));

			return View::make('comments._form', compact('comment'));
		}

		/**
		 * Store a newly created resource in storage.
		 *
		 * @return Response
		 */
		public function store($post_id)
		{
			return $this->comments->store( $post_id, Input::all() );
		}

		/**
		 * Display the specified resource.
		 *
		 * @param  int  $id
		 * @return Response
		 */
		public function show($post_id, $id)
		{
			return $this->comments->findById($post_id, $id);
		}

		/**
		 * Show the form for editing the specified resource.
		 *
		 * @param  int  $id
		 * @return Response
		 */
		public function edit($post_id, $id)
		{
			$comment = $this->comments->findById($post_id, $id);

			return View::make('comments._form', compact('comment'));
		}

		/**
		 * Update the specified resource in storage.
		 *
		 * @param  int  $id
		 * @return Response
		 */
		public function update($post_id, $id)
		{
			return $this->comments->update($post_id, $id, Input::all());
		}

		/**
		 * Remove the specified resource from storage.
		 *
		 * @param  int  $id
		 * @return Response
		 */
		public function destroy($post_id, $id)
		{
			$this->comments->destroy($post_id, $id);
			return '';
		}

	}

It does not get much simpler than that, all it is doing is handing the input data to the repository, and taking the response from that, and handing it to the view.

> Note: notice that we added a few more "use" statements to the top of the file, to support the other classes that we are using.  Do not forget this when you are working within a namespace.

The only thing that is a bit tricky in this controller is the constructor.  Notice we are passing in a typed variable, yet there is not point that we have access to the instantiation of this controller to actually insert that class... welcome to dependency injection!  What we are actually doing here is hinting to our controller that we have a dependency needed to run this class.  Laravel will then take the type hint that we provided, and run `App::make('PostRepositoryInterface')`, and insert what is returned into the constructor of the controller for us.

The observant of you will notice that what we are trying to pass in as a dependency is an interface, and as you know, an interface cannot be instantiated.  This is where it gets cool, we are actually going to tell Laravel that whenever an instance of `PostRepositoryInterface` is requested, to actually return something else.

Open up your `app/routes.php` file and add the following to the top of the file
	
	App::bind('PostRepositoryInterface', 'EloquentPostRepository');
	App::bind('CommentRepositoryInterface', 'EloquentCommentRepository');

Now if you were to ever change your repository to instead use a different ORM than Eloquent, or maybe a file-based driver, all you have to do is change these 2 lines and you are good to go.

PostRepositoryInterface and CommentRepositoryInterface must actually exist, and the bindings must actually implement that interface.  So let's create them now:

app/repositories/PostRepositoryInterface.php

	<?php

	interface PostRepositoryInterface {
		public function findById($id);
		public function findAll();
		public function paginate();
		public function store($data);
		public function update($id, $data);
		public function destroy($id);
		public function validate($data);
		public function instance();
	}

app/repositories/CommentRepositoryInterface.php

	<?php

	interface CommentRepositoryInterface {
		public function findById($post_id, $id);
		public function findAll($post_id);
		public function store($post_id, $data);
		public function update($post_id, $id, $data);
		public function destroy($post_id, $id);
		public function validate($data);
		public function instance();
	}


Now that we have our 2 interfaces built, our implementations contain each of these methods.  Let's build those now.

app/repositories/EloquentPostRepository.php

	<?php

	class EloquentPostRepository implements PostRepositoryInterface {

		public function findById($id)
		{
			$post = Post::with(array(
					'comments' => function($q)
					{
						$q->orderBy('created_at', 'desc');
					}
				))
				->where('id', $id)
				->first();

			if(!$post) throw new NotFoundException('Post Not Found');
			return $post;
		}

		public function findAll()
		{
			return Post::with(array(
					'comments' => function($q)
					{
						$q->orderBy('created_at', 'desc');
					}
				))
				->orderBy('created_at', 'desc')
				->get();
		}

		public function paginate($limit = null)
		{
			return Post::paginate($limit);
		}

		public function store($data)
		{
			$this->validate($data);
			return Post::create($data);
		}

		public function update($id, $data)
		{
			$post = $this->findById($id);
			$post->fill($data);
			$this->validate($post->toArray());
			$post->save();
			return $post;
		}

		public function destroy($id)
		{
			$post = $this->findById($id);
			$post->delete();
			return true;
		}

		public function validate($data)
		{
			$validator = Validator::make($data, Post::$rules);
			if($validator->fails()) throw new ValidationException($validator);
			return true;
		}

		public function instance($data = array())
		{
			return new Post($data);
		}

	}
	

app/repositories/EloquentCommentRepository.php

	<?php

	class EloquentCommentRepository implements CommentRepositoryInterface {

		public function findById($post_id, $id)
		{
			$comment = Comment::find($id);
			if(!$comment || $comment->post_id != $post_id) throw new NotFoundException('Comment Not Found');
			return $comment;
		}

		public function findAll($post_id)
		{
			return Comment::where('post_id', $post_id)
				->orderBy('created_at', 'desc')
				->get();
		}

		public function store($post_id, $data)
		{
			$data['post_id'] = $post_id;
			$this->validate($data);
			return Comment::create($data);
		}

		public function update($post_id, $id, $data)
		{
			$comment = $this->findById($post_id, $id);
			$comment->fill($data);
			$this->validate($comment->toArray());
			$comment->save();
			return $comment;
		}

		public function destroy($post_id, $id)
		{
			$comment = $this->findById($post_id, $id);
			$comment->delete();
			return true;
		}

		public function validate($data)
		{
			$validator = Validator::make($data, Comment::$rules);
			if($validator->fails()) throw new ValidationException($validator);
			return true;
		}

		public function instance($data = array())
		{
			return new Comment($data);
		}

	}

If you take a look in our repositories, there are a few Exceptions that we are throwing, which are not native, nor do they belong to Laravel.  Those are custom Exceptions that we are using to simplify our code.  By using custom Exceptions, we are able to easily halt the progress of the application if certain conditions are met.  For instance if a post is not found, we can just toss a NotFoundException, and the application will handle it accordingly, but not by showing a 500 error as usual, instead we are going to setup custom error handlers.

First let's define the custom Exceptions.  Create a file in your `app` folder called `errors.php`

	touch app/errors.php

app/errors.php

	<?php

	class PermissionException extends Exception {
		
		public function __construct($message = null, $code = 403)
		{
			parent::__construct($message ?: 'Action not allowed', $code);
		}

	}

	class ValidationException extends Exception {

		protected $messages;

		public function __construct($validator)
		{
			$this->messages = $validator->messages();
			parent::__construct($this->messages, 400);
		}

		public function getMessages()
		{
			return $this->messages;
		}

	}

	class NotFoundException extends Exception {

		public function __construct($message = null, $code = 404)
		{
			parent::__construct($message ?: 'Resource Not Found', $code);
		}

	}

Very simple Exceptions, notice for the ValidationException, we can just pass it the failed validator instance, and it will handle the error messages accordingly!

Now we need to define our error handlers, that will be called when one of these Exceptions are thrown.

app/filters.php

	...

	/**
	 * General HttpException handler
	 */
	App::error( function(Symfony\Component\HttpKernel\Exception\HttpException $e, $code)
	{
		$headers = $e->getHeaders();

		switch($code)
		{
			case 401:
				$default_message = 'Invalid API key';
				$headers['WWW-Authenticate'] = 'Basic realm="CRM REST API"';
			break;

			case 403:
				$default_message = 'Insufficient privileges to perform this action';
			break;

			case 404:
				$default_message = 'The requested resource was not found';
			break;

			default:
				$default_message = 'An error was encountered';
		}

		return Response::json(array(
			'error' => $e->getMessage() ?: $default_message
		), $code, $headers);
	});

	/**
	 * Permission Exception Handler
	 */
	App::error(function(PermissionException $e, $code)
	{
		return Response::json($e->getMessage(), $e->getCode());
	});

	/**
	 * Validation Exception Handler
	 */
	App::error(function(ValidationException $e, $code)
	{
		return Response::json($e->getMessages(), $code);
	});

	/**
	 * Not Found Exception Handler
	 */
	App::error(function(NotFoundException $e)
	{
		return Response::json($e->getMessage(), $e->getCode());
	});


We must now let our auto-loader know about these new files.  So we must tell Composer where to check for them:

composer.json

	{
	    "require": {
	        "laravel/framework": "4.0.*",
	        "way/generators": "dev-master",
	        "twitter/bootstrap": "dev-master",
	        "conarwelsh/mustache-l4": "dev-master"
	    },
	    "require-dev": {
	        "phpunit/phpunit": "3.7.*",
	        "mockery/mockery": "0.7.*"
	    },
	    "autoload": {
	        "classmap": [
	            "app/commands",
	            "app/controllers",
	            "app/models",
	            "app/database/migrations",
	            "app/database/seeds",
	            "app/tests/TestCase.php",
	            "app/repositories",
	            "app/errors.php"
	        ]
	    },
	    "scripts": {
	        "post-update-cmd": "php artisan optimize"
	    },
	    "minimum-stability": "dev"
	}

We must now tell Composer to actually check for these files, and include them in the auto-load registry.

	composer dump-autoload

Great, so we have completed our controllers and our repositories, the last 2 items in our MVRC that we have to take care of are our models and views, both of which are pretty straight forward.

app/models/Post.php

	<?php

	class Post extends Eloquent {

	    protected $fillable = array(
	    	'title', 'content', 'author_name'
	    );

	    public static $rules = array(
	    	'title'       => 'required',
			'author_name' => 'required'
	    );

	    public function comments()
		{
			return $this->hasMany('Comment');
		}

	}

app/models/Comment.php

	<?php

	class Comment extends Eloquent {

		protected $fillable = array(
			'post_id', 'content', 'author_name'
		);

		public static $rules = array(
			'post_id'     => 'required|numeric',
			'content'     => 'required',
			'author_name' => 'required'
		);

		public function post()
		{
			return $this->belongsTo('Post');
		}

	}


As far as views are concerned, I am just going to mark up some simple bootstrap-friendly pages.  Remember to change each files extension to `.mustache` though, since our generator thought that we would be using `.blade.php`.  We are also going to create a few "partial" views, using the Rails convention of prefixing them with an `_` to signify a partial.

> Note: I skipped a few views, as we will not be using them in this tutorial.

public/views/posts/index.mustache

	{{#posts}}
		{{> posts._post}}
	{{/posts}}

public/views/posts/show.mustache

	<article>
		<h3>
			{{ post.title }} {{ post.id }}
			<small>{{ post.author_name }}</small>
		</h3>
		<div>
			{{ post.content }}
		</div>
	</article>

	<div>
		<h2>Add A Comment</h2>
		{{> comments._form }}

		<section data-role="comments">
			{{#post.comments}}
				<div>
					{{> comments._comment }}
				</div>
			{{/post.comments}}
		</section>
	</div>

public/views/posts/_post.mustache

	<article data-toggle="view" data-target="posts/{{ id }}">
		<h3>{{ title }} {{ id }}</h3>
		<cite>{{ author_name }} on {{ created_at }}</cite>
	</article>

public/views/posts/_form.mustache

	{{#exists}}
		<form action="/v1/posts/{{ post.id }}" method="post">
			<input type="hidden" name="_method" value="PUT" />
	{{/exists}}
	{{^exists}}
		<form action="/v1/posts" method="post">
	{{/exists}}
		
		<fieldset>

			<div class="control-group">
				<label class="control-label"></label>
				<div class="controls">
					<input type="text" name="title" value="{{ post.title }}" />
				</div>
			</div>

			<div class="control-group">
				<label class="control-label"></label>
				<div class="controls">
					<input type="text" name="author_name" value="{{ post.author_name }}" />
				</div>
			</div>

			<div class="control-group">
				<label class="control-label"></label>
				<div class="controls">
					<textarea name="content">{{ post.content }}"</textarea>
				</div>
			</div>

			<div class="form-actions">
				<input type="submit" class="btn btn-primary" value="Save" />
			</div>

		</fieldset>
	</form>

public/views/comments/_comment.mustache

	<h5>
		{{ author_name }}
		<small>{{ created_at }}</small>
	</h5>
	<div>
		{{ content }}
	</div>

public/views/comments/_form.mustache

	{{#exists}}
		<form class="form-horizontal" action="/v1/posts/{{ comment.post_id }}/{{ id }}" method="post">
			<input type="hidden" name="_method" value="PUT" />
	{{/exists}}
	{{^exists}}
		<form class="form-horizontal" action="/v1/posts/{{ comment.post_id }}" method="post">
	{{/exists}}
		
		<fieldset>

			<div class="control-group">
				<label class="control-label">Author Name</label>
				<div class="controls">
					<input type="text" name="author_name" value="{{ comment.author_name }}" />
				</div>
			</div>

			<div class="control-group">
				<label class="control-label">Comment</label>
				<div class="controls">
					<textarea name="content">{{ comment.content }}</textarea>
				</div>
			</div>

			<div class="form-actions">
				<input type="submit" class="btn btn-primary" value="Save" />
			</div>

		</fieldset>
	</form>







Great, we have all of our API components in place.  Let's run our unit tests to see where we are at!

	vendor/phpunit/phpunit/phpunit.php

Your first run of this test should pass with flying (green) colors.  However, if you were to run this test again, you will notice that it fails now with a handful of errors, and that is because our repository tests actually tested the database, and in doing so deleted some of the records our previous tests used to assert values.  This is an easy fix though, all we have to do is tell our tests that they need to re-seed the database after each test.  In addition, we did not receive a noticable error for this, but we did not close Mockery after each test either, this is a requirement of Mockery that you can find in their docs.  So let's add both missing methods.

open up `app/tests/TestCase.php` and add the following 2 methods

	public function setUp()
	{
		parent::setUp();
		$this->seed();
	}

	public function tearDown()
	{
		Mockery::close();
	}

This is great, we now said that at every "tear down", which is run after each test completes, to re-seed the database.  However we still have one problem, everytime you re-seed, it is only going to append new rows to the tables, our tests are looking for items with a row ID of 1, so we still have a few changes to make.  We just need to tell the database to truncate our tables when seeding:

app/database/seeds/CommentsTableSeeder.php

	<?php

	class CommentsTableSeeder extends Seeder {

		public function run()
	    {
	        $comments = array(
	            array(
	                'content'     => 'Lorem ipsum Nisi dolore ut incididunt mollit tempor proident eu velit cillum dolore sed',
	                'author_name' => 'Testy McTesterson',
	                'post_id'     => 1,
	                'created_at'  => date('Y-m-d H:i:s'),
	                'updated_at'  => date('Y-m-d H:i:s'),
	            ),
	            array(
	                'content'     => 'Lorem ipsum Nisi dolore ut incididunt mollit tempor proident eu velit cillum dolore sed',
	                'author_name' => 'Testy McTesterson',
	                'post_id'     => 1,
	                'created_at'  => date('Y-m-d H:i:s'),
	                'updated_at'  => date('Y-m-d H:i:s'),
	            ),
	            array(
	                'content'     => 'Lorem ipsum Nisi dolore ut incididunt mollit tempor proident eu velit cillum dolore sed',
	                'author_name' => 'Testy McTesterson',
	                'post_id'     => 2,
	                'created_at'  => date('Y-m-d H:i:s'),
	                'updated_at'  => date('Y-m-d H:i:s'),
	            ),
	        );

	        //truncate the comments table when we seed
	        DB::table('comments')->truncate();
	        DB::table('comments')->insert($comments);
	    }

	}


app/database/seeds/PostsTableSeeder.php

	<?php

	class PostsTableSeeder extends Seeder {

		public function run()
	    {
	        $posts = array(
	            array(
	                'title'       => 'Test Post',
	                'content'     => 'Lorem ipsum Reprehenderit velit est irure in enim in magna aute occaecat qui velit ad.',
	                'author_name' => 'Conar Welsh',
	                'created_at'  => date('Y-m-d H:i:s'),
	                'updated_at'  => date('Y-m-d H:i:s'),
	            ),
	            array(
	                'title'       => 'Another Test Post',
	                'content'     => 'Lorem ipsum Reprehenderit velit est irure in enim in magna aute occaecat qui velit ad.',
	                'author_name' => 'Conar Welsh',
	                'created_at'  => date('Y-m-d H:i:s'),
	                'updated_at'  => date('Y-m-d H:i:s'),
	            )
	        );

	        //truncate the posts table each time we seed
	        DB::table('posts')->truncate();
	        DB::table('posts')->insert($posts);
	    }

	}

	
Now you should be able to run the tests any number of times, and get passing tests each time!

























## Backbone App


















