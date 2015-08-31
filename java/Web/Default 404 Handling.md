# Default 404 Handling

如果没有`route`匹配到客户端的请求,那么`Vert.x-Web`会发送一个404的信号错误.

你可以自行手动设置`error handler`, 或者选择我们提供的`error handler`, 如果没有设置`error handler`，`Vert.x-Web`会返回一个基本的404回应

