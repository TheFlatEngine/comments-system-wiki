
[](#laracomments)laraComments
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

Extract the files from the archive you have downloaded from CodeCanyon into a vendor folder in your project root.

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

### [](#2-add-commenter-trait-to-your-user-model)2\. Add Commenter trait to your User model

Add the `Attributes` trait to your User model so that you can retrieve user avatar and user's profile link (why? we are have mention feature so everytime people mention each other, they will able to visit user's profile too):

    class User extends Authenticatable
    {
        use Notifiable;
        
        protected $appends = [
            'artwork_url', 'permalink_url'
        ];
        
        protected $fillable = [
            'name', 'email', 'password',
        ];

        protected $hidden = [
            'password', 'remember_token',
        ];

        protected $casts = [
            'email_verified_at' => 'datetime',
        ];
        
        
        
        //Custom code
        public function getArtworkUrlAttribute($value)
        {
            //Change to your avatar generator
            $size = 80;
            return "https://www.gravatar.com/avatar/" . md5( strtolower( $this->id ) ) . "?d=retro&s=" . $size;
        }
    
        /**
         * @param $value
         * @return string
         * Link to user profile here
         */
    
        public function getPermalinkUrlAttribute($value)
        {
            //you can return a router like route('user.profile', ['id' => $this->id])
            return url($this->id);
        }
    }

### [](#5-custom-comment-policy-optional)5\. Custom comment policy (optional)

If you need, you can overwrite default comment policy class:

Then register policy in `AuthServiceProvider`:

And add policy prefix to comments.php config

`'policy\_prefix' => 'comments\_custom',`

### [](#publish-config--configure-optional)Publish Config & configure (optional)

In the `config` file you can specify:

*   where is your User model located; the default is `\App\User::class`
*   where is your Comment model located; the default is `\App\Comment::class`
*   policy prefix, you can create custom policy class and extends `tizis\laraComments\Policies\CommentPolicy;`
*   allow tags for html filter
*   API prefix

Publish the config file (optional):

`php artisan vendor:publish --provider="tizis\\laraComments\\Providers\\ServiceProvider" --tag=config `
### [](#publish-views-customization)Publish views (customization)

The default UI is made for `Bootstrap 4`, but `you can change it` however you want.

All view examples include js/css files for correct working. `The possibility of conflict` with your scripts and styles.

php artisan vendor:publish --provider="tizis\\laraComments\\Providers\\ServiceProvider" --tag=views 

[](#usage)Usage
---------------

### [](#1-backend-rendering)1\. Backend rendering:

In the view where you want to display comments, place this code and modify it:

In the example above we are setting argument the `model` as class of the book model.

Behind the scenes, the package detects the currently logged in user if any.

If you open the page containing the view where you have placed the above code, you should see a working comments form.

### [](#2-frontend-rendering-api)2\. Frontend rendering (API):

To disable API routes by default, set the `route.root => null` config value.

#### [](#1-description)1\. Description

Sometimes additional processing of content is necessary before transmission over API.

#### [](#2-config)2\. Config:

        'api' => [
            'get' => [
                'preprocessor' => [
                    'comment' =>  App\Helpers\CommentPreprocessor\Comment::class,
                    'user' =>  App\Helpers\CommentPreprocessor\User::class 
                    ...
                ]
            ]
        ]
    

#### [](#3-contract)3\. Contract

Create preprocessor class and implement `ICommentPreprocessor` interface:

**Examples**:

Comment:

    
    namespace App\Helpers\CommentPreprocessor;
    
    use tizis\laraComments\Contracts\ICommentPreprocessor;
    
    class Comment implements ICommentPreprocessor
    {
       public function process($comment): array
       {
           return 'Hi, ' . $comment . '!';
       }
    }
              
    

User:

    
    namespace App\Helpers\CommentPreprocessor;
    
    use tizis\laraComments\Contracts\ICommentPreprocessor;
    
    class User implements ICommentPreprocessor
    {
       public function process($user): array
       {
           $user->name = $user->name . '[Moderator]' 
           return $user;
       }
    }
              
    

#### [](#4-example)4\. Example:

Without preprocessing:

    $comment = 1;
    echo $comment; // 1
    
    $user = Auth::user();
    echo $user->name; // user1
    

With preprocessing:

    $comment = 1;
    echo $comment; // Hi, 1 !
    
    $user = Auth::user();
    echo $user->name; // user1[Moderator]
    

[](#features-of-commentable-model)Features of Commentable model
---------------------------------------------------------------

*   Scope withCommentsCount()

#### [](#example)Example:

    /**
     * Add comments_count attribute to model
     */
    Posts::withCommentsCount()->orderBy('id', 'desc')->get() 
    

[](#static-helper)Static Helper
-------------------------------

`use tizis\laraComments\Http\CommentsHelper;`

#### [](#methods)Methods:

*   getNewestComments(default $take = 10, default $commentable\_type = null)
*   getCommenterRating(int $userId, \[optional Carbon $cacheTtl\])
*   moveCommentTo(CommentInterface $comment, ICommentable $newCommentableAssociate)
*   moveCommentToAndRemoveParentAssociateOfRoot(CommentInterface $comment, ICommentable $newCommentableAssociate)

#### [](#example-1)Example:

    CommentsHelper::getNewestComments(20) // Return last 20 comments
    CommentsHelper::getNewestComments(20, Book::class) // Return last 20 comments of Book model
    

[](#examples)Examples
---------------------

This repository include only `bootstrap4` template, but you can create you own UI. This is just a example of package features.

This is example of `backend`rendering, `this way have bad performance` when 100+ comments on post due to the need to check user permissions (reply, edit, delete etc) for each comment.

`A good idea` is use API and build UI with Vue js (or any other library) with verification of user permissions (only for UI) on frontend.
