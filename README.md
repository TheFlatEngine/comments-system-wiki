[](#comments)Comments Engine
=============================

This package can be used to comment on any model you have in your application.

### [](#features)Features

*    View comments
*    Add comment
*    Delete comment
*    Edit comment
*    Reply to comment
*    React comments (like, haha, wow, etc...)
*    Mention friends in comments
*    Ajax load
*    Dark/Light theme
*    Emoji built-in
*    HTML filter customization (using HTMLPurifier)

[](#requirements) Requirements
-----------------------------

*   PHP 7.2 +
*   Laravel 5.8 +
*   jQuery 3+
*   atwho https://github.com/ichord/At.js
*   Javascript template https://github.com/blueimp/JavaScript-Templates
*   jQuery Ajax Form https://github.com/malsup/form

Seen like many but don't worry, we included everything (except jQuery) inside the package. You just need to use it.

[](#installation)Installation
-----------------------------

Extract the files from the archive you have downloaded from CodeCanyon into root folder of your project.

Edit your composer.json file and add a local path repository:

    "repositories": [
        {
            "type": "path",
            "url": "./theflatengine/comments"
        }
    ]


In your terminal run:

    composer require theflatengine/comments *@dev

### [](#1-run-migrations)1\. Run migrations

We need to create the table for comments.

For 5.4 and bellow, add the service provider:

    // config/app.php
    'providers' => [
        ...
        TheFlatEngine\Comments\CommentsServiceProvider::class,
    ];
For 5.2 and bellow, you have to publish the migrations with:

    php artisan vendor:publish --provider="TheFlatEngine\Comments\CommentsServiceProvider" --tag=migrations

Run the migrate command to create the necessary tables:

    php artisan migrate
Publish the assets files with:

    php artisan vendor:publish --provider="TheFlatEngine\Comments\CommentsServiceProvider" --tag=public
You can publish the config-file with:

    php artisan vendor:publish --provider="TheFlatEngine\Comments\CommentsServiceProvider" --tag=config
Additionally you may want to clear the config, cache, etc:

    php artisan config:clear
    php artisan route:clear
    php artisan cache:clear
    php artisan view:clear

### [](#2-add-commenter-trait-to-your-user-model)2\. Add Attributes to your User model

Add the `Attributes` to your User model so that you can retrieve user avatar and user's profile link (why? we are have mention feature so every time people mention each other, they will able to visit user's profile too), Your User model should look like:

    class User extends Authenticatable
    {
        use Notifiable;
        
        //Add this -->
        protected $appends = [
            'artwork_url', 'permalink_url'
        ];
        //End
        
        ...
        
        //Add this -->
        public function getArtworkUrlAttribute($value)
        {
            //Change this to your avatar generator
            $size = 80;
            return "https://www.gravatar.com/avatar/" . md5( strtolower( $this->id ) ) . "?d=retro&s=" . $size;
        }
   
        public function getPermalinkUrlAttribute($value)
        {
            //you can return a router like route('user.profile', ['id' => $this->id])
            return url($this->id);
        }
        //End
    }

### [](#5-custom-comment-policy-optional)5\. Custom comment policy (optional)

If you need, you can overwrite default comment policy class, publish comments config:

    php artisan vendor:publish --provider="TheFlatEngine\Comments\CommentsServiceProvider" --tag=config

### [](#publish-config--configure-optional)Publish Config & configure (optional)

In the `config` file you can specify:

*   `rections` A list of reaction can be use.
  
*   `perPage` The number of comment load each page
   
*   `replyPreload` The number of reply will be load along with comment
   
*   `min_chars` The minimum of characters can be post
   
*   `max_chars` The maximum of characters can be post            
   
*   `allow_link` If set `true` the url like `http://google.com` will be automatically convert to click able link
   
*   `allow_edit` Allow user to edit their comments
   
*   `allow_delete` Allow user to delete their comments    
  
*   `commentable_type` A white list of Commentable type model, for example: App\User,App\Post


### [](#publish-views-customization)Publish views (customization)

You can change the design however you want.

All view examples include js/css files for correct working. `The possibility of conflict` with your scripts and styles.

    php artisan vendor:publish --provider="TheFlatEngine\Comments\CommentsServiceProvider" --tag=views 

After the command processed, all blade tempalte will be copied in to table `resources\views\vendor\TheFlatEngine`

[](#usage)Usage
---------------

### [](#1-frontend-rendering-api)1\. Frontend rendering (API):
Place this code into your main html.

    <!-- Comment System Style -->
    <link rel="stylesheet" href="{{ asset('assets/css/comments.css') }}">
    <link rel="stylesheet" href="{{ asset('assets/css/reaction.css') }}">
    <!-- End System Style -->
    <meta name="csrf-token" content="{{ csrf_token() }}" />

And then javascript library, remember to add jQuery too, you can choose any jQuery version which higher 3.0.0
    
    <!-- Comments System Library -->
    <script src="{{ asset('assets/js/commentsCore.js') }}" type="text/javascript"></script>
    <script src="{{ asset('assets/js/commentsApp.js') }}" type="text/javascript"></script>
    <!-- End Comments System Library -->
    
    
Then finally load the comment, for example User profile comment

    @include('TheFlatEngine::comments.index', ['object' => (Object) ['id' => 1, 'type' => 'App\User', 'title' => 'Kate Woodman']])

Or load comment by pure html

    <a data-toggle="comments" data-target="#user-comments-1">Load Comment</a>
    <div id="user-comments-1" data-commentable-type="App\User" data-commentable-id="1" class="hide"></div>
