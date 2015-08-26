Default 404 Handling
If no routes match for any particular request, Vert.x-Web will signal a 404 error.

This can then be handled by your own error handler, or perhaps the augmented error handler that we supply to use, or if no error handler is provided Vert.x-Web will send back a basic 404 (Not Found) response.

