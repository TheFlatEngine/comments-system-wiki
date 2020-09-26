
[](#laracomments)laraComments
=============================

This package can be used to comment on any model you have in your application.

### [](#features)Features

*    View comments
*    Create comment
*    Delete comment
*    Edit comment
*    Reply to comment
*    Authorization rules | with customization
*    View customization
*    Dispatch events
*    Likes | Dislikes | Comment rating
*    API for basic function: get, update, delete, create
*    HTML filter customization (using HTMLPurifier)

[](#upgrade-guides--)[Upgrade guides →](#upgrade-guides)
--------------------------------------------------------

*   [From 2.x.x to 3.0](#from-2xx-to-30)

[](#requirements)Requirements
-----------------------------

*   php 7.1 +
*   laravel 5.6 +
*   `Your application should contain auth module.`
    *   Laravel 6.x: [https://laravel.com/docs/6.x/authentication](https://laravel.com/docs/6.x/authentication)
    *   Laravel 5.6: [https://laravel.com/docs/5.6/authentication](https://laravel.com/docs/5.6/authentication)

[](#installation)Installation
-----------------------------

composer require tizis/lara-comments 

### [](#1-run-migrations)1\. Run migrations

We need to create the table for comments.

`php artisan migrate`

### [](#2-add-commenter-trait-to-your-user-model)2\. Add Commenter trait to your User model

Add the `Commenter` trait to your User model so that you can retrieve the comments for a user:

use tizis\\laraComments\\Traits\\Commenter;
     
class User extends Authenticatable {   
	use ..., Commenter;   

### [](#3-create-comment-model)3\. Create Comment model

`use tizis\\laraComments\\Entity\\Comment as laraComment;`

`class Comment extends laraComment
{
}`

### [](#4-add-commentable-trait-and-the-icommentable-interface-to-models)4\. Add `Commentable` trait and the `ICommentable` interface to models

Add the `Commentable` trait and the `ICommentable` interface to the model for which you want to enable comments for:

`use tizis\\laraComments\\Contracts\\ICommentable;
use tizis\\laraComments\\Traits\\Commentable;`
     
class Post extends Model implements ICommentable {        
   use Commentable;        

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

1.  Build with semantic ui  
    [![2222d](https://user-images.githubusercontent.com/16865573/48430226-0124c680-e799-11e8-9341-daac331236b2.png)](https://user-images.githubusercontent.com/16865573/48430226-0124c680-e799-11e8-9341-daac331236b2.png)
2.  Build with bootstrap 4  
    [![3333](https://user-images.githubusercontent.com/16865573/48430227-0124c680-e799-11e8-8cdb-8dd042155550.png)](https://user-images.githubusercontent.com/16865573/48430227-0124c680-e799-11e8-8cdb-8dd042155550.png)

[](#upgrade-guides)Upgrade guides
---------------------------------

### [](#from-2xx-to-30)From 2.x.x to 3.0

`commentable_type` and `commentable_id` request attributes was merged into single `commentable_encrypted_key`

You need to replace these deprecated attributes.

Example:

    Old /bootstrap4/form.blade.php
    <input type="hidden" name="commentable_type" value="\{{ get_class($model) }}"/>
    <input type="hidden" name="commentable_id" value="{{ $model->id }}"/>
    
    

    New /bootstrap4/form.blade.php
    <input type="hidden" name="commentable_encrypted_key" value="{{ $model->getEncryptedKey() }}"/>
