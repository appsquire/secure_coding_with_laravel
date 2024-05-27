# Secure Coding with Laravel

## Introduction

Coding is a dangerous job.  Hackers are around every corner, eager to poke and prod every line of code created and published by a developer, always looking for that elusive vulnerability that will lead to immense wealth and power!  Bwahahaha!  But, coding doesn’t have to be dangerous if you follow the rules of the street.  Follow along and we'll teach you some of the tricks of the trade for writing secure code in one of AppSquire’s favorite web frameworks; [Laravel](https://laravel.com).  (At the time of writing this blog, Laravel 11 was the latest version of the framework)

## Laravel's Security

Laravel is inherently a very secure framework.  Its massive open source community is constantly publishing patches and updates to keep its codebase secure and relevant to the current security landscape.  Vulnerabilities typically exist in websites created with the framework rather than the framework itself.  We’ll look at a few security issues that developers may overlook when creating a Laravel web application.  First, we’ll look at writing secure database queries, then we’ll learn about writing validators to protect against dangerous inputs.  Finally, we’ll delve into writing security policies for authorization.

## Writing Secure Database Queries

Database query injection attacks are a very common attack method used when attempting to exfiltrate sensitive database information from a server.  Sometimes, they can even be used to obtain remote-code execution (RCE) on a server.  Query injection vulnerabilities are caused by incorrect/missing user input validation leading to database query manipulation; which in turn leads to sensitive data leaks or unauthorized data changes.  Laravel’s powerful and extremely easy to use ORM, Eloquent, contains the tools needed to combat these attacks.  In raw PHP, parameterized queries are used to sanitize user input so it can’t be used to hijack queries.  Parameterized queries are usually quite time consuming to set up, however.  Eloquent performs this parameterization automatically.  It allows something that would look like this with parameterized queries (SQL query sample)
```
$username = [value directly obtained from user];
$stmt = mysqli_prepare($dbc, "SELECT * FROM users WHERE username = ?");
mysqli_stmt_bind_param($stmt, "s", $username);
mysqli_stmt_execute($stmt);
$row = mysqli_stmt_fetch($stmt);
```

To simply look like this in Eloquent with automatically parameterized input.
```
$username = [value directly obtained from user];
User::where(‘username’, $username)->first();
```
## Validating User Inputs

Although Laravel helps keep your database queries safe, you should always validate user inputs; both for database queries and anywhere else user inputs are being used to control backend functionalities.  The number one rule on the street is to never trust user input.  If no input validation is implemented for file uploads, hackers could DDOS your server or possibly even cause a remote code execution (RCE) if they can upload (and afterwards open) PHP files.  File name inputs that aren’t sanitized could possibly lead to path traversal attacks.  Even ignoring the security aspect of input validation, it prevents the user-supplied input from being gibberish, such as a user inputting the word “cheeseburger” as their age.  Writing validators is simple and easy but they are very powerful.  A manually created validator is shown below and there is an even simpler automatic version that can be run as an extension directly on a request object.
```
$userData = [‘title’ => ‘some title’, ‘file’ => [some file]];
Validator::make($userData, [
   'title' => 'required|string|max:255',
   'file' => 'file|size:100|mimes:jpg,bmp,png',
])->validate();
```
With this code, simple validation controls, separated by the pipe character, prevent titles from being outrageously long or being omitted.  Files are forced to be within a specified size and range of file types.  There are many validator controls available to fit any use case; from regex checking to email format checking and everything in between.

## Implementing Authorization with Security Policies

Moving on from validation, implementing authorization in a Laravel application is also easy with its extremely useful and robust security policy functionalities. With Laravel’s security policies and “gates”, backend functions and model viewing/editing can be blocked from prying eyes and only allow access to users who have been granted rights.  After a few simple configuration steps, a security policy check, allowing only a user to change their own notes, will end up looking something like the below code.
```
public function update(?User $user, Note $note): bool
{
    return $user?->id === $note->author_user_id;
}
``` 
The check can be easily called in any function that has a reference to the note model with the gate authorize function. In this example the check would look like `Gate::authorize('update', $note);`.

Laravel will automatically pass the current user and note model to the security policy function and verify that the user is authorized to update the note.  If the check fails, Laravel will throw a 403 exception and go no further.  The security policies can also be used as middleware and run the function before a page is even reached.  For example, the code below wouldn’t allow an unauthorized user to view a secret note page.
```
Route::get('/secret-note/{note}', function (Note $note) {
   // Page where the user can view a secret note
})->middleware('can:view,note');
```
These examples only scratch the surface of what Laravel’s authorization framework allows you to do.  You can conditionally render blade files and much much more, but this should be enough to get you started.

## Conclusion

The aforementioned functionalities are just some of the more common ones that every developer should be aware of but there is a lot more that Laravel has to offer including CSRF protection and rate-limiting.  We’ve only just begun in the world of thwarting hackers with Laravel, but you should now have a good start on exploring the rest on your own.  Now go put the knowledge you’ve just gained to good use and keep the streets safe from those nosy hackers! 

If you want to learn more or need help with an app idea or existing project, [contact us](https://www.appsquire.com/#Contact-Us) today.
