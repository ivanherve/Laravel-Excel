<!--

PLEASE READ: FILLING IN THE TEMPLATE IS REQUIRED!
Issues that do not include enough information might not be picked up.

Have you read Laravel-Excel's 
contributing guidelines (https://laravel-excel.maatwebsite.nl/docs/3.0/getting-started/contributing)
and Code Of Conduct (https://github.com/Maatwebsite/Laravel-Excel/blob/3.0/CODE_OF_CONDUCT.md)?
By filing an Issue, you are expected to comply with it, including treating everyone with respect.

Please prefix your issue with one of the following: [BUG] [PROPOSAL] [QUESTION].

-->

### Prerequisites

<!--

Put an X between the brackets if you have done the following:

-->

* [X] Able to reproduce the behaviour outside of your code, the problem is isolated to Laravel Excel.
* [X] Checked that your issue isn't already filed.
* [X] Checked if no PR was submitted that fixes this problem.

### Versions

<!-- Please be as exact and complete as possible when proving version numbers -->

* PHP version: 7.1.3<!-- put your PHP version here -->
* Laravel/lumen-framework version: 5.7<!-- put your Laravel version here -->
* Package version: ^3.1<!-- put Laravel Excel package version here -->

### Description

<!-- Describe the issue -->
As described upper, I'm trying to export data in an excel file and download it but I'm getting this error,
```Access to fetch at 'http://localhost:8000/download' from origin 'http://localhost:3000' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.```

### Steps to Reproduce

<!-- How can this issue be reproduced? Provide an Excel file or reproduction repository to help us reproduce the issue easily.  -->
run ``php artisan make:export UsersExport --model=User`` to get a UserExport.php file which looks like this:
```
<?php

namespace App\Exports;

use App\User;
use Maatwebsite\Excel\Concerns\FromCollection;

class UsersExport implements FromCollection
{
    /**
    * @return \Illuminate\Support\Collection
    */
    public function collection()
    {
        return User::all();
    }
}
```
and create this function in your the controller file: 
```
public function excel()
    {
        return Excel::download(new UsersExport(),'users.xlsx');
    }
```
Then add in the middleware folder the CorsMiddleware file:
```<?php

namespace App\Http\Middleware;

use Closure;

class CorsMiddleware
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        $headers = [
            'Access-Control-Allow-Origin'      => '*',
            'Access-Control-Allow-Methods'     => 'POST, GET, OPTIONS, PUT, DELETE',
            'Access-Control-Allow-Credentials' => 'true',
            'Access-Control-Max-Age'           => '86400',
            'Access-Control-Allow-Headers'     => 'Content-Type, Authorization, X-Requested-With, api_token'
        ];

        if ($request->isMethod('OPTIONS'))
        {
            return response()->json('{"method":"OPTIONS"}', 200, $headers);
        }

        $response = $next($request);
        foreach($headers as $key => $value)
        {
            $response->header($key, $value);
        }

        return $response;
    }
}
```

and add it to the ``bootstrp/app.php``:
```
$app->middleware([
    App\Http\Middleware\CorsMiddleware::class
]);
```

Finnally add in route/web.php: ``$router->get('/download, 'CommunicationController@excel');``


**Expected behavior:**

<!-- What you expect to happen -->
I expected an excel file to be downloaded 

**Actual behavior:** 

<!-- What actually happens. Please include screenshots, strack traces and anything that can help us understand the issue. -->
I'm getting this on the console of google chrome each time I click on the button that runs the download link
![image](https://user-images.githubusercontent.com/23102351/55387042-66573980-5539-11e9-8b45-cbefb0f9a485.png)
I just don't undertand why the CORS block this request and not the others. I also have other functions that works properly thanks to all I added on CorsMiddleware. Do I need to add other parameters ? I so on other similar issues that they had to add parameters but they were using old versions of laravel-excel.

Thank you for helping me
