Timeout handler
Vert.x-Web includes a timeout handler that you can use to timeout requests if they take too long to process.

This is configured using an instance of TimeoutHandler.

If a request times out before the response is written a 408 response will be returned to the client.

Here’s an example of using a timeout handler which will timeout all requests to paths starting with /foo after 5 seconds:

router.route("/foo/").handler(TimeoutHandler.create(5000));
