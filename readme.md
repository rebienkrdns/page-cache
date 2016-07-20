# Laravel Page Cache

This package allows you to easily cache responses as static files on disk for lightning fast page loads.

- [Introduction](#introduction)
- [Installation](#installation)
  - [Service Provider](#service-provider)
  - [Middleware](#middleware)
  - [URL rewriting](#url-rewriting)
- [Usage](#usage)
  - [Using the middleware](#using-the-middleware)
  - [Clearing the cache](#clearing-the-cache)
- [License](#license)

---

## Introduction

While static site builders such as [Jekyll](https://jekyllrb.com/) and [Jigsaw](http://jigsaw.tighten.co/) are extremely popular these days, some still prefer to build dynamic PHP sites. This allows them to easily add dynamic functionality wherever needed, and also means that there's no build step involved in pushing updates to the site.

That said, for truly static pages on a site there really is no reason to have to boot up a full PHP app just to serve a static page. Serving a simple HTML page from disk is infinitely faster and less taxing on the server.

The solution? Full page caching. Using the middleware included in this package, you can selectively cache the response to disk for any given request. Subsequent calls to the same page will be served directly as a static HTML page!

## Installation

Install the `page-cache` package with composer:

```
$ composer require silber/page-cache
```

### Service Provider

Open `config/app.php` and add a new item to the `providers` array:

```php
Silber\PageCache\LaravelServiceProvider::class,
```

### Middleware

Open `app/Http/Kernel.php` and add a new mapping to the `routeMiddleware` property:

```php
protected $routeMiddleware = [
    'page-cache' => Silber\PageCache\Middleware\CacheResponse::class,
    /* ... keep the existing mappings here */
];
```

If you want to cache _all_ successful GET requests, you can instead add it to the `web` middleware group:

```php
protected $middlewareGroups = [
    'web' => [
        \Silber\PageCache\Middleware\CacheResponse::class,
        /* ... keep the existing middleware here */
    ],
];
```

The middleware is smart enough to only cache responses with a 200 HTTP status code, and only for GET requests.

### URL rewriting

In order to serve the static files directly once they've been cached, you need to properly configure your web server to check for those static files.

If you're using **nginx**, you should update your `location` block's `try_files` directive to include a check in the `page-cache` directory:

```nginxconf
location / {
    try_files $uri $uri/ /page-cache/$uri.html /index.php?$query_string;
}
```

If you're using **apache**, add the following before the block labeled `Handle Front Controller`:

```apacheconf
# Serve Cached Page If Available...
RewriteCond %{DOCUMENT_ROOT}/page-cache%{REQUEST_URI}.html -f
RewriteRule . page-cache%{REQUEST_URI}.html [L]
```

## Usage

### Using the middleware

> **Note:** If you've added the middleware to the global `web` group, you can skip this part entirely.

To cache the response of a given request, use the `page-cache` middleware:

```php
Route::get('posts/{slug}', [
    'uses' => 'PostController@show',
    'middleware' => 'page-cache',
]);
```

Every post will now be cached to a file under the `public/page-cache` directory, closely matching the URL structure of the request. All subsequent  requests for this post will be served directly from disk, not even hitting your app.

### Clearing the cache

Since the responses are cached to disk as static files, any updates to those pages in your app will not be reflected on your site. To update pages on your site, you should clear the cache with the following command:

```
php artisan page-cache:clear
```

As a rule of thumb, it's good practice to add this to your deployment script. That way, whenever you push an update to your site the page cache will be automatically cleared.

If you're using [Forge](https://forge.laravel.com)'s Quick Deploy feature, you should add this line to the end of your Deploy Script. This'll ensure that the cache is cleared whenever you push an update to your site.

## License

The Page Cache package is open-sourced software licensed under the [MIT license](http://opensource.org/licenses/MIT)
