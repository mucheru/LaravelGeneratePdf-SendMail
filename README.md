step 1:Download laravel
composer create-project --prefer-dist laravel/laravel blog
step2: Modify the routes
Route::get('/', 'TestController@index');
Route::get('/report', 'TestController@daily_report')->name('report');
step 3:Add laravel-Pdf Package
composer require niklasravnsborg/laravel-pdf
step 4:Configure the package in the config/app.php
'providers' => [
	// ...
	niklasravnsborg\LaravelPdf\PdfServiceProvider::class
]
'aliases' => [
	// ...
	'PDF' => niklasravnsborg\LaravelPdf\Facades\Pdf::class
]
Step 5:Publish the package
php artisan vendor:publish
Step 6:Create the controller TestController and paste the code
namespace App\Http\Controllers;

use App\Mail\CheckUser;
use App\User;
use Carbon\Carbon;
use Illuminate\Http\Request;
use PDF;

class TestController extends Controller
{
    public function index()
    {   
       return view('welcome');
    }

    public function daily_report(Request $request)
    {
       $start_date = Carbon::parse($request->start_date)
                             ->toDateTimeString();

       $end_date = Carbon::parse($request->end_date)
                             ->toDateTimeString();

       $data['users'] = User::whereBetween('created_at',[$start_date,$end_date])->get();
       
       $data['start_date'] = Carbon::parse($request->start_date)
                             ->toDayDateTimeString();
       $data['end_date'] = Carbon::parse($request->end_date)
                             ->toDayDateTimeString();
       
       $count = User::whereBetween('created_at',[$start_date,$end_date])->count();

       if( $count < 1 ) {
        session()->flash('message','There is no user between those date!');
        return redirect()->back();
       }

       $pdf = PDF::loadView('test', $data, [
                   'format' => 'A4'
              ]);

        \Mail::send('test', $data, function($message) use ($pdf){
            $message->from('xxx.com*');
            $message->to('xxx.org');
            $message->subject('Date wise user report');
            $message->attachData($pdf->output(),'document.pdf');
        });

       $pdf->SetProtection(['copy', 'print'], '', 'pass');
       return $pdf->stream('document.pdf'); 

    }
}

Step 7:
Create the views as shown in the resouce folder.
Step 8:
Create seeder for users table
step 9:
php artisan serve.
