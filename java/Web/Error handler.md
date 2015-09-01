# Error handler

你可以自己提供一个`ErrorHandler`用于处理`error`异常,否则的话`Vert.x-Web`会包含一个包装好的`pretty error handler`用于响应错误页面.

如果你自己想要设置一个`ErrorHandler`, 那么你只需要将其设置成`failure handler`就可以了.

