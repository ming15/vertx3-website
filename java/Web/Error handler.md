Error handler
You can render your own errors using a template handler or otherwise but Vert.x-Web also includes an out of the boxy "pretty" error handler that can render error pages for you.

The handler is ErrorHandler. To use the error handler just set it as a failure handler for any paths that you want covered.

