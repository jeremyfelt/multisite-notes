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
1. If not multisite, return `wp_dashboard_quota()` without doing anything.

## wp-admin/includes/deprecated.php

1. Helps determine whether to use `user_level` or `capabilities` when retrieving user meta in the deprecated `get_author_user_ids()`
	* Applicable to both closed and open networks. Also deprecated.
1. Helps to determine whether to use `user_level` or `capabilities` when retrieving user meta in the deprecated `get_editable_user_ids()`
	* Applicable to both closed and open networks. Also deprecated.
1. Helps to determine whether to use `user_level` or `capabilities` when retrieving user meta in the deprecated `get_nonauthor_user_ids()`
	* Applicable to both closed and open networks. Also deprecated.
1. Used to help find the correct user meta key in the deprecated WP_User_Search class.
	* Applicable to both closed and open networks. Also deprecated.

## wp-admin/includes/export.php

1. Used to return `network_home_url()` when `wxr_site_url()` fires.
	* Applicable to both closed and open networks.

## wp-admin/includes/file.php

1. Used in a `@uses` comment for `wp_handle_upload()`
1. Used to show different error messaging in `wp_handle_upload()`. If multisite is not enabled, A suggestion to check your php.ini file is shown.
	* Applicable to both open and closed networks.
	* Would think this messaging could be improved to show different messaging to a certain level of user so that troubleshooting becomes easier.
1. When uploading a file, delete the `dirsize_cache` if multisite is enabled.
	* Applicable to both closed and open networks.

## wp-admin/includes/media.php

1. Fires an action when an upload will exceed the defined upload space quota for a site.
	* Applicable to both closed and open networks.

## wp-admin/includes/misc.php

1. Avoid saving rewrite rules in `.htaccess` if multisite is active.
	* Applicable to both open and closed networks.
	* Kind of a bummer that we can't offer this to multisite installations, though everyone should use nginx anyway.
1. Avoid saving rewrite rules for IIS if multisite is active.
	* Applicable to both open and closed networks.

## wp-admin/includes/plugin.php

1. Used in `@uses` doc for `_get_dropins()`
1. Used to add `sunrise.php`, `blog-deleted.php`, `blog-inactive.php`, `blog-suspended.php` to the dropins array.
	* This could be changed for close networks to not deal with the deleted, inactive, and suspended dropins.
1. Returns false if not multisite in `is_plugin_active_for_network()`
	* Applicable to both closed and open networks.
1. Handle network wide plugin activation if this is multisite.
	* Applicable to both closed and open networks.
1. Handle network wide plugin deactivation if this is multisite.
	* Applicable to both closed and open networks.
1. Include network activated plugins in array of plugins to validate in `validate_active_plugins()` if multisite is enabled.

## wp-admin/includes/schema.php

1. Used first to set the `$is_multisite` variable inside `wp_get_db_schema()` in combination with the `WP_INSTALLING_NETWORK` constant.
1. `$is_multisite` is first used to determine if `$global_tables` should be made up of `$users_multi_table` and `$usermeta_table` or `$users_single_table` and `$usermeta_table`
	* Seems applicable to both open and closed networks, though see notes on registration and activation elsewhere.
1. `$is_multisite` is used again when the scope for the schema is global, all, and by default to add the `$ms_global_tables` to the configuration.
	* The schema laid out by `$ms_global_tables` could be reduced if a closed network does not want to allow activations and registrations. This may be more trouble than it's worth.
1. In `populate_options`, used to determine what DB version to use in a single site install.
	* Single site only. Applicable to both open and closed networks.
1. In `populate_options`, used to set `blogdescription` and `permalink_structure` options.
	* Applicable to both open and closed networks.
1. In `populate_network()`, used to set `$site_admins` array during multisite installation.
	* Occurs during network installation only.
	* Applicable to both open and closed networks.
1. In `populate_network()`, used to set the network meta option for `upload_space_check_disabled`.
	* Applicable to both open and closed networks.
