# Uses of is_multisite()

## wp-activate.php

1. Possible redirect to `wp-login.php?action=register` if not multisite.
	* This check should be for an open multisite network.
	* A closed multisite network should redirect to home.
