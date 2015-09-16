# File System

Vert.x lets you manipulate files on the file system. File system operations are asynchronous and take a handler function as the last argument. This function will be called when the operation is complete, or an error has occurred.

The argument passed into the handler is an instance of org.vertx.java.core.AsyncResult.

Synchronous forms

For convenience, we also provide synchronous forms of most operations. It's highly recommended the asynchronous forms are always used for real applications.

The synchronous form does not take a handler as an argument and returns its results directly. The name of the synchronous function is the same as the name as the asynchronous form with Sync appended.

copy

Copies a file.

This function can be called in two different ways:

copy(source, destination, handler)
Non recursive file copy. source is the source file name. destination is the destination file name.

Here's an example:

vertx.fileSystem().copy("foo.dat", "bar.dat", new AsyncResultHandler<Void>() {
    public void handle(AsyncResult ar) {
        if (ar.succeeded()) {
            log.info("Copy was successful");
        } else {
            log.error("Failed to copy", ar.cause());
        }
    }
});
copy(source, destination, recursive, handler)
Recursive copy. source is the source file name. destination is the destination file name. recursive is a boolean flag - if true and source is a directory, then a recursive copy of the directory and all its contents will be attempted.

move

Moves a file.

move(source, destination, handler)

source is the source file name. destination is the destination file name.

truncate

Truncates a file.

truncate(file, len, handler)

file is the file name of the file to truncate. len is the length in bytes to truncate it to.

chmod

Changes permissions on a file or directory.

This function can be called in two different ways:

chmod(file, perms, handler).
Change permissions on a file.

file is the file name. perms is a Unix style permissions string made up of 9 characters. The first three are the owner's permissions. The second three are the group's permissions and the third three are others permissions. In each group of three if the first character is r then it represents a read permission. If the second character is w it represents write permission. If the third character is x it represents execute permission. If the entity does not have the permission the letter is replaced with -. Some examples:

rwxr-xr-x
r--r--r--
chmod(file, perms, dirPerms, handler).
Recursively change permissionson a directory. file is the directory name. perms is a Unix style permissions to apply recursively to any files in the directory. dirPerms is a Unix style permissions string to apply to the directory and any other child directories recursively.

props

Retrieve properties of a file.

props(file, handler)

file is the file name. The props are returned in the handler. The results is an object with the following methods:

creationTime(): Time of file creation.
lastAccessTime(): Time of last file access.
lastModifiedTime(): Time file was last modified.
isDirectory(): This will have the value true if the file is a directory.
isRegularFile(): This will have the value true if the file is a regular file (not symlink or directory).
isSymbolicLink(): This will have the value true if the file is a symbolic link.
isOther(): This will have the value true if the file is another type.
Here's an example:

vertx.fileSystem().props("foo.dat", "bar.dat", new AsyncResultHandler<FileProps>() {
    public void handle(AsyncResult<FileProps> ar) {
        if (ar.succeeded()) {
            log.info("File props are:");
            log.info("Last accessed: " + ar.result().lastAccessTime());
            // etc
        } else {
            log.error("Failed to get props", ar.cause());
        }
    }
});
lprops

Retrieve properties of a link. This is like props but should be used when you want to retrieve properties of a link itself without following it.

It takes the same arguments and provides the same results as props.

link

Create a hard link.

link(link, existing, handler)

link is the name of the link. existing is the exsting file (i.e. where to point the link at).

symlink

Create a symbolic link.

symlink(link, existing, handler)

link is the name of the symlink. existing is the exsting file (i.e. where to point the symlink at).

unlink

Unlink (delete) a link.

unlink(link, handler)

link is the name of the link to unlink.

readSymLink

Reads a symbolic link. I.e returns the path representing the file that the symbolic link specified by link points to.

readSymLink(link, handler)

link is the name of the link to read. An usage example would be:

vertx.fileSystem().readSymLink("somelink", new AsyncResultHandler<String>() {
    public void handle(AsyncResult<String> ar) {
        if (ar.succeeded()) {
            log.info("Link points at  " + ar.result());
        } else {
            log.error("Failed to read", ar.cause());
        }
    }
});
delete

Deletes a file or recursively deletes a directory.

This function can be called in two ways:

delete(file, handler)
Deletes a file. file is the file name.

delete(file, recursive, handler)
If recursive is true, it deletes a directory with name file, recursively. Otherwise it just deletes a file.

mkdir

Creates a directory.

This function can be called in three ways:

mkdir(dirname, handler)
Makes a new empty directory with name dirname, and default permissions `

mkdir(dirname, createParents, handler)
If createParents is true, this creates a new directory and creates any of its parents too. Here's an example

vertx.fileSystem().mkdir("a/b/c", true, new AsyncResultHandler<Void>() {
    public void handle(AsyncResult ar) {
        if (ar.suceeded()) {
            log.info("Directory created ok");
        } else {
            log.error("Failed to mkdir", ar.cause());
        }
    }
});
mkdir(dirname, createParents, perms, handler)
Like mkdir(dirname, createParents, handler), but also allows permissions for the newly created director(ies) to be specified. perms is a Unix style permissions string as explained earlier.

readDir

Reads a directory. I.e. lists the contents of the directory.

This function can be called in two ways:

readDir(dirName)
Lists the contents of a directory

readDir(dirName, filter)
List only the contents of a directory which match the filter. Here's an example which only lists files with an extension txt in a directory.

vertx.fileSystem().readDir("mydirectory", ".*\\.txt", new AsyncResultHandler<String[]>() {
    public void handle(AsyncResult<String[]> ar) {
        if (ar.succeeded() {
            log.info("Directory contains these .txt files");
            for (int i = 0; i < ar.result().length; i++) {
              log.info(ar.result()[i]);
            }
        } else {
            log.error("Failed to read", ar.cause());
        }
    }
});
The filter is a regular expression.

readFile

Read the entire contents of a file in one go. Be careful if using this with large files since the entire file will be stored in memory at once.

readFile(file). Where file is the file name of the file to read.

The body of the file will be returned as an instance of org.vertx.java.core.buffer.Buffer in the handler.

Here is an example:

vertx.fileSystem().readFile("myfile.dat", new AsyncResultHandler<Buffer>() {
    public void handle(AsyncResult<Buffer> ar) {
        if (ar.succeeded()) {
            log.info("File contains: " + ar.result().length() + " bytes");
        } else {
            log.error("Failed to read", ar.cause());
        }
    }
});
writeFile

Writes an entire Buffer or a string into a new file on disk.

writeFile(file, data, handler) Where file is the file name. data is a Buffer or string.

createFile

Creates a new empty file.

createFile(file, handler). Where file is the file name.

exists

Checks if a file exists.

exists(file, handler). Where file is the file name.

The result is returned in the handler.

vertx.fileSystem().exists("some-file.txt", new AsyncResultHandler<Boolean>() {
    public void handle(AsyncResult<Boolean> ar) {
        if (ar.succeeded()) {
            log.info("File " + (ar.result() ? "exists" : "does not exist"));
        } else {
            log.error("Failed to check existence", ar.cause());
        }
    }
});
fsProps

Get properties for the file system.

fsProps(file, handler). Where file is any file on the file system.

The result is returned in the handler. The result object is an instance of org.vertx.java.core.file.FileSystemProps has the following methods:

totalSpace(): Total space on the file system in bytes.
unallocatedSpace(): Unallocated space on the file system in bytes.
usableSpace(): Usable space on the file system in bytes.
Here is an example:

vertx.fileSystem().fsProps("mydir", new AsyncResultHandler<FileSystemProps>() {
    public void handle(AsyncResult<FileSystemProps> ar) {
        if (ar.succeeded()) {
            log.info("total space: " + ar.result().totalSpace());
            // etc
        } else {
            log.error("Failed to check existence", ar.cause());
        }
    }
});
open

Opens an asynchronous file for reading \ writing.

This function can be called in four different ways:

open(file, handler)
Opens a file for reading and writing. file is the file name. It creates it if it does not already exist.

open(file, perms, handler)
Opens a file for reading and writing. file is the file name. It creates it if it does not already exist and assigns it the permissions as specified by perms.

open(file, perms, createNew, handler)
Opens a file for reading and writing. file is the file name. It createNew is true it creates it if it does not already exist.

open(file, perms, read, write, createNew, handler)
Opens a file. file is the file name. If read is true it is opened for reading. If write is true it is opened for writing. It createNew is true it creates it if it does not already exist.

open(file, perms, read, write, createNew, flush, handler)
Opens a file. file is the file name. If read is true it is opened for reading. If write is true it is opened for writing. It createNew is true it creates it if it does not already exist. If flush is true all writes are immediately flushed through the OS cache (default value of flush is false).

When the file is opened, an instance of org.vertx.java.core.file.AsyncFile is passed into the result handler:

vertx.fileSystem().open("some-file.dat", new AsyncResultHandler<AsyncFile>() {
    public void handle(AsyncResult<AsyncFile> ar) {
        if (ar.succeeded()) {
            log.info("File opened ok!");
            // etc
        } else {
            log.error("Failed to open file", ar.cause());
        }
    }
});
AsyncFile

Instances of org.vertx.java.core.file.AsyncFile are returned from calls to open and you use them to read from and write to files asynchronously. They allow asynchronous random file access.

AsyncFile implementsReadStream and WriteStream so you can pump files to and from other stream objects such as net sockets, http requests and responses, and WebSockets.

They also allow you to read and write directly to them.

Random access writes

To use an AsyncFile for random access writing you use the write method.

write(buffer, position, handler).

The parameters to the method are:

buffer: the buffer to write.
position: an integer position in the file where to write the buffer. If the position is greater or equal to the size of the file, the file will be enlarged to accomodate the offset.
Here is an example of random access writes:

vertx.fileSystem().open("some-file.dat", new AsyncResultHandler<AsyncFile>() {
    public void handle(AsyncResult<AsyncFile> ar) {
        if (ar.succeeded()) {
            AsyncFile asyncFile = ar.result();
            // File open, write a buffer 5 times into a file
            Buffer buff = new Buffer("foo");
            for (int i = 0; i < 5; i++) {
                asyncFile.write(buff, buff.length() * i, new AsyncResultHandler<Void>() {
                    public void handle(AsyncResult ar) {
                        if (ar.succeeded()) {
                            log.info("Written ok!");
                            // etc
                        } else {
                            log.error("Failed to write", ar.cause());
                        }
                    }
                });
            }
        } else {
            log.error("Failed to open file", ar.cause());
        }
    }
});
Random access reads

To use an AsyncFile for random access reads you use the read method.

read(buffer, offset, position, length, handler).

The parameters to the method are:

buffer: the buffer into which the data will be read.
offset: an integer offset into the buffer where the read data will be placed.
position: the position in the file where to read data from.
length: the number of bytes of data to read
Here's an example of random access reads:

vertx.fileSystem().open("some-file.dat", new AsyncResultHandler<AsyncFile>() {
    public void handle(AsyncResult<AsyncFile> ar) {
        if (ar.succeeded()) {
            AsyncFile asyncFile = ar.result();
            Buffer buff = new Buffer(1000);
            for (int i = 0; i < 10; i++) {
                asyncFile.read(buff, i * 100, i * 100, 100, new AsyncResultHandler<Buffer>() {
                    public void handle(AsyncResult<Buffer> ar) {
                        if (ar.succeeded()) {
                            log.info("Read ok!");
                            // etc
                        } else {
                            log.error("Failed to write", ar.cause());
                        }
                    }
                });
            }
        } else {
            log.error("Failed to open file", ar.cause());
        }
    }
});
If you attempt to read past the end of file, the read will not fail but it will simply read zero bytes.

Flushing data to underlying storage.

If the AsyncFile was not opened with flush = true, then you can manually flush any writes from the OS cache by calling the flush() method.

This method can also be called with an handler which will be called when the flush is complete.

Using AsyncFile as ReadStream and WriteStream

AsyncFile implements ReadStream and WriteStream. You can then use them with a pump to pump data to and from other read and write streams.

Here's an example of pumping data from a file on a client to a HTTP request:

final HttpClient client = vertx.createHttpClient.setHost("foo.com");

vertx.fileSystem().open("some-file.dat", new AsyncResultHandler<AsyncFile>() {
    public void handle(AsyncResult<AsyncFile> ar) {
        if (ar.succeeded()) {
            final HttpClientRequest request = client.put("/uploads", new Handler<HttpClientResponse>() {
                public void handle(HttpClientResponse resp) {
                    log.info("Received response: " + resp.statusCode());
                }
            });
            AsyncFile asyncFile = ar.result();
            request.setChunked(true);
            Pump.createPump(asyncFile, request).start();
            asyncFile.endHandler(new VoidHandler() {
                public void handle() {
                    // File sent, end HTTP requuest
                    request.end();
                }
            });
        } else {
            log.error("Failed to open file", ar.cause());
        }
    }
});
Closing an AsyncFile

To close an AsyncFile call the close() method. Closing is asynchronous and if you want to be notified when the close has been completed you can specify a handler function as an argument.