1. In `populate_network()`, used to set the network meta option for `ms_files_rewriting`.
	* Applicable to both open and closed networks.
1. During upgrade from single site to multisite, setup the `$current_site` global and other data under the assumption that this site will be the main site on the new network. This does not occur on future uses of `populate_network()`.
	* Applicable for both open and closed networks.

## wp-admin/includes/theme.php

1. Show update messaging for themes in single site mode.
	* This is single site specific, but could possibly be changed to included closed networks if the management of themes at individual site levels change at all.

## wp-admin/includes/update.php

1. Show a core update nag in `update_nag()` if this is multisite and the user can update core.
	* Applicable to both open and closed networks.
1. Show update messaging in individual plugin rows if in the network admin or if in single site.
	* Applicable to both open and closed networks.
	* This use is more specific to single site. We use `is_network_admin` as the multisite check here.

## wp-admin/includes/upgrade.php

1. Used to determine if first post content should be retrieved from network options.
	* Applicable to both open and closed networks.
1. Used to determine if the first comment content should be retrieved from network options.
	* Applicable to both open and closed networks.
1.Used to determine if first page content should be retrieved from network options.
	* Applicable to both open and closed networks.
1. Set an option to show the welcome panel in single site mode. We use an else to handle multisite cases and to show a welcome panel for all non-super admin users.
	* Applicable to both open and closed networks.
	* It would be worth a look at the welcome panel messaging. I'm not sure if it has open network specific language in it.
1. If multisite, flush rewrite rules to pick up the new page.
	* Applicable to both open and closed networks.
1. Used to fire `upgrade_network()` in `wp_upgrade()` if multisite is enabled and this is the main site.
	* Applicable to both open and closed networks.
1. If multisite, update or set the db_version in the blog_versions table.
	* Applicable to both open and closed networks.
1. In `upgrade_280()`, retrieve all options from the main options table and create copies for individual site options.
	* Old and specific. Applicable to both open and closed networks.
1. In `upgrade_300()`, add a network level option for `siteurl` if this is the main site and a very old DB version.
	* Old and specific. Applicable to both open and closed networks.
1. In `pre_schema_upgrade()`, upgrade the signups and blogs table.
	* Old and specific. Applicable to both open and closed networks.

## wp-admin/includes/user.php

1. In `edit_user()`, don't allow a user with edit user caps to change their own user role to something without those caps.
	* Applicable to both closed and open networks.
1. In `wp_delete_user()`, use `remove_user_from_blog()` rather than deleting the user completely.
	* Applicable to both closed and open networks.
	* As stated elsewhere, it may be interesting to handle deletion of users in a closed network when a site admin has the ability.

## wp-admin/maint/repair.php

1. Determine if repair on the `sitecategories` table should be attempted.
	* Applicable to both closed and open networks. Old.

## wp-admin/network.php

1. Used to determine where to redirect after multisite has been installed during initial page load.
	* Applicable to both closed and open networks. Used during installation.
1. As part of `network_step2()`, used to show original configuration steps if multisite is already installed.
	* Applicable to both closed and open networks.
1. Used as a single site check to determine if the "Enabling the Network" messaging should show in `network_step2()`.
	* Applicable to both closed and open networks.
1. Used in `network_step2()` with the `ms_files_rewriting` option to determine what rules should be written to web config.
	* Applicable to both closed and open networks.
	* Slightly strange as it wouldn't fire during initial installation.
1. Used in `network_step2()` with the `ms_files_rewriting` option to determine what rules should be written to `.htaccess`.
	* Applicable to both closed and open networks.
	* Slightly strange as it wouldn't fire during initial installation.
1. Used as a single site check in `network_step2()` to explain that the network will be enabled once you complete these steps.
	* Applicable to both closed and open networks.
	* Could benefit from a less confusing `is_single_site()` or `is_installing_network()` or something.
1. Determine which network step should fire on the initial page load.
	* Applicable to both closed and open networks.

