# Request body handling
The BodyHandler allows you to retrieve request bodies, limit body sizes and handle file uploads.

You should make sure a body handler is on a matching route for any requests that require this functionality.
```java
router.route().handler(BodyHandler.create());
```

### Getting the request body
If you know the request body is JSON, then you can use getBodyAsJson, if you know it’s a string you can use getBodyAsString, or to retrieve it as a buffer use getBody.

Limiting body size
To limit the size of a request body, create the body handler then use setBodyLimit to specifying the maximum body size, in bytes. This is useful to avoid running out of memory with very large bodies.

If an attempt to send a body greater than the maximum size is made, an HTTP status code of 413 - Request Entity Too Large, will be sent.

There is no body limit by default.

### Merging form attributes
By default, the body handler will merge any form attributes into the request parameters. If you don’t want this behaviour you can use disable it with setMergeFormAttributes.

### Handling file uploads
Body handler is also used to handle multi-part file uploads.

If a body handler is on a matching route for the request, any file uploads will be automatically streamed to the uploads directory, which is file-uploads by default.

Each file will be given an automatically generated file name, and the file uploads will be available on the routing context with fileUploads.

Here’s an example:
```java
router.route().handler(BodyHandler.create());

router.post("/some/path/uploads").handler(routingContext -> {

  Set<FileUpload> uploads = routingContext.fileUploads();
  // Do something with uploads....

});
```
Each file upload is described by a FileUpload instance, which allows various properties such as the name, file-name and size to be accessed.

