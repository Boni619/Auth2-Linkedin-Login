



#Install the Laravel UI package for Auth

composer require laravel/ui

php artisan ui bootstrap --auth

#Install the js dependencies using the following command.

npm install && npm run dev

#Install the Cashier package for Stripe dependencies.

composer require laravel/cashier

#Create an essential database migrations

 php artisan migrate

 #Get and Set Stripe API Keys

// .env
STRIPE_KEY=your key here
STRIPE_SECRET=your secret here

// services.php
'stripe' => [
    'model'  => App\User::class,
    'key' => env('STRIPE_KEY'),
    'secret' => env('STRIPE_SECRET'),
],

Also, you need to add the Billable Trait inside the User.php model.

// User.php
use Laravel\Cashier\Billable;
class User extends Authenticatable
{
    use Billable;
}

#Create Plans on Stripe Dashboard

#Now create blade file,

resources >> views folder

#for webhooks stripe

composer require spatie/laravel-stripe-webhooks

php artisan vendor:publish --provider="Spatie\StripeWebhooks\StripeWebhooksServiceProvider" --tag="config"

php artisan vendor:publish --provider="Spatie\WebhookClient\WebhookClientServiceProvider" --tag="migrations"

php artisan migrate


#create routes for webhook

Route::stripeWebhooks('webhook-route-configured-at-the-stripe-dashboard');

search VerifyCsrfToken.php

protected $except = [
    'webhook-route-configured-at-the-stripe-dashboard',
];

#create webhooks in stripe > developer > webhooks

same route as create in web.php

#create a job

 php artisan make:job StripeWebhooks/HandleChargeableSource

 #The key should be the name of the stripe event type where but with the . replaced by _. The value should be the fully qualified classname.

 namespace App\Jobs\StripeWebhooks;

use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;
use Spatie\WebhookClient\Models\WebhookCall;

class HandleChargeableSource implements ShouldQueue
{
    use InteractsWithQueue, Queueable, SerializesModels;

    /** @var \Spatie\WebhookClient\Models\WebhookCall */
    public $webhookCall;

    public function __construct(WebhookCall $webhookCall)
    {
        $this->webhookCall = $webhookCall;
    }

    public function handle()
    {
        // do your work here

        // you can access the payload of the webhook call with `$this->webhookCall->payload`
    }
}

#After having created your job you must register it at the jobs array in the stripe-webhooks.php config file

// config/stripe-webhooks.php

'jobs' => [
    'source_chargeable' => \App\Jobs\StripeWebhooks\HandleChargeableSource::class,
],