## wp-admin/options-discussion.php

1. Used to display a message that signup has been disabled. Only registered users can comment.
	* Could apply to both open and closed networks, though the lines blur a bit. We might want to allow registration to comment as a user of the site, but not register in the same way that it may have been handled before.

## wp-admin/options-general.php

1. Used when single site to show messaging about the WordPress URL and Site URL being the same or different.
	* Applies as a single site indicator.
1. Used to display the option for WordPress URL, Site URL, and admin notification information on a single site.
	* Applies as a single site indicator.
	* The registration options here may be useful or provide some guidance depending on how open networks are handled in multisite. What does registration really mean?
1. If this is multisite and multiple languages are available in `get_available_languages()`, show a drop down for the site specific option.
	* Applicable to open and closed networks.

## wp-admin/options-media.php

1. If a single site, show messaging around the folder and path used for uploading files.
	* Applicable to single sites only.
1. If a single site, show the option for upload folder and path.
	* Applicable to single sites only.

## wp-admin/options-permalink.php

1. Set a blog prefix of `/blog` if this is a subdirectory install of multisite and is on the main site.
	* Applicable to both open and closed networks.
1. If single site, show messaging around updating your `web.config` or `.htaccess`.
	* Applicable to single sites only.
1. Replace any other mention of `blog` in the various permalink bases if multisite and if a subdirectory install and if on the main site.
	* Applicable to both open and closed networks.
1. If single site, show messaging to communicate troubles around updating rewrite rules due to file permissions.
	* Applicable to single sites only.

## wp-admin/options.php

1. Used to determine if admin email change requests should be processed.
	* Applicable to both open and closed networks.
1. If multisite and not a super admin and an action of update, display a cheating message. This page is not normally accessed directly.
	* Applicable to open and closed networks.
1. If single site, whitelist different options for updating than if multisite.
	* Applicable to open and closed networks.
1. If options are being saved and this is multisite and you are not a super admin, display an insufficient permissions error.
	* Applicable to open and closed networks.

## wp-admin/plugin-editor.php

1. If this is multisite and this file is not being accessed in the `wp-admin/network/` area, redirect to the network admin url version.
	* Applicable to both open and closed networks.

## wp-admin/plugin-install.php

1. If this is multisite and this file is not being accessed in the `wp-admin/network/` area, redirect to the network admin url version.
	* Applicable to both open and closed networks.

## wp-admin/plugins.php

1. If activating a plugin and this is multisite and the page is not being accessed under `wp-admin/network/` and this is a network only plugin, redirect to `self_admin_url()`.
	* Applicable to both open and closed networks.
1. When bulk activating, if multisite, check for any network only plugins and unset them from the bulk activation array.
	* Applicable to both open and closed networks.
1. If single site, or if the user can install plugins and is in the `wp-admin/network` area, show a link for 'Add New'.
	* Applicable to both open and closed networks.
	* If any capabilities were changed for site administrators to install plugins outside of `is_network_admin()`, this could change.

## wp-admin/post-new.php

1. If multisite, always do `_admin_notice_post_locked()` on `admin_footer`.
	* Applicable to both closed and open networks.

## wp-admin/post.php

1. If multisite, always do `_admin_notice_post_locked()` on `admin_footer`.
	* Applicable to both closed and open networks.

## wp-admin/theme-editor.php

1. If this is multisite and this page is not loaded under `wp-admin/network`, redirect to the network admin URL.
	* Applicable to both closed and open networks.

## wp-admin/theme-install.php

1. If this is multisite and this page is not loaded under `wp-admin/network`, redirect to the network admin URL.
	* Applicable to both closed and open networks.

## wp-admin/themes.php

1. If this is multisite and the current user can install themes, show a message that theme installation can only be done from the network admin.
	* Applicable to closed and open networks.
	* But wouldn't it be nice if a trusted site admin on a closed network could install their own themes from the theme page.
