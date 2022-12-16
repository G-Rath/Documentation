---
title: WordPress Developer's Guide
subtitle: WordPress Best Practices
description: A list of suggestions for developing WordPress sites on Pantheon.
cms: "WordPress"
contenttype: [guide]
categories: [manage]
newcms: [wordpress]
audience: [development]
product: [--]
integration: [--]
tags: [workflow, security, composer]
reviewed: "2022-05-16"
layout: guide
showtoc: true
permalink: docs/guides/wordpress-developer/wordpress-best-practices
anchorid: wordpress-best-practices
---

This article provides suggestions, tips, and best practices for developing and managing WordPress sites on the Pantheon platform.

## Development

* We recommend using an [IDE](https://en.wikipedia.org/wiki/Comparison_of_integrated_development_environments#PHP), or a text editor designed for development like [Atom](https://atom.io/), [Sublime Text](https://www.sublimetext.com/), or [Brackets](https://github.com/adobe/brackets/).

* Do not modify core WordPress files as it can cause unintended consequences, and can [prevent you from updating your site regularly](/core-updates#apply-upstream-updates-manually-from-the-command-line-to-resolve-merge-conflicts). If you need to modify any WP functionality, do it as a custom or [Must Use](/guides/wordpress-configurations/mu-plugin) plugin, which adheres to the [WP.org Plugin best practices](https://developer.wordpress.org/plugins/the-basics/best-practices/).

* Use [Redis](/guides/object-cache). Redis is an open-source, networked, in-memory, key-value data store that can be used as a drop-in caching backend for your WordPress site. Pantheon makes it super simple and you'll be able to cache a lot of database queries in WordPress.

* Use [wp-cfm](/guides/wordpress-configurations/wp-cfm). It lets you store settings from the `wp_options` table in Git and pull it into the database. A lot of WordPress stuff is option-heavy and you can spend a lot of time trying to figure out what you missed between environments. This is true for all WordPress sites, but especially helpful on Pantheon where you have at least three environments you will need to reconfigure every time.

* Use [Grunt](https://gruntjs.com) or [Gulp](https://github.com/gulpjs/gulp) to aggregate JS/CSS on your local development environment rather than relying on the server to do it for you. This helps speed up your workflow by minimizing redundant tasks.

* When developing custom plugins or themes, it is best to abide by the [WordPress Coding Standards](https://make.wordpress.org/core/handbook/best-practices/coding-standards/) for efficiency and ease of collaboration.

### Plugins

* Add [Composer](/guides/composer) and pull your WordPress plugins from [wpackagist.org](https://wpackagist.org/). WordPress Packagist mirrors the WordPress.org plugin repository and adds a composer.json file so things play nice. It makes future debugging much simpler should you need to switch between multiple plugin or WordPress versions to see what caused something to break. While [committing Composer dependencies is generally not recommended](https://getcomposer.org/doc/faqs/should-i-commit-the-dependencies-in-my-vendor-directory.md), you will have to commit the dependencies that Composer downloads on Pantheon since running `composer install` on the environments is not supported (just as Git submodules are not supported).

* If you have a custom plugin that retrieves a specific post (or posts), instead of using `wp_query()` to retrieve it, use the `get_post()` function. While wp_query has its uses, the [get_post](https://developer.wordpress.org/reference/functions/get_post/) function is built specifically to retrieve a WordPress Post object, and does so very efficiently.

* Don't use plugins that create files vital to your site logic that you aren't willing to track in Git. Sometimes they're dumped in uploads, sometimes not, and you'll likely have difficulty trying to figure it out later. You'd be surprised how many uploads-type plugins rely on `.htaccess` files — avoid those as well.


### Themes

* In your theme, use a simple PHP `include()` instead of WordPress's [get_template_part()](https://codex.wordpress.org/Function_Reference/get_template_part). The overhead is heavy if your use case is simply adding in another sub-template file. For example:

  ```php
  <?php get_template_part('content', 'sidebar'); ?>
  <?php include('content-sidebar.php'); ?>
  ```
  
#### Manage License Keys for Themes or Plugins

There are many plugins and themes in WordPress that require license keys. Since Dev and Multidev are the only writable environments in SFTP mode, it is best practice to associate the license key in a domain so you can easily update and deploy the updates to Test and Live environments. 


## Testing

* Run [Launch Check](/guides/wordpress-pantheon/wordpress-launch-check) to review errors and get recommendations on your site's configurations.

* Automate testing with [Behat](/guides/behat). Adding automated testing into your development workflow will help you deliver higher quality WordPress sites.

## Live

* We recommend using HTTPS. For more information, see [HTTPS on Pantheon's Global CDN](/guides/global-cdn/https)

* Verify [Global CDN caching](/guides/global-cdn/test-global-cdn-caching) is working on your site.

* Follow our [Frontend Performance](/guides/frontend-performance) guide to tune your WordPress site.

## Avoid XML-RPC Attacks

The `/xmlrpc.php` script is a potential security risk for WordPress sites. It can be used by bad actors to brute force administrative usernames and passwords, for example. This can be surfaced by reviewing your site's `nginx-access.log` for the Live environment. If you leverage [GoAccess](/guides/logs-pantheon/nginx-access-logs), you might see something similar to the following:

```none
2 - Top requests (URLs)                                  Total: 366/254431

Hits Vis.     %   Bandwidth Avg. T.S. Cum. T.S. Max. T.S. Data
---- ---- ----- ----------- --------- --------- --------- ----
2026   48 0.77%   34.15 KiB   1.27  s  42.74 mn  38.01  s /xmlrpc.php
566   225 0.21%   12.81 MiB   4.08  s  38.45 mn  59.61  s /
262    79 0.10%  993.71 KiB   2.32  s  10.14 mn  59.03  s /wp-login.php
```

Pantheon recommends disabling XML-RPC, given the WordPress Rest API is a stronger and more secure method for interacting with WordPress via external services.

Pantheon blocked requests to `xmlrpc.php` by default in the [WordPress 5.4.2 core release](/changelog/2020/07#wordpress-542). If your version of WordPress is older than this, you can block `xmlrpc.php` attacks by applying your [upstream updates](/core-updates).

### Enable XML-RPC via Pantheon.yml

<Alert title="Note"  type="info" >

XML-RPC is not recommended on the Pantheon platform. Pantheon does not support XML-RPC if it is enabled. 

</Alert>

You can re-enable access to XML-RPC for tools and plugins that require it, such as [Jetpack](https://jetpack.com/) or the WordPress mobile app. 

<Partial file="jetpack-enable-xmlrpc.md" />

### Disable XML-RPC via a Custom Plugin

This method has the advantage of being toggleable without deploying code, by activating or deactivating a custom plugin. The result of creating and activating this plugin is that exploitable XMLRPC methods will no longer be available via POST requests.

1. [Set the connection mode to SFTP](/guides/sftp) for the Dev or target Multidev environment via the Pantheon Dashboard or with [Terminus](/terminus):

  ```bash{promptUser: user}
  terminus connection:set <site>.<env> sftp
  ```

1. Use [Terminus](/terminus) and [WP-CLI's `scaffold plugin`](https://developer.wordpress.org/cli/commands/scaffold/plugin/) command to create  a new custom plugin.

  In the following example, replace `my-site` with your Pantheon site name, and `disable-xmlrpc` with your preferred name for this new plugin:

  ```bash{promptUser: user}
  terminus wp my-site.dev -- scaffold plugin disable-xmlrpc
  ```

1. Add the following lines to the main PHP plugin file:

  ```php:title=wp-content/plugins/disable-xmlrpc/disable-xmlrpc.php
  # Disable /xmlrpc.php
  add_filter('xmlrpc_methods', function () {
    return [];
  }, PHP_INT_MAX);
  ```

	If your site uses a nested web root directory, you must include that directory in the path. For example, if your nested web root is `/wp`, use `/wp/xmlrpc.php` instead of `/xmlrpc.php` 

1. Activate the new plugin from within the WordPress admin dashboard, or via Terminus and WP-CLI:

  ```bash{promptUser: user}
  terminus wp my-site.dev -- plugin activate disable-xmlrpc
  ```

1. Commit your work, deploy code changes then activate the plugin on Test and Live environments.

## Avoid WordPress Login Attacks

<Partial file="wp-login-attacks.md" />

## Disable Anonymous Access to WordPress Rest API

The WordPress REST API is enabled for all users by default. To improve the security of a WordPress site, you can disable the WordPress REST API for anonymous requests, to avoid exposing admin users. This action improves site safety and reduces unexpected errors that can result in compromised WordPress core functionalities.

The following function ensures that anonymous access to your site's REST API is disabled and that only authenticated requests will work. You can add this code sample to a theme's `functions.php` file or to a must-use plugin:

```php
// Disable WP Users REST API for non-authenticated users (allows anyone to see username list at /wp-json/wp/v2/users)
add_filter( 'rest_authentication_errors', function( $result ) {
	if ( true === $result || is_wp_error( $result ) ) {
		return $result;
	}

	if ( ! is_user_logged_in() ) {
		return new WP_Error(
			'rest_not_logged_in',
			__( 'You are not currently logged in.' ),
			array( 'status' => 401 )
		);
	}

	return $result;
});
```

## Security Headers

Pantheon's Nginx configuration [cannot be modified](/guides/platform-considerations/platform-site-info#htaccess) to add security headers, and many solutions (including plugins) written about security headers for WordPress involve modifying the `.htaccess` file for Apache-based platforms.

There are plugins for WordPress that do not require `.htaccess` to set security headers, but header specifications may change more rapidly than the plugins can keep up with. In those cases, you may want to define the headers yourself.

Adding code like the example below in a plugin (or [mu-plugin](/guides/wordpress-configurations/mu-plugin)) can help add security headers for WordPress sites on Pantheon, or any other Nginx-based platform. Do not add this to your theme's `functions.php` file, as it will not be executed for calls to the REST API.

The code below is only an example to get you started. You'll need to modify it to match your needs, especially the Content Security Policy. Tools like [SecurityHeaders.com](https://securityheaders.com) can help to check your security headers, and link to additional information on how to improve your security header profile.

```php
function additional_securityheaders( $headers ) {
  if ( ! is_admin() ) {
    $headers['Referrer-Policy']             = 'no-referrer-when-downgrade'; //This is the default value, the same as if it were not set.
    $headers['X-Content-Type-Options']      = 'nosniff';
    $headers['X-XSS-Protection']            = '1; mode=block';
    $headers['Permissions-Policy']          = 'geolocation=(self "https://example.com") microphone=() camera=()';
    $headers['Content-Security-Policy']     = "script-src 'self'";
    $headers['X-Frame-Options']             = 'SAMEORIGIN';
  }

  return $headers;
}
add_filter( 'wp_headers', 'additional_securityheaders' );
```

**Note:** Because the headers are applied by PHP code when WordPress is invoked, they will not be added when directly accessing assets like `https://example.com/wp-content/uploads/2020/01/sample.json`.