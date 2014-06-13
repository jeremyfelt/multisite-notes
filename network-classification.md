# Thoughts

* There are many times when `! is_multisite()` is used purely to perform single site behavior. While going down the road of closed and open networks, we may want to consider something along the lines of `is_single_site()` that makes it easier to follow when something specific to single site is happening. This could make even more sense if we start doing something like `is_multisite( 'closed' )` or `is_closed_network()` or ...

# Uses of is_multisite()

As of changeset 28572, there are 255 uses of `is_multisite()` in core.

## wp-activate.php

1. Possible redirect to `wp-login.php?action=register` if not multisite.
	* This check should be for an open multisite network.
	* A closed multisite network should redirect to home.

## wp-login.php

1. Used to determine what the login header URL and title should be. In single site, this is a link to `https://wordpress.org` and `Powered by WordPress`. In multisite, the network's home URL and name are used.
	* This seems like appropriate behavior for closed and open networks.
1. If `retrieve_password()` in multisite, the network's name is used rather than the site's name when sending the password reset email.
	* This seems like appropriate behavior for closed and open networks.
1. Used when handling `action=register` to reroute to `wp-signup.php` if multisite is enabled.
	* This should only redirect for open multisite networks.
1. For `action=login` (and as a default), `is_multisite()` is used twice to determine where to redirect a logged in user.
	* This seems valid for open and closed networks. It may be worth considering what `get_active_blog_for_user()` means in this scenario.

## wp-settings.php

1. Used to initialize multisite with `ms-blogs.php` and `ms-settings.php` if enabled.
	* This is necessary for open and closed networks.
1. Used to load additional multisite specific functionality through `ms-functions.php`, `ms-default-filters.php`, and `ms-deprecated.php`.
	* This is necessary for open and closed networks.
1. Used to load network activated plugins.
	* This is necessary for open and closed networks.
1. Used to fire `ms_cookie_constants()`
	* This is necessary for open and closed networks.
1. Used to check a site's status via `ms_site_check()` before loading it.
	* This is valid for open and closed networks, though the flow in `ms_site_check()` around inactive, deleted, etc... "blogs" is worth looking at.

## wp-signup.php

1. Used in the same fashion as `wp-activate.php` to redirect to `wp-login.php?action=register` if not multisite.
	* This should exist on an open multisite network.
	* A closed multisite newtwork should redirect to home.

## wp-admin/about.php

1. Used to determine what language to display for the link back to updates.
	* I'm not entirely sure this should exist at all. If it does, seems to work for open and closed networks.

## wp-admin/admin-header.php

1. Used to add a `multisite` class to `<body>` in the admin header.
	* This is valid for both open and closed networks.

## wp-admin/admin.php

1. Used to redirect to `upgrade.php` if a DB version mismatch is detected and this is not multisite.
	* This is valid for both open and closed networks.

## wp-admin/index.php

1. Used in determining what help message to provide for the dashboard. If multisite, the message about recent and popular plugins is left out.
	* This could be modified to change if a site has the ability to install new plugins.

## wp-admin/menu.php

1. Add "My Sites" to the menu.
	* This is applicable to open and closed networks.
1. Help determine if `wp_get_update_data()` should fire. Currently for non-multisite or for super admins.
	* This is applicable to open and closed networks.
1. If not multisite, updates for plugins, themes, and core are shown in the menu.
	* This is applicable to open and closed networks.
1. If not multisite, add the 'Editor' link to the appearance menu for editing theme files.
	* This is applicable to open and closed networks.
1. If not multisite, show a counter next to plugins to indicate available updates.
	* This is applicable to open and closed networks.
1. If not multisite, add links for 'Add New' and 'Editor' under plugins.
	* This could be modified if a site has the ability to install new plugins. It would likely not make sense to add the Editor.
1. Add a 'Delete Site' option to the tools menu if not the main site.
	* This is applicable to open and closed networks.
1. If not yet multisite, but `WP_ALLOW_MULTISITE` is defined, show a Network Setup option under tools.
	* This is applicable to open and closed networks.

## wp-admin/ms-delete-site.php

1. A check for multisite is performed before any site deletion logic is processed.
	* This is applicable to open and closed networks.

## wp-admin/my-sites.php