1. Used as a single site flag to set an install themes cap check in `_wpThemeSettings`
	* Applicable to single sites.
1. Used as a single site flag to set an install themes URL in `_wpThemeSettings`.
	* Applicable to single sites.
1. If single site and the user has the install_themes capability, show an 'Add New' option.
	* Applicable to single site.
1. If single site or if the current user can manage network themes, show any error messaging.
	* Applicable to both closed and open networks.
	* Would benefit from any `is_single_site()` change.
1. If single site and can edit themes and any themes are broken, show messaging.
	* Applicable to single site.

## wp-admin/update-core.php

1. If this is multisite and this page is not loaded under `wp-admin/network`, redirect to the network admin URL.

## wp-admin/user-edit.php

1. If multisite, check if the site administrator is allowed to edit any user.
	* Applicable to both closed and open networks.
1. If multisite and on a user's own profile page and an option is set to confirm the profile email change, update the user and send.
	* Applicable to both closed and open networks.
	* But this is kind of confusing.
1. If multisite and on a user's own profile page and the option to dismiss a confirmation email is selected, don't send an email (?).
	* Applicable to both closed and open networks.
	* But this is also kind of confusing.
1. If multisite, update the email address in signups if present.
	* Applicable to open networks. Not necessary in a closed network.
1. If multisite, handle a request to revoke super admin for a user.
	* Applicable to both closed and open networks.
1. If multisite, and a current user cannot create users, but can promote users, show an 'Add Existing' option when on a user edit page that is not the user's.
	* Applicable to open and closed networks.
1. If multisite and in the network admin area and the current user can manage network options, show the option to grant or revoke super admin rights.
	* Applicable to open and closed networks.

## wp-admin/user-new.php

1. If multisite and the current user cannot create or promote users, show a cheating message.
	* Applicable to open and closed networks.
1. If multisite, create functions for `admin_created_user_email()` and `admin_created_user_subject()`, both invitation related emails to new users.
	* Applicable to open networks.
	* Seems less applicable to closed networks. I wouldn't invite a user, I would add them.
1. If multisite, use `wpmu_validate_user_signup()` to validate provided new user information, sign them up, and then activate the signup if the no confirmation option is selected.
	* Applicable to open networks.
	* In a closed network, so much of this is clunky.
1. If multisite and the user can both create and promote users, set a `$do_both` flag to be used when deciding what options to show for adding a new user.
	* Applicable to both closed and open networks.
1. If multisite, provide different messaging for adding new and existing users.
	* Applicable to both closed and open networks.
1. If multisite, filter whether to enable user auto-complete for non super admins in multisite.
	* Applicable to both closed and open networks.
1. If multisite, process the possible `$_GET['update']` values and provide appropriate messaging.
	* Some separation will be required for open and closed networks. These messages include text referencing invitations and confirmation links.
1. If multisite, determine whether to show messaging for adding an existing user. Then show the form for adding the user.
	* Applicable to both closed and open networks.
	* Some changes could be made to language around confirmation email in a closed vs open network.
1. Comment marking end of `is_multisite()` block.
1. If single site, show fields for First Name, Last Name, Website, and possibly password fields.
	* This is single site specific right now, but could be useful for closed networks where individual site administrators want to add more information about users as they are being created.
1. Comment marking end of `is_multisite()` block.
1. If multisite and super admin, add an option to skip the confirmation email.
	* Applicable to open networks.
	* The handling of this confirmation should be changed in general for closed networks.

## wp-admin/users.php

1. If multisite, show "Remove" user language instead of "Delete" user language.
	* Applicable to open networks. May be interesting to change some of the expectations here for closed networks.
1. When modifying your own user, used to check if you are a super admin if you try editing your user role to something that cannot promote users when your current role can promote users.
	* Applicable to open and closed networks.
1. If this is multisite and a user is being promoted that is not a member of the blog, throw a cheating message.
	* Applicable to open and closed networks.
