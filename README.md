# using-ajax-in-laravel-5-form-request-error-handling
Using AJAX In Laravel 5 With Form Request Error Handling
Using AJAX In Laravel 5 With Form Request Error Handling
Introduction
Here is a simple and lightweight method to submit form data via AJAX. This lightweight method will also be able to handle Laravel’s form request error messages. No additional client side validation plugins will be necessary for error handling. In this tutorial, we will be using Laravel 5 along with the Bootstrap and jQuery libraries.

What is AJAX?
AJAX is stands for Asynchronous JavaScript and XML and allows for web apps to send and retrieve data from the server asynchronously without affecting the current web page. AJAX helps improves user experience by not having to fully reload a web page on every request which then reduces bandwidth.

Outline of using AJAX in Laravel 5
-Form blade template
-Form validation request
-Routing
-AJAX script
-Controller logic
-Create a form in blade view
-First off, let’s start by creating a form with inputs so that we can post to our server. For this example let’s use a simple login form with an email and password field. Don’t forget to import the Bootstrap and jQuery libraries.

 
<form method="POST" action="{{ url('login') }}" id="login_form">
 
    <input name="_token" type="hidden" value="{{ csrf_field() }}">
 
    <div class="form-group">
        <label for="email" class="sr-only">Email Address</label>
        <input class="form-control" placeholder="Email Address" name="email" type="email">
        <span class="help-block email-error"></span>
    </div>
 
    <div class="form-group">
        <label for="email" class="sr-only">Password</label>
        <input class="form-control" placeholder="Password" name="password" type="password">
        <span class="help-block password-error"></span>
    </div>
 
    <button class="btn btn-primary" type="submit">Log In</button>
 
</form>
 
It’s a simple form with an id selector of “login_form”. I’ll be posting to a “login” post route. For error handling I have two empty span tags with the classes “email-error” and “password-error”. This form be located in my “login” view in the root of my resources > views folder.

Form validation request
Next, let’s create a form request for validation. I’ll be using the native form request provided by Laravel.

php artisan make:request LoginRequest
 
Open up LoginRequest and add your own validation rules and messages.


<?php
 
namespace App\Http\Requests;
 
use Illuminate\Foundation\Http\FormRequest;
 
class LoginRequest extends FormRequest
{
    public function authorize()
    {
        return true;
    }
 
    public function rules()
    {
        return [
            'email' => 'required|email',
            'password' => 'required',
        ];
    }
 
    public function messages()
    {
        return [
            'email.email' => 'Please enter a valid email address',
            'email.required' => 'Email address is required',
            'password.required' => 'Password is required',
        ];
    }
}
 
Proper routing
Before we move on, let’s just make sure our routes are good.

 
<?php
 
Route::get('login', ['as' => 'login', 'uses' => 'PagesController@getLogin']);
Route::post('login', ['as' => 'login', 'uses' => 'PagesController@postLogin']);
 
As you can see, I’m just using a simple GET and POST route. The GET route simply displays the form we created and the POST route will handle the login logic. Both of which reside in my “PagesController”. You can put it anywhere else you like.

Onto the AJAX script
This is my lightweight AJAX script that I pretty much recycle in most (if not all) of my web applications that I build.

 
$(document).ready(function () {
 
    var form = $('#login_form');
 
    form.submit(function(e) {
 
        e.preventDefault();
        $.ajax({
            url     : form.attr('action'),
            type    : form.attr('method'),
            data    : form.serialize(),
            dataType: 'json',
            success : function ( json )
            {
                // Success
                // Do something like redirect them to the dashboard...
            },
            error: function( json )
            {
                if(json.status === 422) {
                    var errors = json.responseJSON;
                    $.each(json.responseJSON, function (key, value) {
                        $('.'+key+'-error').html(value);
                    });
                } else {
                    // Error
                    // Incorrect credentials
                    // alert('Incorrect credentials. Please try again.')
                }
            }
        });
    });
 
});
 
First, I selected the form by id and localizing it into a variable labeled form. Then I’m building up the AJAX by simply grabbing the action and method from the form itself. Then I’m grabbing the data from the form and serializing it. I have a two callbacks: success and error. In the error callback, if the status is 422 (unprocessable entity), then I’m going to loop through the response and dynamically display the error messages in our span tag by its class “name” and “-error”. If the error status is 400 then that means the credentials were incorrect. I’m just simply going to alert the user. On success, you can handle it however you like. For example, you can redirect the user to their dashboard or something.

The controller logic
In my “PagesController” I have two methods. A getLogin method and a postLogin method. The “getLogin” method simply returns the login view and the “postLogin” method handles the login logic.

 
<?php
 
namespace App\Http\Controllers;
 
use App\Http\Requests\LoginRequest;
 
class PagesController extends Controller
{
    public function getLogin()
    {
        return view('login');
    }
 
    public function postLogin(LoginRequest $request)
    {
        if (Auth::attempt(['email' => $request->get('email'), 'password' => $request->get('password')])) {
            return response()
                ->json([
                    'message' => 'Success',
                    'status' => 200
                ], 200);
        } else {
            return response()
                ->json([
                    'message' => 'Incorrect Credentials',
                    'status' => 400
                ], 400);
        }
    }
}
 
Conclusion
That’s pretty much it! This concludes my tutorial on using AJAX in Laravel 5 with error handling. This is extremely recyclable and you can use it in any form post.
