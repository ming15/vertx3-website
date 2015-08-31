# Serving favicons

`Vert.x-Web`包含一个`FaviconHandler`来响应`favicons`.

`Favicons`可以被指定为文件系统里的一个路径,或者`Vert.x-Web`会默认的在`classpath`搜索名为`favicon.ico`的文件.这意味着你需要在你的应用程序的jar包绑定该`favicon`