1. If multisite, handle a dodelete request. User deletion is not allowed from this screen. Redirect.
	* Applicable to open and closed networks.
	* Though it would be interesting if users could be deleted in some instances.
1. If multisite, handle a delete request. User deletion is not allowed from this screen. Redirect.
	* Applicable to open and closed networks.
	* See note above.
1. If single site, handle a doremove request. Users cannot be removed in single site, only deleted.
	* Would benefit from an `is_single_site()` method.
1. If single site, handle a remove request. Users cannot be removed in single site, only deleted.
	* Would benefit from an `is_single_site()` method.
1. If multisite, and a current user cannot create users, but can promote users, show an 'Add Existing' option when on a user edit page that is not the user's.
	* Applicable to open and closed networks.

## wp-admin/user/admin.php

1. If single site, redirect to the admin URL.
	* Applicable to single site.

## wp-includes/admin-bar.php

1. Used in `wp_admin_bar_site_menu()` to add an `Edit Site` link to the admin bar if the user can manage sites and is a site admin.
	* Applicable to both closed and open networks.
1. In `wp_admin_bar_my_sites_menu()`, don't show the my sites menu for single site users.
	* Would benefit from an `is_single_site()` method.

## wp-includes/cache.php

1. Used when construction `WP_Object_Cache` to set `$this->multisite`, adding more to the list of uses to check...
	* Applicable for both closed and open networks.
1. Used as `$this->multisite` when deciding what blog prefix to use during object cache construction.
	* Applicable for both closed and open networks.
	* Each use of `$this->multisite` is only to determine if the blog prefix should be used as part of the key.

## wp-includes/canonical.php

1. If multisite, redirect to the signup location if `wp-register.php` is requested.
	* Applicable to open networks.
	* A different process should be provided for closed networks that have no registration.

## wp-includes/capabilities.php

1. If multisite, super admins have all capabilities by definition.
	* Applicable to both closed and open networks.
1. If multisite, `edit_user` and `edit_users` caps are allowed only for super admins.
	* Applicable to open networks.
	* May be worth discussing the role of a site admin in a closed network.
1. Used as part of a single site, super admin check when determining if unfiltered uploads should be allowed.
	* Applicable to open networks.
	* May be worth discussing the role of a site admin in a closed network and how unfiltered uploads could be used.
1. Do not allow unfiltered HTML if multisite and the user is not a super admin.
	* Applicable to open networks.
	* May be worth discussing the role of a site admin in a closed network.
1. Do not allow non super admins in multisite to edit files, plugins, or themes.
	* Applicable to open and closed networks.
	* There **may** be an argument for allowing site admins to edit files at the site level, but that's way risky.
1. Do not allow non super admins to update, delete, install plugins, themes, or core.
	* Applicable to open networks.
	* Changes around plugins and themes could be interesting for site admins on closed networks.
1. If multisite, check for and set the proper cap for `manage_network_plugins`
	* Applicable to open networks.
	* If changes were made around site admins, this cap may be adjusted a bit.
1. If multisite and not a super admin, do not allow the `delete_users` or `delete_user` cap.
	* Applicable to open networks.
	* Worth discussing the roles of site admins and whether a user deleted from a site could be deleted completely.
1. If multisite and a super admin and `add_new_users` is a network option (?), add the `create_users` capability.
	* Applicable to open networks.
	* The super_admin logic could change on closed networks.
1. In `current_user_can_for_blog()`, determines if we should `switch_to_blog()`
	* Applicable for both closed and open networks.
1. Also in `current_user_can_for_blog()`, determine if we should `restore_current_blog()`
	* Applicable for both closed and open networks.
1. Used in `is_super_admin()` to call `get_super_admins()` for building the super admin list.
	* Applicable for both closed and open networks.

## wp-includes/class-wp-admin-bar.php

1. When initializing the admin bar, if multisite, use `get_active_blog_for_user()` to figure out what domain to show.
	* Applicable for both closed and open networks.

## wp-includes/class-wp-theme.php

