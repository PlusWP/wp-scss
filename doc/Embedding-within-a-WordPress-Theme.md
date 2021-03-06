# Embedding within a WordPress Theme

For some reasons, you want to bundle `wp-scss` plugin within a theme or a plugin. 2 things to know:

1. it’s totally feasible and it’s officially supported;
1. theme bundling will take over plugin bundling, or even the official plugin.

## Basics of embedding

`wp-scss` plugin provides a special file for embedding. It does the dirty job and let you the hand on what to do before dispatching stuff.

### First: embedding
The first part of embedding is… embedding:

```php
// wp-content/themes/your-theme/functions.php

require dirname(__FILE__) . '/vendor/wp-scss/bootstrap-for-theme.php';
```

At this point, the plugin is available these way:
* `$WPScssPlugin` variable available within the global scope;
* `WPScssPlugin::getInstance()` will always return you the active plugin instance, whatever the scope is (longer but safer).

### Second: configuring

You can do whatever you want with the plugin. It hasn’t been  initialized yet. Deal with folders, change compilation strategy or whatever.

You can register any SCSS stylesheet, it has no incidence.
There won’t be parsed at this moment.

### Third: initializing

You now have to plug `wp-scss` on WordPress events to make the plugin work.

This is as simple as the following code:

```php
// wp-content/themes/your-theme/functions.php

$scss = WPScssPlugin::getInstance();
$scss->dispatch();
```

The `dispatch` method deals with everything you need.

You’re done!

## Manual registration of scheduled tasks

However, by embedding the plugin, you have to manually activate the garbage collector. This feature cleans every compiled file older than 5 days, every day.

Regarding of where you embed the plugin, you have to **register the task only once**:

```php
// wp-content/themes/your-theme/functions.php
// …

$scss->install();
```

The same way, you also have to unregister the garbage collector at the relevant moment:

```php
// wp-content/themes/your-theme/functions.php
// …

$scss->uninstall();
```

Within a plugin, it would looks like this:

```php
// wp-content/themes/your-theme/functions.php
// …

register_activation_hook(__FILE__, array($scss, 'install'));
register_deactivation_hook(__FILE__, array($scss, 'uninstall'));
```

## If the plugin is installed aside

Remember that if `wp-scss` exists in the `wp-content/plugins` folder, you can use this code as dependency of your theme or plugin.

To always rely on the latest up-to-date version, you should detect the existence of the plugin **after all plugins have been loaded**. You could then bundle your own `wp-scss` copy as legacy fallback, just in case.

```php
// wp-content/themes/my-theme/functions.php

add_action('plugins_loaded', 'register_scss_fallback');

function register_scss_fallback(){
	if (!class_exists('WPScssPlugin')){
		require dirname(__FILE__) . '/vendor/wp-scss/bootstrap-for-theme.php';
		WPScssPlugin::getInstance()->dispatch();
		// we’re done, everything works as if the plugin is activated
	}
}
```
