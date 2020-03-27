# Laravel Queue Monitor

[![Latest Stable Version](https://img.shields.io/packagist/v/romanzipp/laravel-queue-monitor.svg?style=flat-square)](https://packagist.org/packages/romanzipp/laravel-queue-monitor)
[![Total Downloads](https://img.shields.io/packagist/dt/romanzipp/laravel-queue-monitor.svg?style=flat-square)](https://packagist.org/packages/romanzipp/laravel-queue-monitor)
[![License](https://img.shields.io/packagist/l/romanzipp/laravel-queue-monitor.svg?style=flat-square)](https://packagist.org/packages/romanzipp/laravel-queue-monitor)
[![Code Quality](https://img.shields.io/scrutinizer/g/romanzipp/laravel-queue-monitor.svg?style=flat-square)](https://scrutinizer-ci.com/g/romanzipp/laravel-queue-monitor/?branch=master)
[![Build Status](https://img.shields.io/scrutinizer/build/g/romanzipp/laravel-queue-monitor.svg?style=flat-square)](https://scrutinizer-ci.com/g/romanzipp/Laravel-Queue-Monitor/build-status/master)

This package offers monitoring like [Laravel Horizon](https://laravel.com/docs/horizon) for database queue.

## Features

* Monitor all jobs like [Laravel Horizon](https://laravel.com/docs/horizon), but not only for redis
* Handles failed jobs with exception
* Support for milliseconds
* Model for Queue Monitorings

## Installation

```
composer require romanzipp/laravel-queue-monitor
```

**If you use Laravel 5.5+ you are already done, otherwise continue.**

Add Service Provider to your app.php configuration file:

```php
romanzipp\QueueMonitor\Providers\QueueMonitorProvider::class,
```

## Configuration

Copy configuration to config folder:

```
$ php artisan vendor:publish --provider="romanzipp\QueueMonitor\Providers\QueueMonitorProvider"
```

Migrate the Queue Monitoring table. The table name itself can be configured in the config file.

```
$ php artisan migrate
```

## Usage

To monitor a job, simply add the `romanzipp\QueueMonitor\Traits\QueueMonitor` Trait.

```php
use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use romanzipp\QueueMonitor\Traits\QueueMonitor; // <---

class ExampleJob implements ShouldQueue
{
    use Dispatchable;
    use InteractsWithQueue;
    use Queueable;
    use SerializesModels;
    use QueueMonitor; // <---
}
```

### Set progress

You can set a **progress value** (0-100) to get an estimation of a job progression.

This example job loops through a large amount of users and updates it's progress value with each chunk iteration.

```php
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Database\Eloquent\Collection;
use romanzipp\QueueMonitor\Traits\QueueMonitor;

class ExampleJob implements ShouldQueue
{
    use QueueMonitor;

    public function handle()
    {
        $usersCount = User::count();

        $perChunk = 50;
        $currentChunk = 0;

        User::query()
            ->chunk($perChunk, function (Collection $users) use (&$currentChunk, $perChunk, $usersCount) {

                $this->queueProgress(++$currentChunk * $perChunk / $usersCount * 100);

                foreach ($users as $user) {
                    // ...
                }
            });
    }
}
``` 

### Set custom data

This package also allows to set custom data on the monitoring model.

```php
use Illuminate\Contracts\Queue\ShouldQueue;
use romanzipp\QueueMonitor\Traits\QueueMonitor;

class ExampleJob implements ShouldQueue
{
    use QueueMonitor;

    public function handle()
    {
        $this->queueData(['foo' => 'Bar']);
        
        // WARNING! This is overriding the monitoring data
        $this->queueData(['bar' => 'Foo']);
    }
}
``` 

### Retrieve processed Jobs

```php
use romanzipp\QueueMonitor\Models\Monitor;

/** @var Monitor $job */
$job = Monitor::query()->first();

// Exact start & finish dates with milliseconds
$job->getStartedAtExact();
$job->getFinishedAtExact();

// If the job is still running, get the estimated seconds remaining
$job->getRemainingSeconds();

// Retrieve any data that has been set while execution
$job->getData();
```

### Model Scopes

```php
use romanzipp\QueueMonitor\Models\Monitor;

// Filter by Status
Monitor::failed();
Monitor::succeeded();

// Filter by Date
Monitor::lastHour();
Monitor::today();

// Chain Scopes
Monitor::today()->failed();
```

## ToDo

- [ ] Add Job & Artisan Command for automatic cleanup of old database entries

## Upgrading from 1.0 to 2.0

### The Monitor Model

```diff
- ->basename()
- ->basename
+ ->getBaseame()
```

```diff
- ->parsed_data
+ ->getData()
```

```diff
- remaing_seconds
+ getRemainingSeconds()
```

```diff
- startedAtExact()
+ getStartedAtExact()
```

```diff
- finishedAtExact()
+ getFinishedAtExact()
```

----

This package was inspired by gilbitron's [laravel-queue-monitor](https://github.com/gilbitron/laravel-queue-monitor) which is not maintained anymore.