1. Always return true in `WP_Theme`'s `is_allowed()` if single site. Otherwise get themes that are allowed on the network and the site.
	* Applicable for both closed and open networks.
	* Could be interesting to figure out the logic for this if site admins are ever allowed to install themes.
1. If single site in `WP_Theme`'s `get_allowed_on_site`, use `get_current_blog_id()`.
	* Applicable for both closed and open networks.

## wp-includes/class-wp-xmlrpc-server.php

1. In `wp_getUsersBlogs()`, if single site, return the value of `blogger_getUsersBlogs()`
	* Applicable to both closed and open networks.
1. In `blogger_getUsersBlogs()`, if multisite, return value of `_multisite_getUsersBlogs()`. This retrieves the user's sites.
	* Applicable to both closed and open networks.

## wp-includes/default-constants.php

1. If multisite, define a slightly larger memory limit.
	* Applicable to both closed and open networks.

## wp-includes/deprecated.php

1. Used in deprecated `wp_admin_bar_dashboard_view_site_menu()` to determine what dashboard URL to use for a user.
	* Deprecated. Applicable to open and closed networks.

## wp-includes/functions.php

1. If multisite is enabled and `ms_files_rewriting` is disabled, obey the value of `UPLOADS`.
	* Applicable to open and closed networks.
1. Additional logic around multisite and upload directories.
	* Applicable to open and closed networks.
1. Used in `is_main_site()` to return true if single site.
	* Applicable for single site.
1. Used in `is_main_network()` to return true if single site.
	* Applicable for single site.
1. Used in `global_terms_enabled` to return false if single site.
	* Applicable for single site.

## wp-includes/l10n.php

1. Used in `get_locale()` to help determine what locale to use when multisite is enabled.
	* Applicable to both open and closed networks.
1. Used in `load_default_textdomain` when multisite is enabled to determine which text domain to use.
	* Applicable to both open and closed networks.

## wp-includes/link-template.php

1. In `get_home_url()`, switch to a specified site before looking up the home URL.
	* Applicable to both open and closed networks.
1. In `get_site_url()`, switch to a specified site before looking up the network URL.
	* Applicable to both open and closed networks.
1. In `network_site_url()`, return `site_url()` if single site. Otherwise determine what the current network URL is.
	* Applicable to both open and closed networks.
1. In `network_home_url()`, return `home_url()` if single site. Otherwise determine what the current network's home URL is.
	* Applicable to both open and closed networks.
1. In `network_admin_url()`, return `admin_url()` if single site. Otherwise determine what the current network's admin URL is.
	* Applicable to both open and closed networks.
1. In `get_dashboard_url()`, return `admin_url()` as the user's dashboard URL if this is a single site installation. Otherwise determine which site is the user's primary site and route there.
	* Applicable to both open and closed networks.

## wp-includes/load.php

1. In `wp_not_installed()`, an error message is shown if the requested site is not installed properly and we are not in the middle of installing.
	* Applicable to both open and closed networks.
1. In `wp_get_active_and_valid_plugins()`, add active network plugins to the array of active plugins.
	* Applicable to both open and closed networks.
1. The definition of `is_multisite()`.
	* Applicable to everything.

## wp-includes/media-template.php

1. If multisite is enabled and upload space is not available, include upload limit exceeded messaging in the inline uploader template.
	* Applicable to both open and closed networks.

## wp-includes/media.php

1. Used to determine if upload limit is exceeded in Plupload default parameters.
	* Applicable to both open and closed networks.

## wp-includes/ms-deprecated.php

1. Used 3 times in a comment for `is_site_admin()`, as this was a method of determining if multisite was enabled before 3.0.0.
	* Applicable to both open and closed networks.

## wp-includes/ms-files.php

1. Used only to redirect if multisite is not enabled.

## wp-includes/ms-functions.php

1. In `get_active_blog_for_user()`, return the only site if single site. If multisite, determine what the user's primary site is and return that.
	* Applicable to both open and closed networks.

