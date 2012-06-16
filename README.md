# VisualPHPUnit

VisualPHPUnit is a visual front-end for PHPUnit.  It offers the following features:

* A stunning front-end which organizes test and suite results
* The ability to view unit testing progress via graphs
* An option to maintain a history of unit test results through the use of snapshots
* Enumeration of PHPUnit statistics and messages
* Convenient display of any debug messages written within unit tests
* Sandboxing of PHP errors
* The ability to generate test results from both a browser and the command line

## Screenshots

TODO

## Requirements

VisualPHPUnit only supports PHPUnit v3.5 and above.

## Installation

1. Download and extract (or git clone) the project to a web-accessible directory.
2. Change the permissions of `app/resource/cache` to 777.

```bash
# from the project root
chmod 777 app/resource/cache
```

3. Open `app/config/bootstrap.php` with your favorite editor.
    1. Within the `$config` array, change `pear_path` so that it points to the directory where PEAR is located.
    2. Within the `$config` array, change `test_directory` so that it points to the root directory where your unit tests are stored.

## Web Server Configuration

### nginx

Place this code block within the `http {}` block in your `nginx.conf` file:

```nginx

    server {
	    server_name     vpu;
	    root            /srv/http/vpu/app/public;
	    index           index.php;

	    access_log      /var/log/nginx/vpu_access.log;
	    error_log       /var/log/nginx/vpu_error.log;

	    location / {
            try_files $uri /index.php;
	    }

	    location ~ \.php$ {
            fastcgi_pass    unix:/var/run/php-fpm/php-fpm.sock;
            fastcgi_index   index.php;
            fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include         fastcgi_params;
	    }
    }
```

Note that you will have to change the `server_name` to the name you use in your hosts file. You will also have to adjust the directories according to where you installed the code. In this configuration, /srv/http/vpu/ is the project root. The public-facing part of VisualPHPUnit, however, is located in app/public within the project root (so in this example, it's /srv/http/vpu/app/public).

### Apache

In your `httpd.conf` file, locate your `DocumentRoot`. It will look something like this:

```apache
DocumentRoot "/srv/http"
```

Now find the `<Directory>` tag that corresponds to your `DocumentRoot`. It will look like this:

```apache
<Directory "/srv/http">
```

Within that tag, change the `AllowOverride` setting:

```apache
AllowOverride All
```

Ensure that your `DirectoryIndex` setting contains index.php:

```apache
DirectoryIndex index.php
```

Now uncomment the following line:

```apache
Include conf/extra/httpd-vhosts.conf
```

Edit your conf/extra/httpd-vhosts.conf file and add the following code block:

```apache
<VirtualHost *:80>
    DocumentRoot "/srv/http/vpu/app/public"
    ServerName vpu
    ErrorLog "/var/log/httpd/vpu_error.log"
    CustomLog "/var/log/httpd/vpu_access.log" common
    <Directory /srv/http/vpu/app/public>
        Options +FollowSymLinks
    </Directory>
</VirtualHost>
```

Note that you will have to change the `ServerName` to the name you use in your hosts file. You will also have to adjust the directories ( in `DocumentRoot`, as well as the `<Directory>` tag) according to where you checked out the code. In this configuration, /srv/http/vpu/ is the project root. The public-facing part of VisualPHPUnit, however, is located in app/public within the project root (so in this example, it's /srv/http/vpu/app/public).

Within your project's public root, create an .htaccess file (in our case, it'd be located at /srv/http/vpu/app/public/.htaccess) and paste the following block inside:

```apache
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !favicon.ico$
    RewriteRule ^(.*)$ index.php [QSA,L]
</IfModule>
```

### Restart Your Web Server

Restart your web server, and then point your browser at the server name you chose above!

## Project Configuration (optional)

VPU comes with many of its features disabled by default.  In order to take advantage of them, you'll have to modify a few more lines in `app/config/bootstrap.php`.

1. If you'd like to enable graph generation, you will have to do the following:
    1. Within the `$config` array, change `store_statistics` to `true`.  If you'd like, you can keep this set as `false`, though you will have to change the 'Store Statistics' option to 'Yes' on the UI if you want the test statistics to be used in graph generation.
    2. Run the migration `app/resource/migration/01_CreateSchema.sql` against a MySQL database.
        - Note that this will automatically create a database named `vpu` with the tables needed to save your test statistics.
    3. Within the `$config` array, change the settings within the `db` array to reflect your database settings.
        - Note that if you're using the migration described above, `database` should remain set to `vpu`.
        - The `plugin` directive should not be changed.

2. If you'd like to enable snapshots, you will have to do the following:
    1. Within the `$config` array, change `create_snapshots` to `true`.  If you'd like, you can keep this set as `false`, though you will have to change the 'Create Snapshots' option to 'Yes' on the UI if you want the test results to be saved.
    2. Within the `$config` array, change `snapshot_directory` to a directory where you would like the snapshots to be saved.
        - Note that this directory must have the appropriate permissions in order to allow PHP to write to it.
        - Note that the dropdown list on the 'Archives' page will only display the files found within `snapshot_directory`.

3. If you'd like to enable error sandboxing, you will have to do the following:
    1. Within the `$config` array, change `sandbox_errors` to `true`.  If you'd like, you can keep this set as `false`, though you will have to change the 'Sandbox Errors' option to 'Yes' on the UI if you want the errors encountered during the test run to be sandboxed.
    2. Within the `$config` array, change `error_reporting` to reflect which errors you'd like to have sandboxed.  See PHP's manual entry on [error_reporting](http://php.net/manual/en/function.error-reporting.php) for more information.

4. If you'd like to use a PHPUnit XML configuration file to define which tests to run, you will have to do the following:
    1. Within the `$config` array, change `xml_configuration_file` to the path where the configuration file can be found.
        - Note that if you leave this set to `false`, but select 'Yes' for the 'Use XML Config' option on the UI, VPU will complain and run with the tests chosen in the file selector instead.
    2. Modify your PHPUnit XML configuration file to include this block:
       ```xml
       <!-- This is required for VPU to work correctly -->
       <listeners>
         <listener class="PHPUnit_Util_Log_JSON"></listener>
       </listeners>
       ```

5. If you'd like to load any bootstraps, you will have to do the following:
    1. Within the `$config` array, list the paths to each of the bootstraps within the `bootstraps` array.

## Version Information

TODO

## Feedback

Feel free to send any feedback you may have regarding this project to NSinopoli@gmail.com.

## Credits

Special thanks to Matt Mueller (http://mattmueller.me/blog/), who came up with the initial concept, wrote the original code (https://github.com/MatthewMueller/PHPUnit-Test-Report), and was kind enough to share it.

Thanks to Mike Zhou, Hang Dao, Thomas Ingham, and Fredrik Wollsén for their suggestions!
