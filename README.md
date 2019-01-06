Sitemap for Laravel
===================

[![Build Status](https://travis-ci.org/dwightwatson/sitemap.png?branch=master)](https://travis-ci.org/dwightwatson/sitemap)
[![Total Downloads](https://poser.pugx.org/watson/sitemap/downloads.svg)](https://packagist.org/packages/watson/sitemap)
[![Latest Stable Version](https://poser.pugx.org/watson/sitemap/v/stable.svg)](https://packagist.org/packages/watson/sitemap)
[![Latest Unstable Version](https://poser.pugx.org/watson/sitemap/v/unstable.svg)](https://packagist.org/packages/watson/sitemap)
[![License](https://poser.pugx.org/watson/sitemap/license.svg)](https://packagist.org/packages/watson/sitemap)

Sitemap is a slightly smarter alternative for generating sitemaps for dynamic sites. As opposed to manually defining all your routes or letting a crawler run wild through your site, this package lets you pass in your Eloquent models/queries directly and generate sitemaps on-the-fly. In addition, if you have more than 50,000 routes (and you're hitting Google's sitemap limit) this package will automatically break your sitemaps with an index sitemap.

Read more about sitemaps and how to use them efficiently on [Google Webmaster Tools](https://support.google.com/webmasters/answer/156184?hl=en).

## Installation
First, require the package through Composer.

```sh
composer require watson/sitemap
```

Then, install the sitemap generator command, where you will define your sitemap's routes.

```sh
php artisan sitemap:install
```

### Older versions
Please visit the other branches of this repo for Laravel 4.x or PHP 5.x support. Be aware that older versions of this package required manual route definitions.

## Usage

Open up your new service provider in `app\Console\Commands\GenerateSitemapCommand.php`;

You're more than welcome to move or rename this as you like, as long as it continues to extend the base generate command provided by this package.

```php
use App\{Post, User};
use Watson\Sitemap\Registrar;
use Watson\Sitemap\Commands\GenerateCommand;
use Watson\Sitemap\Enums\{ChangeFrequency, Priority};

class SitemapGenerateCommand extends GenerateCommand
{
    /**
     * Register the sitemaps tags.
     *
     * @param  \Watson\Sitemap\Registrar  $sitemap
     * @return void
     */
    public function register(Registrar $sitemap): void
    {
        // Add a specific path.
        $sitemap->add('/contact')
            ->changes(ChangeFrequency::NEVER);

        // Path is just an alias for add.
        $sitemap->path('/contact')
            ->priority(Priority::HIGHEST);

        // Or use a named route.
        $sitemap->path(route('users.show', 1))
            ->modified(now()->subDays(2));

        // You can also pass in a model, it'll use updated_at for the lastModified attribute.
        $sitemap->model(User::class, function ($user) {
            return route('users.show', $user);
        });

        $sitemap->model(Post::whereNotNull('published_at'))
            ->changes(ChangeFrequency::DAILY)
            ->priority(Priority::HIGHEST)
            ->withRoute(function ($post) {
                return route('posts.show', $post);
            });
    }
}
```

To generate your sitemap(s) simply call the `generate:sitemap` command.

```sh
php artisan generate:sitemap
```

If you want to generate the sitemap dynamically on a schedule you can simply register the command with Laravel's scheduler.

```php
/**
 * Define the application's command schedule.
 *
 * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
 * @return void
 */
protected function schedule(Schedule $schedule)
{
    $schedule->command(Commands\GenerateSitemapCommand::class)->daily();
}
```

### Change freqencies
Constant values for change frequencies are provided by `Watson\Sitemap\Enums\ChangeFrequency`.

* `ChangeFrequency::ALWAYS`
* `ChangeFrequency::HOURLY`
* `ChangeFrequency::DAILY`
* `ChangeFrequency::WEEKLY`
* `ChangeFrequency::MONTHLY`
* `ChangeFrequency::YEARLY`
* `ChangeFrequency::NEVER`

### Priorities
Constant values for change frequencies are provided by `Watson\Sitemap\Enums\Priority`.

* `Priority::HIGHEST` - equivilant of 1
* `Priority::HIGHER` - equivilant of 0.85
* `Priority::HIGH` - equivilant of 0.7
* `Priority::MEDIUM` - equivilant of 0.5
* `Priority::LOW` - equivilant of 0.3
* `Priority::LOWER` - equivilant of 0.15
* `Priority::LOWEST` - equivilant of 0