## wp-includes/option.php

1. In `wp_load_alloptions()`, if single site, retrieve `alloptions` from cache. If multisite, retrieve `alloptions` from the database.
	* Applicable to both open and closed networks.
1. In `wp_load_alloptions()`, if single site, save `alloptions` to cache. If multisite, don't.
	* Applicable to both open and closed networks.
1. In a comment for `wp_load_core_site_options()`
1. If single site or using external object cache or installing, return null in `wp_load_core_site_options()`. Otherwise, load options and provide them in cache.
	* Applicable to both open and closed networks.
1. In `get_site_option()` used to check for single site. If single site, retrieve the option with `get_option()`. If multisite, `get_site_option()` retrieves the network level option from sitemeta.
	* Applicable to both open and closed networks.
1. In `add_site_option()`, use `add_option()` if single site, otherwise set the network level option in the sitemeta table.
	* Applicable to both open and closed networks.
1. In `delete_site_option()`, use `delete_option()` if single site, otherwise delete the network level option from the sitemeta table.
	* Applicable to both open and closed networks.
1. In `update_site_option()`, use `update_option()` if single site, otherwise update the network level option in the sitemeta table.
	* Applicable to both open and closed networks.

## wp-includes/post.php

1. In `wp_delete_attachment()`, delete the `dirsize_cache` transient if multisite is enabled.
	* Applicable to both open and closed networks.

## wp-includes/rewrite.php

1. Add rewrite rules for registration and signup if multisite is enabled in and this is the main site.
	* Applicable to open networks.
	* Likely an opportunity for different behavior in a closed network.

## wp-includes/theme.php

1. In `wp_get_themes()`, if multisite, process requests for the type of allowed theme to retrieve - network, site, all - for an individual site.
	* Applicable to open and closed networks.

## wp-includes/update.php

1. In `wp_version_check()`, if multisite, set multisite specific reporting data for the API call.
	* Applicable to open and closed networks.

## wp-includes/user.php

1. In `wp_authenticate_spam_check()`, check if the authenticated user has been marked as a spammer or if the user's primary blog has been marked as spam.
	* Applicable to open networks.
	* May not have a place in a closed network.
1. In `WP_User_Query::prepare_query()`, add `user_url` to search columns if this is not multisite or if this is multisite and is not a large user installation.
	* Applicable to both open and closed networks.
1. In `WP_User_Query::prepare_query()`, used to help determine if a cap meta query should be added specific to the site.
	* Applicable to both open and closed networks.
1. In `get_blogs_of_user()`, if single site, set the current site data to the `$blogs` array and return. If multisite, use user meta to build a list of sites and return that array.
	* Applicable to both open and closed networks.

## wp-includes/wp-db.php

1. Used in `init_charset()` if it exists. Sets the charset to UTF8 and the DB collate options.
	* Applicable to both open and closed networks.
1. Used in `set_prefix()` to help determine what table prefix to set.
	* Applicable to both open and closed networks.
1. Used in `get_blog_prefix()` to determine how to build the base prefix for an individual site.
	* Applicable to both open and closed networks.
1. Used twice in `WPDB::tables()` to determine when multisite global tables should be merged into the tables array.
	* Applicable to both open and closed networks.
1. Handle the display of a database error slightly different if multisite is enabled in `WPDB::print_error()`.
	* Applicable to both open and closed networks.

## wp-admin/network/about.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/admin.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/credits.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/edit.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/freedoms.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/index.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/plugin-editor.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/plugin-install.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/plugins.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/profile.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/settings.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/setup.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/site-info.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/site-new.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/site-settings.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/site-themes.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/site-users.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/sites.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/theme-editor.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/theme-install.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/themes.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/update-core.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/update.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/upgrade.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/user-edit.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/user-new.php

1. Used only to redirect if multisite is not enabled.

## wp-admin/network/users.php

1. Used only to redirect if multisite is not enabled.