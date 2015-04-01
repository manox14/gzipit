**GzipIt** is a single file solution to combine, minimize and gzip CSS and JavaScript assets thus reducing needed network bandwidth, page load times and server load. By using **GzipIt** one can implement most important (for the average web site) techniques of the famous [Best Practices for Speeding Up Your Web Site](http://developer.yahoo.com/performance/rules.html) for a web site.

> Project was inspired by [CSS and Javascript Combinator](http://rakaz.nl/code/combine) by Niels Leenheer. [PHP version](http://github.com/rgrove/jsmin-php/) of [Douglas Crockford's JSMin](http://www.crockford.com/javascript/jsmin.html) and [CssMin](http://code.google.com/p/cssmin/) are bundled into source code with licence notices leaved intact. Browser detection code is partially taken from [Minify](http://code.google.com/p/minify/).


# Features #
  * _**Single PHP file**_, easy installation, [PHP accelerators](http://en.wikipedia.org/wiki/List_of_PHP_accelerators) friendly
  * **No changes** in the existing development and deployment process, no scripts to run on any change in CSS or JavaScript file, GzipIt tries to be as transparent for developer as possible
  * Multiple files are **combined in one** thus minimizing number of needed connections
  * CSS and JavaScript **code is minimized** by removing comments, spaces and line feeds
  * All content is served **gzip-compressed** if client supports this
  * HTTP headers for **agressive caching on client** are added to the response, **ETags** are supported too
  * Files can be collected into **unlimited number of 'assets'** for even easier referencing from HTML
  * Processed data can be **cached on the disk** to reduce server load
  * Friendly error messages for the quick debug

# Requirements #
  * PHP 5.x
  * `zlib` extension (usually installed with PHP)
  * `mod_rewrite` is recommended but not required

# Installation #
  * Download, unpack and drop the `gzipit.php` file into the root of your web site
  * Open file in any editor and set three parameters to reflect you folder layout:

```
// Directory where output files will be cached (can be placed outside of document root)
define('GZIPIT_DIR_CACHE', dirname(__FILE__) . '/tmp');

// Directory where original CSS files are stored (sub directories are accessible too)
define('GZIPIT_DIR_CSS', dirname(__FILE__) . '/css');			

// Directory where original CSS files are stored (sub directories are accessible too)
define('GZIPIT_DIR_JS', dirname(__FILE__) . '/js');				
```
  * Make folder you specified in `GZIPIT_DIR_CACHE` writable from PHP.

# Test the installation #
GzipIt accepts three HTTP GET parameters: `type`, `files` (used only together) and `assets`. Please check next section for instructions how to use assets, the remaining two are simple:
  * `type` can have one of two possible values `css` (for CSS files) or `javascript` (for JavaScript)
  * `files` should be comma-delimited list of files from the folders you specified in `GZIPIT_DIR_CSS` or `GZIPIT_DIR_JS` parameters

To test that GzipIt works in your environment you can try to access the URL (ajust it for you domain):

```
http://www.example.com/gzipit.php?type=css&files=script1.js,script2.js
```

You can reference files in sub-folders too (notice absence of leading slash):
```
http://www.example.com/gzipit.php?type=css&files=script1.js,forms/script2.js
```

If you get your scripts minified in your browser you can change URLs in HTML to the new location with one small recommended addition:

```
http://www.example.com/gzipit.php?type=css&files=script1.js,forms/script2.js&v=1.0
```

The `v=1.0` is not used by GzipIt. Its purpuse is to force client cache flush when you change it.

Such simple install has following pros:
  * It is really simple

...but it has some cons to:
  * If you have a lot of files to combine, you can hit URL length limit (for example it is just 255 characters for Apache on Windows)
  * If you have your CSS in subfolders you will have problems referencing images. The image URLs are relative to CSS file location. So if you have slashes in URL, you will have to use absolute URLs or reference images from the root of the site.

# Assets #
Asset is a 'preset' of two parameters (`type` and `files`) and can be specified either in `gzipit.php` file itself or in external file.

If you specify this parameter (default is empty string):

```
define('GZIPIT_ASSETS_FILE', 'assets.php');
```
the specified file will be included and variable called `$GZIPIT_ASSETS` will be used, otherwise you can specify this variable in `gzipit.php`.
For both cases syntax is the same:

```
$GZIPIT_ASSETS = array(
	'css-default' => array(
		'type' => 'css',
		'files' => array(
			'file1.css',
			'file2.css',
			...
		)
	),
	'js-default' => array(
		'type' => 'javascript',
		'files' => array(
			'file1.js',
			'file2.js',
			...
		)
	),
);
```

Have you noticed how similar assets are to `type` and `files` parameters?

To use assets you should specify only one parameter in the URL:

```
http://www.example.com/gzipit.php?asset=css-default
```

or, even better:

```
http://www.example.com/gzipit.php?asset=css-default&v=1.0
```

Such approach is much better than specifing all files in URL, becase:

  * URL length is not a problem
  * Unlimited number of assets is possible

But there is one disadvantage left:
  * Some proxy servers consider URLs with GET parameters as dynamic and doesn't cache them.

So, let's move to the even better option.

# Enabling Apache's `mod_rewrite` for assets #

Place the following into you `.htaccess`:

```
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
RewriteRule ^asset-(.*?)\.v(.*?)\.(css|js)$ gzipit.php?asset=$1 [L]
</IfModule>
```

After that you will be able to replace this (example):

```
<link type="text/css" href="/gzipit.php?asset=css-default&v=1.0" rel="stylesheet" />
<script type="text/javascript" src="/gzipit.php?asset=js-default&v=1.0"></script>
```

With nice and clean URLs:

```
<link type="text/css" href="/asset-css-default.v1.css" rel="stylesheet" />
<script type="text/javascript" src="/asset-js-default.v1.js"></script>
```

# nginx configuration #

When nginx is the only web server (no Apache backend), rewrites can be configured like this:

```
server {
        server_name  example.com www.example.com;
        listen   80;
        
        location / {
               root   /www/example.com;
               index  index.php index.html index.htm;
               if (!-e $request_filename) {
			rewrite ^\/asset-(.*?)\.v(.*?)\.(css|js)$ /gzipit.php?asset=$1 last;
               }
        }        

	... // Other sections
}
```

# Changelog #
  * 2012-03-30 — v1.2, bugfix for IE6,7,8
  * 2012-03-09 — v1.1, bugfixes, constants are now prefixed with `GZIPIT_` instead of `CP_`
  * 2011-03-20 — v1.0, first stable version, PHP 5.3 support
  * 2010-10-03 — v1.0-beta1, initial release

# ToDo #
  * Update of the bundled libraries
  * Support for eAccelerator\APC\memcached to store processed files in their cache
  * Check if [phar](http://www.php.net/manual/en/intro.phar.php) is really an option

# Limitations #
  * For even more efficient serving CSS and JavaScript you should consider performing combining, minimizing and gzipping beforehand, so files will be served directly by web server without any need for PHP. If you have really high traffic site you should already know and use this :).