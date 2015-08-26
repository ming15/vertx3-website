Using blocking handlers
Sometimes, you might have to do something in a handler that might block the event loop for some time, e.g. call a legacy blocking API or do some intensive calculation.

You can’t do that in a normal handler, so we provide the ability to set blocking handlers on a route.

A blocking handler looks just like a normal handler but it’s called by Vert.x using a thread from the worker pool not using an event loop.

You set a blocking handler on a route with blockingHandler. Here’s an example:

router.route().blockingHandler(routingContext -> {

  // Do something that might take some time synchronously
  service.doSomethingThatBlocks();

  // Now call the next handler
  routingContext.next();

});
By default, any blocking handlers executed on the same context (e.g. the same verticle instance) are ordered - this means the next one won’t be executed until the previous one has completed. If you don’t care about orderering and don’t mind your blocking handlers executing in parallel you can set the blocking handler specifying ordered as false using blockingHandler.

