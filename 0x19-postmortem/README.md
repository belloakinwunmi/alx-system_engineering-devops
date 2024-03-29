When the project 0x019 postmortem task was released, an outage occurred on an isolated node container running a dedicated web server. GET requests on the server led to 500 Internal Server errors when the expected response was an HTML file defining a simple executable command.

Debugging Process

	1: Checked running processes using ps aux. Two apache2 processes - root and www-data - were properly running.

	2: Looked in the sites-available folder of the /etc/apache2/ directory. Determined that the web server was serving content located in /var/www/html/.

	3: In one terminal, ran strace on the PID of the root Apache process. In another, curled the server. Expected great things... only to be disappointed. strace gave no useful information.

	4: Repeated step 3, except on the PID of the www-data process. Kept expectations lower this time... but was rewarded! strace revealed an -1 ENOENT (No such file or directory) error occurring upon an attempt to access the file /var/www/html/wp-includes/class-wp-locale.phpp.

	5: Looked through files in the /var/www/html/ directory one by one, using Vim pattern matching to try and locate the erroneous .phpp file extension. Located it in the wp-settings.php file. (Line 137, require_once( ABSPATH . WPINC . '/class-wp-locale.php' );).

	6: Removed the trailing p from the line.

	7: Tested another curl on the server. 200 A-ok!

	8: Wrote a Puppet manifest to automate fixing of the error.
    	Summation

A typo caused the glitch in the system. In full, the WordPress app was encountering a critical error in wp-settings.php when trying to load the file class-wp-locale.phpp. The correct file name, located in the wp-content directory of the application folder, was class-wp-locale.php.

The patch involved a simple fix on the typo, removing the trailing p.
 
Prevention

This outage was not a web server error, but an application error. To prevent such outages moving forward, please keep the following in mind.

Test the application before deploying. This error would have arisen and could have been addressed earlier had the app been tested.

Note that in response to this error, I wrote a Puppet manifest 0x17-web_stack_debugging_3 to automate the fixing of any such identical errors should they occur in the future. The manifest replaces any phpp extensions in the file /var/www/html/wp-settings.php with php.

