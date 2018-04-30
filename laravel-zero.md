theme: Work Projectors
footer: Laravel _Zero_ - Laravel Austin
build-lists: true
slidenumbers: true

# [fit] Laravel **Zero**

---

# [fit] Micro-framework 
# [fit] for **command-line** applications

Provides an elegant starting point for your console application

^ Built on top of the Laravel components
^ You can optionally install Laravel's Eloquent and logging
^ Laravel Zero is unofficial, and customized for optimized building of command line applications

---

# [fit] Installation

* `composer global require "laravel-zero/installer"`
* `composer create-project --prefer-dist laravel-zero/laravel-zero movie-cli
`

^ Much like Laravel, Laravel Zero has its own installer. But you can also use the composer create-project syntax

---

# [fit] Commands

By default Laravel **_Zero_** comes with a `InspiringCommand` as an example

`php <your-app-name> make:command <NewCommand>`

---

## [fit] Desktop Notifications

![fit inline](https://raw.githubusercontent.com/nunomaduro/laravel-desktop-notifier/stable/docs/icon.png)

```
$this->notify("Hello Web Artisan", "Love beautiful..", "icon.png");
```

---

## [fit] Tasks

![fit right](https://raw.githubusercontent.com/nunomaduro/laravel-console-task/master/docs/example.png)

```php
$this->task("Installing Laravel", function () {
    return true;
});

$this->task("Doing something else", function () {
    return false;
});
```

---

## [fit] Interactive Menus

![fit right](https://raw.githubusercontent.com/nunomaduro/laravel-console-menu/master/docs/example.png)

```php
$option = $this->menu('Pizza menu', [
    'Freshly baked muffins',
    'Freshly baked croissants',
    'Turnovers, crumb cake, cinnamon buns, scones',
])->open();

$this->info("You have chosen the option number #$option");
```

```php
$this->menu($title, $options)
    ->setForegroundColour('green')
    ->setBackgroundColour('black')
    ->setWidth(200)
    ->setPadding(10)
    ->setMargin(5)
    ->setExitButtonText("Abort")
    // remove exit button with
    // ->disableDefaultItems()
    ->setUnselectedMarker('❅')
    ->setSelectedMarker('✏')
    ->setTitleSeparator('*-')
    ->addLineBreak('<3', 2)
    ->addStaticItem('AREA 2')
    ->open();
```

---

# [fit] Service Providers

Laravel **_Zero_** recommends using **Service Providers** for defining concrete implementations

---

# [fit] Config

Laravel **_Zero_** utilizes the same configuration system as Laravel.

```php
'production' => true,
```

---

# [fit] Database

Laravel **_Zero_** allows you to install a Laravel's Eloquent component

`php <your-app-name> app:install database`

^ Along with Eloquent, you also gets Laravel's database migrations and database seeding

---

# [fit] Logging

`php <your-app-name> app:install log`

^ This one is pretty straight forward, it's the same logging component that comes with Laravel

---

# [fit] Filesystem

Laravel **_Zero_** ships with the Filesystem component by default

```php
use Illuminate\Support\Facades\Storage;

Storage::put("reminders.txt", "Task 1");
```

---

# [fit] Scheduler

Laravel **_Zero_** ships with the **Task Scheduling** system of Laravel

`* * * * * php /path-to-your-project/your-app-name schedule:run >> /dev/null 2>&1`

```php
public function schedule(Schedule $schedule): void
{
    $schedule->command(static::class)->everyMinute();
}
```

^ To schedule tasks you need to define their schedule in the schedule method.

---

# [fit] Environment Configuration

Laravel **_Zero_** also supports **DotEnv** based environment configuration

`php <your-app-name> app:install dotenv`

^ Much like Laravel, Laravel Zero supports `.env` environment configuration

---

# [fit] Collision

A detailed and intuitive error handler framework for console applications

![fit inline](https://raw.githubusercontent.com/nunomaduro/collision/stable/docs/example.png)

^ Collision is built on top of Whoops

---

# [fit] Tinker

Laravel **_Zero_** is a powerful REPL for Laravel

^ Jorge Gonzalez made it available for Laravel Zero as well

---

# [fit] Tests

```php
use Tests\TestCase;
use Illuminate\Support\Facades\Artisan;

class InspiringCommandTest extends TestCase
{
    /**
     * A basic test example.
     */
    public function testInspiringCommand(): void
    {
        Artisan::call('inspiring');

        $this->assertContains('Leonardo da Vinci', Artisan::output());
    }
}
```

^ By default, Laravel Zero ships with an Integration suite for testing your command line applications

---

# [fit] DEMO