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
