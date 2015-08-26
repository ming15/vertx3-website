Serving static resources
Vert.x-Web comes with an out of the box handler for serving static web resources so you can write static web servers very easily.

To serve static resources such as .html, .css, .js or any other static resource, you use an instance of StaticHandler.

Any requests to paths handled by the static handler will result in files being served from a directory on the file system or from the classpath. The default static file directory is webroot but this can be configured.

In the following example all requests to paths starting with /static/ will get served from the directory webroot:

router.route("/static/*").handler(StaticHandler.create());
For example, if there was a request with path /static/css/mystyles.css the static serve will look for a file in the directory webroot/static/css/mystyle.css.

It will also look for a file on the classpath called webroot/static/css/mystyle.css. This means you can package up all your static resources into a jar file (or fatjar) and distribute them like that.

When Vert.x finds a resource on the classpath for the first time it extracts it and caches it in a temporary directory on disk so it doesn’t have to do this each time.

Configuring caching
By default the static handler will set cache headers to enable browsers to effectively cache files.

Vert.x-Web sets the headers cache-control,last-modified, and date.

cache-control is set to max-age=86400 by default. This corresponds to one day. This can be configured with setMaxAgeSeconds if required.

If a browser sends a GET or a HEAD request with an if-modified-since header and the resource has not been modified since that date, a 304 status is returned which tells the browser to use its locally cached resource.

If handling of cache headers is not required, it can be disabled with setCachingEnabled.

When cache handling is enabled Vert.x-Web will cache the last modified date of resources in memory, this avoids a disk hit to check the actual last modified date every time.

Entries in the cache have an expiry time, and after that time, the file on disk will be checked again and the cache entry updated.

If you know that your files never change on disk, then the cache entry will effectively never expire. This is the default.

If you know that your files might change on disk when the server is running then you can set files read only to false with setFilesReadOnly.

To enable the maximum number of entries that can be cached in memory at any one time you can use setMaxCacheSize.

To configure the expiry time of cache entries you can use setCacheEntryTimeout.

Configuring the index page
Any requests to the root path / will cause the index page to be served. By default the index page is index.html. This can be configured with setIndexPage.

Changing the web root
By default static resources will be served from the directory webroot. To configure this use setWebRoot.

Serving hidden files
By default the serve will serve hidden files (files starting with .).

If you do not want hidden files to be served you can configure it with setIncludeHidden.

Directory listing
The server can also perform directory listing. By default directory listing is disabled. To enabled it use setDirectoryListing.

When directory listing is enabled the content returned depends on the content type in the accept header.

For text/html directory listing, the template used to render the directory listing page can be configured with setDirectoryTemplate.

Disabling file caching on disk
By default, Vert.x will cache files that are served from the classpath into a file on disk in a sub-directory of a directory called .vertx in the current working directory. This is mainly useful when deploying services as fatjars in production where serving a file from the classpath every time can be slow.

In development this can cause a problem, as if you update your static content while the server is running, the cached file will be served not the updated file.

To disable file caching you can provide the system property vertx.disableFileCaching with the value true. E.g. you could set up a run configuration in your IDE to set this when runnning your main class.