1. A check for multisite is performed before showing the My Sites page.
	* This is applicable to open and closed networks.

## wp-admin/includes/admin.php

1. Used to load `wp-admin/includes/ms.php` and `wp-admin/includes/ms-deprecated.php`
	* This is applicable to open and closed networks.

## wp-admin/includes/ajax-actions.php

1. Used before fully firing `wp_ajax_autocomplete_user()`. `wp_die( -1 )` is used if not multisite.
	* This is applicable to open and closed networks.

## wp-admin/includes/class-wp-importer.php

1. Used to determine if `switch_to_blog()` should be used when `set_blog()` fires during an import.
	* This is applicable to open and closed networks.

## wp-admin/includes/class/wp-plugins-list-table.php

1. Used when preparing items for the plugin list table. If multisite is enabled, the advanced plugins list table will display.
	* This is likely applicable to open and closed networks, though could be used to display slightly different data.
1. Used to filter out network only plugins.
	* This is applicable to open and closed networks.
1. Used to help show the update and delete options in either single site mode or in the network admin.
	* The behavior of this could change somewhat depending on what restrictions are changed as part of a closed network. This does apply at some level to both open and closed.
1. Do not show `plugin_rows()` if multisite and not in the network admin **and** looking at mustuse or dropin plugins.
	* This could change, though is probably applicable to both closed and open networks. It depends on what access a site on a closed network should have to mu-plugins and drop-in plugins.
1. Used to prevent the plugin deletion option from appearing in admin area of an individual site.
	* This could change if more management flexibility is given to individual sites in a closed network.
1. Used in combination with `$screen->in_admin( 'network' )` to determine if the edit plugin option should be shown for an individual site.
	* This could change if more management flexibility is given to individual sites in a closed network. This also seems scary. :)

## wp-admin/includes/class-wp-themes-list-table.php

1. Used to guide a network admin to enable or install more themes for an individual site if only one is available in the list table.
	* This could change if more management flexibility is given to individual sites for installing themes in a closed network.
1. Used to show theme deletion options for single site only.
	* This could change if individual sites are given the ability to add and remove their own themes.

## wp-admin/includes/class-wp-upgrader-skins.php

1. Used to determine if a plugin should be network activated once installed by a network administrator.
	* This is applicable to both open and closed networks.

## wp-admin/includes/class-wp-upgrader.php

1. Enter maintenance mode if multisite is enabled and more than one plugins is being bulk upgraded.
	* Applicable to both open and closed networks.
	* There's a @todo here around only kicking this in for individual sites if possible. That could be a cool feature, especially if sites on a closed network have more control.
1. Enter maintenance mode if multisite is enabled and more than one theme is being bulk upgraded.
	* Applicable to both open and closed networks.
	* As with plugins, there is a @todo around maintenance mode for individual sites.

## wp-admin/includes/class-wp-users-list-table.php

1. Show a "Remove" option to remove users in the users list table rather than the option to delete users from the network entirely.
	* This could change if sites are more autonomous around their users. An option to delete a user could be shown if that user is not a member of any other sites on the network.
1. Used when displaying rows in the users list table. If it is multisite and the user has no role on this site, the user is not shown.
	* This seems to be less applicable with closed networks. Open networks can have various registration needs that need to be fulfilled before a user should appear on the site's user list. Closed networks want the user to appear immediately. It may be possible that nothing changes directly here, but that modified closed network behavior as a whole causes a change.
1. When not multisite, show the delete user option on a single row.
	* The determination to display this differently could change one day in a closed network configuration.
1. When multisite, show the remove user option on a single row.
	* The determination to display this differently could change one day in a closed network configuration.

## wp-admin/includes/dashboard.php

1. If not multisite, or if in the network admin, bring in a feed of popular plugins.
	* This use is purely for single site detection. Another check is used for network admin.
	* This could change if more plugin management options are provided to sites on a closed network.

## wp-admin/includes/deprecated.php

## wp-admin/includes/export.php

## wp-admin/includes/file.php

## wp-admin/includes/media.php

## wp-admin/includes/misc.php

## wp-admin/includes/plugin.php

## wp-admin/includes/schema.php

## wp-admin/includes/theme.php

## wp-admin/includes/update.php

## wp-admin/includes/upgrade.php

## wp-admin/includes/user.php

