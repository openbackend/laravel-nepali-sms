# Laravel Nepali SMS Gateway Integrator

[![Latest Version on Packagist](https://img.shields.io/packagist/v/openbackend/nepali-sms.svg?style=flat-square)](https://packagist.org/packages/openbackend/nepali-sms)
[![Total Downloads](https://img.shields.io/packagist/dt/openbackend/nepali-sms.svg?style=flat-square)](https://packagist.org/packages/openbackend/nepali-sms)
[![License](https://img.shields.io/packagist/l/openbackend/nepali-sms.svg?style=flat-square)](https://packagist.org/packages/openbackend/nepali-sms)

A comprehensive Laravel package for integrating popular Nepali SMS gateways with advanced features like OTP verification, bulk messaging, delivery tracking, and more.

## Supported SMS Gateways

- **Sparrow SMS** - Popular SMS gateway in Nepal
- **Sparrow A2P** - Application-to-Person messaging
- **Ebeema SMS** - Reliable SMS service provider
- **EasySMS** - Simple SMS integration
- **SMS24x7** - 24/7 SMS service
- **Custom Gateway** - Add your own SMS provider

## Features

- 🚀 **Multiple Gateway Support** - Switch between different SMS providers
- 📱 **OTP Generation & Verification** - Built-in OTP system with expiration
- 📊 **Delivery Tracking** - Track message delivery status
- 🔄 **Queue Support** - Send SMS asynchronously using Laravel queues
- 📈 **Bulk Messaging** - Send messages to multiple recipients
- 🎯 **Template Support** - Pre-defined message templates
- 📝 **Logging** - Comprehensive logging for debugging
- 🔒 **Rate Limiting** - Prevent abuse with configurable rate limits
- 📊 **Analytics** - SMS sending statistics and reports
- 🌐 **Multi-language** - Support for English and Nepali text
- 🔄 **Failover** - Automatic fallback to backup gateways
- 📱 **Laravel Notifications** - Integration with Laravel notification system

## Requirements

- PHP 8.1 or higher
- Laravel 10.0 or higher (supports Laravel 12)

## Installation

Install the package via Composer:

```bash
composer require openbackend/nepali-sms
```

Publish the configuration file:

```bash
php artisan vendor:publish --provider="OpenBackend\NepaliSms\NepaliSmsServiceProvider"
```

Run the migrations to create necessary tables:

```bash
php artisan migrate
```

## Configuration

Configure your SMS gateways in `config/nepali-sms.php`:

```php
return [
    'default' => env('NEPALI_SMS_DEFAULT_GATEWAY', 'sparrow'),
    
    'gateways' => [
        'sparrow' => [
            'driver' => 'sparrow',
            'token' => env('SPARROW_SMS_TOKEN'),
            'from' => env('SPARROW_SMS_FROM', 'SPARROW'),
        ],
        
        'sparrow_a2p' => [
            'driver' => 'sparrow_a2p',
            'client_id' => env('SPARROW_A2P_CLIENT_ID'),
            'client_secret' => env('SPARROW_A2P_CLIENT_SECRET'),
            'from' => env('SPARROW_A2P_FROM'),
        ],
        
        'ebeema' => [
            'driver' => 'ebeema',
            'username' => env('EBEEMA_USERNAME'),
            'password' => env('EBEEMA_PASSWORD'),
            'from' => env('EBEEMA_FROM'),
        ],
    ],
    
    'otp' => [
        'length' => 6,
        'expiry' => 300, // 5 minutes
        'template' => 'Your OTP code is: {otp}. Valid for {expiry} minutes.',
    ],
    
    'rate_limit' => [
        'enabled' => true,
        'max_attempts' => 5,
        'decay_minutes' => 1,
    ],
];
```

Add your gateway credentials to `.env`:

```env
NEPALI_SMS_DEFAULT_GATEWAY=sparrow

# Sparrow SMS
SPARROW_SMS_TOKEN=your_sparrow_token
SPARROW_SMS_FROM=YourApp

# Sparrow A2P
SPARROW_A2P_CLIENT_ID=your_client_id
SPARROW_A2P_CLIENT_SECRET=your_client_secret
SPARROW_A2P_FROM=YourApp

# Ebeema SMS
EBEEMA_USERNAME=your_username
EBEEMA_PASSWORD=your_password
EBEEMA_FROM=YourApp
```

## Basic Usage

### Sending a Simple SMS

```php
use OpenBackend\NepaliSms\Facades\NepaliSms;

// Send SMS using default gateway
NepaliSms::to('9841234567')
    ->message('Hello from Nepal!')
    ->send();

// Send SMS using specific gateway
NepaliSms::gateway('sparrow')
    ->to('9841234567')
    ->message('Hello from Sparrow!')
    ->send();
```

### Sending Bulk SMS

```php
$recipients = ['9841234567', '9851234567', '9861234567'];

NepaliSms::to($recipients)
    ->message('Bulk message to all recipients')
    ->send();
```

### OTP Generation and Verification

```php
// Generate and send OTP
$otp = NepaliSms::generateOtp('9841234567');

// Verify OTP
$isValid = NepaliSms::verifyOtp('9841234567', '123456');

if ($isValid) {
    // OTP is valid
} else {
    // Invalid or expired OTP
}
```

### Using Templates

```php
// Create a template
NepaliSms::createTemplate('welcome', 'Welcome {name} to our platform!');

// Send using template
NepaliSms::to('9841234567')
    ->template('welcome', ['name' => 'John'])
    ->send();
```

### Queue Support

```php
// Send SMS asynchronously
NepaliSms::to('9841234567')
    ->message('This will be sent via queue')
    ->queue();

// Send with delay
NepaliSms::to('9841234567')
    ->message('This will be sent after 5 minutes')
    ->delay(now()->addMinutes(5))
    ->queue();
```

## Advanced Features

### Laravel Notifications Integration

Create a notification class:

```php
php artisan make:notification WelcomeMessage
```

```php
use OpenBackend\NepaliSms\Channels\NepaliSmsChannel;
use OpenBackend\NepaliSms\Messages\NepaliSmsMessage;

class WelcomeMessage extends Notification
{
    public function via($notifiable)
    {
        return [NepaliSmsChannel::class];
    }

    public function toNepaliSms($notifiable)
    {
        return NepaliSmsMessage::create()
            ->message('Welcome to our platform!')
            ->gateway('sparrow');
    }
}
```

Send notification:

```php
$user->notify(new WelcomeMessage());
```

### Delivery Tracking

```php
$message = NepaliSms::to('9841234567')
    ->message('Track this message')
    ->send();

// Check delivery status
$status = NepaliSms::getDeliveryStatus($message->id);

// Get detailed delivery report
$report = NepaliSms::getDeliveryReport($message->id);
```

### Analytics and Reports

```php
// Get SMS statistics for today
$stats = NepaliSms::getStatistics();

// Get statistics for specific date range
$stats = NepaliSms::getStatistics(
    from: now()->subDays(7),
    to: now()
);

// Get gateway-wise statistics
$gatewayStats = NepaliSms::getGatewayStatistics('sparrow');
```

### Rate Limiting

```php
// Check if user can send SMS
if (NepaliSms::canSendSms('9841234567')) {
    NepaliSms::to('9841234567')
        ->message('Rate limited message')
        ->send();
} else {
    // Rate limit exceeded
}
```

### Failover Support

```php
// Configure multiple gateways with priority
NepaliSms::gateways(['sparrow', 'ebeema', 'easysms'])
    ->to('9841234567')
    ->message('This will try multiple gateways if one fails')
    ->send();
```

## Artisan Commands

### Test SMS Gateway

```bash
php artisan nepali-sms:test --gateway=sparrow --to=9841234567
```

### Generate OTP

```bash
php artisan nepali-sms:otp --generate --phone=9841234567
```

### Verify OTP

```bash
php artisan nepali-sms:otp --verify --phone=9841234567 --code=123456
```

### View Statistics

```bash
php artisan nepali-sms:stats
```

### Clear Old SMS Logs

```bash
php artisan nepali-sms:cleanup --days=30
```

## Events

The package fires several events that you can listen to:

- `MessageSending` - Before sending SMS
- `MessageSent` - After SMS is sent successfully
- `MessageFailed` - When SMS sending fails
- `OtpGenerated` - When OTP is generated
- `OtpVerified` - When OTP is verified
- `OtpExpired` - When OTP expires

Example listener:

```php
Event::listen(MessageSent::class, function ($event) {
    Log::info('SMS sent successfully', [
        'to' => $event->message->to,
        'gateway' => $event->message->gateway,
        'id' => $event->message->id,
    ]);
});
```

## Testing

Run the test suite:

```bash
composer test
```

Run tests with coverage:

```bash
composer test-coverage
```

## Security

If you discover any security-related issues, please email security@openbackend.dev instead of using the issue tracker.

## Contributing

Please see [CONTRIBUTING.md](CONTRIBUTING.md) for details.

## License

The MIT License (MIT). Please see [License File](LICENSE) for more information.

## Credits

- [Rudra Ramesh](https://github.com/rudraramesh)
- [OpenBackend](https://github.com/openbackend)
- [All Contributors](../../contributors)

## Support

- [Documentation](https://github.com/openbackend/laravel-nepali-sms/wiki)
- [Issues](https://github.com/openbackend/laravel-nepali-sms/issues)
- [Discussions](https://github.com/openbackend/laravel-nepali-sms/discussions)
